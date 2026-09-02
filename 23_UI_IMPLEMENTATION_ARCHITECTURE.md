# 23_UI_IMPLEMENTATION_ARCHITECTURE.md
## Block Battles — UI Implementation Architecture

**Governing documents:** `08_UI_UX_MASTER_SPECIFICATION.md`, `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md`, `18_TECHNICAL_ARCHITECTURE.md`, `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md`, `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document takes the wireframes and UX flows from Layer B and dictates the exact Unity Canvas, Component, and script structures required to build them. It enforces a strict Passive View (Model-View-Presenter) architecture to ensure AI coding agents can safely wire UI logic without corrupting game state.

---

## 1. UI Architecture Philosophy

**"The UI is Dumb. The UI is Reactive."**

1.  **Passive View:** A UI script (View) never calculates math, modifies player data, or decides if an action is valid. It only reads data and sends intents (Commands).
2.  **No Polling:** UI elements must never use `Update()` to check if the score has changed. They must subscribe to the `EventBus`.
3.  **Component Isolation:** Do not build a massive `UIManager.cs` that holds references to every Text object in the game. Each functional panel (e.g., `EnemyHUDView`) manages its own lifecycle and references.
4.  **Text Mesh Pro:** All text MUST use `TextMeshProUGUI`. Legacy `Text` components are forbidden.

---

## 2. Screen Registry

For every UI state, a specific Prefab and Controller exists.

| Prefab | Controller (Presenter) | Data Source | Opens On... | Closes On... |
|---|---|---|---|---|
| `pf_MainMenu` | `MainMenuView` | `SaveService` | Scene Load | Run Start |
| `pf_BattleHUD` | `BattleHUDView` | `RunModel`, `EventBus`| Battle Start | Run Over |
| `pf_SettingsModal` | `SettingsView` | `SettingsManager` | Settings Button | Close/Back |
| `pf_RelicSelect` | `RelicSelectionView`| `RunManager` | Battle Win | Choice confirmed |
| `pf_GameOverModal`| `DefeatView` | `RunManager` | `BoardLockedEvent`| End Run tapped |

---

## 3. UI Layer Structure (Canvas Sorting)

To prevent Z-fighting and ensure Modals always block gameplay, UI is strictly sorted via `Canvas.sortingOrder`.

| Layer | Sorting Order | Description | Contains |
|---|---|---|---|
| **Gameplay Board**| `0` | World space, handled outside UI | Board, Pieces |
| **Persistent HUD** | `10` | Static elements, non-blocking | Pause button, Meta-currency |
| **Gameplay HUD** | `20` | Dynamic combat info | Enemy HP, Objective bars |
| **Modal Layer** | `50` | Blocks input beneath it via Raycast | Pause, Relics, Game Over |
| **Feedback Layer** | `80` | Non-blocking ephemeral text | Damage Numbers, Popups |
| **Transition Layer**| `100` | Absolute top, blocks everything | Screen Fades (`pf_Transition`) |

---

## 4. Screen Lifecycle

Every UI View inherits from a base `UIView` class that standardizes its lifecycle:

1.  **`Initialize()`:** Caches internal `GetComponent` references.
2.  **`Show(data)`:** Binds the provided data to the UI elements. Subscribes to `EventBus`. Triggers the entrance animation (e.g., DOTween fade/scale). Sets `CanvasGroup.interactable = true`.
3.  **`Hide()`:** Reverses animation. Sets `CanvasGroup.interactable = false`. Unsubscribes from `EventBus`.
4.  **`Dispose()`:** Destroys the GameObject (or returns to pool).

---

## 5. UI State Management

Because the UI is isolated, a lightweight `UIOrchestrator` (living in the Presentation Layer) manages the Modal stack.

- If the player opens `Settings` over the `PauseMenu`, the Orchestrator tracks both. 
- When the Android Hardware Back Button is pressed, the Orchestrator pops the top-most modal and calls `Hide()` on it.
- **Rule:** Opening a Modal automatically disables `GraphicRaycaster` on the Gameplay HUD to prevent accidental piece drops behind the modal.

---

## 6. Data Binding (Event-Driven)

We do not use complex MVVM data-binding frameworks (too heavy/magical for simple AI logic). We use direct event subscriptions.

```csharp
public class EnemyHUDView : MonoBehaviour {
    [SerializeField] private TextMeshProUGUI _hpText;
    [SerializeField] private Image _hpFill;

