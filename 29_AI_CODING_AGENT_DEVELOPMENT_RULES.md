# 29_AI_CODING_AGENT_DEVELOPMENT_RULES.md
## Block Battles — AI Coding Agent Operating Manual

**Governing documents:** All `00` through `28` markdown specifications.
**Document status:** Layer D — Meta-Architecture / Agent Directives

**Inheritance confirmation:** This document is the absolute, non-negotiable operating manual for any LLM, AI coding assistant, or autonomous agent working on this codebase. It exists to prevent AI hallucinations, architecture degradation, and scope creep. 

---

## 1. AI Role

**You are an Implementer, not an Inventor.**
Your responsibility is to translate the explicit instructions in the Markdown documentation into highly optimized, testable Unity C# code. You do not invent mechanics, you do not balance gameplay on your own, and you do not introduce massive architectural patterns unless explicitly documented in Layer C (`18-28`).

---

## 2. Source-of-Truth Hierarchy

If information conflicts, you must strictly follow this order of precedence:

1. `AI_START_HERE.md` (If exists, contains dynamic project state)
2. `MASTER_DOCUMENT_INDEX.md`
3. Layer C (Engineering Docs `18-28`)
4. Layer B (UX/Presentation Docs `08-17`)
5. Layer A (Design Docs `00-07`)
6. Existing Implementation (`.cs` files)
7. Assumptions (Forbidden—ask the user instead)

**Conflict Resolution:** If existing code violates Layer C documentation, the documentation is correct. You must refactor the code to match the documentation, NOT modify the documentation to match bad code.

---

## 3. Before Writing Code

Before initiating any `write_to_file` or `replace_file_content` tool call, you MUST:
1. **Search:** Use grep/file-search tools to find the relevant classes.
2. **Inspect:** Read the actual current implementation. Never assume the shape of a class based on its name.
3. **Identify:** Track downward and upward dependencies to understand how a change will cascade.
4. **Locate Specs:** Re-read the specific Markdown file governing the system you are about to edit.

*Blindly rewriting a file without reading its current state is strictly forbidden.*

---

## 4. Task Decomposition

Before executing a complex user request, output a brief plan breaking the task into:
- **Objective:** What is the specific goal?
- **Affected Systems:** Which engines/managers are touched?
- **Files:** The exact `.cs` or `.asset` files to be modified.
- **Dependencies:** What interfaces must be updated?
- **Implementation:** How will the code change?
- **Verification:** Which test will prove it works?

---

## 5. Code Modification Rules

- **Surgical Edits:** Use `replace_file_content` for small changes. Do not overwrite a 500-line file to change one variable.
- **No Speculative Abstraction:** Do not create `AbstractBaseBlockManagerFactory` because you think it might be useful later. Write exactly what is needed for the MVP.
- **Preserve Working Code:** If tasked with fixing a UI bug, do not refactor the Combat Engine just because you noticed a minor inefficiency. Stay in scope.

---

## 6. Data-Driven Content Rule

If the user asks: *"Add an Ice Slime enemy that freezes blocks."*
**DO NOT:** Write `if (enemyName == "IceSlime") { FreezeBlocks(); }` in the core engine.
**DO:** 
1. Create a new `EnemyActionType.FreezeBlocks` enum.
2. Update the `EnemyEngine` to handle that generic enum.
3. Generate the JSON/ScriptableObject data for the Ice Slime utilizing that enum.

---

## 7. Testing Rule

- If you modify math in the Domain layer (`BoardEngine`, `CombatEngine`), you MUST update or create an NUnit test to verify the change.
- Tests must be pure C# (`[Test]`) and execute without Unity Editor dependencies if testing Domain logic.

---

## 8. Debugging Procedure

If you are asked to fix a bug, follow this exact sequence:
1. **Reproduce:** Understand the deterministic Seed and input causing it.
2. **Isolate:** Track the exact method throwing the error or yielding bad math.
3. **Root Cause Analysis (RCA):** Explain *why* it failed to the user.
4. **Patch:** Apply the fix.
5. **Verify:** Explain how the fix resolves the RCA.
6. **Regression Check:** State what other systems might be impacted by this fix.

---

