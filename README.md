# Knight RPG

2D side-scrolling idle RPG made with Unity 6. The player enters the main stage from the start scene, automatically battles monsters, earns gold and experience, upgrades stats, and preserves progression through a JSON save system.

![Gameplay screenshot](Assets/Screenshots/lobby_repaired_play_game_view.png)

## Overview

- Genre: 2D side-scrolling auto-battle RPG / idle RPG
- Engine: Unity 6 `6000.3.10f1`
- Render pipeline: Universal Render Pipeline, 2D Renderer
- Platform target: Windows PC
- Main scenes: `Assets/Scenes/Start Scene.unity`, `Assets/Scenes/Main Game.unity`

## Key Features

- Auto-battle loop with player/enemy targeting, attack timing, hit handling, death handling, and reward flow
- Stage progression and balance data separated into ScriptableObject-driven configuration
- Monster spawning through weighted monster data and Unity object pooling
- Player growth through level, experience, gold, stat upgrades, equipment bonuses, and skills
- Skill system with loadout UI, skill data assets, effect types, and VFX playback
- Equipment system with slot-based equipment data and stat bonuses
- Gameplay windows for stat, skill, and equipment panels with centralized window state management
- Save/load system using `ISaveable`, `SaveManager`, `SaveData`, and JSON file storage
- Scene bootstrap and dependency registration through a lightweight custom `DIContainer`

## Gameplay Loop

```text
Start Scene
    -> Main Game
    -> Auto movement / enemy detection
    -> Auto battle
    -> Monster defeat
    -> Gold and EXP rewards
    -> Stat, equipment, and skill growth
    -> Save progression
```

## Technical Highlights

### Custom Runtime Architecture

The project uses a small custom dependency container instead of an external DI framework. `GameLifetimeScope` resolves important scene objects and manually injects dependencies into systems such as `AutoBattleController`, `SkillManager`, and `SaveManager`.

### Data-Driven Combat

Monsters, stages, rewards, skills, equipment, and level values are separated into data assets or balance classes so gameplay can be tuned without rewriting the core battle loop.

### Save System

Progression data is collected from systems that implement `ISaveable`, serialized into `SaveData`, and stored through `JsonFileSaveStorage`. Save requests are routed through `SaveEvents.RequestSave()`.

### UI Window Management

Gameplay panels are coordinated through `GameplayWindowManager` and `IGameplayWindow`, preventing stat, skill, and equipment windows from fighting over open/close state.

## Main Scripts

| Area | Scripts |
| --- | --- |
| Core | `GameManager`, `GameBootstrap`, `DIContainer`, `RootLifetimeScope` |
| Battle | `AutoBattleController`, `AutoBattleUnit`, `AutoBattleAI`, `AutoBattleSensor2D` |
| Stage/Reward | `StageManager`, `StageBalanceData`, `RewardBalanceData`, `BattleRewardService` |
| Growth | `PlayerResources`, `StatUpgradeManager`, `LevelBalanceData` |
| Skills | `SkillManager`, `SkillData`, `SkillVfxPlayer`, `SkillLoadoutUI` |
| Equipment | `EquipmentManager`, `EquipmentData`, `EquipmentSlotUI` |
| Save | `SaveManager`, `SaveData`, `JsonFileSaveStorage`, `ISaveable` |
| UI | `GameplayWindowManager`, `PlayerStatusUI`, `ExpBarUI`, `CoinTextCounter` |

## Project Structure

```text
Assets/
+-- Scenes/
|   +-- Start Scene.unity
|   +-- Main Game.unity
+-- _Project/
|   +-- Art/
|   +-- Prefabs/
|   +-- Scripts/
|   |   +-- Core/
|   |   +-- Gameplay/
|   |   +-- Save/
|   |   +-- Skills/
|   |   +-- Equipment/
|   |   +-- Editor/
|   +-- README.md
+-- Screenshots/
```

## How to Run

1. Install Unity `6000.3.10f1`.
2. Clone this repository.
3. Open the project from Unity Hub.
4. Open `Assets/Scenes/Start Scene.unity`.
5. Press Play.

## Build

The project includes these scenes in Build Settings:

- `Assets/Scenes/Start Scene.unity`
- `Assets/Scenes/Main Game.unity`

Example Windows build command:

```powershell
& "C:\Program Files\Unity\Hub\Editor\6000.3.10f1\Editor\Unity.exe" -batchmode -nographics -quit -projectPath . -buildWindows64Player "Builds\New-RPG.exe" -logFile "Logs\codex-unity-windows-build.log"
```

## Portfolio Notes

This project focuses on implementing the core systems of an idle RPG rather than only presenting a visual prototype. The main engineering work includes:

- Designing a reusable auto-battle flow
- Splitting combat, reward, stage, skill, equipment, and save responsibilities into separate systems
- Using ScriptableObject data for gameplay tuning
- Managing runtime dependencies without relying on a third-party DI package
- Persisting player progression through a structured save pipeline

## Assets

This project uses third-party/free asset packs for sprites, effects, and UI resources. They are included for portfolio and prototype purposes. See each asset folder for the original package files and license information where available.
