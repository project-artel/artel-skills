---
name: artel-game-functions
description: >-
  Add cheats and behaviors to a Unity game that uses the ARTEL SDK, so the ARTEL QA agent can
  jump to a scene, set up a test precondition, or keep giving per-frame input in a real-time
  fight. Use when the user asks to "add a cheat", "치트키 만들어줘", "go to a scene with a cheat",
  "kill all enemies cheat", "show the ending", "auto attack for the QA agent", "behavior",
  "[ArtelCheat]", "[ArtelBehavior]", or wants the QA agent to stop failing with SETUP-FAILED.
---

# ARTEL game functions: cheats and behaviors

A **cheat** is a method the QA agent calls once to change game state (load a scene, unlock a
stage, defeat the enemies on screen). A **behavior** is an `IEnumerator` the agent starts and that
keeps running every frame inside the game until it ends or is stopped (keep casting spells, keep
passing turns). Together they are called game functions.

The agent uses cheats to reach a test case's precondition fast. It must never use one to produce
the value a step verifies, and the QA author can forbid one per test case. Write cheats with that
in mind: each one should set up a state, not fake a result.

## Before you start

1. **The SDK must contain the feature.** Game functions ship in `kr.artel.sdk` from the ARTEL-1052
   change (project-artel/artel-sdk#142). Until that is merged into `develop`, install the package
   from that branch:

   ```text
   https://github.com/project-artel/artel-sdk.git?path=/Packages/kr.artel.sdk#feat/sdk-artelcheat-artelbehavior-등록과-invoke_cheat-start_behavior-action-game_function_catalog-frame-추가-ARTEL-1052
   ```

   Check: `Packages/kr.artel.sdk/Runtime/GameFunctions/ArtelCheatAttribute.cs` exists in the
   package. If it does not, stop and tell the user; nothing below will compile.
2. **Read the game before writing anything.** List the scenes in
   `ProjectSettings/EditorBuildSettings.asset`, then find every `SceneManager.LoadScene` call, the
   save code (`PlayerPrefs` keys), the singletons (`static ... Instance`), and the methods the UI
   buttons call. A good cheat calls those same methods.
3. Put game functions in their own file, for example `Assets/ArtelBench/<Game>GameFunctions.cs`,
   in an assembly that references `Artel.Runtime` (the default `Assembly-CSharp` does).

## Write a cheat

```csharp
using Artel;

public static class MyGameFunctions
{
    [ArtelCheat("Sets the unlocked stage (0 village .. 4 boss) and saves it. Takes effect on the map screen.")]
    static string UnlockStage([CheatRange(0, 4)] int stage)
    {
        MapMove.StagePosition = stage;          // the field the game itself reads
        SaveLoadController.Instance?.SavePlayData();
        PlayerPrefs.Save();
        return $"stage_position={MapMove.StagePosition}";   // the agent sees this text
    }
}
```

Rules the weaver enforces at compile time (a mistake is a compile error, not a runtime surprise):

| Rule | Detail |
|---|---|
| Parameter types | `int`, `float`, `bool`, `string`, or an `enum`. Range with `[CheatRange(min, max)]`; a C# default value makes the parameter optional |
| Return type | cheat: `void`, `string` (the message the agent reads) or `IEnumerator`; behavior: `IEnumerator` |
| Name | the method name in snake_case (`UnlockStage` → `unlock_stage`); override with `name:`; duplicates are an error |
| Static or instance | instance methods must be on a `MonoBehaviour`; the agent passes a `target` selector when several are live |
| `when:` | names a parameterless method on the same type returning `bool` or `string` (`null` = available, otherwise the reason) |
| Release builds | nothing is generated; the attributes are inert and the actions are refused |

### Pick parameter types the agent cannot get wrong

- **Use an enum for a closed set.** The catalog publishes enum values, so the agent picks one
  instead of inventing a scene name. A bad value comes back as
  `bad argument scene: expected one of: Title, Story, ...`.
- **Put a range on every number.** `[CheatRange(0, 4)]` turns a typo into
  `bad argument stage: must be between 0 and 4` instead of a broken save.

### Write `when` for every cheat that only works somewhere

```csharp
static string HasLiveEnemy()
{
    if (TurnBattleSystem.Instance == null) return "not in a battle scene";
    foreach (var enemy in Object.FindObjectsOfType<Enemy>())
        if (enemy.IsAlive) return null;
    return "no live enemy";
}

[ArtelCheat("Defeats every slime alive in the current wave with Enemy.Kill().", when: nameof(HasLiveEnemy))]
static string KillAllSlimes() { ... }
```

The SDK evaluates `when` once a second and shows the agent `Cheats now` / `Cheats not now
(reason)`, and refuses a call with `unavailable: <reason>`. Without it the agent tries the cheat on
the title screen and your code throws.

## Recipes that worked (WordVenture)

The full file is [`references/wordventure-example.cs`](references/wordventure-example.cs).

| Need | Cheat | How |
|---|---|---|
| Go to any scene | `go_to_scene(scene: GameScene, stage: int = 0)` | `SceneManager.LoadScene(name)`; for the battle scene first set the stage the map would have set |
| Unlock a stage | `unlock_stage(stage)` | write the static field the map reads, then save |
| Defeat the enemies on screen | `kill_all_slimes()` | call the game's own `Enemy.Kill()` (the normal death path), not `Destroy` |
| See the ending | `show_ending()` | put the save in the "boss cleared" state, then load `EndingScene` |
| Heal | `heal_player(amount)` | the game's damage method with a negative value, guarded by `when` |

Lessons from building them:

1. **Recreate what the skipped path would have set.** Loading the battle scene from the title
   skipped the map, which is where the game creates its `StageDataSingleton` and sets
   `stagePosition`. The cheat creates the singleton when it is missing:

   ```csharp
   static StageDataSingleton EnsureStageData()
   {
       if (StageDataSingleton.Instance == null)
           new GameObject("StageDataSingleton (cheat)").AddComponent<StageDataSingleton>();
       return StageDataSingleton.Instance;
   }
   ```

   For every scene you can jump to, ask what the previous scene normally leaves behind
   (singletons, static fields, `PlayerPrefs`) and set it in the cheat.
2. **Make the save agree with the scene.** `show_ending` sets `MapMove.StagePosition = 5` (all five
   stages cleared) and saves before loading `EndingScene`, because that is the state the real
   ending leaves. A cheat that only loads the scene leaves a save that contradicts it.
3. **Use the game's own methods.** `Enemy.Kill()` runs the death animation, wave bookkeeping and
   victory check, so killing the last slime of the last wave still leads to `GameClearScene`.
   `Destroy(enemy)` would leave the wave waiting forever.
4. **Return what changed.** `killed=1`, `stage_position=4`, `loading=TurnBattleScene stage=0`. The
   agent reads the message and the screen; a number lets it notice that a wave held one slime, not
   five.

## Write a behavior

```csharp
[ArtelBehavior("Casts spells with the hand cards the way the UI does, and ends the turn when no pair is left.", group: "combat")]
static IEnumerator AutoCastSpells([CheatRange(1, 100)] int maxCasts, ArtelBehaviorContext context)
{
    var casts = 0;
    try
    {
        while (true)
        {
            if (TurnBattleSystem.Instance == null) { context.Complete($"battle ended after {casts} casts"); yield break; }
            if (InteractionLock.IsLocked) { context.Report("waiting: interaction locked"); yield return null; continue; }
            // ... one step of play through the same calls the UI makes ...
            context.Report($"cast {casts}");
            yield return null;          // next frame
        }
    }
    finally
    {
        CardManager.Inst?.CancelDrag(); // runs on stop, timeout, scene change
    }
}
```

- One loop iteration is one frame. `yield return null` and `WaitForSeconds` work.
- Always end yourself with `context.Complete(reason)` when the job is done, and always clean up
  in `finally`; the SDK disposes the iterator on every end (stop, timeout, `replaced`,
  `target_gone`, `unavailable`, `run_ended`).
- `group:` makes behaviors exclusive: starting one stops the running one in the same group.
- Wait out the game's own input locks (`InteractionLock` here) instead of fighting them.
- A behavior calls game code, not input. Say so in the description, because the agent must not
  use it for a step that verifies the controls themselves.

## Verify

1. **Compile in the editor**, then make a development build (`BuildOptions.Development`). Release
   builds contain no game functions.
2. **Check the generated registration** in the build:

   ```bash
   strings -a <Build>/<Game>_Data/Managed/Assembly-CSharp.dll | grep -c __artel_call_
   strings -a -el <Build>/<Game>_Data/Managed/Assembly-CSharp.dll | grep -E "go_to_scene|kill_all"
   ```

   Names are stored as UTF-16, so plain `strings` does not find them; use `-el`.
3. **Check the catalog reached the server** once the game is connected (any revision above 0;
   startup registrations already bump it past 1):

   ```bash
   curl -s "$API/api/projects/<p>/game-builds/<b>/game-functions" -H "Authorization: Bearer $TOKEN"
   ```

   Every cheat and behavior must be listed with the parameters, ranges and enum values you wrote.
4. **Call each cheat once.** On a local orchestration server you can send an action straight to a
   connected instance through the internal port (never exposed in a deployment):

   ```bash
   curl -s -X POST http://127.0.0.1:8091/internal/action/<instanceId> -H 'Content-Type: application/json' \
     -d '{"type":"ACTION","id":9001,"actions":[{"id":1,"jsonrpc":"2.0","method":"invoke_cheat",
          "params":[{"name":"go_to_scene","args":{"scene":"Battle","stage":0}}]}]}'
   grep -o '"requestId":9001,.*' <orchestration log>
   ```

   Also call each one where it must be refused (outside a battle, a bad enum, an out-of-range
   number) and confirm the refusal text. Elsewhere, run a short QA scenario whose precondition
   needs the cheat and read the `invoke_cheat` rows in the run log.

Results seen on WordVenture (2026-10-16): `go_to_scene Battle` → `loading=TurnBattleScene stage=0`;
`kill_all_slimes` → `killed=1` per wave; `show_ending` → `loading=EndingScene stage_position=5`;
`kill_all_slimes` on the ending screen → `unavailable: not in a battle scene`.

## Failure messages the agent will see

| Message | Cause |
|---|---|
| `cheats are not available in this build` | release build |
| `unknown cheat: <name>` | not registered (typo, or the file is not compiled) |
| `bad argument <param>: <why>` | wrong type, out of range, unknown enum value |
| `unavailable: <reason>` | the `when` method said no |
| `no active <Component>` / `<n> <Component> objects; pass target as one of: ...` | instance cheat without a single live target |
| `cheat not allowed in this run: <name>` | the run or test case does not allow it (server side) |
| `<ExceptionType>: <message>` | your cheat threw; fix the cheat |

## References

- [Package README, "Cheats and behaviors for the QA agent"](https://github.com/project-artel/artel-sdk/blob/develop/Packages/kr.artel.sdk/README.md) — attribute reference (in develop once artel-sdk#142 merges)
- [Protocol](https://github.com/project-artel/artel-sdk/blob/develop/docs/protocol.md) — `invoke_cheat`, `start_behavior`, `stop_behavior`, `extend_behavior`, `GAME_FUNCTION_CATALOG`, `BEHAVIOR_ENDED`
- [ADR 0007](https://github.com/project-artel/artel-sdk/blob/develop/docs/adr/0007-game-registered-cheats-and-behaviors.md) — why cheats are generic actions and not part of the SDK vocabulary
- [Design and wire contract](https://github.com/project-artel/artel/tree/main/.plan/general) — `2026-10-13-game-registered-cheats-for-the-qa-agent.md`, `2026-10-13-game-registered-behaviors-the-agent-runs-in-background.md`, `2026-10-14-game-functions-wire-contract.md` (in main once artel#20 merges)
- [WordVenture implementation](https://github.com/project-artel/word-venture/blob/main/Assets/Scripts/Artel/WordVentureGameFunctions.cs) — the same file as `references/wordventure-example.cs` (in main once word-venture#10 merges)
