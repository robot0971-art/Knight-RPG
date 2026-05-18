# 프로젝트 가이드

이 문서는 이 Unity 프로젝트에서 기능을 추가하거나 수정할 때 따라야 할 현재 기준입니다. 예전 문서에 있던 VContainer 중심 설명은 현재 코드와 맞지 않으므로, 이 문서는 실제 구현을 기준으로 정리합니다.

## 프로젝트 개요

- 장르: 2D 횡스크롤 자동 전투 RPG / 방치형 전투 게임.
- 엔진: Unity 6 `6000.3.10f1`.
- 렌더링: Universal Render Pipeline, 2D Renderer.
- 주요 씬: `Assets/Scenes/Start Scene.unity`, `Assets/Scenes/Main Game.unity`.
- 핵심 흐름: 시작 씬에서 메인 게임으로 진입한 뒤, 플레이어와 몬스터가 자동 전투를 진행하고 골드, 경험치, 스탯, 장비, 스킬 상태를 저장합니다.

## 실제 기술 스택

- DI: 외부 DI 프레임워크가 아니라 프로젝트 자체 `DIContainer`를 사용합니다.
- 전역 상태: `GameManager.Instance`와 `DIContainer.Global`을 함께 사용합니다.
- 수동 주입: `GameLifetimeScope`가 씬 오브젝트를 찾아 등록하고 `Construct(...)` 메서드로 필요한 의존성을 연결합니다.
- 오브젝트 풀: `UnityEngine.Pool.ObjectPool<T>`를 사용합니다.
- 입력: New Input System을 사용합니다. 현재 이동 입력은 `Keyboard.current` 기반입니다.
- 저장: `ISaveable`, `SaveManager`, `JsonFileSaveStorage`, `SaveData` 구조를 사용합니다.
- 데이터: 몬스터, 스킬은 `ScriptableObject` 데이터 에셋을 사용합니다.

## 폴더 구조

- `Assets/_Project/Scripts/Core`
  - `GameManager`, `GameBootstrap`, `GameState`, `DIContainer`, `RootLifetimeScope`.
- `Assets/_Project/Scripts/Gameplay`
  - 자동 전투, 스테이지, 플레이어 리소스, 스탯 강화, HUD, 배경 스크롤, 경험치/골드 UI.
- `Assets/_Project/Scripts/Save`
  - 저장 인터페이스, 저장 데이터, JSON 파일 저장소, 저장 이벤트.
- `Assets/_Project/Scripts/Skills`
  - 스킬 데이터, 스킬 매니저, 스킬 UI, 스킬 VFX, 전투 타겟 인터페이스.
- `Assets/_Project/Scripts`
  - 장비 데이터, 장비 매니저,  UI.
- `Assets/_Project/Scripts/Editor`
  - 에디터 전용 보조 도구. 런타임 코드와 분리합니다.

## 아키텍처 기준

- `GameManager`는 게임 상태(`Booting`, `Title`, `Playing`, `Paused`, `GameOver`)와 골드를 관리합니다.
- `GameBootstrap`은 `GameManager`가 없으면 생성하고 시작 상태를 설정합니다.
- `DIContainer`는 간단한 전역 레지스트리입니다. 등록된 객체가 없으면 `FindFirstObjectByType<T>()`로 fallback을 시도합니다.
- `GameLifetimeScope`는 메인 게임 씬의 핵심 참조를 해결하고 `AutoBattleController`, `SkillManager`, `SaveManager`에 의존성을 주입합니다.
- `RootLifetimeScope`는 현재 전역 `GameManager` 등록 정도만 담당합니다. VContainer의 `LifetimeScope`가 아닙니다.

## 코딩 규칙