    private void OnEnable() {
        EventBus<DamageDealtEvent>.OnEvent += UpdateHP;
    }
    
    private void OnDisable() {
        EventBus<DamageDealtEvent>.OnEvent -= UpdateHP;
    }

    private void UpdateHP(DamageDealtEvent evt) {
        _hpText.text = $"{evt.newCurrentHP} / {evt.maxHP}";
        _hpFill.fillAmount = (float)evt.newCurrentHP / evt.maxHP;
        // Trigger shake animation here
    }
}
```

---

## 7. Objective UI

- **Prefab:** `pf_ObjectivePanel`
- **Behavior:** Reads `ObjectiveData`. When `ObjectiveProgressedEvent` fires, the fill bar lerps to the new value over 200ms. 
- **Completion:** On `ObjectiveCompletedEvent`, the bar flashes green (`CLR-SEM-SUCCESS`), and the `pf_VFX_ObjectiveBurst` particle system plays over the UI transform.

---

## 8. Enemy HP UI

- **Prefab:** `pf_EnemyHUD`
- **Behavior:** Contains portrait, name, and HP bar. 
- **Damage Numbers:** `DamageDealtEvent` instantiates a `pf_DamageText` prefab at the anchor point of the EnemyHUD, which floats upward and fades out. This is managed by a `FloatingTextController` to prevent overlapping numbers.

---

## 9. Relic Choice UI

- **Prefab:** `pf_RelicSelectModal` contains a Horizontal Layout Group with 3 `pf_RelicCardView` children.
- **Behavior:** 
  1. Opens on `BattleCompletedEvent`.
  2. Receives an array of 3 random `RelicData` objects from the Domain layer.
  3. Binds Data: Icon, Name, and Description text is localized and populated.
  4. On Click: Sends `CommandSelectRelic(relicID)` to the Application layer, then calls `Hide()`.

---

## 10. Victory & Game Over UI

- **Victory (`pf_VictoryModal`):** Shows Run stats (floors cleared, total score). Simple, triumphant, fast.
- **Game Over (`pf_DefeatModal`):** 
  - Dynamic Title based on defeat reason ("Board Locked" vs "Out of Moves").
  - (MVP+) Contains the "Watch Ad to Continue" Revive button. The button's `interactable` state is bound to `MonetizationService.IsAdReady()`.

---

## 11. Settings UI

- **Prefab:** `pf_SettingsModal`
- **Controls:** `UnityEngine.UI.Slider` for volumes, `UnityEngine.UI.Toggle` for Haptics and Reduced Motion.
- **Binding:** Subscribes to `SettingsManager.OnSettingsChanged` to update slider visuals. Moving a slider triggers `SettingsManager.SetVolume(val)` instantly (No "Save" button required).

---

## 12. Tutorial UI

- **Prefab:** `pf_TutorialTooltip`
- **Behavior:** A reusable floating tooltip (Text + Arrow). 
- **Positioning:** Receives a `RectTransform` target and anchors itself relative to that target (e.g., points at the first tray piece). 
- **Dismissal:** Destroys itself as soon as the relevant `EventBus` action occurs (e.g., `PiecePickedUpEvent`).

---

## 13. Responsive Layout

Block Battles targets a 9:16 portrait layout but must support iPads (3:4) and ultra-tall phones (9:21).

1. **Canvas Scaler:** 
   - UI Scale Mode: `Scale With Screen Size`
   - Reference Resolution: `1080 x 1920`
   - Screen Match Mode: `Match Width Or Height` (Set to 0.5 for balanced scaling, or 0 to strictly match width on tall phones).
2. **Anchoring:** Never use fixed pixel positions. 
   - The Tray is anchored `Bottom-Center`. 
   - The Enemy HUD is anchored `Top-Center`. 
   - The Board bounds fit within the remaining flexible middle space.

---

## 14. Safe Areas

All full-screen UI panels MUST have a root child object named `SafeArea` with a `SafeAreaFitter` script attached. This script modifies the `RectTransform`'s `anchorMin` and `anchorMax` to match `Screen.safeArea`, preventing UI from rendering under iPhone notches or Android punch-holes.

---

## 15. Animation Hook Architecture

As required by `12` (Game Feel) and `22` (Events), the UI is responsible for telling the core game state when it is done animating.

```csharp
public void PlayEnemyHitAnimation() {
    // UI animates the hit (using DOTween)
    transform.DOShakePosition(0.2f, strength: 5f).OnComplete(() => {
        // Signals back to Orchestrator to unlock the game state
        EventBus<UIAnimationCompleteEvent>.Raise(new UIAnimationCompleteEvent());
    });
}
```
*Rule:* Game logic never waits `yield return new WaitForSeconds(0.2f)`. It always waits for the UI to signal completion.

---

## 16. Input Handling

- **GraphicRaycaster:** Required on `Canvas_HUD` and `Canvas_Modals`.
- **Optimization:** REMOVE the `GraphicRaycaster` from static UI (like background art). Uncheck `Raycast Target` on all `Text` and `Image` components that do not need to be clicked. This massively improves UI performance on mobile.
- **EventSystem:** One `EventSystem` prefab initialized in the `00_Boot` scene.

---

## 17. Accessibility

- **Contrast:** Verified visually; UI relies on `10_COLOR_SYSTEM` tokens.
- **Labels:** Any button using an Icon instead of Text (e.g., the Pause button) must have an invisible `TextMeshProUGUI` component or a custom `AccessibilityLabel` script attached for mobile screen readers to announce the intent.
- **Reduced Motion:** If `SettingsManager.ReducedMotion` is true, DOTween animations on UI panels bypass their durations (set to 0ms) and snap directly to their final states.

---

## 18. Localization Integration

All `TextMeshProUGUI` components holding static text must be accompanied by a `LocalizedText` component.

```csharp
[RequireComponent(typeof(TextMeshProUGUI))]
public class LocalizedText : MonoBehaviour {
    public string localizationKey; // e.g., "ui.btn.start"

