# Task 4 — Automation Design

## 1. Project Structure and Page Object Model

### Design Approach

I would use the Page Object Model (POM) to separate UI interactions from test scenarios and business assertions. 
The design would also use reusable components for tabs and field types to avoid duplication and keep the framework maintainable as the console grows.

The reference HTML page would be used only to understand the UI structure and behavior. Automated tests would target the production application, 
not the reference page.

### Proposed Project Structure

```text
bank-console-automation/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── framework/
│   │           ├── pages/
│   │           │   ├── BankConsolePage.java
│   │           │   └── components/
│   │           │       ├── TabComponent.java
│   │           │       └── ConfigField.java
│   │           ├── fields/
│   │           │   ├── TextField.java
│   │           │   ├── CheckboxField.java
│   │           │   └── DropdownField.java
│   │           ├── data/
│   │           │   └── ConfigurationDataProvider.java
│   │           └── support/
│   │               ├── WaitHelper.java
│   │               └── ConfigSnapshotManager.java
│   └── test/
│       ├── java/
│       │   └── tests/
│       │       ├── ConfigurationRegressionTest.java
│       │       └── ConfigurationPersistenceTest.java
│       └── resources/
│           ├── test-data/
│           └── config/
├── reports/
├── pom.xml
└── README.md
```

### Page Object Responsibilities

- `BankConsolePage`: Represents the overall console, navigation, Save action, and access to tabs.
- `TabComponent`: Handles selecting a tab and waiting until its content is ready.
- `ConfigField`: Provides common operations for locating and interacting with a field by its stable wrapper.
- `TextField`, `CheckboxField`, and `DropdownField`: Encapsulate control-specific operations.
- `ConfigurationDataProvider`: Loads configuration scenarios from external test data.
- `ConfigSnapshotManager`: Captures the original values of authorized test configuration and supports safe restoration.
- Test classes: Define scenarios, expected outcomes, and assertions without containing low-level locator logic.

I would avoid creating one large page class containing all tabs and fields. Each reusable component would have one clear responsibility, 
while test classes would focus on business behavior.

## 2. Locator Strategy

### Stable Locators

The child control identifiers, such as `slot="field-145"`, are dynamically generated and must not be used as primary locators.

Instead, I would locate the stable parent wrapper using its visible label or a unique descriptive attribute. 
I would then identify the child control within that wrapper.

Conceptually:

```text
Find field wrapper by stable label
              |
              v
Inspect child control type
              |
       +------+------+
       |      |      |
       v      v      v
     Input Checkbox Dropdown
```

For example, the field wrapper could be located by a stable label such as "Daily Transfer Limit". 
The actual locator syntax would be based on the production DOM structure and validated during implementation.

### Handling Different Control Types

I would use a common field interface with operations such as:

- `getValue()`
- `setValue(value)`
- `isVisible()`
- `isEnabled()`

Each control implementation would handle its own behavior:

- **Text input:** Clear the existing value and enter the new value.
- **Checkbox:** Read the current checked state and change it only when it differs from the expected value.
- **Dropdown:** Select the expected option and verify the selected value.

The common interface would allow the test to handle different field types consistently, 
while each implementation would preserve the correct control-specific behavior.

Before interacting with a field, the framework would verify that the wrapper resolves to exactly one intended field. 
Ambiguous or missing locators should fail clearly rather than interact with an arbitrary element.

## 3. Handling Dynamic DOM

Tab switching, scrolling, and re-rendering can replace field elements and cause stale element references.

I would address this using the following strategy:

1. Locate the tab using a stable visible name or descriptive attribute.
2. Click the tab and wait for its content to become visible and ready.
3. Locate each field when it is needed instead of storing long-lived element references.
4. Use explicit waits for meaningful conditions, such as visibility, enabled state, or a completed save operation.
5. If a stale element occurs, re-locate the field and retry only when the action is safe and the failure is transient.
6. After scrolling, re-check element readiness before interacting with controls.
7. Wait for the save operation to complete and for a reliable success indicator or persisted state.

I would not use fixed sleeps as the primary synchronization mechanism, dynamic child IDs as primary locators, 
or broad retry loops that hide genuine failures. I would also avoid catching exceptions and marking a test as passed without verifying the expected outcome.

For operations that may have side effects, retries must be carefully controlled to avoid duplicate updates.

## 4. Test Data Strategy

### Externalized Configuration

Test values would be stored outside the test logic in structured configuration files, 
such as JSON or YAML, with environment-specific settings maintained separately.

A configuration scenario would define:

- Tab name
- Field label or stable field key
- Control type
- Expected value
- Validation rules, where applicable

The test would read this data and use the appropriate field component to apply and verify the values.

### Required Regression Scenario

The initial regression scenario would update at least 12 fields across at least four tabs. For example:

