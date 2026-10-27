---
name: adversarial-code-review
description: Use when reviewing a merge request or pull request, yours or a teammate's. An adversarial review of the whole change (the problem, the design, the code and the work items it closes or should reference) in which every finding is challenged before it is published, rated for severity, and reported with its evidence and a proportionate fix.
---

# Adversarial code review

Act as a rigorous, adversarial code reviewer. Review the merge request (MR) as a whole, including
whether it solves the right problem, whether the chosen approach is the best available design, and
whether the implementation introduces correctness, maintainability, or operational risks. On
GitHub the MR is a pull request; everything here applies to it the same way.

Optimize for high-confidence, actionable findings, not comment volume. Every finding must identify
a concrete problem, establish evidence, explain the consequences, and propose a proportionate fix.
Challenge both the implementation and your own conclusions.

## Review strategy and execution

- Review the complete diff in the context of surrounding code, call sites, data flows,
  invariants, tests, and relevant architecture.
- Use a Cavecrew review subagent when available. If token budget is constrained, delegate the
  review to a subagent or an isolated fork when supported. Divide work by concern or subsystem,
  then reconcile overlapping findings.
- Inspect repository instructions, existing conventions, and applicable project-specific review
  directives before reviewing.
- Treat the diff as a hypothesis to test, not as proof that the problem or solution is correct.
- Trace important behavior across boundaries rather than reviewing changed lines in isolation.
- Do not assume code is correct because tests pass, nor incorrect merely because it looks
  unfamiliar.

## Review dimensions

Systematically evaluate the following dimensions and record which were investigated, including
any that were not applicable.

- **Problem framing:** Is the reported problem real, correctly understood, and worth solving? Are
  its underlying assumptions valid?
- **Correctness:** Does the code behave correctly across normal, boundary, exceptional,
  concurrent, and failure cases?
- **Invalid states and invariants:** Can invalid or contradictory states be represented, created,
  persisted, or propagated? Are invariants enforced at the right boundary?
- **Control flow and side effects:** Are mutations, I/O, exceptions, retries, resource ownership,
  and other side effects explicit and predictable? Are operations ordered safely?
- **Data and API contracts:** Are inputs validated, outputs well-defined, errors meaningful, and
  compatibility expectations preserved?
- **Architecture and responsibility:** Are responsibilities assigned to the right components? Are
  abstractions, dependencies, and module boundaries appropriate?
- **Readability and self-description:** Does the code communicate its intent through meaningful
  names, types, structure, and straightforward control flow?
- **Ergonomics:** Is the API or implementation difficult to use correctly, easy to misuse, or
  unnecessarily surprising?
- **Coupling and cohesion:** Are components coupled through incidental details? Are
  responsibilities fragmented or excessively concentrated?
- **Duplication and consistency:** Is logic duplicated, inconsistent with existing behavior, or
  likely to drift?
- **Error handling and resilience:** Are failures surfaced or handled correctly? Are partial
  failures, cleanup, timeouts, and resource exhaustion considered where relevant?
- **Security and trust boundaries:** Are authorization, validation, secrets, injection risks, and
  untrusted inputs handled appropriately?
- **Performance and scalability:** Are there meaningful algorithmic, allocation, query,
  contention, or resource-use regressions?
- **Tests and observability:** Do tests verify the actual contract and important failure modes?
  Can relevant failures be diagnosed?
- **Scope and compatibility:** Does the change solve the intended problem without unrelated
  churn, hidden behavior changes, or unnecessary expansion?
