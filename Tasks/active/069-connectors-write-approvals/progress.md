# Progress: Connectors — every outside write is a proposal only the connection owner approves

## Checklist
- [x] proposeOnly for writes
- [x] MCP proposal filing
- [x] connection_owner gate
- [x] Send-what-was-approved + single attempt
- [x] Tests

## Last step
Built and committed locally on `feat/069-connectors-write-approvals` (branched from 068). `tests/connector-write-approvals.test.js` passes 19/19; the 120 agent, MCP, proposal, Slack and connector suites pass. Not pushed.

## Blockers
None. Like 068, a write resolves the owner's connection through the 063 seam, so on this branch it runs only against the fake provider until track A's resolver lands.

## Log

### 2026-10-28
- Task created from the split of 056; acceptance criteria drawn from the 056 PRD.
- Assigned to track B in the two-track split; depends only on its own track and the 063 contract.

### 2026-10-30
- Moved to active; branch `feat/069-connectors-write-approvals` cut from 068.
- Write tools (`kind: 'write'`) now need a `card(input)`, or the provider's tools do not load (`Modules/Connectors/tools.js`).
- Agents: each write is a registry action with `proposeOnly: true` and the new gate `connection_owner`, so no autonomy level makes it and it is never grantable or approvable "always" (`Modules/Agents/registry/connectorTools.js`). A run files it with the run's person as `connectionOwner` (`engine/graph.js`); the generic propose route refuses a connector write.
- MCP: each write is a tool behind `connectors:write` with `alwaysProposed`, so a call is filed as a proposal and the reply says nothing was sent and that it waits in the Inbox (`Modules/Mcp/connectorTools.js`, `tools.js`, `propose.js`). Approval asks the grant or token again for `connectors:write` (`Mcp/approval.js`).
- Gate: only `connectionOwner` approves; an owner or admin is refused, and the Inbox lists the row as locked for everyone else (`Modules/Connectors/writes.js`, `proposals.js`, `approverRights.js`). Anyone who may decline still can.
- Send: `Modules/Connectors/writes.js` checks the input against the tool's schema on the owner's own connection when filed, stores the params with that `connectionId` and the card (target, title, whole text, attachments) on the change. The executor sends exactly those params, once: an attempt is claimed on the proposal (`sendAttempts`) before the provider is called, and the answer or error is kept in `delivery`. An edit is checked again and saved back as pending with a new card (`reshown: true`), never sent on the same click.
- Card UI is left to 078; the backend payload is `changes[].card`.
- Windows run: 5 conventions suites (role-playbooks, style-check, ai-core-boundary, plain-words, ci-e2e-shards) fail the same way with these changes stashed, so none come from this task.
