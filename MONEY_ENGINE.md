# MONEY_ENGINE.md

## Purpose

The money engine is the most trust-sensitive part of HouseMate.

It must be deterministic, testable, auditable, and independent from the LLM.

## Currency representation

Store currency amounts as integer minor units where possible.

Example:
฿2106.57 → 210657 satang.

Never use JavaScript floating-point arithmetic for final money allocation.

Use integer arithmetic or a decimal library.

## Core accounting equation

For a member:

`net = amount_paid_for_household - allocated_share - settlements_sent?`

Recommended clearer ledger approach:

For each expense:
- payer receives credit equal to total paid,
- each participant receives debit equal to allocated share.

For each settlement:
- sender receives credit for amount sent,
- receiver receives debit for amount received.

Then sum ledger entries per member.

Choose one sign convention and document it in code.

## Equal split

Example:
Total = 1000
Participants = 3

Ideal:
333.333...

Actual minor-unit allocation must total exactly 1000.

Use deterministic remainder distribution.

Example:
334
333
333

Remainder recipient selection must be deterministic:
- stable sorted participant IDs, or
- configured ordering.

Do not randomly assign rounding differences.

## Exact split

User enters each member share.

Validation:
sum(exact shares) == total.

If not:
block confirmation and show difference.

## Percentage split

Validation:
percent sum == 100%.

Use decimal-safe calculation.

Allocate rounding remainder deterministically.

## Ratio split

Example:
A: 1
B: 2
C: 1

Total ratio = 4.

Allocate total proportionally.

## Selected participants

Selected participants are simply the participant set used by the split method.

Non-selected household members receive zero share.

## Hybrid split

Required because this is a key real-world use case.

Definitions:

- `bill_total`
- `individual_charges`
- `shared_adjustments`
- `credits/deductions`
- `shared_participants`

One supported model:

1. Start with bill total.
2. Reserve explicit individual charges.
3. Reserve shared adjustments if they must be deducted before base sharing.
4. Apply credits/deductions according to explicit rule.
5. Compute remaining shared amount.
6. Split shared amount across selected shared participants.
7. Add each person's explicit individual charge.
8. Apply person-specific credits/deductions.
9. Validate final allocated total equals bill total.

## Electricity acceptance scenario

Input:

Total:
2106.57

Individual:
Ethan 5
Henry 60
Donut 125
Johnny 180
Rio 20

Other adjustment:
204

Special/adjustment total:
390 + 204 = 594

Shared remainder:
2106.57 - 594 = 1512.57

Shared participants:
9

Base share:
1512.57 / 9 = 168.063333...

Displayed approximate:
168.06

However, final stored member shares must still sum exactly to 2106.57 after allocation and rounding.

If the 204 is not associated with a specific person, the UI must ask what it represents:
- shared amount already handled separately,
- individual charge,
- deduction,
- credit,
- excluded/non-household amount.

The system must not guess accounting semantics silently.

## Expense edit rules

Editing:
- title only → no recalculation.
- amount → requires recalculation.
- participants → requires recalculation.
- split config → requires recalculation.
- payer → changes ledger but not shares.

Always show new preview before saving meaningful financial changes.

## Voiding

Do not hard-delete a finalized expense.

Void it:
- preserve original,
- exclude from active balances,
- write audit event.

## Settlement suggestion algorithm

Input:
all member net balances.

Goal:
reduce number of transfers while preserving totals.

Algorithm:
- separate creditors and debtors,
- greedily match largest debtor and creditor,
- create suggested transfers,
- continue until all near zero.

Suggestions are not actual settlements until users record them.

## Required money tests

- equal split exact division
- equal split remainder
- exact split
- invalid exact split
- percentage split
- percentage remainder
- ratio split
- selected participants
- hybrid split
- negative/credit cases
- payer included
- payer excluded
- settlement effect
- voided expense
- edited expense
- 1 participant
- large amount
- zero amount rejection
- currency precision
- duplicated confirmation/idempotency
