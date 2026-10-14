# Task 1 - Test Design & Risk Coverage

## Scope

This task covers the testing approach for the Local Fund Transfer feature in a mobile banking application.

The main focus areas are:

- Financial correctness and transaction integrity.
- Authentication and transfer authorization.
- Transfer limits and fee calculation.
- Failure handling and duplicate prevention.
- Transaction confirmation and customer notification.

Testing will be risk-based, prioritizing scenarios that could result in financial loss, unauthorized transfers, or inconsistent transaction records.

## 1.1 Questions, Assumptions, and Clarifications

Before designing the test cases, I would clarify the following key points with the Business Analyst (BA) or Product Owner (PO). 
These questions focus on gaps that could affect financial correctness, security, and transaction reliability.

### 1. Account and Beneficiary Eligibility

**Question:** Which source accounts and registered beneficiaries are eligible for transfers? Can blocked, inactive, or restricted accounts be used?

**Why it matters:** To prevent unauthorized transfers and invalid transactions.

**Assumption:** Only active, eligible source accounts and valid registered beneficiaries can be used.

### 2. Amount and Transfer Limits

**Question:** What are the minimum transfer amount and supported decimal precision? 
How are the per-transaction limit of 10,000 and daily limit of 20,000 calculated, and when does the daily limit reset?

**Why it matters:** Incorrect validation or limit calculations could allow excessive transfers or reject valid transactions.

**Assumption:** The amount must be positive and follow the supported currency precision. 
The specified limits must be enforced, while the exact daily reset rules require confirmation.

### 3. Fees and Available Balance

**Question:** How are transfer fees calculated, are they included in the transfer limits, 
and what should happen if the customer cannot cover both the transfer amount and its fee?

**Why it matters:** Incorrect fee handling or balance validation could cause financial discrepancies or unexpected charges.

**Assumption:** The configured fee is applied exactly once, and the transfer must not execute if the available balance is insufficient to cover the required amount.

### 4. OTP and Authorization

**Question:** What are the OTP expiration, retry, and resend rules, and must the OTP be linked to the specific transfer details?

**Why it matters:** Weak OTP validation could allow unauthorized transfers.

**Assumption:** Only a valid, unexpired OTP associated with the current transfer can authorize execution. Retry and resend rules require confirmation.

### 5. Duplicate Requests and Transaction Status

**Question:** What should happen if the customer retries after a timeout or submits the same transfer more than once? How are pending and failed transfers handled?

**Why it matters:** Duplicate requests or unclear transaction states could result in duplicate debits, inconsistent records, or unnecessary customer retries.

**Assumption:** A transfer must not execute more than once for the same request. 
The system must provide a reliable transaction status and handle uncertain outcomes without creating duplicates.

### 6. Confirmation and Notifications

**Question:** What information must appear on the confirmation screen and in the SMS, and what should happen if SMS delivery fails after a successful transfer?

**Why it matters:** Incorrect confirmation details can mislead customers, while notification failures can cause confusion and support issues.

**Assumption:** Confirmation details must match the recorded transaction. An SMS failure must not reverse or repeat a successful transfer, 
and error messages must not expose sensitive information.

## 1.2 Risk Assessment and Prioritization

The following risks are prioritized based on their potential financial impact, security implications, and effect on customer trust.

### 1. Duplicate Transfers or Inconsistent Processing

**Risk:** A transfer is executed more than once after retries, timeouts, or system failures.

**Potential Impact:** Customers may face duplicate deductions or uncertainty about their funds. The bank may experience financial losses,disputes,and reconciliation issues.

**Priority:** Critical

### 2. Unauthorized Transfers

**Risk:** OTP validation fails, or account eligibility checks allow an unauthorized transfer.

**Potential Impact:** Customers may suffer financial loss, while the bank faces fraud exposure and reputational damage.

**Priority:** Critical

### 3. Incorrect Financial Validation

**Risk:** Transfer limits, fees, or available balance are calculated incorrectly.

**Potential Impact:** Customers may face unexpected charges or incorrect balances. The bank may experience financial discrepancies or policy violations.

