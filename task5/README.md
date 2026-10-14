# Task 5 — Quality Strategy & Release Judgement

## 5.1 Go / No-Go Recommendation

**Recommendation: No-Go for the currently planned release until the critical readiness conditions below are met.** 
The announced launch date is a business constraint, not evidence of product quality.

The current evidence does not support a QA Green status. Regression passes at only 68%, approximately 30 failures remain uninvestigated, 
UAT is incomplete and unsigned, and the FX integration has received only one round of happy-path testing. 
These gaps create unacceptable uncertainty for an international money-movement journey.

The two open critical onboarding/KYC defects are outside the stated release scope, but they share the customer profile service. 
I would require an impact assessment, including API and integration regression, before concluding that they cannot affect this release.

I would reconsider the decision only when:
- All critical and high-risk transfer scenarios pass, including authorization, limits, fees, FX rate handling, insufficient funds,
  duplicate submissions, timeouts/retries, and correct debit/credit outcomes.
- All critical regression failures are investigated, with release-relevant failures fixed and retested. Remaining failures have documented causes,
  risk assessments, and explicit approval from the appropriate stakeholders.
- The KYC defects are assessed for shared-service impact, with sufficient evidence that the transfer journey is not affected, or the defects are resolved.
- UAT is completed and formally signed off by the client's authorized business owner.
- No unresolved release-blocking defects remain, and the release decision is recorded with evidence and named business/technical risk owners.

If these conditions cannot be met within 72 hours, I would recommend postponing the release rather than treating the deadline as a reason to accept unknown risks.

## 5.2 Next 72 Hours — Priorities and Trade-offs

**Priority 1 — Establish the actual risk (first 4–8 hours).** Review regression failures, classify them by severity and affected functionality, 
and reproduce the highest-risk failures. Do not label failures as flaky without evidence. Review logs, test data, environment stability, and recent changes. 
Assess the shared customer profile service and the two critical KYC defects with developers.

**Priority 2 — Validate money movement and FX.** Work with developers and integration engineers to verify FX rate source, currency pairs, rate freshness, 
rounding, fees, and the final amount shown to the customer. Test successful transfers and negative/edge cases, including invalid or unavailable rates, 
timeouts, retries, duplicate requests, limits, and consistent transaction status and account balances across relevant APIs and services.

**Priority 3 — Complete focused regression and UAT.** Run risk-based regression around transfer creation, validation, authorization, FX, fees, 
failure handling, and transaction records. Pair experienced QA engineers with the two newer team members for critical flows and review their results. 
Agree on the remaining UAT scripts and completion times with the client’s business team.

**Priority 4 — Make and communicate the release decision.** Maintain a single release-readiness report listing evidence, open defects, residual risks, 
owners, and deadlines. Hold a joint go/no-go review with QA, engineering, delivery, and the client’s authorized decision-makers.

**What I would drop or defer:** low-risk cosmetic checks, non-release features, broad exploratory work unrelated to the transfer journey, 
and refactoring that does not reduce immediate release risk. I would not drop critical-path testing, failure/recovery testing, defect investigation, 
or UAT merely to meet the date.

## 5.3 Communication

**To the delivery manager:** I would respond in writing that QA cannot currently be declared Green. I would provide the evidence, the main risks, 
the recovery plan, accountable owners, and the earliest time for reassessment. 
I would explain that the launch announcement does not change the quality evidence and request a formal go/no-go decision if the risks remain unresolved.

**To the client:** I would provide a clear, factual readiness update, explain the remaining validation and UAT activities, agree on completion milestones, 
and communicate the release recommendation through the agreed governance process. I would avoid internal blame or unverified claims.

The level of detail and wording may differ, but the facts, severity of risks, unresolved blockers, and final readiness status must remain consistent. 
I would never provide a Green status to one audience while communicating material unresolved risks to another.

## 5.4 Preventing Recurrence and Making Quality Visible

I would introduce the following practices:

- **Daily automated regression triage:** assign each failure an owner and root-cause category, track flaky tests separately, and require evidence before quarantine. Review trends and fix recurring instability rather than ignoring it.
- **Release-readiness gates:** agree on entry/exit criteria, risk-based coverage, defect thresholds, and required UAT sign-off before announcing a release date.
- Involve QA in release planning and assess shared-service dependencies during impact analysis.
- **Earlier integration testing:** test FX and other external dependencies in a production-like staging environment throughout the sprint,
- including negative, timeout, retry, and reconciliation scenarios. Use a short daily cross-functional risk review during the final release window.

I would report three metrics to the client:

1. **Risk-weighted critical-scenario pass rate:** passed critical scenarios divided by executed critical scenarios, with blocked and untested scenarios shown separately.
2. **Open release-blocking defects:** count by severity, affected journey, owner, and age, with trends over time.
3. **UAT completion and sign-off:** completed scripts versus planned scripts, results by business-critical flow, and formal approval status.

I would not use raw test-case counts, overall pass rate alone, or automation percentage as proof of release readiness. 
These can hide untested critical paths, unstable tests, and unresolved business risks. Metrics must support a decision, 
not make the release appear healthier than the evidence justifies.
