# Job Screening Workflow: General Implementation Plan

Created October 26, 2026.

Source of requirements: [JOB_SCREENING_DESIGN.md](JOB_SCREENING_DESIGN.md).

This plan describes future implementation work. It does not implement the workflow, create a database, start screening, connect accounts, or schedule runs. The design document remains authoritative; proposals and unresolved decisions below must not silently change its agreed rules.

## 1. Intended outcome

The user opens this project in an agent session and asks it to run screening. The main agent follows local Markdown instructions, checks database readiness, collects today's listings from the three configured GitHub sources, and coordinates up to three fresh posting reviewers at once.

Each collected encounter is saved. New jobs are screened, completed jobs are skipped indefinitely, and exhausted reading failures remain available for manual review. The database supports later filtering and export to the selected output format.

The initial implementation should use a local command-line interface backed by a database access layer. SQLite is the recommended database. A separate HTTP server, custom model API integration, dashboard, and automatic schedule are outside the initial scope.

## 2. Proposed components and responsibilities

| Component | Responsibility |
| --- | --- |
| Main agent | Follow startup instructions, collect source listings, manage assignments, save results, and summarize the run. |
| Source readers | Interpret the three GitHub list formats and their displayed dates; retain all of today's eligible rows. |
| Identity resolver | Resolve employer links, apply configured company labels, and extract URL IDs or use full-URL fallback. |
| Local database interface | Validate inputs and enforce persistence, duplicate handling, work reservation, and result transitions. |
| Fresh posting reviewer | Read one employer posting, apply the rules, and return a compact structured result. |
| Export operation | Retrieve selected records and produce the agreed output without modifying screening history. |
| Markdown instructions | Explain startup, operation contracts, screening rules, and recovery behavior. |

Keep repeatable identity and database rules in the local interface. Use the agent for navigating and interpreting job postings. Subagents return results; the main agent performs routine database writes through the interface.

## 3. Phase 1: Establish contracts and project structure

### Work

- Choose the implementation language/runtime based on the local environment and maintenance needs; the design has not fixed one.
- Define the database location, generated-output location, and configuration location relative to a stable project root.
- Define structured inputs, outputs, and error categories for local operations before implementing them.
- Define the review-result contract: job identity, employer URL, title, location, read outcome, screening decision, reason, relevant evidence, and uncertainty notes.
- Separate the meanings of a new encounter, previously known identity, active assignment, completed screening, and manual-review status.
- Establish an initial company/account mapping and URL extraction rules for the Roblox, Amazon, and Microsoft examples in the design.

### Deliverables

- A small project layout and operation contracts.
- A documented distinction between accepted decisions and implementation proposals.
- An explicit list of unresolved behaviors that affect the next phase.

### Completion check

A reviewer can trace one source row through identity resolution, screening, and persistence without relying on an undocumented interpretation of “already scanned.”

## 4. Phase 2: Implement persistence and the local interface

### Work

Implement storage for these logical records; exact table boundaries can be finalized during this phase:

| Record | Information to retain |
| --- | --- |
| Run | Run identifier, fixed Los Angeles run date, start/end times, and completion or interruption status. |
| Source retrieval | Source, retrieval time, success/failure, and collection counts or error details. |
| Job | Company label, original company name, identity kind/value, employer URL, current work state, and completed result where available. |
| Encounter | Run, source listing reference/link, source-displayed date, observed job information, resolved job reference when available, and handling outcome. |
| Assignment/attempt | Target job, owning run/reviewer, lifecycle times, reading attempts, and returned result or failure. |

- Enforce uniqueness of `(company_key, identity_kind, identity_value)` for resolved jobs.
- Store IDs and full-URL fallback values as text without truncation.
- Preserve encounters even when an employer link or company identity cannot yet be resolved; do not invent a completed job record.
- Preserve distinct encounters across sources and runs while making a repeated persistence call for the same encounter safe to retry.
- Keep completed results immutable during routine runs. Repeated saves return the existing completion instead of overwriting it.
- Keep database errors distinct from “not found.”

Expose a small set of workflow operations:

| Operation | Required behavior |
| --- | --- |
| Initialize | Create first-use storage without resetting existing history. |
| Validate readiness | Verify the expected database, schema, and access; fail clearly when unavailable. |
| Begin/finish run | Record the fixed run date and eventual run status. |
| Record retrieval/encounter | Save source collection results and every eligible encounter. |
| Check history | Return completed, pending, assigned, manual-review, or unknown identity state. |
| Reserve work | Atomically check eligibility and assign one job to one reviewer. |
| Save screening | Validate and save a completed keep/exclude result against its assignment. |
| Record failure | Save an attempt or exhausted failure without marking it successfully screened. |
| Inspect/export | Read records and filter results without erasing history. |

### Completion check

Duplicate records, repeated saves, failed lookups, and competing reservations behave predictably. No browser activity or long-running agent work occurs inside a database transaction.

