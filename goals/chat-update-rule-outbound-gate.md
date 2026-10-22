# Goal: A chat edit can never make an existing rule send mail data somewhere new without the user doing it themselves in the rule editor

## Observable Truths
- [x] T1 When chat updateRule would add or change a FORWARD, SEND_EMAIL, REPLY or CALL_WEBHOOK action, nothing is written and the tool tells the user to make that change in the rule editor — exists: `changesOutboundBehavior` + `OUTBOUND_RULE_EDIT_ERROR` in `update-rule-tool.ts` — wired: runs before `partialUpdateRule` / `updateRuleActions` / `setRuleEnabled` — evidence: `npx vitest run utils/ai/assistant/chat-rule-tools.test.ts` → "blocks adding a FORWARD/SEND_EMAIL/REPLY/CALL_WEBHOOK action", "blocks changing the recipient…", "blocks changing other fields (cc, bcc, webhookUrl, delay)…" pass; same tests fail (6 failed) against the pre-fix file
- [x] T2 When chat updateRule changes the conditions of a rule that has an outbound action, nothing is written and the same message is returned — exists: condition branch of `changesOutboundBehavior` — wired: checks `updates.actions ?? originalActions` — evidence: "blocks widening the conditions of a rule that forwards mail" passes; fails on pre-fix code
- [x] T3 Safe chat edits still work: rename, pause/enable, label/archive/draft action changes, condition changes on rules without outbound actions — exists: unchanged write path — wired: existing `updateRuleTool` tests — evidence: `npx vitest run utils/ai/assistant utils/rule utils/risk` → 28 files, 345 tests passed, incl. "allows changing conditions and non-sending actions on rules without outbound actions"
- [x] T4 Re-sending an unchanged outbound action (model copies existing actions) is not blocked — exists: check runs on `effectiveUpdates` and compares via `normalizeActionForComparison` — wired: after `normalizeRuleUpdates` — evidence: "allows renaming and pausing a rule that forwards mail when the copied forward action is unchanged" passes (rename + setRuleEnabled called, updateRuleActions not called)

## Key Links
- Check must run on `effectiveUpdates` (after copied-field stripping), else T4 breaks.
- Check must run before any write; a partial write (conditions saved, actions refused) is the failure mode.
- Tool error must be visible to the model so it relays the editor hint, not retry in a loop.

## Out of scope, not done
- `createRule` lets a static-recipient FORWARD through without confirmation (risk check rates it low) — separate follow-up.
- Tool description still says direct rule edits are "already confirmed"; not changed to keep the model-facing surface unchanged. The error message carries the guidance.
- INTEGRATION actions are not gated: they post to the user's own connected integration, not an attacker-chosen destination.
- Confirm-card flow for update_rule (user chose block + editor).
- Other audit findings.
