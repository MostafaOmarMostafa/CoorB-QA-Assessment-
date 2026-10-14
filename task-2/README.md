# Task 2 — Exploratory Testing & Bug Reporting

## Application 1: ParaBank

### Bug 1: Bill Payment Accepts Zero and Negative Amounts

**Environment**
- Application: ParaBank Demo
- URL: https://parabank.parasoft.com/parabank/index.htm
- Browser: Google Chrome
- Date: October 14, 2026

**Preconditions**
- The user is registered and logged in.
- A valid source account is available.
- The Bill Pay page is accessible.

**Steps to Reproduce**
1. Log in to ParaBank.
2. Navigate to the Bill Pay page.
3. Enter valid payee details.
4. Enter `0` as the payment amount.
5. Submit the payment.
6. Repeat the test using `-10` as the payment amount.

**Actual Result**

The application accepts both zero and negative payment amounts and displays a successful payment confirmation. 
The negative payment was also recorded in Account Activity.

**Expected Result**

The application should reject zero and negative payment amounts and display a validation message requiring the amount to be greater than zero.

**Expectation Basis**

Standard bill-payment business rules. No explicit written requirement was available during exploratory testing.

**Severity:** High

**Priority:** High

**Business Impact**

Invalid payment requests may be recorded as successful transactions and may affect transaction history or account balances incorrectly.

**Evidence**

During testing, a payment of `-$10.00` was accepted and appeared in Account Activity. A zero-amount payment was also accepted. Screenshots are not attached.

---

## Application 2: Altoro Mutual

### Bug 2: Transfer Funds Allows Transfers Exceeding the Available Balance

**Environment**
- Application: Altoro Mutual Demo
- URL: https://demo.testfire.net/
- Browser: Google Chrome
- Date: October 14, 2026

**Preconditions**
- The user is logged in.
- The source and destination accounts are different.
- The source account balance is visible.

**Steps to Reproduce**
1. Log in to Altoro Mutual.
2. Open Account History for account `800016 Savings`.
3. Observe the available balance of `$76,564,656.00`.
4. Navigate to Transfer Funds.
5. Select `800016 Savings` as the source account.
6. Select `800002 Savings` as the destination account.
7. Enter `$80,000,000` as the transfer amount.
8. Submit the transfer.
9. Reopen Account History for `800016 Savings`.

**Actual Result**

The application displays the following successful transfer confirmation:

`8.0E7 was successfully transferred from Account 800016 into Account 800002`

After the transfer, the source account's Ending Balance and Available Balance are both displayed as `-$3,435,344.00`.

**Expected Result**

The application should reject a transfer amount that exceeds the available balance and display an appropriate validation message. 
The transfer should not be completed.

**Expectation Basis**

Standard funds-transfer business rules. No explicit written requirement was available during exploratory testing.

**Severity:** High

**Priority:** High

**Business Impact**

Allowing transfers that exceed available funds may cause unauthorized overdrafts and inaccurate account balances.

**Evidence**

The following results were observed during testing:
- Before transfer: available balance of `$76,564,656.00`.
- Transfer confirmation: `$80,000,000` transfer reported as successful.
- After transfer: available balance of `-$3,435,344.00`.

Screenshots are not attached. The demo environment also contains unusual pre-existing account data, so the underlying cause has not been independently verified.

---

## Exploratory Testing Approach

Exploratory testing focused on financial transaction validation, boundary values, and account-balance consistency.

The following areas were explored:

- Bill payments using zero and negative amounts.
- Funds transfers using an amount greater than the displayed available balance.
- Transaction confirmation messages.
- Account history and resulting balances.
- Validation behavior for invalid transaction amounts.

The observed behavior was compared against standard financial transaction rules. Reproducible steps, actual results, expected results, severity, priority, 
and potential business impact were documented for each finding.

Testing was limited to functional behavior in the provided demo applications. No security or load testing was performed.

## Conclusion

Two potential functional defects were identified involving financial transaction validation:

1. ParaBank accepts zero and negative bill-payment amounts.
2. Altoro Mutual allows a transfer exceeding the displayed available balance.

Both findings should be investigated further to confirm their underlying causes and assess their impact on transaction integrity.

**Evidence limitation:** Findings are documented from observations made during testing. 
Screenshots are not attached, and the underlying causes have not been independently verified.