## 5. Phase 3: Implement today's source collection and identity resolution

### Work

- Support the three exact sources linked in the design document.
- Fix the run date in `America/Los_Angeles` once at startup, including runs that cross midnight.
- Read each source's current list and collect only rows whose explicit date matches the run date or whose relative age is `0d`.
- Use the date shown on the GitHub list, not the employer date, repository commit date, or first-seen date.
- Log and exclude missing or unrecognizable source dates. Do not silently guess.
- Collect all qualifying rows without a 50-job cap. There is no initial backlog or missed-day recovery.
- Handle source-specific table structure, including repeated-company markers and multiple application links, without mixing adjacent rows.
- Save each source's success or failure distinctly, so a failed retrieval cannot look like a successful empty list.
- Resolve aggregator links to employer posting URLs through available permitted tools.
- Apply explicit company/account mappings, preserving the originally displayed name.
- Extract IDs only from recognized employer URL components. When no ID can be extracted, use the full employer URL and a distinct fallback identity kind.
- Never substitute visible page IDs or structured metadata IDs, truncate URLs, or deduplicate by title/description similarity.
- Route conflicting URL IDs, unknown company mappings, and unresolved employer links according to the decisions established for those cases.

### Completion check

For controlled source samples, every today's row is accounted for and older rows do not enter candidate history. The same identity from multiple sources produces multiple encounters but only one eligible job assignment. Different URL IDs remain different jobs even when all descriptive fields match.

## 6. Phase 4: Implement assignment and run coordination

### Work

- Build a persistent queue from today's collected, resolved, eligible identities.
- Record an encounter for completed jobs while skipping their description review indefinitely.
- Record encounters for manual-review jobs without automatically retrying them.
- Reserve a job before launching its reviewer; do not rely on a list cursor to prevent duplicate work.
- Keep at most three reviewers active. Launch a fresh reviewer for the next queued posting when one finishes.
- Save each completed result immediately rather than waiting for the entire run.
- If spawning or persistence fails, preserve enough assignment information to explain and recover the interruption.
- Require a result to correspond to its assigned job and active assignment. Do not let a late result overwrite a different completed assignment.
- Confirm an old reviewer has stopped before releasing interrupted work for reassignment. A timeout alone must not create overlapping reviewers.
- Record run outcomes and distinguish completed processing from partial processing or failed source collection.

### Completion check

Two competing assignment requests cannot both acquire the same job. Interrupted processing retains completed results, and recovery does not silently re-review completed jobs or restart exhausted manual-review jobs.

The policy for unfinished work from a previous run date remains unresolved. Do not automatically drain an older queue as part of today's run before that policy is settled.

## 7. Phase 5: Implement isolated posting review

### Work

- Use one fresh subagent per posting, without inherited conversation history where the available tools support that behavior.
- Give the reviewer only the task-specific rules, assignment information, and employer posting URL needed for its job.
- Verify that the selected agent environment supports the required delegation and page-reading tools. Missing capabilities must be reported, not silently replaced with a workflow that violates the design.
- Read the target posting's complete relevant description, including qualifications and eligibility language.
- Keep its title and requirements separate from recommended jobs, navigation, and unrelated page sections. Do not blindly remove page endings that may contain relevant restrictions.
- Treat a redirect to a general jobs page or an incomplete description as an unsuccessful read.
- Use permitted retrieval/browser methods and respect explicit access restrictions.

Apply the exact agreed rules:

| Check | Required behavior |
| --- | --- |
| Title inclusion | Case-insensitive substring match for `grad`, `graduate`, `graduates`, `early`, or `2026` in the employer title. Do not automatically change the year. |
| Internship exclusion | Whole-word match for `intern`, `interns`, `internship`, or `internships` in the title overrides inclusion. |
| Location | Keep U.S. roles and remote roles regardless of country; display remote locations as `Remote`. |
| Restrictions | Exclude explicit sponsorship refusals and explicit citizenship/security-clearance requirements. |
| Missing restrictions | Keep otherwise qualifying roles when a successfully read posting does not mention restrictions. |
| Ambiguous restrictions | Keep otherwise qualifying roles and include a note describing the ambiguity. |
| Open applications | Identify explicitly closed postings; do not infer an open job from a failed page load. |

- Allow one retry after a temporary reading failure. If both attempts fail, return `Could not check` and retain the job for manual review.
- Return compact structured findings with evidence for exclusions and uncertainty notes. Do not return the full page to the main conversation.
- Keep a read failure separate from an exclusion, and validate reviewer output before saving a completed decision.

### Completion check

A manually reviewed sample demonstrates that title boundaries, internship exclusions, remote exceptions, explicit versus ambiguous restrictions, and related-job sections are handled as specified. A failed read never becomes an assumed sponsorship-positive result.

## 8. Phase 6: Write startup and operating instructions

Create the operational Markdown documents only when implementing the workflow. Proposed documents are:

