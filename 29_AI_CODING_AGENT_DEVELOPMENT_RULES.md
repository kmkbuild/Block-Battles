# 29_AI_CODING_AGENT_DEVELOPMENT_RULES.md
## Block Battles — AI Coding Agent Operating Manual

**Governing documents:** All `00` through `28` markdown specifications.
**Document status:** Layer D — Meta-Architecture / Agent Directives

**Inheritance confirmation:** This document is the absolute, non-negotiable operating manual for any LLM, AI coding assistant, or autonomous agent working on this codebase. It exists to prevent AI hallucinations, architecture degradation, and scope creep. 

---

## 1. AI Role

**You are an Implementer, not an Inventor.**
Your responsibility is to translate the explicit instructions in the Markdown documentation into highly optimized, testable Flutter, Flame, and Dart code. You do not invent mechanics, you do not balance gameplay on your own, and you do not introduce massive architectural patterns unless explicitly documented in Layer C (`18-28`).

---

## 2. Source-of-Truth Hierarchy

If information conflicts, you must strictly follow this order of precedence:

1. `AI_START_HERE.md` (If exists, contains dynamic project state)
2. `MASTER_DOCUMENT_INDEX.md`
3. Layer C (Engineering Docs `18-28`)
4. Layer B (UX/Presentation Docs `08-17`)
5. Layer A (Design Docs `00-07`)
6. Existing Implementation (`.dart` files)
7. Assumptions (Forbidden—ask the user instead)

**Conflict Resolution:** If existing code violates Layer C documentation, the documentation is correct. You must refactor the code to match the documentation, NOT modify the documentation to match bad code.

---

## 3. Project Workspace

- **Documentation:** `E:\Block Buster`
- **Game Project:** `E:\block battles` (when created, all code operations happen here)
- **Always run:** `flutter pub get` after modifying `pubspec.yaml`.
- **Always run:** `dart format lib/ test/` after writing Dart code.
- **Always run:** `flutter analyze` before declaring a task complete.
- **Always run:** `flutter test` before declaring a task complete.
- **Device testing:** `flutter run` on a connected Android device or emulator.

---

## 4. Before Writing Code

Before initiating any `write_to_file` or `replace_file_content` tool call, you MUST:
1. **Search:** Use `grep_search` to find the relevant classes and functions.
2. **Inspect:** Read the actual current implementation. Never assume the shape of a class based on its name.
3. **Identify:** Track downward and upward dependencies to understand how a change will cascade.
4. **Locate Specs:** Re-read the specific Markdown file governing the system you are about to edit.

*Blindly rewriting a file without reading its current state is strictly forbidden.*

---

## 5. Task Decomposition

Before executing a complex user request, output a brief plan breaking the task into:
- **Objective:** What is the specific goal?
- **Affected Systems:** Which engines/managers are touched?
- **Files:** The exact `.dart` files to be modified.
- **Dependencies:** What interfaces/providers must be updated?
- **Implementation:** How will the code change?
- **Verification:** Which test will prove it works?

---

## 6. Code Modification Rules

- **Surgical Edits:** Use `replace_file_content` for small changes. Do not overwrite a 500-line file to change one variable.
- **No Speculative Abstraction:** Do not create `AbstractBaseBlockManagerFactory` because you think it might be useful later. Write exactly what is needed for the MVP.
- **Preserve Working Code:** If tasked with fixing a UI bug, do not refactor the Combat Domain just because you noticed a minor inefficiency. Stay in scope.

---

## 7. Data-Driven Content Rule

If the user asks: *"Add an Ice Slime enemy that freezes blocks."*
**DO NOT:** Write `if (enemyName == "IceSlime") { FreezeBlocks(); }` in the core engine.
**DO:** 
1. Create a new `EnemyActionType.freezeBlocks` enum in Dart.
2. Update the `EnemyService` to handle that generic enum.
3. Update the `EnemyCatalog` dart file or JSON schema for the Ice Slime utilizing that enum.

---

## 8. Testing Rule

- If you modify math in the Domain layer (`BoardService`, `CombatService`), you MUST update or create a Dart test (`dart test`) to verify the change.
- Tests must be pure Dart and execute without Flutter/Flame dependencies if testing Domain logic.

---

## 9. Debugging Procedure

If you are asked to fix a bug, follow this exact sequence:
1. **Reproduce:** Understand the deterministic Seed and input causing it.
2. **Isolate:** Track the exact function throwing the error or yielding bad math.
3. **Root Cause Analysis (RCA):** Explain *why* it failed to the user.
4. **Patch:** Apply the fix.
5. **Verify:** Explain how the fix resolves the RCA.
6. **Regression Check:** State what other systems might be impacted by this fix.

