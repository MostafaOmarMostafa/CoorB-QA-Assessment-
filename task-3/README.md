# Task 3 — API Testing

## Part A: Exchange Rate API Testing

### 1. Objective

The objective is to validate the exchange rate API using Postman, verify response correctness, check response time, validate requested currencies, 
and test how the API handles unsupported currency codes.

### 2. API Details

- **HTTP Method:** GET
- **Endpoint:** `https://open.er-api.com/v6/latest/{{baseCurrency}}`
- **Tool:** Postman
- **Authentication:** Not required
- **Environment:** FX Testing Environment

### 3. Environment Variables

| Variable | Purpose |
|---|---|
| `baseCurrency` | Specifies the base currency for the request |
| `targetCurrencies` | Defines the target currencies to validate |
| `maxResponseTime` | Sets the maximum acceptable response time in milliseconds |
| `capturedSarRate` | Stores the SAR rate extracted from the response |

Variables are used to make the request reusable and reduce hard-coded values.

### 4. Test Scenarios and Assertions

The Postman test script covers the following validations:

1. Verify that the HTTP status code is 200.
2. Verify that the response time is below the configured threshold.
3. Verify that the response result indicates success.
4. Verify that the returned base currency matches the configured base currency.
5. Verify that the response contains a rates object.
6. Verify that the configured target currencies exist and have positive numeric rates.
7. Verify that the exchange rate for the base currency equals 1.
8. Capture the SAR exchange rate and verify that it is positive and numeric.

### 5. Test Data

The external CSV file contains the following base currencies:

- USD
- EUR
- GBP

The same request can be reused for different base currencies by updating the `baseCurrency` environment variable.

The CSV file was prepared as test data. Automated CSV-driven execution through the Postman Collection Runner was not completed.

### 6. Negative Testing

An unsupported currency code should be tested to observe the API provider's actual behavior.

The test should verify whether the provider rejects the request or returns a fallback response. 
A successful HTTP status alone is not sufficient to confirm that the requested currency was processed correctly.

For a banking application, unsupported currency codes should be handled explicitly and must not result in an unintended or misleading exchange rate.

### 7. Limitations and Improvements

- Automated data-driven execution using the CSV file remains to be completed.
- Negative testing should be executed and the actual response documented.
- Additional schema validation could verify required fields and data types.
- Exchange rate freshness should be checked using the provider's update timestamps.
- Response-time thresholds should be based on agreed non-functional requirements.

---

## Part B: Payment API Testing

### 3.1 Highest-Risk Test Scenarios

For `POST /api/v1/payments`, I would prioritize scenarios that could cause financial loss, unauthorized transfers, 
duplicate payments, or inconsistent account balances.

**1. Valid Payment**
- Submit a valid payment with correct sender, beneficiary, amount, and currency.
- Verify the response, transaction ID, final status, and corresponding balance changes.

**2. Insufficient Funds**
- Submit a payment when the sender's available balance is lower than the requested amount.
- Verify that the payment is rejected and no debit is applied.

**3. Amount and Currency Validation**
- Test zero, negative, malformed, and unsupported-precision amounts.
- Test unsupported currencies and invalid currency codes.
- Verify that invalid requests are rejected without financial side effects.

**4. Transaction Limits**
- Test payments at, below, and above the per-transaction limit.
- Test daily limits, including transactions that reach or exceed the remaining daily allowance.
- Verify that the configured limits are enforced consistently.

**5. Authorization**
- Attempt to initiate a payment from an account the authenticated user is not authorized to access.
- Verify that unauthorized requests are rejected and no account data or funds are exposed.

**6. Duplicate Requests**
- Submit the same payment request more than once.
- Verify that duplicate processing is prevented where idempotency is required.

**7. Concurrent Payments**
- Submit concurrent payments against the same account.
- Verify that balance updates are consistent and the account cannot spend more than the available funds.

I would prioritize these scenarios based on financial impact, likelihood, and the possibility of irreversible consequences.

### 3.2 Idempotency and Retry Handling

Payment APIs must handle timeouts and retryable server errors safely because a client may not know whether the original request was processed.

I would use an `Idempotency-Key` to identify a unique payment operation.

**Test approach:**

1. Submit a payment request with a unique idempotency key.
2. Simulate a timeout or a retryable server error after the request is sent.
3. Retry the same request using the same idempotency key.
4. Verify that the payment is not processed twice and the sender is not debited twice.
5. Verify that the retry returns the original result or the current transaction status, according to the API contract.
6. Reuse the same key with a different payload and verify that the API rejects it or handles it according to the documented contract.

A timeout or HTTP 503 response does not necessarily mean that the payment failed. Before retrying an uncertain payment, 
I would check its status or retry safely using the same key.

### 3.3 Verifying Actual Money Movement

An HTTP 201 response indicates that the API reports successful resource creation, 
but it does not independently prove that the financial transaction completed correctly.

I would verify the outcome using the following checks:

1. **Payment Record:** Confirm that the transaction exists with the correct transaction ID, sender, beneficiary, amount, and currency.
2. **Sender Balance:** Verify that the sender was debited by the correct amount exactly once.
3. **Beneficiary Balance:** Verify that the beneficiary received the expected amount, where applicable.
4. **Transaction Status:** Confirm that the payment reached the expected final state, such as `COMPLETED`, according to the system's workflow.
5. **Ledger Entries:** Verify that the debit and credit entries are consistent and reconcile correctly.
6. **Duplicate Check:** Confirm that retries or failures did not create duplicate transactions or additional debits.
7. **Audit and Reconciliation:** Review authorized transaction records, audit logs, and reconciliation evidence to investigate inconsistencies.

I would use authorized database queries and available transaction or ledger records to validate the results. If processing is asynchronous, 
I would wait for or poll the transaction status according to the API contract rather than assuming that the payment completed immediately.

The final acceptance decision would depend on consistent transaction status, account balances, and ledger evidence—not only on the HTTP response.

---

## Submission Files

The Task 3 folder should contain:

- `README.md` — Test approach, assertions, limitations, and Part B answers.
- `task-3-collection.json` — Exported Postman Collection.
- `task-3-environment.json` — Exported Postman Environment.
- `data.csv` — External currency test data.

## Conclusion

The testing approach focuses on API response validation, reusable test configuration, negative testing, and financial transaction integrity. 
For payment APIs, correctness must be verified through transaction records, balances, idempotency behavior, and ledger consistency in addition to HTTP responses.