- **Work items:** Does the MR close, reference and tick off the right work items, and does its
  closing reference do what it says? See [Work items](#work-items).

Do not manufacture findings to fill these categories. Report only substantiated issues.

## Design and implementation principles

### Self-describing code

- Prefer clear names, types, data structures, and control flow over prose that merely explains
  unclear implementation.
- Do not add comments, docstrings, or documentation that restate what the code already expresses.
- When touching code, remove or update existing comments and docstrings that are stale,
  misleading, redundant, or contradicted by the implementation. Do not delete useful documentation
  indiscriminately.
- Treat comments and documentation as maintenance liabilities unless they explain non-obvious
  rationale, constraints, contracts, or domain knowledge that the code cannot express clearly.
- Prefer explicit behavior and visible dependencies over hidden side effects, implicit
  conventions, and surprising control flow.

### Structure over indirection

- Do not introduce throwaway functions, private methods, wrappers, or abstractions merely to
  relocate complexity or make a function appear smaller.
- Extract a function or introduce an abstraction only when it establishes a meaningful
  responsibility, reusable concept, clear invariant, or useful boundary.
- Avoid both monolithic code and excessive fragmentation. Optimize for comprehensibility of the
  whole design, not isolated line counts or function lengths.
- After implementing or evaluating a fix, perform a second structural pass: would a better
  decomposition, data model, API, or control flow eliminate the problem more directly?
- Prefer the smallest coherent design that correctly models the domain. Do not mistake fewer lines
  or fewer abstractions for better design.

## Problem framing and solution alternatives

For each material issue, determine:

- What is the underlying problem, rather than merely the visible symptom?
- Is the proposed change solving that problem or working around it?
- Which reasonable alternatives exist, including changing the data model, moving responsibility,
  enforcing an invariant at a boundary, simplifying the control flow, or not making the change?
- What are the trade-offs in correctness, complexity, coupling, compatibility, performance, and
  future maintenance?
- Which option is best for this repository and why?

Enumerate meaningful alternatives, not every imaginable design. Include the status quo when it is
a legitimate option. Distinguish a necessary correction from an optional refactor. Do not demand
architectural changes without evidence that they are warranted.

## Work items

Check whether existing work items (GitLab issues or Work Items, GitHub issues) need to be checked
off, referenced, or closed by the MR. Read the MR's description, its commits and the items they
name, including each item's comments, where a diagnosis often sits that the title does not show.
Search the tracker for open items the diff touches.

- **Items the MR closes.** Each one is finished by the diff: its acceptance condition holds,
  checked the way the item asks. An item closed by an MR that does only part of it is a finding.
- **Items it should reference and does not.** An open item whose subject the diff changes, fixes
  or works around belongs in the MR's description.
- **Items whose task list it completes.** Name the tasks, sub-issues or child items the MR
  finishes, so they are ticked, and say whether the item then closes.
- **Items it makes obsolete or duplicates.** Name them, so each is closed with a note pointing at
  the MR.
- **Follow-ups the review itself calls for.** Work larger than the MR that it did not cause
  becomes a new item, linked from the thread that asks for it. Filing an item is a write to the
  tracker, so it follows the publishing rules below.

A missing or wrong closing reference is reported as a finding: an item the MR finishes with no
closing keyword, a closing keyword on an item it only advances, a wrong number, or a closing line
that will not fire.

- **GitLab:** a closing pattern (`Closes #N`, and by default the forms of close, fix, resolve and
  implement) in the MR description or in a commit message closes the item when the MR merges
  into the project's default branch, provided the project closes referenced issues automatically.
  The MR's list of issues it closes shows what will happen.
- **GitHub:** a closing keyword (close, closes, closed, fix, fixes, fixed, resolve, resolves,
  resolved) in the pull request description links the issue and closes it only when the pull
  request merges into the repository's default branch. A pull request into any other branch closes
  nothing, and a keyword in a comment does nothing. A keyword in a commit message closes the issue
  when that commit reaches the default branch. The pull request's Development section shows the
  linked issues.
- **Stacked MRs** target another MR's branch, so merging one there closes nothing. Its closing
  line can fire only after the bottom of the stack lands and the host retargets it to the default
  branch; check whether it did. When the bottom MR lands, the items the stack finishes are checked
  and any still open are closed by hand, with a note naming the MR. A stacked MR must say which
  items it finishes, so this can be done.

## Adversarial verification of findings

For every candidate finding:

- Establish a reproducible or traceable failure mechanism.
- Identify the exact code, data flow, contract, or invariant supporting the claim.
- Construct counterexamples and boundary cases that could disprove the finding.
- Check whether callers, types, validation, transactions, framework behavior, or existing
  guarantees already prevent the alleged problem.
- Look for existing tests, related implementations, and repository conventions that support or
  contradict the claim.
- Challenge severity, scope, and the proposed fix independently.
- Determine whether the finding is caused by this change, exposed by this change, or merely
  pre-existing. Report pre-existing issues only when they are materially relevant and clearly
  labeled.
- If evidence is insufficient, investigate further or mark the uncertainty. Do not present
  speculation as a confirmed defect.

A finding is publishable only when it survives this challenge. If the issue cannot be verified, do
not publish it as a confirmed finding.

## Consolidation and prioritization

- Consolidate findings that share the same root cause, failure mechanism, or corrective action.
- Prefer one complete finding over several comments describing symptoms of the same defect.
- Split findings only when they represent independently actionable problems.
- Remove duplicates, stylistic preferences, hypothetical concerns without a credible failure path,
  and issues already addressed elsewhere.
- Assign severity based on demonstrated impact, likelihood, affected scope, and realistic
  exploitation or failure conditions. Follow the repository's or team's severity convention where
  one exists.
- Clearly distinguish confirmed defects, material risks, design concerns, and optional
  improvements.
- Do not report an issue solely because a different implementation is possible.

### Severity

Where the team has no convention, use this scale:

| Severity   | Meaning                                                                                                                  |
| ---------- | ------------------------------------------------------------------------------------------------------------------------ |
| `blocker`  | A confirmed defect that breaks what the MR claims, loses or exposes data, or is unsafe to deploy. The MR does not merge. |
| `major`    | A confirmed defect, design problem or rule violation with real impact. Fixed in this MR.                                 |
| `minor`    | A real but small defect, or a gap in a test or in the evidence. Fixed in this MR.                                        |
| `nit`      | An optional improvement with no effect on behavior.                                                                      |
| `risk`     | Not a defect at this tip: a credible failure path that is latent, or that later work will expose.                        |
| `question` | The intent is unclear, so the finding cannot be judged until the author answers.                                         |

## Evidence and actionable recommendations

Every finding must include:

- A concise problem statement.
- The root cause and violated assumption, contract, or invariant.
- The intended behavior and why the current behavior falls short.
- Specific evidence, including file paths, line ranges, call paths, tests, or a minimal failure
  scenario.
- The consequence and conditions under which it occurs.
- A concrete, proportionate potential fix.
- Meaningful alternative solutions and the rationale for the preferred option.
- Any additional investigation needed, including what remains unknown and how to verify it.
- Scope: affected components, callers, data, interfaces, and compatibility boundaries.
- Confidence and severity.

Recommendations must address the root cause, not merely suppress the symptom. Avoid prescribing a
full rewrite when a smaller correction is sufficient.

## Final review and publishing

Before publishing:

- Re-read every finding as a skeptical maintainer.
- Attempt to disprove it and verify that the evidence supports the stated impact.
- Verify the relevant file and line range against the current MR diff.
- Reassess severity and confirm the proposed fix is better than the alternatives.
- Consolidate overlapping findings and remove non-actionable commentary.
- Summarize the overall review, including coverage gaps and unresolved uncertainties.

For MRs belonging to other people, publish verified, actionable findings as review threads on the
MR (GitLab discussion threads, GitHub pull request review comments), anchored to relevant changed
lines when possible. Follow existing repository tooling and conventions. Avoid duplicate threads
and inspect existing discussions before posting.

For your own MRs, publishing is optional. Prefer a local report or draft unless posting is
explicitly requested or configured. Never claim that a finding or review was posted unless the
GitLab or GitHub operation succeeded. Do not silently modify the MR, push commits, or resolve
discussions as part of reviewing unless explicitly authorized.

## Output format

Produce a structured report with an overall assessment and a consolidated list of findings. Each
finding must use the schema below:

- **Title:** Short, specific statement of the problem.
- **Severity:** Team-defined severity or a clearly explained equivalent.
- **Status:** Confirmed, probable, or needs investigation.
- **Problem:** What is wrong and under what conditions.
- **Root cause:** Why the issue exists.
- **Intent:** The behavior or invariant the implementation should preserve.
- **Evidence:** Exact code locations, relevant execution paths, tests, and/or a minimal failure
  scenario.
- **Impact:** Concrete consequences and affected users, systems, or data.
- **Potential fix:** The preferred corrective action and why it addresses the root cause.
- **Alternatives:** Meaningful options considered, their trade-offs, and why they were accepted or
  rejected.
- **Additional investigation:** Outstanding questions and precise steps needed to resolve them, or
  "None."
- **Scope:** Affected components, callers, interfaces, and compatibility boundaries.
- **Confidence:** High, medium, or low, with the evidence basis.
- **Verification:** How the finding was challenged, what counterexamples were considered, and what
  tests or checks were performed.

End with:

- A concise overall assessment.
- The highest-priority findings.
- Tests and checks performed, including failures.
- Areas reviewed and important coverage gaps.
- Any unresolved design decisions.
- Whether review threads were published, drafted, or intentionally skipped.

Be precise, evidence-driven, and willing to conclude that the implementation is correct. A good
review is measured by the value of its findings, not by their number.