**Priority:** High

### 4. Incorrect Transaction Status or Financial Records

**Risk:** The application reports an incorrect status, or the customer is debited without the transfer being completed correctly.

**Potential Impact:** Customers may lose access to funds or receive misleading information. The bank may face reconciliation issues and customer complaints.

**Priority:** Critical

### 5. Incorrect Confirmation or SMS Failure

**Risk:** Confirmation details are inaccurate, or the SMS fails after a successful transfer.

**Potential Impact:** Customers may become uncertain about the transaction and retry unnecessarily, increasing support requests and reducing trust.

**Priority:** Medium-High

### Risk-Based Testing Priorities

I would test critical risks first, focusing on authorization, duplicate prevention, and financial consistency. Next, I would validate limits, fees, 
and available balance. Finally, I would verify confirmation details, error messages, and SMS behavior.

Financial outcomes must be verified against authoritative transaction and account records, not only the information displayed in the mobile application.

## 1.3 Test Cases

The following test cases focus on the highest-risk scenarios, including financial correctness, authentication, limits, fees, duplicate prevention, 
and failure handling.

### TC01 - Successful Local Fund Transfer

**Preconditions:** The customer is authenticated, the source account and beneficiary are eligible, sufficient funds are available, 
and the required OTP can be obtained.

**Test Steps:**
1. Select the source account and registered beneficiary.
2. Enter a valid amount and optional purpose note.
3. Confirm the transfer using a valid OTP.

**Expected Result:** The transfer is executed exactly once, the correct amount and fee are applied, the transaction reference is generated, 
the confirmation details are accurate, and an SMS is sent.

### TC02 - Invalid Transfer Amount

**Preconditions:** The customer has access to an eligible source account and registered beneficiary.

**Test Steps:** Try submitting zero, negative, malformed, or unsupported-precision amounts.

**Expected Result:** Invalid amounts are rejected with an appropriate message. No transfer is executed, and no funds or fees are deducted.

### TC03 - Per-Transaction Limit

**Preconditions:** The customer has sufficient funds and has not exceeded the daily limit.

**Test Steps:** Submit a transfer at the 10,000 limit and another exceeding 10,000.

**Expected Result:** A transfer of exactly 10,000 is accepted if all other conditions are satisfied. 
Any amount exceeding 10,000 is rejected without executing the transfer or deducting funds. This assumes the per-transaction limit is inclusive.

### TC04 - Daily Transfer Limit

**Preconditions:** The customer has completed successful transfers totaling 19,000 during the applicable day, and sufficient funds are available.

**Test Steps:**
1. Attempt a transfer of 1,000 to reach the daily limit of 20,000.
2. If the first transfer succeeds, attempt another transfer of 1 to exceed the daily limit.
   
**Expected Result:** Assuming the daily limit is inclusive, the transfer of 1,000 is accepted if all other rules are satisfied.
Any additional transfer exceeding the daily limit is rejected without an unauthorized debit. 
The daily limit calculation and reset behavior must follow the confirmed business rules.

### TC05 - Fee Calculation and Insufficient Balance

**Preconditions:** The applicable fee configuration is known, and the source account balance is controlled.

**Test Steps:** Execute a valid transfer with sufficient funds, 
then attempt a transfer where the customer can cover the principal amount but not the principal plus the applicable fee.

**Expected Result:** The fee is calculated and charged exactly once. If the total required amount cannot be covered, 
the transfer is rejected without an unintended partial debit.

### TC06 - OTP Validation

**Preconditions:** A transfer has been initiated, and OTP validation is required. Test data is available to verify invalid, expired, 
and valid OTP scenarios independently.

**Test Steps:**
1. Attempt confirmation with an invalid or expired OTP.
2. In a separate test execution, attempt to use an OTP associated with a different transfer.
3. Start a fresh eligible transfer and confirm it using the correct OTP.

**Expected Result:** Invalid, expired, or mismatched OTPs must not authorize a transfer. A valid OTP must authorize only its intended transfer.
Retry limits and account-locking behavior must follow the approved security rules. No unauthorized debit must occur.

