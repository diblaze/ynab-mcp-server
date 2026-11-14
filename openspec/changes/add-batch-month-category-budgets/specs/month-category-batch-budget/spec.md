## Purpose

Let an assistant set the budgeted amounts of up to 50 categories in one budget month from a single reviewed preview and a single explicit human approval, finishing in about a minute, with every item verified, the batch stopped at the first failure, and partial outcomes recoverable.

## ADDED Requirements

### Requirement: Batch month budget write
The system SHALL set the budgeted amount of every listed category in one month to its proposed absolute dollar amount after one approved preview, and report each item in dollars.

#### Scenario: Batch applies every item
- **GIVEN** two categories in a month with budgeted amounts that differ from the proposed amounts
- **WHEN** the user approves one preview for the batch and the tool runs with that preview token and confirmed true
- **THEN** both categories read back with the proposed budgeted amounts
- **AND** the result reports status completed and, per item, the category, current and proposed amounts in dollars, status applied and outcome verified_applied

### Requirement: Batch preview shows current and proposed amounts
The system SHALL show, in the preview, the current and the proposed budgeted amount for every item in the batch.

#### Scenario: Preview lists every item
- **GIVEN** a batch of two categories in a month
- **WHEN** the batch is previewed
- **THEN** the preview lists both categories with their current and their proposed budgeted amounts
- **AND** no write is sent

### Requirement: Invalid batch rejected before preview
The system SHALL refuse to issue a preview for a batch that cannot be applied unambiguously.

#### Scenario: Duplicate category refused
- **GIVEN** a batch that lists the same category twice
- **WHEN** the batch is previewed
- **THEN** the preview fails with an error naming the duplicate category and no token is issued

#### Scenario: Unknown category refused
- **GIVEN** a batch with a category id that does not exist in that month
- **WHEN** the batch is previewed
- **THEN** the preview fails with an error naming that category and no token is issued

#### Scenario: Oversized batch refused
- **GIVEN** a batch with more than 50 items
- **WHEN** the batch is previewed
- **THEN** the preview fails with an input validation error and no token is issued

#### Scenario: Malformed month refused
- **GIVEN** a month that is not the first day of a month in YYYY-MM-DD form
- **WHEN** the batch is previewed
- **THEN** the preview fails with the month format error and no token is issued

### Requirement: Stale batch aborts before any write
The system SHALL refuse to write any item when the budgeted amount of any item changed after the preview.

#### Scenario: Changed item after preview
- **GIVEN** a previewed batch
- **WHEN** one item's budgeted amount is changed outside the tool before execution
- **THEN** execution fails with a stale-preview error
- **AND** no write is sent and no operation is journaled

### Requirement: Batch stops at the first failure
The system SHALL stop at the first item that cannot be verified as applied, never retry it automatically, and report every item's status.

#### Scenario: Write failure stops the batch
- **GIVEN** an approved batch of two categories where the first write is rejected by YNAB
- **WHEN** the batch runs
- **THEN** the result reports status partial with recovery required
- **AND** the first item is reported pending and the second item is never written

#### Scenario: Final readback mismatch reported
- **GIVEN** an approved batch where a written item's stored amount differs from the write response by the time of the final readback
- **WHEN** the batch finishes writing
- **THEN** the result reports status conflict with recovery required
- **AND** that item is reported conflict with the observed amount in dollars

#### Scenario: Final readback unavailable
- **WHEN** every item was written but the final month read fails
- **THEN** the result is `status: unknown` with `outcome: final_readback_unavailable` and `recovery_required: true`

#### Scenario: Partial batch resumes after new approval
- **GIVEN** a partial batch from a rejected write on its second item
- **WHEN** the user previews and approves resume_operation for that operation
- **THEN** only the second item is written and the operation reports completed

### Requirement: Unchanged items are not written
The system SHALL record an item whose current budgeted amount already equals the proposed amount as unchanged without sending a write for it.

#### Scenario: Unchanged item skipped
- **GIVEN** a batch where one item's proposed amount equals its current amount
- **WHEN** the approved batch runs
- **THEN** that item is reported applied with outcome no_change
- **AND** no write is sent for that item

### Requirement: Batch reads the month, not each item
The system SHALL execute an approved batch of N items with at most N + 4 YNAB requests: one month read for the stale check, one month read before writing, one write per changed item, one month read for the final readback, and one item read only after a failed write.

#### Scenario: Request count for a full batch
- **GIVEN** an approved batch of three changed items
- **WHEN** the batch runs without failures
- **THEN** execution sends exactly three month reads and three writes and no per-item reads

### Requirement: Batch tool gated like every write
The system SHALL expose the batch tool only when writes are enabled and SHALL require a preview token and confirmed true to run it.

#### Scenario: Read-only session hides the batch tool
- **GIVEN** a session without write access
- **WHEN** the tools are listed
- **THEN** the batch tool is absent
- **AND** with writes enabled it is present, annotated writable, and its input requires previewToken and confirmed true