---

## 10. Forbidden AI Behaviors

You will be immediately halted and corrected if you exhibit these behaviors:
- **Inventing Mechanics:** Adding rules not in `01-07`.
- **God Classes:** Putting new logic into a global manager instead of the correct modular Riverpod provider (`21`).
- **UI Logic:** Making Flutter widgets calculate damage (`23`).
- **Using Physics:** Using Flutter `GestureDetector` with raw physical coordinates for board placement instead of a mapped mathematical grid (`19`).
- **Hiding Errors:** Wrapping broken code in `try { } catch { }` without logging or fixing the root cause.
- **Using setState:** Using `setState()` or `StatefulWidget` for global game state (use Riverpod).
- **Mixing Concerns:** Putting game rules inside Flame components.
- **Domain Pollution:** Making domain services depend on Flame, Flutter, or UI packages (`lib/domain/` must have ZERO imports from Flutter/Flame).

---

## 11. Documentation Update Rule

The markdown files are living documents.
- If you add a new Relic Trigger Enum, you MUST update `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md` to reflect the new schema.
- If you add a new UI Modal, you MUST update the registry in `23_UI_IMPLEMENTATION_ARCHITECTURE.md`.

---

## 12. Change Log Procedure

Whenever you complete a coding step, you must explicitly log what was changed using the format in Section 22. Do not leave the user guessing what files you touched.

---

## 13. Human Approval Boundaries

You MUST stop and ask for human approval before:
- Adding a new package to `pubspec.yaml`.
- Deleting a Core Domain file.
- Modifying the Hive schema (`RunSaveData`) in a way that breaks backwards compatibility without writing a Migration script.
- Changing the overall color palette or visual aesthetic.

---

## 14. Dependency Management

- Obey Layer Boundaries. `lib/domain/` has no UI imports. `lib/presentation/` has UI imports.
- Use Riverpod Provider Injection for core logic (`21`). 

---

## 15. Asset Generation Rules

- Strictly enforce `snake_case.dart` naming for all Dart files and assets.
- Ensure generated UI sprites are white/grayscale for dynamic tinting if required by design.

---

## 16. UI Consistency Rules

- **Passive View:** Flutter widgets use Riverpod `watch()` and update visuals automatically (`23`).
- **Safe Area:** All full-screen screens must use the Flutter `SafeArea` widget.
- **Text:** Use standard Flutter `Text` widgets with the custom font defined in `pubspec.yaml`.

---

## 17. Performance Rules

- **Zero Allocation Gameplay:** You MUST avoid creating new heavy objects (like complex collections or allocations) inside the game loop or during dragging (`26`). Reuse objects where mathematically appropriate.
- **Object Pooling:** If you need to spawn hundreds of VFX particles, use Flame's pooling or component reuse mechanisms.

---

## 18. Security / Privacy Rules

- Do not introduce arbitrary analytics tracking without checking `16_SETTINGS`.
- Do not store plaintext PII (Personally Identifiable Information) in Hive boxes.

---

## 19. Git Rules

When executing git commands (if authorized):
- **Branching:** Create a branch for the feature (e.g., `feat/relic-system`).
- **Commit Style:** Use Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`).
- **Size:** Commit small, atomic changes.

---

## 20. Completion Definition

A task is ONLY complete when:
1. `flutter analyze` returns 0 issues.
2. The logic adheres perfectly to the Markdown architecture.
3. `dart test` passes.
4. `flutter test` passes.
5. The requested feature works correctly at runtime.
6. Any altered schemas are updated in the Markdown documentation.

---

## 21. AI Task Response Format

When you finish a work session, output your summary in this exact format:

```text
**TASK**: [Brief description of what was asked]
**CHANGED**: [High-level summary of what you did]
**WHY**: [Brief justification for the architectural approach]
**FILES**:
- [List of modified files]
- [List of created files]
**TESTS**: [Which tests were run/added to verify the change]
**RESULT**: [Success/Failure/Compilation Status]
**KNOWN ISSUES**: [Anything the user needs to know or fix manually]
**DOCUMENTATION IMPACT**: [Which .md files were updated, or "None"]
```

---

## 22. Final AI Engineering Constitution

1. **Read Before You Write:** Never edit a file blindly.
2. **Obey the Architecture:** Dependencies point inward. Domain Logic is decoupled from Flutter/Flame.
3. **Protect Performance:** No heavy allocations in the game loop.
4. **Data Over Code:** Expand the game through declarative catalogs, not massive `if/else` statements.
5. **Ask, Don't Guess:** If the documentation is missing a critical variable, ask the human. 

---

**End of `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md`.**
