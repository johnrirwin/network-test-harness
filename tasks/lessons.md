# Lessons Learned

Use this file to capture durable lessons after user corrections so future work improves over time.

## Entry template
- Date:
- Context:
- Correction:
- Preventive rule:

## Entries
- Date: 2026-05-04
  Context: GitHub actions for this repository
  Correction: Use the `johnrirwin` GitHub account, not `jirwin-lb`.
  Preventive rule: Before any GitHub CLI write action, verify the active account is `johnrirwin`; if auth is invalid or a different account is active, stop and re-authenticate before proceeding.
