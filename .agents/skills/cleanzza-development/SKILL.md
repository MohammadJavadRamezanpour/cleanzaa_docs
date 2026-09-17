---
name: cleanzza-development
description: Use for any Cleanzza implementation, bug fix, refactor, API or database change, CI or infrastructure work, test, code review, feature documentation, or release task. Read and apply the current docs/development_rules.md before acting. Do not use for unrelated projects.
---

# Cleanzza development

The canonical development contract is `docs/development_rules.md` in the separate [Cleanzza documentation repository](https://github.com/MohammadJavadRamezanpour/cleanzaa_docs). This skill is an entry point, not a copy of those rules.

1. At the start of each Cleanzza development task, locate and read the current rules file. From the workspace root it is `docs/development_rules.md`; from the frontend or backend repository it is `../docs/development_rules.md`; from the documentation repository it is `development_rules.md`.
2. Follow the sections that apply to the task. In particular, distinguish the Git-flow rules for the frontend and backend repositories from the documentation repository's separate Git workflow.
3. Read `technical_architecture.md` and `phase_1_product_scope.md` in the documentation repository when the task touches architecture or Phase 1 behavior. Record unresolved product decisions instead of silently choosing a rule.
4. Before reporting completion, verify the applicable tests, documentation, API contract, migrations, CI checks, and delivery state required by the current development rules. Report any requirement that could not be verified.

If the documentation checkout is unavailable, obtain the current rules from the Cleanzza documentation repository before changing project code. Do not rely on memory or this skill as a substitute for the rules file.
