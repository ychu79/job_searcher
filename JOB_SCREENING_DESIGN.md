# Personal New-Grad Software Engineering Job Screening Workflow

Session design summary, updated October 26, 2026.

This document records the decisions and proposals discussed in this chat. It is a design document, not an implementation or an instruction to begin screening. No code, account connections, scheduling, or job screening has been requested as part of saving this document.

## 1. Purpose and priorities

Create a workflow for finding open software engineering new-grad positions, screening employer postings, and producing a shortlist for personal use.

The user prioritizes:

- Simple setup and an easy manual start.
- Finding more potentially suitable jobs over strict accuracy.
- Excluding explicit sponsorship refusals and explicit citizenship or security-clearance requirements.
- Collecting only postings marked as posted today on the GitHub sources.
- Recording every collected encounter while avoiding repeated screening of the same job identity.
- Isolating each posting's review so details from previous jobs do not accumulate in the reviewer's context.

The initial geographic scope is U.S. jobs, with an explicit exception: retain remote jobs regardless of country, subject to the other screening rules.

Google Sheets is the preferred final destination. A local file that can be transferred manually is acceptable. The final delivery and update process remains undecided.

## 2. Discovery sources

Use these three GitHub sources to discover postings:

1. [SimplifyJobs / New-Grad-Positions](https://github.com/SimplifyJobs/New-Grad-Positions)
2. [SpeedyApply / 2027-SWE-College-Jobs — U.S. new-grad positions](https://github.com/speedyapply/2027-SWE-College-Jobs/blob/main/NEW_GRAD_USA.md)
3. [Jobright / 2026-Software-Engineer-New-Grad](https://github.com/jobright-ai/2026-Software-Engineer-New-Grad)

Screen the job on the employer's career website or employer-linked recruiting platform. The repository row is a discovery aid, not the final source for the title, description, or location decision.

Observations from the source inspection during this conversation, which may change:

- Simplify included sponsorship, citizenship, and closed-application markers, but also non-U.S. locations and categories beyond software engineering.
- SpeedyApply's U.S. list included both 2026 and 2027 roles and titles not necessarily limited to new graduates.
- Jobright stated that its list covered only the last seven days. Its links led to Jobright pages, and its entries included internships and non-U.S. locations.

Consequently, membership in a repository does not by itself establish eligibility or that applications remain open. Disappearance from a rolling list does not establish closure.

### Agreed collection window: today only

Every run, including the first, collects only listings marked as posted today on the GitHub source lists. The GitHub listing's displayed date or age controls collection; do not substitute the employer's posting date, first-seen date, or repository commit date.

- Establish the run's date in `America/Los_Angeles` at startup and keep it fixed even if the run crosses midnight.
- For explicit source dates, include only dates matching the run's date.
- For relative ages, treat `0d` as today.
- Log and exclude listings with missing or unrecognizable source dates rather than guessing.
- Collect all matching listings from all three sources at the beginning of the run. There is no 50-posting collection limit.
- Reading a source document containing older rows is fine; only today's rows enter the candidate collection and encounter history.
- Record each source's retrieval time and whether collection succeeded. A retrieval failure must not be represented as a successful collection containing zero jobs.

There is no initial backlog scan, missed-day catch-up, or historical recovery. The user explicitly accepts missing jobs that appear and disappear between runs. These decisions supersede the earlier recommendation to collect each full current list and let history alone determine eligibility for review.

## 3. Agreed screening rules

### 3.1 Title inclusion

Read the target job's title on the employer posting. Match case-insensitively and allow matches anywhere within a word.

The title must contain at least one of:

- `grad`
- `graduate`
- `graduates`
- `early`
- `2026`

The longer graduate terms overlap with `grad`; they are retained here to preserve the user's stated rule.

Do not automatically add other inclusion terms or replace `2026` with the current year. A title containing only `2027` does not pass unless another inclusion term matches.

Accepted limitations of this heuristic:

- Titles such as “Software Engineer I,” “Junior Software Engineer,” or “Entry Level Software Engineer” can fail without one of the listed substrings.
- A title containing `2026` can pass even when it is not actually a new-grad role.
- Substring matching can admit terms such as “undergraduate.”

### 3.2 Internship exclusion

Exclude titles containing any of these as whole words, ignoring case:

- `intern`
- `interns`
- `internship`
- `internships`

This exclusion takes precedence over title inclusion. For example, “Software Engineer Intern — 2026” is excluded.

Apply this rule to the target title, not mentions of internships elsewhere in the description or in recommended jobs.

### 3.3 Location

- Keep jobs located in the U.S.
- Keep jobs identified as remote regardless of country, including an explicit non-U.S. restriction.
- Record remote roles as `Remote` in the location field.
- Exclude non-U.S. jobs that are not remote.

The remote exception is deliberate. The workflow does not need to establish that the user is eligible to work from the specified remote country.

### 3.4 Sponsorship, citizenship, and clearance

Exclude only explicit requirements or refusals in the target posting:

- Visa sponsorship is unavailable, including statements refusing sponsorship now or in the future.
- Citizenship is required.
- A security clearance is required, including an explicit requirement to obtain one.

Do not exclude merely because:

- The employer asks whether an applicant needs sponsorship.
- Clearance is preferred rather than required.
- The posting explicitly says no clearance is required.
- A restriction applies to a different recommended job.

If a successfully read posting does not mention these restrictions, the user wants to assume no citizenship or clearance requirement and that sponsorship is available for screening purposes. Keep the role if it otherwise passes.

Ambiguous restriction wording also remains eligible, with a note explaining the uncertainty. For example, a statement that the employer “may not be able” to support sponsorship is not treated as an unequivocal refusal. Keep the job if all other rules pass; this does not override an explicit requirement or refusal elsewhere in the target posting.

The assistant recommended labeling this outcome “No restrictions found” rather than “Sponsorship confirmed.” That wording distinguishes the screening assumption from verified employer support; the final output wording has not been selected.

### 3.5 Open applications

The intended shortlist is for open roles. The proposed workflow checks the employer posting for closure or an unavailable application.

A loading failure is not evidence that a job is open, closed, or free of restrictions. A redirect to a general careers page must not be treated as a successful read of the original posting.

## 4. Deduplication and screening history

### Agreed identity rule

Use only:

**Consistent company label + URL-derived identity**

The identity is either a posting ID extracted from the employer URL or, when no ID can be extracted, the complete employer posting URL.

- Same company and same ID: skip if already successfully screened.
- Same company and different IDs: screen both, even if titles and descriptions are identical.
- Different URL posting IDs represent different postings, even if they actually refer to the same underlying job or share an internal requisition ID.
- Different companies and the same ID: screen both.
- A previously screened pair that reappears with a new posting date or changed description: skip indefinitely.
- Still record each encounter with a previously screened job; skip only the repeated description review, not the history write.

Do not use title similarity, description similarity, or whole-URL similarity to discard postings. Extract the ID from the URL and compare the company/ID pair. The user explicitly rejected similarity-based matching because distinct positions can share wording.

### Agreed ID extraction: URL only

Use only the employer career-site or employer-linked recruiting-platform URL to extract the posting ID. Do not use visible page IDs or structured job metadata as a fallback or to override the URL ID. This supersedes the earlier proposal to inspect page text or metadata for identity.

Use a site-specific URL rule: a designated path segment or explicitly named job-ID parameter. Do not simply select any number in a URL. Store the extracted ID as text, preserving prefixes and leading zeros.

The examples discussed are:

| Company | Employer posting URL | URL-derived posting ID | Extraction rule |
| --- | --- | --- | --- |
| Roblox | [Example posting](https://careers.roblox.com/jobs/8261393?gh_jid=8261393) | `8261393` | Segment after `/jobs/`; `gh_jid` contains the same value in this example. |
| Amazon | [Example posting](https://www.amazon.jobs/en/jobs/10586548/jr-software-development-engineer-detroit-mi-jr-developer-program) | `10586548` | Segment immediately after `/jobs/`. |
| Microsoft | [Example posting](https://apply.careers.microsoft.com/careers/job/1970393556923390) | `1970393556923390` | Segment immediately after `/careers/job/`. |

During the discussion, Roblox's visible page displayed `46811`, while its URL contained `8261393`. Under the agreed rule, use `8261393`; the visible ID does not affect deduplication. Amazon's visible ID matched its URL ID. Microsoft's page could not be retrieved, including one retry, but its URL still supplied an identifiable posting ID.

Extracting an ID does not mean screening succeeded. An inaccessible posting can have a known identity and still have the status `Could not check`.

If a source uses an aggregator link, obtain the employer posting URL before extracting the identity; do not use the aggregator's record ID. Use the full-URL fallback for a format with no extractable ID. Conflicting job-ID components or an inability to obtain an employer URL should be reported rather than guessed; the exact handling of these cases remains to be decided.

### Agreed fallback: complete URL

When the employer URL has no extractable posting ID, use the company label plus that complete URL as the identity. The user accepted retaining the complete link instead of truncating it.

- Do not truncate the identity: distinct postings can share a long URL prefix.
- Do not substitute an ID from visible page text or structured metadata.
- Distinguish extracted-ID identities from full-URL identities so the two types cannot collide accidentally.
- Different full fallback URLs are different identities, including differences in tracking parameters. Do not silently introduce whole-URL normalization or similarity matching.
- A shortened URL is acceptable for display only; retain the complete identity and original URL in storage.

These postings can proceed through normal screening. A missing extracted ID is not by itself a failed or unrecognized posting once the full employer URL supplies its identity.

### Agreed company naming

Assign consistent company labels to recognized employer career sites or recruiting accounts, while retaining the displayed company name separately. This avoids treating source variants such as “Amazon” and “Amazon.com Services LLC” as different companies when they refer to the same configured employer account.

Normalize capitalization and extra whitespace within those labels. Use explicit mappings rather than automatically merging parents, subsidiaries, or similar company names. For shared recruiting-platform domains, the employer account matters, not just the domain. The initial mappings and handling of unknown accounts remain implementation details to settle.

### Local database persistence

Persist job records, every collected encounter, and screening attempts in a local database. SQLite is the recommended engine for the single-machine workflow; it does not require a separately running database server. The exact schema, runtime, and database location are not implemented or finalized.

Subagents return review results, and the main agent uses the local database interface to save them and maintain the combined output. Centralizing writes avoids having reviewers independently update shared records.

### Agreed access approach: local operations documented in Markdown

Provide a small local command-line interface backed by a database access layer. No running HTTP API service is needed. Each invocation can open the database, perform an operation, return a structured result, and exit.

Markdown instructions should explain the operations, required inputs, response meanings, and error handling. Routine work should not require the agent to compose raw SQL. Identity validation, uniqueness, status transitions, and duplicate-save behavior belong in the interface and database constraints rather than relying only on the agent to remember rules.

The operations discussed are:

- Initialize or validate the database, according to the startup rules.
- Record an encounter and check screening history.
- Reserve an eligible job for one reviewer.
- Save a completed screening and its supporting details.
- Record an unsuccessful attempt and transition exhausted failures to manual review.
- Inspect records for troubleshooting and export results for filtering.

Proposed operation behavior:

- A repeated save of an already completed identity preserves the existing screening result and reports that it was already screened.
- Repeating a save after an uncertain response does not create duplicate results or silently overwrite history.
- A failed lookup is a database error, never evidence that the job is new.
- Removing a job from a displayed shortlist does not erase its screening history.
- Corrections, resets, or deletion are explicit maintenance actions, not generic routine writes that bypass the skip rule.

### Recording encounters separately from screening

The user confirmed that every encounter should be recorded, including previously completed and failed jobs. This does not authorize repeatedly reviewing completed descriptions.

The proposed data organization is:

| Information | Purpose |
| --- | --- |
| Job record | One identity and its current work state and screening result. |
| Appearance history | Each collected encounter, its run and source, source-displayed date, and whether review was skipped or scheduled. |
| Screening attempts | Reviewer assignment, attempt times, success or failure, and supporting details. |

Keep discovery history, work state, and screening outcome separate. A job can have been seen before but never successfully screened. Likewise, a repeated encounter can refer to either a kept or an excluded job. A single status such as “first scan/already scanned/unrecognized” cannot capture all of these combinations reliably.

Illustrative job-record fields, subject to schema design:

| Field | Purpose |
| --- | --- |
| `record_id` | Automatically generated internal primary key. |
| `company_name` | Original company name for display and review. |
| `company_key` | Consistent configured company label, normalized for capitalization and whitespace. |
| `identity_kind` | Extracted URL posting ID or full-URL fallback. |
| `identity_value` | Extracted posting ID or the complete fallback URL, stored as text. |
| `posting_url` | Employer posting URL retained as evidence. |
| `id_source` | URL component or extraction rule used, such as `/jobs/{id}` or `gh_jid`. |
| `title`, `location` | Information read from the target posting. |
| `work_status` | Pending, assigned, completed, or awaiting manual review. |
| `decision`, `reason` | Keep/exclude decision and supporting explanation when screening completes. |
| `screened_at` | Completion date/time; unsuccessful attempts need a separate attempt timestamp. |

The recommended schema uses an internal generated primary key plus a unique constraint on `(company_key, identity_kind, identity_value)`. The identity kind accounts for the newly agreed full-URL fallback. The exact schema and primary-key implementation remain to be finalized.

The completed-screening check requires both a matching identity and a completed result. Both kept and excluded decisions count as completed. An in-progress assignment or unsuccessful attempt does not count as successful screening, but each has its own reason to avoid launching another automatic reviewer.

Completed identities are skipped indefinitely. Jobs awaiting manual review after exhausted reading attempts are also skipped automatically until the user explicitly requests another attempt. In both cases, save new appearances and preserve the existing result or failure record.

## 5. Agent workflow and context isolation

The user prefers a subagent for each posting because a single long-running reviewer could retain irrelevant details from earlier jobs.

The agreed run flow is:

1. Validate startup readiness and fix the run's date in `America/Los_Angeles`.
2. Collect all listings marked today on the three GitHub sources; do not collect an older backlog.
3. Record every collected encounter, establish its company and URL identity, and check history.
4. Skip repeated review of completed jobs and jobs awaiting manual review. Do not duplicate an active assignment.
5. Queue eligible new postings and run up to three fresh reviewers concurrently, one posting per reviewer.
6. Each reviewer reads its target employer posting and returns a compact decision with supporting details.
7. Save each result immediately as it finishes; continue through the eligible queue and prepare output for filtering.

If an employer URL becomes available only after resolving a source link, derive the identity and check history at that point before completing a redundant review. Do not derive identity from the page body. Persist discovery information even if the employer link cannot be resolved; do not invent a completed screening.

### Collection size versus concurrency

Collect all of today's source-listed postings at the beginning of the run. There is no 50-job total or batch checkpoint requirement. Limit active reviewers to three, replacing each finished reviewer with a fresh reviewer for another queued posting. All new eligible postings can be processed in that run, subject to interruptions or execution limits.

Saving completed results immediately allows later work to avoid repeating completed reviews. It does not change the today-only collection policy or automatically authorize collecting an older backlog. The treatment of unfinished pending work after the run date changes remains an open question.

### Proposed persistent work queue

A cursor alone is insufficient because the same identity can appear in multiple sources and work can be interrupted. The recommended local interface records an assignment before launching a reviewer:

- Check eligibility and reserve the job in one atomic database operation.
- Record the assigned reviewer and run, then finish the transaction before browsing begins.
- Other assignment requests for the same active identity return that it is already assigned.
- Save a result against the corresponding assignment and clear or finish that assignment consistently.
- For interrupted work, confirm the previous reviewer has stopped before reassigning it. A timeout alone should not launch overlapping reviewers.

The exact reservation and recovery fields remain implementation details. A cursor may be useful for source traversal or progress display, but it is not the protection against duplicate assignments.

### Proposed subagent contract

- Start without inheriting the parent's conversation history, where the available delegation tools support this.
- Receive the rules or their local file path and one target posting URL.
- Review one posting only; do not reuse that thread for subsequent postings.
- Return company, URL identity and its kind, title, location, employer URL, decision, and a short supporting reason.
- Use distinct outcomes such as `Keep`, `Exclude`, and `Could not check`.
- Return evidence for exclusions and describe failed or incomplete reads.
- Let the main agent write the combined results to avoid conflicting edits.

The goal is separate review context. Subagents do not automatically guarantee accuracy or isolation from shared files, and their creation may inherit history unless explicitly configured otherwise. The initial reviewer concurrency limit is three.

Subagents add model/tool usage and coordination overhead. The user prefers simplicity and context isolation; cost and time targets have not been set.

### AI review versus pre-checks

Options discussed were AI review of each full posting, deterministic pre-checks followed by AI review, and AI review of keyword excerpts only.

The recommended starting approach is AI review of the complete relevant job description, with lightweight identity/title checks where practical. Excerpt-only review was discouraged because it can omit restrictions or necessary context. URL-only ID extraction is agreed; the description-retrieval method and code architecture have not been selected.

## 6. Reading the correct posting

An agent's view depends on its tools: extracted text/HTML, browser page structure and text, or screenshots with browser interaction. It does not necessarily see the page as a person would.

The user specifically raised the risk of confusing recommended jobs with the current posting. Proposed safeguards:

- Identify the target using its main title, employer, URL, and job ID where available.
- Read its description, qualifications, and applicable eligibility statements.
- Ignore recommended-job cards, related roles, navigation, and their requirements.
- Do not follow recommended-job links as part of screening the assigned posting.
- Do not blindly remove the page footer or everything below a fixed point: relevant eligibility language can appear late in the page.
- If the target cannot be isolated or its description cannot be read sufficiently, return `Could not check`.

Separate subagents address cross-posting context accumulation; these page-boundary rules address multiple jobs appearing on the same page.

## 7. Failures, retries, and access restrictions

The user approved **one retry** after a failed reading attempt.

Allow one additional attempt for a temporary reading failure. If both attempts fail, record `Could not check`, move the job to manual review, and continue. A failed read must not trigger the assumption that restrictions are absent.

Website access discussion distinguished:

- Crawler directives such as `robots.txt`.
- Website terms concerning automated access.
- Technical restrictions such as authentication, rate limits, and bot challenges.

There is no universal “AI access” switch. Training crawlers, search crawlers, and user-initiated access may be treated differently. A `robots.txt` directive is not itself an authorization mechanism, and permission for this workflow cannot be inferred merely from a rule about another crawler.

The assistant recommended permitted official feeds/APIs or accessible employer postings when available, otherwise manual review. It did not propose bypassing explicit access restrictions. Using a browser to render JavaScript is distinct from changing tools to evade a denial.

Do not automatically retry those jobs on future runs. Record later collected encounters, preserve the failed-attempt history, and wait for an explicit user request to attempt them again.

## 8. Starting the workflow

The desired user experience is to open the local GitHub project checkout in Codex and ask it to read the rules and perform screening.

The proposed organization is:

- A short `AGENTS.md` that directs Codex to the workflow and requests delegation.
- A detailed Markdown rules file.
- Markdown startup instructions and a reference for the local database operations.
- A local database for persistent screening history and files for exported results.

These files are proposals; this summary does not create or activate that setup.

### Startup and initialization

With the recommended SQLite engine there is no database server to start. The startup instructions should distinguish first-use initialization from validation of existing history:

1. Locate the project and its designated database path.
2. Verify that the local database interface and required runtime are available.
3. On first use, initialize the database and required tables without destroying existing data.
4. On later runs, verify that the expected database and tables are usable.
5. Start collection and screening only after these checks succeed.

A missing database after initial setup must not silently become a new empty database: a wrong path would make all jobs appear unseen. Initialization must preserve existing history. If database validation, lookup, or persistence fails, report the error and stop the affected workflow rather than treating it as empty history or claiming that a result was saved.

Markdown explains the process; the local interface should enforce the data rules. No database or startup interface has been created yet.

In a Codex session already using ChatGPT sign-in, an interactive workflow can use that existing access rather than a separately configured OpenAI API application. Available tools, permissions, usage limits, and website accessibility still apply. Markdown instructions do not install missing tools or enable inaccessible services.

## 9. Results and Google Sheets

Google Sheets is preferred, but integration is not finalized. The user explicitly deferred the final output/update step.

Options discussed:

| Option | Setup and interaction |
| --- | --- |
| Google OAuth connection | Authorize access through Google without putting the account password in the project. |
| Service account | Share a spreadsheet with a dedicated automation account; requires separate authentication and Google Cloud setup. |
| Local TSV file | Copy tab-separated rows into Sheets, or use File → Import. No Google account connection for the workflow. |
| Local Markdown | Readable fallback, less suitable for direct spreadsheet transfer. |

TSV was recommended as the simplest initial handoff. Copying the text and pasting it into a selected cell can transfer rows and columns; it is not literally a single action. A future copy-to-clipboard convenience was discussed but not requested for implementation.

Suggested shortlist columns, not yet finalized:

- Company
- Job ID
- Identity kind, so full-URL fallback records can be distinguished
- Title
- Location
- Employer application URL
- Screening note
- Checked date

Retain rejected decisions, unreadable attempts, and repeated encounters in the database so results can later be filtered. Keep ambiguous restriction statements in the shortlist with notes when other rules pass. The exact presentation of encounter status, excluded and unreadable lists, application notes, and append/replace behavior remains undecided. Scheduling is also deferred.

## 10. Remaining decisions

1. Exact database schema, location, runtime, and local command interface; SQLite is the recommended engine.
2. Initial company/account mappings and handling of unknown company labels, conflicting URL ID components, or unresolved employer links.
3. Queue recovery details, any per-posting effort limit beyond one reading retry, and whether unfinished work from a prior date may be resumed under the today-only policy.
4. Final output format, columns, filters, and Google Sheets transfer method.
5. Output append/replace behavior and any application-tracking fields.
6. Exact tool setup for retrieving employer postings and handling necessary browser rendering.

Resolved since the previous revision: full-URL fallback without truncation; consistent company mappings; recording every encounter without repeating completed screening; manual review after exhausted failures; retaining ambiguous restriction wording; today-only collection on every run; and three concurrent reviewers without a 50-job collection cap.

No graduation date, personal work-authorization timeline, or broader experience-based eligibility filter was established. The agreed title and explicit-restriction rules govern the intended first version.

## 11. References consulted during the discussion

These links supported the discussion; they have not been revalidated as part of saving this summary.

- [Codex subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- [Codex AGENTS.md instructions](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [Codex authentication](https://learn.chatgpt.com/docs/auth)
- [OpenAI computer and browser tools](https://developers.openai.com/api/docs/guides/tools-computer-use)
- [OpenAI crawler distinctions](https://developers.openai.com/api/docs/bots)
- [Google JobPosting structured data and identifiers](https://developers.google.com/search/docs/appearance/structured-data/job-posting)
- [Robots Exclusion Protocol, RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.html)
- [Google Workspace credentials](https://developers.google.com/workspace/guides/create-credentials)
- [Import TSV and other files into Google Sheets](https://support.google.com/docs/answer/40608?hl=en)
- [SQLite initialization and serverless operation](https://www.sqlite.org/zeroconf.html)
- [Appropriate uses for SQLite](https://www.sqlite.org/whentouse.html)
- [SQLite transactions](https://www.sqlite.org/lang_transaction.html)
- [SQLite isolation](https://www.sqlite.org/isolation.html)
