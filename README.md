# Quality Engineering Assessment

## Overview

This repository contains my submission for the Quality Engineering assessment. It covers test design, exploratory testing, API testing, automation architecture, and release readiness.

## Submission Contents

- **Task 1 — Test & Risk:** `task-1/README.md`
- **Task 2 — Exploratory Testing & Bug Reporting:** `task-2/README.md`
- **Task 3 — API Testing:** `task-3/README.md`
- **Task 4 — Automation Design:** `task-4/automation-design.md`
- **Task 5 — Quality Strategy & Release Judgement:** `task-5/README.md`

## Time Spent

The following are honest approximate estimates of the time spent preparing each task:

- **Task 1:** Less than 1 hour.
- **Task 2:** Less than 1 hour.
- **Task 3:** Less than 1 hour.
- **Task 4:** Less than 1 hour.
- **Task 5:** Less than 1 hour.

These estimates reflect the time spent preparing the submissions. They do not imply that every proposed test or automation design was fully executed.

## Assumptions and Questions for the BA / PO

Where requirements were incomplete or ambiguous, I based my approach on the information provided and avoided treating unspecified behavior as confirmed.

In a real project, I would clarify the following:

- **Business rules:** What are the expected outcomes, validation rules, limits, and acceptance criteria for each critical journey?
- **Money movement:** How should fees, FX rates, rounding, timeouts, retries, duplicate requests, and failed or pending transfers be handled?
- **Dependencies:** Can changes or open defects in shared services affect functionality outside their stated scope?
- **Test environment and data:** Which test accounts, permissions, integration endpoints, and data-reset mechanisms are available?
- **Release readiness:** Who owns UAT sign-off, what are the release-blocking criteria, and who formally accepts residual risks?

I would prioritize these questions according to their impact on business behavior, customer funds, and release decisions.

## Deliberate Exclusions and Limitations

- **API testing:** The Postman collection and environment were prepared for review. Data-driven execution and additional negative scenarios were not fully automated or verified as part of the submitted work.
- **Automation design:** The submission describes a proposed maintainable framework; it does not claim that a complete automation framework was implemented.
- **Release strategy:** The recommendation is based on the evidence and risks stated in the supplied scenario. A real release decision would require current test results, defect evidence, and the required stakeholder approvals.
- **Scope prioritization:** Lower-risk or unrelated work was deprioritized in favor of critical business flows and the risks described in each task.

## AI Usage

AI assistance is disclosed in [ai-usage.md](ai-usage.md).

## Closing Note

This submission reflects my approach to risk-based testing, maintainable automation design, API and integration quality, and evidence-based release decisions. I am prepared to explain my reasoning and discuss alternatives during the technical review.
