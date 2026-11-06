# Project workflow rules

The team consists of: Viktor (CTO), Lea (UX Designer), Arno (Architect), Ben (Backend), Frida (Frontend), Dario (DevOps), Quinn (QA), Selma (Security Reviewer), Sina (Security Tester).

## Pipeline for any new feature

1. **ALWAYS start with viktor-cto.** No exceptions. If Viktor rejects, the feature stops.
2. **After Viktor approves**, invoke **lea-ux-designer** and **arno-architect** — these can run in parallel.
   - Lea produces the design spec (`docs/design/<feature-slug>.md`)
   - Arno produces the architecture doc + task breakdown (`docs/architecture/<feature-slug>.md`)
3. **Implementation phase** — invoke ben-backend, frida-frontend, dario-devops in the order dependencies allow. They never modify each other's domain.
   - **frida-frontend** MUST follow Lea's design spec. Deviations from the spec are bugs.
4. **After every implementation task**, ALWAYS invoke quinn-qa before considering the task done.
   - Quinn checks frontend work against BOTH Arno's architecture spec AND Lea's design spec.
5. **selma-security-reviewer** runs proactively on any commit touching auth, payments, user data, file uploads, or external API calls.
6. **sina-security-tester** is invoked explicitly for high-risk features — auth flows, payment flows, admin interfaces, anything handling PII.

## Pipeline diagram

```
PRD → viktor-cto → lea-ux-designer + arno-architect (parallel)
                         ↓                    ↓
                   Design Spec          Task Breakdown
                         ↓                    ↓
                   frida-frontend ←→ ben-backend + dario-devops
                         ↓
                   quinn-qa (after every task, checks spec + design)
                         ↓
                   selma-security-reviewer (proactive on sensitive code)
                         ↓
                   sina-security-tester (explicit for high-risk features)
```

## Hard rules

- No subagent modifies another subagent's domain.
- No implementation without an approved architecture doc.
- No frontend implementation without Lea's design spec.
- No "done" without Quinn's signoff.
- Security findings from Selma or Sina are blocking unless explicitly waived.
- **No control may claim an effect the server does not deliver — and none may claim a restriction the server does not impose.** Both halves are bugs. A switch that saves a preference nothing reads is the first (v2.12 Ship's view). A label telling an admin that un-departmented accounts "have no folder access" when those are the *unrestricted* ones is the second, and it is the more dangerous of the two, because it points attention away from the risky accounts.
- **HTTP-level verification does not count as verification of a UI.** Quinn and Sina drive the API; a real browser is a different environment and hides different bugs. In v2.12 the department grid editor was entirely broken — `structuredClone()` throws on a Svelte `$state` proxy — and every curl-based test passed, because over HTTP the value is never a proxy. Any task that changes a screen needs someone to open that screen.
- **Every agent reports what it did NOT test.** A report that omits its own blind spot reads as full coverage. "I exercised this over HTTP only; I never opened the UI" is the sentence that would have caught the above on the same day.