    private void Start() {
        GetComponent<TextMeshProUGUI>().text = LocalizationService.Get(localizationKey);
    }
}
```
*Expansion Buffer:* All UI buttons and panels are constructed with Horizontal/Vertical Layout Groups to automatically expand if the German or Russian translation is 30% longer than the English text (`15` Sec 18).

---

## 19. UI Performance

- **Canvas Batching:** Modifying a single UI element dirties the entire Canvas, forcing a rebuild.
- **Solution:** Static UI (Backgrounds, Headers) lives on a different Canvas than highly dynamic UI (Damage numbers, HP bars). 
- **Object Pooling:** Damage numbers are pooled. We do not `Instantiate` or `Destroy` TextMeshPro objects during combat; doing so causes fatal GC spikes.

---

## 20. UI Testing

- **PlayMode Tests:** Use Unity's Test Framework to simulate clicks.
```csharp
[UnityTest]
public IEnumerator PauseButton_OpensSettingsModal() {
    // Simulate click on pause button
    pauseButton.onClick.Invoke();
    yield return null;
    // Assert settings modal is active
    Assert.IsTrue(settingsModal.gameObject.activeInHierarchy);
}
```

---

## 21. Final UI Implementation Contract

> **Information is beautiful when it is clean and correct.**
>
> The UI in Block Battles is a faithful, passive reflection of the core mathematical simulation. It enforces strict separation from gameplay logic, relies on standard Unity layout systems for robust responsiveness, and prioritizes localization and accessibility from Day 1. 
> 
> No string is hardcoded. No animation blocks the thread. No piece of text exists without a purpose.

---

**End of `23_UI_IMPLEMENTATION_ARCHITECTURE.md`.**
