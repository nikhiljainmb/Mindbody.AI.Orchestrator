<!-- TEMPLATE — instantiate by copying to tasks/<YYYY-MM-DD-slug>/verification-checklist.md.
     Never edit this template in place. The orchestrator fills it before declaring DONE. -->

# Verification Checklist — <task-id>: <title>

The done gate. FINAL_STATUS.md may not say DONE while any box is unchecked. Judgment calls
(test sufficiency, coverage tier) follow the verification-protocol skill; coverage target
and commands come from ORCHESTRATOR_CONSTRAINTS.md.

- [ ] Build passes: `<command>` → <result>
- [ ] Full test suite passes: `<command>` → <passed>/<total> PASS, 0 failed
- [ ] Coverage of new/changed code >= target: <%> vs <target %> — Tier <1|2|3> per the
      verification-protocol skill
- [ ] Every acceptance criterion checked (from prd.md if present, else plan.md):
      - [ ] AC1: <criterion> → <evidence: command output or observed behavior>
      - [ ] AC2: <criterion> → <evidence>
- [ ] Merged review verdict is APPROVE or APPROVE_WITH_NITS
      (REVIEW_CHECKLIST.md, cycle <N> of <max>)

Any box unchecked → not DONE: run another fix cycle if within the max set in
ORCHESTRATOR_CONSTRAINTS.md, otherwise escalate in FINAL_STATUS.md.
