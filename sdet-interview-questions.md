# SDET Interview Questions & Answers

A practical interview-prep guide for **Software Development Engineer in Test (SDET)** roles.

This guide is intentionally **separate from Playwright** and focuses on the broader areas commonly tested in SDET interviews:

- Testing fundamentals
- Test design
- Coding and DSA
- API testing
- SQL and databases
- Automation framework design
- CI/CD
- Flaky tests and debugging
- Performance and reliability
- Distributed systems
- Security fundamentals
- Situational questions
- Behavioral questions
- Senior SDET / test architecture

> Focus on understanding the reasoning behind each answer rather than memorizing wording.

---

## Table of Contents

1. [SDET Fundamentals](#sdet-fundamentals)
2. [Test Design](#test-design)
3. [Coding and DSA](#coding-and-dsa)
4. [API Testing](#api-testing)
5. [SQL and Databases](#sql-and-databases)
6. [Automation Framework Design](#automation-framework-design)
7. [CI/CD](#cicd)
8. [Flaky Tests and Debugging](#flaky-tests-and-debugging)
9. [Performance and Reliability](#performance-and-reliability)
10. [Distributed Systems](#distributed-systems)
11. [Security Fundamentals](#security-fundamentals)
12. [Situational Questions](#situational-questions)
13. [Behavioral Questions](#behavioral-questions)
14. [Senior SDET / Test Architecture](#senior-sdet--test-architecture)
15. [Top 25 Questions to Master](#top-25-questions-to-master)

---

# SDET Fundamentals

## 1. What is an SDET?

An SDET is a **Software Development Engineer in Test**.

An SDET combines software-development skills with testing and quality-engineering expertise.

Typical responsibilities include:

- Building automation frameworks
- API/backend testing
- Writing test utilities and tools
- CI/CD integration
- Improving product testability
- Debugging failures
- Designing quality strategies
- Collaborating with developers on architecture and reliability

---

## 2. QA Engineer vs SDET?

A simplified distinction:

### QA Engineer

Primarily focuses on validating product quality.

### SDET

Validates quality while also building:

- Test frameworks
- Internal tools
- Automation infrastructure
- Test utilities
- CI integrations
- Testability mechanisms

The exact distinction varies by company.

---

## 3. What is shift-left testing?

Shift-left means moving quality activities earlier in the development lifecycle.

Examples:

- Reviewing requirements early
- Adding unit tests
- Performing static analysis
- Running API and contract tests before UI testing
- Testing during pull-request pipelines
- Reviewing testability during design

The goal is faster feedback and cheaper defect detection.

---

## 4. What is the test pyramid?

A common model:

```text
        E2E
       /   \
 Integration
 /         \
Unit Tests
```

The idea is:

- Many fast unit tests
- Fewer integration tests
- A smaller number of expensive end-to-end tests

The exact shape depends on architecture and business risk.

---

## 5. Severity vs priority?

### Severity

How badly a defect affects the system.

### Priority

How urgently the defect should be fixed.

Example:

A typo on a public homepage may be low severity but high priority.

A serious defect in a rarely used internal feature may temporarily have lower business priority.

---

## 6. Smoke vs regression vs sanity testing?

### Smoke testing

Validates the most critical functionality after a build or deployment.

### Regression testing

Checks that existing functionality still works after changes.

### Sanity testing

Performs narrow validation of a specific change or area.

---

## 7. Functional vs non-functional testing?

### Functional testing

Validates what the system does.

Examples:

- Login
- Checkout
- Search
- Order creation

### Non-functional testing

Validates characteristics of the system.

Examples:

- Performance
- Security
- Reliability
- Accessibility
- Scalability
- Usability

---

## 8. Verification vs validation?

### Verification

Are we building the product correctly?

Examples:

- Reviews
- Static analysis
- Design validation

### Validation

Are we building the correct product?

Examples:

- Executing tests
- User acceptance testing
- End-to-end validation

---

# Test Design

## 9. How would you test a feature with no requirements?

I would:

1. Clarify the business goal.
2. Speak with product and development stakeholders.
3. Inspect related functionality.
4. List assumptions explicitly.
5. Identify happy paths.
6. Explore negative and boundary cases.
7. Consider security, performance, and accessibility where relevant.
8. Document gaps and risks.

Do not invent requirements silently.

---

## 10. How do you decide what to automate?

Consider:

- Business criticality
- Regression frequency
- Repeatability
- Manual effort
- Stability
- Maintenance cost
- Feedback speed
- Risk

High-value, repeatable, deterministic checks are strong candidates.

---

## 11. When should you NOT automate a test?

Examples:

- One-time verification
- Rapidly changing prototypes
- Subjective usability evaluation
- Low-value scenarios with high maintenance cost
- Exploratory testing

Automation should provide more value than maintenance cost.

---

## 12. What is risk-based testing?

Prioritize testing according to risk.

A simple model:

```text
Risk ≈ Probability of Failure × Impact of Failure
```

High-risk areas often include:

- Payments
- Authentication
- Authorization
- Data loss
- Financial calculations
- Critical integrations

---

## 13. What is boundary value analysis?

Boundary value analysis tests values at and around limits.

For a valid range of `1–100`, useful values include:

```text
0
1
2
99
100
101
```

Defects frequently occur near boundaries.

---

## 14. What is equivalence partitioning?

Divide possible inputs into groups expected to behave similarly.

Example:

For age `18–65`:

```text
< 18      -> invalid partition
18–65     -> valid partition
> 65      -> invalid partition
```

You can then test representative values from each partition.

---

## 15. What is pairwise testing?

Pairwise testing reduces combinations by ensuring every pair of parameter values is covered at least once.

It is useful when the total number of combinations is too large to test exhaustively.

---

# Coding and DSA

## 16. Reverse a string.

```ts
function reverseString(value: string): string {
  return value.split('').reverse().join('');
}
```

Be prepared to implement it manually as well.

---

## 17. Find the first non-repeating character.

```ts
function firstUnique(s: string): string | null {
  const counts = new Map<string, number>();

  for (const c of s) {
    counts.set(c, (counts.get(c) ?? 0) + 1);
  }

  for (const c of s) {
    if (counts.get(c) === 1) {
      return c;
    }
  }

  return null;
}
```

Complexity:

```text
Time:  O(n)
Space: O(n)
```

---

## 18. How do you detect duplicates in an array?

```ts
function hasDuplicates(values: number[]): boolean {
  return new Set(values).size !== values.length;
}
```

---

## 19. How would you detect a cycle in a linked list?

Use Floyd's tortoise-and-hare algorithm.

- Slow pointer moves one node at a time.
- Fast pointer moves two nodes at a time.
- If they meet, a cycle exists.

Complexity:

```text
Time:  O(n)
Space: O(1)
```

---

## 20. What should you discuss after solving a coding problem?

Do not stop at working code.

Discuss:

- Time complexity
- Space complexity
- Empty inputs
- Null values
- Duplicates
- Boundary cases
- Invalid input
- Scalability
- Test cases

This shows an SDET mindset.

---

## 21. What is a hash map useful for?

Common uses:

- Frequency counting
- Fast lookup
- Duplicate detection
- Caching
- Mapping IDs to objects

Typical average lookup complexity:

```text
O(1)
```

---

## 22. Array vs linked list?

### Array

- Fast indexed access
- Contiguous memory
- Expensive insertion in the middle

### Linked list

- Sequential access
- Easy insertion/removal when node reference is known
- Extra memory for pointers

---

# API Testing

## 23. What should you validate in an API test?

Do not validate only the status code.

Check:

- HTTP status
- Response body
- Schema
- Headers
- Business rules
- Authorization
- Response time where relevant
- Side effects
- Persistence
- Negative cases

---

## 24. PUT vs PATCH?

### PUT

Typically replaces or updates the full resource representation.

### PATCH

Typically applies a partial update.

Always follow the API contract.

---

## 25. Common HTTP status codes?

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
429 Too Many Requests
500 Internal Server Error
503 Service Unavailable
```

---

## 26. 401 vs 403?

### 401

Authentication is missing or invalid.

### 403

The identity is known but does not have permission.

---

## 27. How would you test authentication and authorization?

Test combinations such as:

```text
No token
Invalid token
Expired token
Valid token
Wrong role
Correct role
Tampered token
```

Verify both response codes and actual resource access.

---

## 28. How would you test rate limiting?

Send requests beyond the documented limit and verify:

```text
429 Too Many Requests
```

Also validate:

- Recovery after the rate-limit window
- Relevant headers
- Retry behavior
- Whether limits apply per user/IP/token as expected

---

## 29. How would you test an asynchronous API?

Example:

```text
POST /jobs
-> 202 Accepted
-> jobId
```

Then check the job state:

```text
GET /jobs/{jobId}
```

Validate:

- Pending state
- Completion
- Failure
- Timeout behavior
- Invalid job ID
- Retry behavior

---

## 30. How would you test a microservice that depends on external APIs?

Combine:

- Unit tests with stubs
- Contract tests
- Integration tests
- Failure injection
- Timeout tests
- Retry validation
- Circuit-breaker testing
- Limited real end-to-end tests

---

## 31. What is contract testing?

Contract testing validates that two services agree on request and response formats and behavior.

It helps detect breaking integration changes earlier than full end-to-end testing.

---

# SQL and Databases

## 32. INNER JOIN vs LEFT JOIN?

### INNER JOIN

Returns only matching rows.

### LEFT JOIN

Returns every row from the left table plus matching rows from the right table.

---

## 33. How do you find duplicate values?

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

## 34. Find the second-highest salary.

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
  SELECT MAX(salary)
  FROM employees
);
```

---

## 35. How would you validate database data after an API request?

Example:

```text
API -> create order
DB  -> verify order persisted
DB  -> validate customer relation
DB  -> validate amount/status
```

Use direct DB validation only when persistence itself is part of what you need to verify.

---

## 36. What is database transaction isolation?

Transaction isolation controls how concurrent transactions observe one another.

Common levels:

```text
Read Uncommitted
Read Committed
Repeatable Read
Serializable
```

Potential concurrency issues include:

- Dirty reads
- Non-repeatable reads
- Phantom reads

---

## 37. Primary key vs foreign key?

### Primary key

Uniquely identifies a row.

### Foreign key

References a key in another table and helps enforce relationships.

---

# Automation Framework Design

## 38. How would you design an automation framework?

Example structure:

```text
tests/
clients/
fixtures/
data/
config/
utils/
reporting/
```

Key design decisions:

- Test ownership
- Data strategy
- Retry policy
- Parallelism
- Dependency management
- Logging
- CI integration
- Coding standards
- Configuration
- Secrets
- Cleanup

---

## 39. What makes an automation framework maintainable?

Good frameworks generally have:

- Clear responsibilities
- Low duplication
- Useful abstractions
- Deterministic tests
- Isolated test data
- Actionable failures
- Separated configuration
- Reusable utilities
- Minimal unnecessary complexity

---

## 40. What design patterns might an SDET use?

Common examples:

```text
Factory
Builder
Strategy
Facade
Page/Component Object
Dependency Injection
Singleton — carefully
```

Explain why a pattern solves a problem rather than merely naming it.

---

## 41. What is data-driven testing?

Data-driven testing runs the same test logic against multiple datasets.

Example:

```text
valid user
invalid password
locked account
expired account
```

It reduces duplication when the behavior is structurally the same.

---

# CI/CD

## 42. Where should automated tests run in CI/CD?

A possible pipeline:

```text
Commit
  ↓
Static checks
  ↓
Unit tests
  ↓
API/component tests
  ↓
Build
  ↓
Deploy
  ↓
Smoke tests
  ↓
Broader regression
```

Fast feedback should happen early.

---

## 43. Your CI pipeline takes two hours. What do you do?

First profile where time is spent.

Then consider:

- Parallelization
- Sharding
- Removing duplicate tests
- Moving scenarios to lower layers
- Caching dependencies
- API setup instead of UI setup
- Selective PR suites
- Nightly full regression

Do not optimize by blindly deleting coverage.

---

## 44. What should cause a deployment to fail?

Quality gates can include:

```text
Build failure
Critical unit-test failure
Contract-test failure
Critical integration failure
Critical E2E failure
Security gate failure
Migration failure
Smoke-test failure
```

The policy should reflect actual business risk.

---

## 45. What is the purpose of a smoke test after deployment?

A post-deployment smoke test checks whether the most critical functionality works in the deployed environment.

Examples:

- App starts
- Login works
- Core API responds
- Database connectivity works
- Critical business flow succeeds

---

# Flaky Tests and Debugging

## 46. What is a flaky test?

A flaky test produces inconsistent results against effectively unchanged application behavior.

It may pass on one run and fail on another.

---

## 47. What commonly causes flaky tests?

Examples:

```text
Race conditions
Shared state
Time-dependent behavior
Unstable environments
Network dependencies
Test-order dependency
Incorrect async handling
Random test data
Resource exhaustion
External-system failures
```

---

## 48. How do you debug a flaky test?

A good process:

1. Collect logs and evidence.
2. Run the test repeatedly.
3. Check test-order dependency.
4. Inspect timings.
5. Check shared state.
6. Investigate infrastructure failures.
7. Isolate the smallest reproducible case.
8. Fix the root cause.

---

## 49. Should flaky tests be retried?

Retries can help with rare transient failures.

They should not become the fix for:

- Race conditions
- Bad synchronization
- Shared state
- Weak test design

Track retry frequency and investigate repeated offenders.

---

## 50. What is the difference between a product bug and a test bug?

### Product bug

The application behaves incorrectly.

### Test bug

The test makes an incorrect assumption, uses bad synchronization, or validates the wrong behavior.

A good SDET diagnoses which layer is actually at fault.

---

# Performance and Reliability

## 51. Load vs stress vs spike vs soak testing?

### Load

Expected workload.

### Stress

Push beyond expected capacity.

### Spike

Sudden large increase in traffic.

### Soak / endurance

Sustained workload for a long period.

---

## 52. What metrics would you inspect during performance testing?

Examples:

```text
Latency
Throughput
Error rate
CPU
Memory
Database connections
Queue depth
Network utilization
p95 latency
p99 latency
```

---

## 53. What is p95 latency?

If p95 is `400 ms`, approximately 95% of measured requests completed in 400 ms or less.

The slowest 5% took longer.

Percentiles are often more useful than averages for understanding tail latency.

---

## 54. What is an SLO?

An SLO is a **Service Level Objective**.

Example:

```text
99.9% of requests should succeed each month.
```

SLOs define reliability goals that can guide testing and monitoring.

---

# Distributed Systems

## 55. How would you test eventual consistency?

Do not assume immediate synchronization.

Example:

```text
Write
  ↓
Observe initial state
  ↓
Poll/wait within defined SLA
  ↓
Verify convergence
```

Also test:

- Delayed propagation
- Duplicated events
- Missing events
- Reordered messages
- Retry behavior

---

## 56. How would you test a message queue?

Validate:

```text
Message produced
Message consumed
Payload correctness
Retry behavior
Dead-letter behavior
Duplicate delivery
Ordering
Consumer failure
Idempotency
```

---

## 57. What is idempotency?

An idempotent operation can be repeated without creating unintended additional effects.

Example:

Retrying a payment with the same idempotency key should not charge the customer twice.

---

## 58. How would you test retries in a distributed system?

Validate:

- Retry count
- Backoff strategy
- Duplicate side effects
- Recovery after transient failure
- Permanent failure behavior
- Logging and observability

---

# Security Fundamentals

## 59. What basic security cases should an SDET consider?

Depending on the system:

```text
Authentication
Authorization
Input validation
SQL injection
XSS
CSRF
Session management
Secrets exposure
Sensitive logs
Rate limiting
Dependency vulnerabilities
```

An SDET does not replace a security specialist, but should understand common risks.

---

## 60. Authentication vs authorization?

### Authentication

Who are you?

### Authorization

What are you allowed to do?

---

## 61. How would you test role-based access control?

Test:

```text
Anonymous user
Basic user
Manager
Admin
```

For each role, verify both:

- Allowed actions
- Forbidden actions

Do not validate only hidden UI elements; verify server-side enforcement too.

---

# Situational Questions

## 62. A production bug escaped even though all tests passed. What do you do?

I would:

1. Understand customer impact.
2. Reproduce the bug.
3. Identify why existing tests missed it.
4. Fix the product.
5. Add protection at the cheapest appropriate test layer.
6. Check whether the issue reveals a broader strategy gap.
7. Improve monitoring if detection was weak.

Do not simply add one E2E test and stop.

---

## 63. Developers say QA is slowing releases. How do you respond?

Look at data:

```text
Pipeline duration
Failure rate
False-positive rate
Escaped defects
Time-to-diagnosis
Test distribution
```

Then improve feedback speed without removing critical risk coverage.

---

## 64. A developer says a bug is "not reproducible." What do you do?

Gather:

- Environment
- Build number
- Logs
- Network data
- Timestamp
- User state
- Input data
- Browser/device
- Correlation ID
- Request ID

Then isolate variables systematically.

---

## 65. Requirements change one day before release. What do you test?

Use risk-based prioritization.

Focus on:

- Changed functionality
- Affected integrations
- Critical business flows
- Regression blast radius
- Rollback/recovery

Clearly communicate what was and was not validated.

---

## 66. You have one hour to test a release. What do you do?

Prioritize:

1. Deployment health
2. Business-critical paths
3. Changed areas
4. Major integrations
5. High-risk regression
6. Monitoring and rollback readiness

Document residual risk explicitly.

---

## 67. Two tests pass alone but fail together. What do you investigate?

Look for shared state:

- Same account
- Same database record
- Same test data
- Shared files
- Global variables
- Order dependency

Tests should own the data they mutate.

---

## 68. Your suite is green only after retries. What does that tell you?

It suggests the suite has unreliable signals.

Investigate:

- Flaky tests
- Timing issues
- Shared state
- Environment instability
- External dependencies

A green build after repeated retries is not equivalent to a healthy test suite.

---

## 69. A release has a known medium-severity bug. Would you block it?

Not automatically.

I would communicate:

- Customer impact
- Frequency
- Workaround
- Scope
- Business risk
- Monitoring
- Rollback options

Release decisions should be risk-based.

---

## 70. A critical test fails five minutes before deployment. What do you do?

First determine whether it is:

- Product failure
- Test failure
- Environment failure
- Data issue

If confidence cannot be restored quickly, communicate the uncertainty and business risk clearly rather than ignoring the failure.

---

# Behavioral Questions

## 71. Tell me about a difficult bug you found.

Use STAR:

```text
Situation
Task
Action
Result
```

Focus on:

- Diagnostic process
- Collaboration
- Technical depth
- Customer/business impact

---

## 72. Tell me about a flaky suite you improved.

Explain:

```text
Baseline problem
How you measured it
Root causes
Technical changes
Result
```

Use real metrics when you have them.

---

## 73. Tell me about a disagreement with a developer.

A strong answer demonstrates:

- Respectful disagreement
- Technical evidence
- Focus on customer risk
- Willingness to reconsider
- Clear final outcome

---

## 74. Tell me about a bug you missed.

Do not claim you have never missed one.

Explain:

1. What happened
2. Why it escaped
3. What you learned
4. What you changed

---

## 75. How do you advocate for quality under deadline pressure?

Discuss tradeoffs:

```text
Risk
Impact
Evidence
Mitigation
Release options
Monitoring
Rollback
```

Quality advocacy is about making risk visible, not automatically blocking everything.

---

## 76. Tell me about a process you improved.

Good examples include:

- Reducing pipeline runtime
- Eliminating flaky tests
- Improving test data
- Adding observability
- Improving release gates
- Creating reusable tooling

Show measurable impact where possible.

---

# Senior SDET / Test Architecture

## 77. How do you measure test-suite health?

Useful indicators include:

```text
Flake rate
Execution duration
Failure signal quality
Time to diagnose
Escaped defects
Coverage of critical risks
Maintenance cost
Mean time to detection
Retry frequency
```

Do not rely only on test count or pass percentage.

---

## 78. How would you improve testability of a system?

Possible improvements:

- Stable APIs
- Dependency injection
- Deterministic clocks
- Feature flags
- Test hooks
- Structured logging
- Correlation IDs
- Metrics
- Controllable external dependencies
- Seedable data

A strong SDET influences product architecture, not just test code.

---

## 79. What is observability and why should an SDET care?

Observability uses system outputs such as:

- Logs
- Metrics
- Traces

to understand internal system behavior.

It helps diagnose:

- Distributed failures
- Async workflows
- Performance issues
- Production incidents
- Test failures

---

## 80. What would you test in a feature-flag rollout?

Test:

```text
Flag OFF
Flag ON
Different user cohorts
Configuration failure
Backward compatibility
Rollback
Cached state
Analytics/monitoring
```

Verify both versions can coexist safely during rollout.

---

## 81. How would you design testing for a payment system?

Cover:

- Successful payment
- Declined payment
- Timeout
- Duplicate requests
- Retries
- Idempotency
- Partial failure
- Refunds
- Currency
- Rounding
- Authorization
- Audit trail
- Webhook duplication
- Reconciliation
- Security
- Concurrency

---

## 82. How would you test a system that depends on third-party services?

Use several layers:

```text
Unit tests -> mocks/stubs
Contract tests -> interface compatibility
Integration tests -> controlled environments
E2E tests -> limited real dependency coverage
```

Also test:

- Timeout behavior
- Retry logic
- Fallbacks
- Rate limiting
- Partial outage
- Error mapping

---

## 83. How would you design a test strategy for microservices?

Consider:

- Unit tests
- Contract tests
- Service-level integration tests
- API tests
- Message/event tests
- Limited E2E tests
- Resilience testing
- Observability validation
- Data consistency
- Failure recovery

Avoid relying entirely on full E2E tests.

---

## 84. How do you decide what belongs in unit, integration, and E2E tests?

Use the cheapest layer that can provide sufficient confidence.

### Unit

Pure logic and isolated behavior.

### Integration

Service/database/component interaction.

### E2E

Critical user and business workflows across the full system.

---

## 85. What is testability?

Testability is how easy a system is to observe, control, isolate, and validate.

A highly testable system usually provides:

- Clear interfaces
- Deterministic behavior
- Good logging
- Dependency injection
- Seedable state
- Controlled clocks
- Stable test hooks

---

# Top 25 Questions to Master

If you are short on time, make sure you can answer these without hesitation:

1. What is an SDET?
2. QA Engineer vs SDET
3. What is shift-left testing?
4. Explain the test pyramid
5. Severity vs priority
6. Smoke vs regression vs sanity
7. How do you decide what to automate?
8. What is risk-based testing?
9. What should you validate in an API test?
10. 401 vs 403
11. PUT vs PATCH
12. How do you test an asynchronous API?
13. INNER JOIN vs LEFT JOIN
14. How do you find duplicates in SQL?
15. How would you design an automation framework?
16. What makes a test framework maintainable?
17. How should tests fit into CI/CD?
18. What causes flaky tests?
19. How do you debug flaky tests?
20. Load vs stress vs spike vs soak
21. What is idempotency?
22. How do you test eventual consistency?
23. How would you test a payment system?
24. How do you measure test-suite health?
25. How do you improve system testability?

---

# Interview Answer Framework

For technical scenario questions, a strong answer often follows this structure:

```text
1. Clarify the expected behavior
2. Identify likely failure modes
3. Explain how you would investigate
4. Choose the appropriate test layer
5. Describe the solution
6. Mention tradeoffs
7. Explain what you would avoid
```

For behavioral questions, use:

```text
STAR

Situation
Task
Action
Result
```

---

# Final Preparation Advice

Do not memorize these answers word for word.

For each important topic, be able to:

- Explain it in your own words
- Give a real example
- Discuss tradeoffs
- Describe one common failure mode
- Explain how you would test it
- Write a small implementation when relevant

That combination is usually much more valuable in an SDET interview than memorizing definitions.
