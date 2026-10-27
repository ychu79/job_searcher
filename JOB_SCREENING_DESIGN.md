# Personal New-Grad Software Engineering Job Screening Workflow

Session design summary, compiled October 26, 2026.

This document records the decisions and proposals discussed in this chat. It is a design document, not an implementation or an instruction to begin screening. No code, account connections, scheduling, or job screening has been requested as part of saving this document.

## 1. Purpose and priorities

Create a workflow for finding open software engineering new-grad positions, screening employer postings, and producing a shortlist for personal use.

The user prioritizes:

- Simple setup and an easy manual start.
- Finding more potentially suitable jobs over strict accuracy.
- Excluding explicit sponsorship refusals and explicit citizenship or security-clearance requirements.
- Avoiding repeated screening of the same company and URL-derived posting ID.
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

The assistant recommended labeling this outcome “No restrictions found” rather than “Sponsorship confirmed.” That wording distinguishes the screening assumption from verified employer support; the final output wording has not been selected.

### 3.5 Open applications

The intended shortlist is for open roles. The proposed workflow checks the employer posting for closure or an unavailable application.

A loading failure is not evidence that a job is open, closed, or free of restrictions. A redirect to a general careers page must not be treated as a successful read of the original posting.

## 4. Deduplication and screening history

### Agreed identity rule

Use only:

**Company name + posting ID extracted from the employer posting URL**

- Same company and same ID: skip if already successfully screened.
- Same company and different IDs: screen both, even if titles and descriptions are identical.
- Different URL posting IDs represent different postings, even if they actually refer to the same underlying job or share an internal requisition ID.
- Different companies and the same ID: screen both.
- A previously screened pair that reappears with a new posting date or changed description: skip indefinitely.

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

If a source uses an aggregator link, obtain the employer posting URL before extracting the identity; do not use the aggregator's record ID. The proposed behavior for an unfamiliar URL format or conflicting ID values is to report unresolved identity rather than guess. Exact conflict-handling rules remain to be decided.

### Local database persistence

The user clarified that the requirement is for the agent to save the results of scanned postings in a local database. A separately running HTTP API service is not required for this requirement. The database technology and access mechanism have not been selected.

The proposed division of responsibility remains: subagents return review results, and the main agent writes them to the local database and maintains the combined output. This avoids having multiple reviewers independently update shared records.

Proposed record fields:

| Field | Purpose |
| --- | --- |
| `record_id` | Automatically generated internal primary key. |
| `company_name` | Original company name for display and review. |
| `company_key` | Company name normalized for capitalization and extra whitespace. |
| `url_job_id` | Posting ID extracted exclusively from the employer posting URL. |
| `posting_url` | Employer posting URL retained as evidence. |
| `id_source` | URL component or extraction rule used, such as `/jobs/{id}` or `gh_jid`. |
| `title`, `location` | Information read from the target posting. |
| `screening_status` | Whether screening completed or could not be completed. |
| `decision`, `reason` | Keep/exclude decision and supporting explanation when screening completes. |
| `screened_at` | Completion date/time; unsuccessful attempts need a separate attempt timestamp. |

The assistant recommended an internal generated primary key plus a unique constraint on `(company_key, url_job_id)`. A composite primary key was also discussed as a valid alternative; the exact database schema has not been explicitly approved.

The skip check must require both a matching company/URL-ID pair and completed screening. Both kept and excluded decisions count as completed. The mere existence of a database row, an in-progress assignment, or a failed attempt must not count as successful screening.

Proposed database operations are checking history, recording completed screenings, recording unsuccessful attempts, and exporting results. Removing a job from a shortlist should not erase its screening history. Any reset for intentional rescreening would be an explicit action, not routine behavior.

### Proposed history handling

- Persist history in the local database between runs.
- Record both kept and excluded postings as successfully screened.
- Keep unsuccessful reads separate from completed screening.
- Have the main agent maintain the shared history and combined output.
- Normalize company names only for capitalization and extra whitespace; do not automatically merge subsidiaries or company aliases.

