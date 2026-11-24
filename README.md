# Question-2-University-Fee-Payments-
# Q2 – University Fee Payments (schema `q2`)

**Purpose:** accurate fee balances, automatic payment audit records, and a daily collection report. No credit balances allowed.

## Objects
| Part | Object | What it does |
|---|---|---|
| (a) | `students`, `payments`, `payment_audit`, `daily_collections` | 3 students (one with no payments, one fully settled), 6 payments |
| (b) | `fn_fee_balance(id)` | `fee_due` − total payments; full fee if none; rejects unknown student |
| (c) | `pr_receive_payment(id, amount, date)` | rejects NULL/zero/negative amounts and amounts above the balance; inserts the payment only (`fee_due` untouched, **no audit insert**) |
| (d) | `trg_payments_audit` | AFTER INSERT row trigger writes payment ID, student ID, amount, timestamp |
| (e) | `v_fee_balances` | student number, name, outstanding balance; settled students excluded (`HAVING balance > 0`) |
| (f) | `idx_payments_student_paid_on (student_id, paid_on)` | composite: equality then range |
| (g) | `pr_refresh_collections(day)` + job `q2_refresh_collections` | upsert; cron `0 1 * * *` for yesterday |

## Run / test
- `SELECT * FROM payment_audit;` should show one row per payment, written by the trigger.
- Try paying 0, NULL, or more than the balance: each must fail.
