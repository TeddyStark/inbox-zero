# Goal: A chat edit can never make an existing rule send mail data somewhere new without the user doing it themselves in the rule editor

## Observable Truths
- [ ] T1 When chat updateRule would add or change a FORWARD, SEND_EMAIL, REPLY or CALL_WEBHOOK action, nothing is written and the tool tells the user to make that change in the rule editor — exists: outbound-action check in `update-rule-tool.ts` — wired: runs before `partialUpdateRule` / `updateRuleActions` — evidence: <filled at the end>
- [ ] T2 When chat updateRule changes the conditions of a rule that has an outbound action, nothing is written and the same message is returned — exists: same check, condition branch — wired: uses merged post-update actions — evidence: <filled at the end>
- [ ] T3 Safe chat edits still work: rename, pause/enable, label/archive/draft action changes, condition changes on rules without outbound actions — exists: unchanged write path — wired: existing tests in `chat-rule-tools.test.ts` — evidence: <filled at the end>
- [ ] T4 Re-sending an unchanged outbound action (model copies existing actions) is not blocked — exists: comparison via existing `normalizeRuleUpdates` — wired: check runs on `effectiveUpdates`, not raw `updates` — evidence: <filled at the end>

## Key Links
- Check must run on `effectiveUpdates` (after copied-field stripping), else T4 breaks.
- Check must run before any write; a partial write (conditions saved, actions refused) is the failure mode.
- Tool error must be visible to the model so it relays the editor hint, not retry in a loop.

## Out of scope, not done
- `createRule` lets a static-recipient FORWARD through without confirmation (risk check rates it low) — separate follow-up.
- Confirm-card flow for update_rule (user chose block + editor).
- Other audit findings.