Local database persistence is agreed. The remaining handling details above were recommended, but not all were explicitly confirmed.

### Unresolved: no extractable URL posting ID

How to handle a posting whose employer URL does not expose a usable job ID remains unresolved. The URL-only decision rules out using a page-body or metadata ID instead.

The proposed default is to retain it as an unresolved record or in a separate review list without deduplicating it. This can produce repeated entries across runs. The user has not approved this default. Do not invent an ID, use the entire URL as a replacement key, or treat all missing-ID records for a company as duplicates.

## 5. Agent workflow and context isolation

The user prefers a subagent for each posting because a single long-running reviewer could retain irrelevant details from earlier jobs.

The proposed design is:

1. The main agent collects candidate links from the three sources.
2. It extracts posting IDs from employer URLs and checks company/ID pairs against the local database where identity is available.
3. It assigns each remaining posting to a fresh subagent, with limited concurrency.
4. The subagent reads the employer posting and applies the same screening rules.
5. The subagent returns a compact result rather than the full page.
6. The main agent records the result in the local database and updates the combined output.

If an employer URL becomes available only after resolving a source link, extract its ID and check history at that point before completing a redundant review. Do not derive identity from the page body. The main agent should also avoid assigning the same known company/ID pair twice within a run.

### Proposed subagent contract

- Start without inheriting the parent's conversation history, where the available delegation tools support this.
- Receive the rules or their local file path and one target posting URL.
- Review one posting only; do not reuse that thread for subsequent postings.
- Return company, URL-derived posting ID if available, title, location, employer URL, decision, and a short supporting reason.
- Use distinct outcomes such as `Keep`, `Exclude`, and `Could not check`.
- Return evidence for exclusions and describe failed or incomplete reads.
- Let the main agent write the combined results to avoid conflicting edits.

The goal is separate review context. Subagents do not automatically guarantee accuracy or isolation from shared files, and their creation may inherit history unless explicitly configured otherwise. The exact concurrency limit has not been chosen.

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

The proposed interpretation is one additional attempt for a temporary failure. After that, record `Could not check` and continue. A failed read must not trigger the assumption that restrictions are absent.

Website access discussion distinguished:

- Crawler directives such as `robots.txt`.
- Website terms concerning automated access.
- Technical restrictions such as authentication, rate limits, and bot challenges.

There is no universal “AI access” switch. Training crawlers, search crawlers, and user-initiated access may be treated differently. A `robots.txt` directive is not itself an authorization mechanism, and permission for this workflow cannot be inferred merely from a rule about another crawler.

The assistant recommended permitted official feeds/APIs or accessible employer postings when available, otherwise manual review. It did not propose bypassing explicit access restrictions. Using a browser to render JavaScript is distinct from changing tools to evade a denial.

The treatment of failed attempts on future runs remains unresolved; the one-retry decision does not establish whether to retry again on a later day.

## 8. Starting the workflow

The desired user experience is to open the local GitHub project checkout in Codex and ask it to read the rules and perform screening.

The proposed organization is:

- A short `AGENTS.md` that directs Codex to the workflow and requests delegation.
- A detailed Markdown rules file.
- A local database for persistent screening history and files for exported results.

These files are proposals; this summary does not create or activate that setup.

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
- Title
- Location
- Employer application URL
- Screening note
- Checked date

Internal history should retain decisions for rejected jobs as well. Whether to expose separate excluded and unreadable lists, preserve application notes, append rows, or produce a fresh output file remains undecided. Scheduling is also deferred.

## 10. Remaining decisions

1. What to do when no posting ID can be extracted from the employer URL, or URL ID components conflict.
2. Whether to retry previously unreadable postings on later runs.
3. Final output format, columns, and Google Sheets transfer method.
4. Output append/replace behavior and any application-tracking fields.
5. Exact tool setup for retrieving employer postings and handling necessary browser rendering.
6. Subagent concurrency and any per-posting effort limit beyond the agreed one retry.
7. Local database technology, exact schema, access mechanism, and company-name normalization details.

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