- 새 런타임 컴포넌트는 특별히 상속을 허용해야 하는 이유가 없다면 `sealed`로 작성합니다.
- 인스펙터에서 설정할 값은 `[SerializeField] private` 필드를 우선 사용합니다.
- 런타임에서 잦은 생성/삭제가 일어나는 오브젝트는 `UnityEngine.Pool` 기반 풀링을 우선 검토합니다.
- 애니메이터 상태명은 반복 호출 구간에서 문자열 직접 호출보다 `Animator.StringToHash` 캐시를 선호합니다.
- 매 프레임 `.material` 접근으로 머티리얼 인스턴스를 늘리지 않습니다. 필요하면 캐시, `sharedMaterial`, `MaterialPropertyBlock`을 사용합니다.
- 씬 참조가 필요한 경우 먼저 인스펙터 연결을 고려하고, 자동 해결이 필요할 때만 `DIContainer.Global.Resolve<T>()` 또는 `FindFirstObjectByType<T>()`를 사용합니다.
- `GameManager.Instance`는 현재 코드에서 사용 중인 fallback이므로 제거하지 말고, 새 코드에서는 가능하면 `DIContainer` 등록 흐름과 충돌하지 않게 사용합니다.

## 전투와 성장

- 전투 흐름은 `AutoBattleController`가 조율합니다.
- 유닛의 체력, 공격력, 공격 속도, 치명타, 경험치, 사망 처리는 `AutoBattleUnit` 중심입니다.
- 몬스터 추가는 `AutoBattleUnit` 상속이 아니라 `MonsterData`와 몬스터 prefab 설정을 기준으로 합니다. `AutoBattleUnit`은 `sealed`입니다.
- 스테이지와 보스 진행은 `StageManager`가 관리합니다.
- 스탯 강화는 `StatUpgradeManager`, 장비 보너스는 `EquipmentManager`, 스킬 효과는 `SkillManager`와 `SkillVfxPlayer`를 확인합니다.

## 저장 시스템

- 저장 대상은 `ISaveable`을 구현합니다.
- 저장 데이터는 `SaveData`에 필드를 추가하고, 각 시스템의 `CaptureSaveData` / `RestoreSaveData`에서 반영합니다.
- 저장 트리거는 `SaveEvents.RequestSave()`를 사용합니다.
- 저장 파일은 기본적으로 `JsonFileSaveStorage("save.json")` 흐름을 따릅니다.

## UI 규칙

- UI 패널은 가능하면 `IGameplayWindow`를 구현해 `GameplayWindowManager`와 함께 동작하게 합니다.
- 스탯 창, 스킬 창, 장비 창은 서로 열림 상태가 충돌하지 않도록 기존 `GameplayWindowManager` 패턴을 확인합니다.
- TextMeshPro와 legacy `Text`가 혼재되어 있으므로 기존 컴포넌트 타입을 먼저 확인하고 맞춰서 수정합니다.

## 에디터/빌드 주의사항

- 에디터 전용 코드는 반드시 `Assets/_Project/Scripts/Editor` 아래에 두거나 `#if UNITY_EDITOR`로 감쌉니다.
- 빌드 포함 씬은 `ProjectSettings/EditorBuildSettings.asset`의 `Start Scene`, `Main Game`입니다.
- Windows 빌드 검증 예시:

```powershell
& "C:\Program Files\Unity\Hub\Editor\6000.3.10f1\Editor\Unity.exe" -batchmode -nographics -quit -projectPath . -buildWindows64Player "Builds\New-RPG.exe" -logFile "Logs\codex-unity-windows-build.log"
```

## 한글 인코딩

- 이 파일은 Windows PowerShell 5에서도 한글이 깨질 가능성을 줄이기 위해 UTF-8 BOM으로 저장합니다.
- 새 Markdown, JSON, C# 파일은 UTF-8을 사용합니다.
- PowerShell에서 한글이 깨져 보이면 파일이 망가진 것이 아니라 콘솔 코드페이지 문제일 수 있습니다. 확인 전 `chcp 65001` 또는 PowerShell 7 사용을 권장합니다.

## 피해야 할 오래된 가이드

- 현재 프로젝트는 VContainer를 사용하지 않습니다. `VContainer`, `[Inject]`, `LifetimeScope` 기반으로 새 구조를 만들지 마세요.
- `IService` 인터페이스는 현재 프로젝트에 없습니다. 새 서비스를 만들 때 문서만 보고 `IService`를 추가하지 마세요.
- `AutoBattleUnit`은 상속 확장 대상이 아닙니다. 몬스터와 스탯 차이는 데이터와 prefab 설정으로 처리합니다.