| Tab | Example fields |
|---|---|
| Onboarding Rules | Automatic onboarding, required documents, approval requirement |
| KYC Thresholds | Basic KYC threshold, enhanced KYC threshold, review requirement |
| Transfer Limits and Fees | Single transfer limit, daily transfer limit, transfer fee |
| Notifications and Audit | Email notifications, SMS notifications, audit logging |

These are illustrative field names; the actual labels, available values, dependencies, and validation rules must be confirmed against the production application.

The scenario would navigate to each tab, update the required fields, save the configuration, reload the page, and verify that all expected values persisted.

### Unique Data and Shared Environments

The test environment is shared, contains restricted data, and cannot be reset on demand. 
Therefore, I would design tests to minimize disruption and avoid destructive cleanup.

- Use synthetic or approved test data only; never use restricted customer information.
- Prefer a dedicated test configuration scope or uniquely identifiable test records where the application supports them.
- Capture the original values of configuration fields before modification.
- Change only the fields owned by the test.
- Restore the original values after verification, when authorized and safe.
- Verify the restoration and report cleanup failures explicitly.
- Avoid parallel execution of tests that modify the same shared configuration.
- Do not assume that resetting the database or restoring a full configuration is permitted.

If safe restoration cannot be guaranteed, the test should run only in an approved isolated configuration scope. 
Shared configuration changes must not silently overwrite another team's work.

## 5. Reusability and Scalability

### Adding a New Tab or Field

The framework should make routine extensions data-driven.

To add a fifth tab, an engineer should normally add the tab's configuration and field definitions to the test data, 
validate the tab's stable locator, and add focused tests only when the new tab introduces distinct business behavior.

To add a twentieth field, the engineer should normally add a field definition and expected value. 
Existing field components should handle the interaction if the field uses a supported control type.

A new field type would require a dedicated implementation of the common field interface and its associated tests.

This approach reduces duplication and keeps test maintenance proportional to the actual change.

### Execution Time

I would start with a small, reliable regression suite and measure execution time. I would optimize only after identifying the main bottlenecks.

Potential improvements include:

- Avoid unnecessary navigation and repeated setup.
- Reuse browser sessions within an appropriate test scope.
- Separate fast validation tests from longer end-to-end persistence tests.
- Collect timing data for navigation, configuration updates, saving, and verification.

I would not remove important persistence checks merely to make the suite faster.

### Parallel Execution

Parallelism should be introduced carefully because multiple tests may modify the same shared configuration.

Independent, read-only tests or tests using isolated configuration scopes can run in parallel. 
Tests modifying shared settings should be serialized or partitioned so that they cannot overwrite each other's changes.

The framework should also ensure that each parallel worker uses isolated browser state and independent test data where required.

### CI Integration and Reporting

I would integrate the suite into CI using a controlled test environment and environment-specific configuration.

The pipeline would:

1. Load the approved environment settings and test data.
2. Execute the selected regression suite.
3. Publish test results and relevant logs.
4. Capture screenshots and diagnostic information for failures.
5. Report failed assertions, stale-element errors, save failures, and persistence mismatches separately.
6. Track execution time and failure trends over time.

As the application grows from 7 to 20 tabs, I would organize execution by feature or risk and allow selective runs for affected areas, 
while retaining a scheduled full regression suite.

I would avoid splitting the suite into many parallel workers until the environment and data strategy can safely support that level of concurrency.

## 6. Tooling and Trade-offs

### Proposed Stack

I would choose Java, Selenium WebDriver, TestNG, and Maven.

- **Java:** Provides a structured language for reusable components, interfaces, and maintainable abstractions.
- **Selenium WebDriver:** Supports browser automation and explicit synchronization for dynamic web applications.
- **TestNG:** Supports test grouping, parameterization, setup and teardown, and controlled parallel execution.
- **Maven:** Manages dependencies, build configuration, and test execution.
- **CI pipeline:** Executes the suite consistently and publishes test reports.
- **Reporting:** Provides readable pass/fail results, screenshots, and diagnostic evidence.

### Trade-offs

Selenium offers mature browser automation capabilities and fits a Java-based enterprise testing stack, 
but it requires disciplined locator design and synchronization. Dynamic applications can become flaky if tests rely on unstable elements or poorly designed waits.

Playwright is a strong alternative because of its built-in auto-waiting and modern browser automation features. 
I would consider it if the team prioritizes faster setup and the organization supports its language and tooling ecosystem. However, 
I would avoid introducing a new stack solely for convenience when an established, supported stack already meets the requirements.

My final choice would consider team expertise, application compatibility, CI support, maintenance cost, 
and long-term ownership—not only the initial implementation effort.

## Final Design Principles

The solution would prioritize stable locators, reusable field components, explicit waits, externalized test data, safe configuration restoration, 
and meaningful persistence assertions.

The primary success criterion is not simply that the UI accepts new values. It is that the automation updates the intended configuration safely, 
saves it successfully, and proves that the expected values remain correct after a page reload.