### TC07 - Duplicate Submission and Retry

**Preconditions:** A valid transfer request is ready for confirmation.

**Test Steps:** Submit the same request multiple times or retry after an uncertain response.

**Expected Result:** The same transfer is not executed more than once. The customer receives a consistent transaction outcome, and duplicate debits are prevented.

### TC08 - Insufficient Balance and Account Eligibility

**Preconditions:** Test accounts are available with insufficient funds or restricted status.

**Test Steps:** Attempt transfers using insufficient funds and an ineligible source account.

**Expected Result:** The system rejects the invalid requests without unauthorized deductions, negative balances, or unintended transaction execution.

### TC09 - Timeout or Downstream Failure

**Preconditions:** A controlled test environment allows a timeout to be simulated after a transfer request reaches the processing service.

**Test Steps:**
1. Submit a valid transfer and simulate a timeout before the customer receives the final response.
2. Verify the authoritative transaction status and account records.
3. Retry safely using the same transfer request, where supported.
4. Repeat the scenario with a request that was not executed before the timeout.

**Expected Result:** If the original transfer was executed, a retry must not create another transfer and 
should return or recover the original outcome where supported. 
If it was not executed, a safe retry may complete it exactly once. The transaction status, debit, and reference must remain consistent. 
The customer must not be told the transfer definitely failed while its outcome remains unknown.

### TC10 - Confirmation and SMS Failure

**Preconditions:** A successful transfer can be completed, and SMS delivery can be simulated as unavailable.

**Test Steps:** Complete a transfer and simulate SMS delivery failure. Verify the transaction record and confirmation details.

**Expected Result:** The transfer remains successful, the confirmation matches the recorded transaction, 
and SMS failure does not trigger a reversal or duplicate transfer. Notification retry or monitoring follows the agreed behavior.

## 1.4 Test Scope, Automation Strategy, and Release Decision

### What Is Deliberately Not Covered

The current scope focuses on the highest-risk transfer scenarios. The following areas are excluded from this initial test cycle:

- Exhaustive UI layouts and device-specific compatibility across all mobile devices.
- Full performance, load, and stress testing.
- Comprehensive security penetration testing.
- Every possible fee configuration and external SMS provider failure combination.

These areas require separate testing activities, dedicated environments, or additional requirements. 
They should not be considered permanently out of scope if business or security risks require further coverage.

### Automation vs. Manual Testing

**Automation:**

I would automate stable, repeatable scenarios such as successful transfers, amount validation, transaction limits, fee calculation, OTP validation, 
duplicate prevention, and financial consistency checks.

API and service-level tests would validate business rules, while integration tests would verify transaction records and failure recovery. 
A small number of UI automation tests would cover the main customer journey and confirmation screen.

All automated financial scenarios should run in a controlled test environment with approved test accounts and data. 
Assertions should verify transaction status and financial records, not rely solely on UI messages. 
Where external services are unavailable, controlled mocks or test doubles may be used, with integration behavior verified separately.

**Manual Testing:**

Manual testing would focus on exploratory scenarios, usability, device-specific behavior, unclear requirements, and investigating unexpected failures. 
Scenarios requiring coordinated external failures may also need controlled manual execution when reliable automation is not available.

Automation would prioritize critical financial and security risks rather than simply maximizing the number of automated tests.

### Release Decision

**Go:** I would recommend release only when critical test cases pass, no unresolved critical defects affect authorization or financial integrity, 
and transaction records remain consistent across successful, failed, and retried requests. Agreed business rules must be validated, 
and remaining risks must be documented and accepted by the appropriate stakeholders.

**No-Go:** I would block release if duplicate transfers, unauthorized execution, incorrect limits or fees, or inconsistent financial records remain possible. 
A successful UI response alone is not sufficient evidence that the feature is safe to release.

### Final QA Position

Release confidence depends on verified financial outcomes, reliable authorization, safe failure recovery, and clear communication to the customer. 
Any remaining high-impact risks must be explicitly reviewed before release approval.