- A short `AGENTS.md` pointing to the startup and screening instructions and requesting the agreed delegation behavior.
- Startup instructions covering runtime checks, database location, first-use initialization, later validation, and run-date selection.
- Screening instructions containing the exact rules and reviewer input/output contract.
- A local-operation reference with actual invocation examples, expected responses, and recovery instructions.

Keep one authoritative operational definition of each rule and cross-reference it to avoid contradictory copies. Update documentation alongside any later changes in behavior.

Startup must distinguish an intentional first initialization from an unexpectedly missing existing database. Do not silently create empty history after a path error or storage loss. Do not begin collection/review when required database checks fail.

### Completion check

A fresh main-agent session can follow the instructions from the project root, validate readiness, and initiate the agreed workflow without inventing SQL, connection settings, or agent instructions.

## 9. Phase 7: Add filtering, export, and a run summary

### Work

- Expose separate filters for new/repeated encounters, work state, screening decisions, ambiguity notes, and manual-review failures.
- Show source collection failures separately from a source having no today's jobs.
- Include counts for encounters, unique resolved jobs, completed-history skips, manual-review skips, new reviews, kept/excluded results, and unfinished work. Keep encounter counts distinct from unique-job counts.
- Finalize output columns and whether the export represents the current run, historical jobs, or both.
- Prefer a local TSV export as the initial proposal for manual Google Sheets transfer; final format remains a user decision.
- Export identity values and long URLs without truncating their stored values. Handle tabs/newlines in descriptive fields so rows remain usable in a spreadsheet.
- Preserve database history regardless of what is filtered out of the exported shortlist.

Direct Google Sheets authorization, append/replace behavior, and application-tracking fields remain separate decisions. Do not make a Google connection a prerequisite for local screening.

### Completion check

The chosen export can be inspected and transferred as intended. Its rows can be traced to stored jobs and encounters, and repeated appearances do not require another description review to appear in the history view.

## 10. Phase 8: Validate the complete workflow

Use focused automated checks for deterministic persistence and collection behavior, plus manual review of a small set of posting decisions. Build these checks alongside their respective phases; this final phase verifies integration rather than repeating every check without a reason.

Important scenarios:

1. First-use initialization preserves any existing data; an unexpected missing database later produces an explicit failure.
2. Today's explicit dates and `0d` rows are included, older/unknown dates are excluded, and the run date stays fixed across midnight.
3. The same company/ID across multiple sources creates multiple encounters and one screening assignment; different IDs with identical descriptions both proceed.
4. A full fallback URL is not truncated, and differing long URL endings remain distinct identities.
5. Repeating the run records new encounters while preserving completed decisions and skipping another review.
6. Competing reservations, interrupted assignments, late results, and repeated save calls do not produce conflicting completions.
7. An initial failed read plus one failed retry leads to manual review; later appearances do not trigger automatic retries.
8. Related-job titles do not affect the target title rule, and relevant eligibility text near the page ending is retained.
9. Ambiguous restriction wording is kept with a note; explicit refusals are excluded; unreadable pages are not interpreted as unrestricted.
10. A source retrieval or database failure is reported honestly instead of producing a misleading successful empty run.

After implementation is authorized, use a small controlled rehearsal before a live run. Any rehearsal limit applies only to validation, not to the production rule of collecting all today's postings. A live first run must still use today's GitHub dates, with no historical backlog.

## 11. Decisions to resolve before the dependent work

| Decision | Needed for |
| --- | --- |
| Runtime, database path, exact schema, and operation names | Persistence implementation and startup instructions. |
| Company/account mappings, conflicting URL IDs, and unresolved employer-link handling | Reliable identity resolution. |
| Available page-reading and fresh-subagent capabilities | Posting review and orchestration. |
| Recovery policy for unfinished work from a previous date | Cross-run queue recovery without violating today-only collection. |
| Source collection failure handling and any effort limits beyond one reading retry | Consistent partial-run behavior. |
| Export format, filters, columns, and current-run versus historical scope | Final output and spreadsheet handoff. |

Resolve ordinary technical choices during implementation and document them. Seek clarification where a choice would change which jobs are collected, reviewed, skipped, or displayed. Scheduling and direct Google Sheets integration can remain deferred while the local workflow is completed.

## 12. Definition of done

- A fresh agent session can follow the documented startup process and invoke the local workflow.
- Only today's GitHub-listed postings are collected, using a fixed Los Angeles run date.
- Every eligible encounter is persisted, including encounters whose screening is skipped.
- Identity follows company mapping plus URL ID/full-URL fallback exactly.
- Completed jobs are skipped indefinitely, and exhausted failures require an explicit retry request.
- At most three fresh reviewers run concurrently, without duplicate active assignments for the same identity.
- Screening applies the exact title, location, restriction, and retry rules.
- Database failures, partial collection, and unresolved records remain visible.
- The chosen result output can be filtered and used, with history preserved.
- Focused validation demonstrates the above behaviors; implementation and documentation agree.