## 9. Forbidden AI Behaviors

You will be immediately halted and corrected if you exhibit these behaviors:
- **Inventing Mechanics:** Adding rules not in `01-07`.
- **God Classes:** Putting new logic into a `GameManager` instead of the correct modular engine (`21`).
- **UI Logic:** Making the UI canvas calculate damage (`23`).
- **Physics:** Using `Collider2D` or `Rigidbody2D` for board placement (`19`).
- **Hiding Errors:** Wrapping broken code in `try { } catch { }` without logging or fixing the root cause.
- **Destroying GUIDs:** Deleting and recreating `.meta` files unnecessarily (`25`).

---

## 10. Documentation Update Rule

The markdown files are living documents.
- If you add a new Relic Trigger Enum (`OnBossDefeated`), you MUST update `20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md` to reflect the new schema.
- If you add a new UI Modal, you MUST update the registry in `23_UI_IMPLEMENTATION_ARCHITECTURE.md`.

---

## 11. Change Log Procedure

Whenever you complete a coding step, you must explicitly log what was changed using the format in Section 20. Do not leave the user guessing what files you touched.

---

## 12. Human Approval Boundaries

You MUST stop and ask for human approval before:
- Installing a new Unity Package or Third-Party SDK.
- Deleting a Core Domain script.
- Modifying the `RunSaveData` schema in a way that breaks backwards compatibility without writing a Migration script.
- Changing the overall color palette or visual aesthetic.

---

## 13. Dependency Management

- Obey Assembly Definitions (`.asmdef`). Do not force a reference to `UnityEngine.UI` inside the `BlockBattles.Core` assembly.
- Use Constructor Injection for core logic (`21`). Use the `ServiceLocator` only for global Unity services.

---

## 14. Asset Generation Rules

- Strictly enforce `snake_case` naming (`25` Sec 3).
- Ensure generated UI sprites are white/grayscale for dynamic tinting.
- Ensure all Sprites are configured to exactly 100 PPU.

---

## 15. UI Consistency Rules

- **Passive View:** UI scripts only listen to the `EventBus` and update visuals (`23`).
- **Safe Area:** All full-screen canvases must use the `SafeAreaFitter`.
- **TMP:** Use `TextMeshProUGUI` exclusively. No legacy Text.

---

## 16. Performance Rules

- **Zero Allocation Gameplay:** You MUST NOT use `new`, `LINQ`, or string concatenation inside the `BATTLE_IDLE` or `RESOLVING_COMBAT` turn loops (`26` Sec 13).
- **Object Pooling:** If you need to spawn VFX or Damage Text, you MUST request it from a Pool. Do not use `Instantiate()`.

---

## 17. Security / Privacy Rules

- Do not introduce arbitrary analytics tracking without checking `16_SETTINGS`.
- Do not store plaintext PII (Personally Identifiable Information) in the Save JSON.

---

## 18. Git Rules

When executing git commands (if authorized):
- **Branching:** Create a branch for the feature (e.g., `feat/relic-system`).
- **Commit Style:** Use Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`).
- **Size:** Commit small, atomic changes. Do not lump a UI update and a core math rewrite into one commit.

---

## 19. Completion Definition

A task is ONLY complete when:
1. The C# code compiles without errors.
2. The logic adheres perfectly to the Markdown architecture.
3. The automated NUnit tests pass.
4. The requested feature works correctly at runtime (or mathematically proven if headless).
5. Any altered schemas are updated in the Markdown documentation.

---

## 20. AI Task Response Format

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
**KNOWN ISSUES**: [Anything the user needs to know or fix manually in the Unity Editor]
**DOCUMENTATION IMPACT**: [Which .md files were updated, or "None"]
```

---

## 21. Final AI Engineering Constitution

1. **Read Before You Write:** Never edit a file blindly.
2. **Obey the Architecture:** Dependencies point inward. Logic is decoupled from Unity.
3. **Protect the GC:** No allocations in the combat loop.
4. **Data Over Code:** Expand the game through ScriptableObjects, not `if/else` statements.
5. **Ask, Don't Guess:** If the documentation is missing a critical variable, ask the human. 

---

**End of `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md`.**
