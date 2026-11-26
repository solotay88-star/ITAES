# IT Asset EOS Scanner (ITAES) — User Requirements for OpenCode

## How to use this file in OpenCode

Copy the prompt in the box below into OpenCode, while you are in your
`opencode-trial` folder. Work on a copy there, not on live asset data.
Use only small made-up sample files for testing. Do not use real asset
lists, customer data, passwords, or API keys in the trial folder.

> Build me a single-file HTML program called the IT Asset EOS Scanner
> (ITAES), following the requirements in this file. It must run in a
> browser in my office, load an asset list from Excel, plain text or
> CSV locally with no upload, fetch End of Support dates live from
> public sources using only non-identifiable data, flag assets past
> End of Support or within 3 months of it, show me a summary and a
> sortable findings table, and export an enhanced Excel file with the
> EOS information and its sources added.
> Build it step by step, test it with small sample files you create,
> and tell me what you tested and what still needs a real asset file
> to check. No server, no build step, no external libraries, no paid
> APIs.

## 1. Who it is for and why

I am an IT auditor. My role requires me to determine whether a
particular IT asset has reached its End of Life and End of Support,
so the organisation can act on assets that are no longer supported.

I have to run this program inside my office environment, as I cannot
access the asset data files outside of my office environment. The best
choice is therefore a single HTML program I can copy into the office
and open in a browser there.

The program is for one auditor working on one asset list at a time.
It is not a shared system, not a server, and not a database. Each
scan is standalone — no history is carried across scans.

## 2. Scope for version 1

### In scope

1. Load one asset list file at a time, by drag and drop or a file
   picker, in Excel (.xlsx or .xls), plain text, or CSV format.
2. Load a separate vendor-to-source mapping file (Excel), maintained
   by the organisation's IT department administrator.
3. Parse the files locally in the browser and show a summary and a
   findings table.
4. Fetch End of Support dates live from public sources, using only
   data that is not identifiable to the organisation.
5. Flag assets past End of Support, and warn on assets within
   3 months of it.
6. Filter, search, and sort the findings.
7. Export an enhanced Excel file for audit workpapers, with the EOS
   information and its sources added.
8. Clear all loaded data from the page.

### Out of scope for version 1

- Loading several asset lists at once and merging them.
- Keeping history across scans, or comparing one scan with another.
- User accounts, logins, saving data on a server, or a database.
- Storing vendor portal credentials, or scraping from behind a
  vendor login.
- Changing any asset or system, or applying any patch.
- A paid API, a subscription, or any charge to run the program.

## 3. Functional requirements

### FR-01 — Load an asset file locally, with no upload

- Accept an asset list in Excel (.xlsx or .xls), plain text, or CSV
  format.
- Support drag and drop onto the page and a normal file picker.
- Show the loaded file name, and after parsing show how many assets
  were found.
- The asset file must never be uploaded anywhere. All parsing must
  happen in the browser on my computer, using FileReader or
  equivalent client-side code.
- If the file is not a valid asset file, show a clear error and do
  not crash the page.

### FR-02 — Parse an asset file flexibly

Asset lists may have been processed by the party collating the
results, so fields may not be in a structured format. Parsing must
be as flexible as the VAT tool's (delimiter detection, header
synonyms, multiple sheets, varying headers), and must produce a
parse report listing what was recognised and what was not.

Each asset usually has these fields:

- Mandatory: name, brand, version, status of usage (in production,
  in testing, decommissioned, etc).
- Optional: description, model, purchased date, commissioned date.
- Identity fallback fields (see FR-03): location, IP address.

- If a row is missing a mandatory field, flag the record as
  incomplete, needing review. Do not drop the row silently.
- A worksheet with more rows than the configured limit, or a cell
  longer than the configured limit, is rejected under NFR-05.

### FR-03 — Asset identity

- An asset is identified by a unique identifier assigned by the
  organisation, where the asset list carries one.
- If there is no such identifier, fall back to: Name of Asset,
  Brand, Model, Version, Location, IP address.
- If the fallback still does not make the asset unique, flag the
  record for attention. Do not merge two records that are not
  proven to be the same asset, and do not count one asset twice.

### FR-04 — Load the vendor-to-source mapping file

- The vendor-to-source mapping is a separate Excel file, maintained
  by the organisation's IT department administrator.
- Load it by file picker, separately from the asset file.
- The mapping says, for a vendor or product, which external source
  to check for its End of Support information.
- Proposed columns for the mapping file, to be confirmed (see §6):
  Brand, Product or name pattern, Source URL, Source type
  (aggregator / vendor page / manual), Last known EOS date, Notes.
- The sources are dynamic: which sources are relevant depends on
  what is maintained in the asset list. There is no fixed shortlist
  of sources.

### FR-05 — Fetch End of Support data live

- The tool fetches EOS data live from the internet itself, rather
  than the auditor looking it up and entering the findings.
- Only data that is not identifiable to the organisation may be
  sent to an external source for referencing, e.g. version number,
  software name, or brand. Never send organisation-identifiable
  data such as hostnames, asset descriptions, dates, or usage
  status.
- Triage with a public aggregator first: endoflife.date for
  software (https://endoflife.date), EOSL.ai for hardware
  (https://eosl.ai). Both are free, public, and need no key.
  Treat an aggregator date as non-authoritative triage, and
  record the aggregator as the source.
- Then check the vendor's own lifecycle page for that product,
  using the source from the mapping file (FR-04).
- Show a progress bar and a status line such as
  "Checked 120 / 500 assets…", and a final line with the date and
  time the check finished, so I can record it in my workpapers.
- Pace requests so as not to overload a source. If a source
  returns an error or a rate-limit response, record the failure
  for that asset only and carry on with the rest. Do not stop the
  whole run.
- If the office internet cannot reach a source, show that per
  asset or in the status line, and keep the parsed results
  visible. Do not show a blank page and do not lose the loaded
  file.

### FR-06 — Detection rule and statuses

- EOS means End of Support. EOL means End of Life.
- A finding is flagged on End of Support, not on End of Life.
- Flag (red) when the End of Support date has been exceeded.
- Amber status when within 3 months before the End of Support
  date.
- More than 3 months before End of Support, the asset is not
  flagged. (Analyst's reading of Solo's answers; to be confirmed
  — see §6.)
- If no EOS is published for a version, note it as "Not published".
  That also means the asset has not met the flag condition: it is
  not flagged.
- A finding is a flag to investigate, not a verdict. The auditor
  investigates further by referencing the external source.

### FR-07 — Extended support

- A vendor may offer extended support for a version that is
  already End of Support. Whether it applies depends on whether
  the organisation has subscribed to it.
- Extended support defaults to No for each asset. The user can
  overwrite that default as input, e.g. by supplying the value in
  the input file or setting it in the tool.
- The tool must show the extended support value for each asset.
- Open question (see §6): how does extended support = Yes affect
  the flag status of an asset past End of Support?

### FR-08 — Login-gated and unknown vendors

Some vendor lifecycle pages need a login (Broadcom/VMware, SAP,
Fortinet). For those, and for a vendor with no known source in the
mapping, use this order:

1. Public aggregators first (endoflife.date, EOSL.ai) as
   non-authoritative triage.
2. Public vendor documents second (KB articles, release notes,
   public EOS notices), via the mapping file.
3. Last-known EOS date held in the mapping file third.
4. Manual check as the last resort: the tool flags the asset as
   needing a manual check and gives a deep link to the vendor
   portal. The auditor checks it with the organisation's login
   and enters the EOS date and source into the tool.

- Each asset must be labelled with which route its EOS date came
  from: aggregator, vendor page, mapping file, or manual check.
- No vendor portal credentials are stored in the tool, and nothing
  is scraped from behind a login.
- For a vendor with no known source in the mapping at all, the
  last resort if all else fails is a manual check.

### FR-09 — Summary dashboard

After a file loads and the check runs, show these counts, as simple
summary cards at the top of the results:

- Total assets
- Flagged (past End of Support)
- Amber (within 3 months of End of Support)
- Not published (no EOS published)
- Incomplete (missing a mandatory field)
- Flagged for attention (identity still not unique)
- Manual check needed

### FR-10 — Findings table

Show one row per asset, with these columns:

| Column | Content |
|---|---|
| Asset ID | Organisation-assigned identifier, or the fallback key (FR-03) |
| Name | Asset name |
| Brand | Brand |
| Model | Model, where present |
| Version | Version |
| Status | Status of usage |
| EOS Date | Published End of Support date, or "Not published" |
| Phase | End of Support / End of Life, where known |
| Result | Flagged, Amber, Not published, Incomplete, or Attention |
| Source | Source URL for the EOS date |
| Route | Aggregator, vendor page, mapping file, or manual check |
| Extended Support | Yes / No |

- Clicking a column heading sorts by that column. Sorting by EOS
  date must sort chronologically, not alphabetically.
- Provide a text search across name, brand, and version.
- Provide a result filter (All, Flagged, Amber, Not published,
  Incomplete, Attention, Manual check needed).
- The table area must scroll, and column headings must stay visible
  while scrolling.

### FR-11 — Export the enhanced file (Excel)

- Export an enhanced version of the source (input) file, in Excel
  (.xlsx) format.
- The enhanced file is a new copy: the original input file is never
  overwritten.
- Keep the input file's columns, and add these columns, in this
  order, after them: Source URL, Date Accessed, Quoted EOS Date,
  Phase, Verified By. More columns may be added later.
- Verified By means who or what verified the EOS date: the source
  for fetched data, or the auditor's name or initials for manually
  entered data (the tool must let the auditor type this when
  entering a date).
- Name the file e.g. `itaes-enhanced-YYYYMMDD.xlsx`.
- The enhanced file is a snapshot at the time the tool ran it.
  EOS data can change over time; the program must not claim a date
  is permanent.

### FR-12 — Clear

- Provide a Clear button that empties the table, the summary, the
  file inputs, and the status line, so the next scan starts clean.

## 4. Non-functional requirements

### NFR-01 — Single file, no install

- Deliver one single HTML file, e.g. `itaes-tool.html`, with inline
  CSS and JavaScript.
- Double-clicking it in Chrome or Edge in the office must open it.
  No server, no Node.js, no build step, no install, no admin rights.
- No external libraries, frameworks, or CDN links. All CSS and
  JavaScript must be inside the file, so it still works offline —
  including the Excel reader (zip and XML parsing in the page
  itself). The only network calls the page may make are the EOS
  fetches in FR-05, FR-08, and the mapping-file sources.

### NFR-02 — Privacy and security

- The asset file, and anything in it, must never leave my computer,
  except for the non-identifiable fields in FR-05 (version, software
  name, brand) sent to an external source for referencing.
- Sensitive data must stay on site.
- No cookies, no tracking, no analytics.
- No vendor portal credentials are stored in the tool.
- Show a clear note on the page: "Your asset file is never uploaded
  — it is parsed only in this browser. Only product name, brand and
  version are sent to public sources to look up End of Support."
- Escape all text from the asset file before putting it into the
  page, so file content cannot inject HTML or script.

### NFR-03 — Usability and performance

- Use plain, readable English, a clean layout, and a large enough
  font for audit review.
- Show results as coloured badges: Flagged red, Amber amber,
  Not published grey, Incomplete yellow, Attention orange.
- The record count varies by organisation and may exceed 5,000 to
  10,000 records. The tool must handle that without freezing the
  browser, and must show progress while fetching.
- Show a short office note on the page listing the domains the
  fetch needs, so IT can allow them if needed: `endoflife.date`,
  `eosl.ai`, plus the vendor domains in the mapping file.

### NFR-04 — Audit evidence

- Record the date and time the check finished, and show it on the
  page, so I can copy it into my workpapers.
- Each asset's EOS date must carry its source, the date accessed,
  and who verified it (FR-11), so the evidence can be traced.
- EOS data can change over time. The program must not claim a date
  is permanent. The enhanced file is a snapshot at the time I ran
  it.

### NFR-05 — Defensive validation of parser inputs

Adopted from the VAT tool (NFR-05). The parser must enforce these
limits, and reject an input that exceeds one with a clear error
naming the limit that was hit — never silently truncate, and never
freeze or crash the browser:

- Max input file: 100 MB. A file larger than 100 MB is rejected
  before parsing.
- Max zip entries: 10,000. A zip-based file (.xlsx, .xlsm) with more
  than 10,000 entries is rejected.
- Max decomposed XML: 250 MB. XML that expands beyond 250 MB after
  decompression is rejected.
- Max worksheet rows: configurable. A worksheet with more rows than
  the configured limit is rejected. Default 100,000 rows.
- Max cell length: configurable. A cell longer than the configured
  limit is rejected. Default 32,767 characters (Excel's per-cell
  maximum).

The two configurable limits must be changeable by the user, and the
current values must be visible where the limit is enforced. These
limits apply to every parser in the tool, including the flexible
parsing in FR-02.

## 5. Acceptance tests (for OpenCode to run in the trial)

OpenCode must create small made-up sample files for testing: a
sample asset file (Excel and CSV) with at least 8 assets, and a
sample vendor mapping file. Do not use a real asset list for this
test. The sample must include: one asset past End of Support, one
within 3 months of End of Support, one more than 3 months away, one
with no EOS published, one missing a mandatory field, one with no
organisation identifier and a non-unique fallback, one with
extended support = Yes, and one from a login-gated vendor.

1. Load the sample Excel file. The asset count must match a manual
   count of the sample, and the parse report must list what was
   recognised.
2. Load the sample CSV file. The same assets must appear.
3. Load a sample with multiple sheets and varying headers. The
   assets must still parse.
4. Load the sample mapping file. Each asset must be matched to the
   right source.
5. Run the check on 3 to 5 assets. The EOS date and status (Flagged
   / Amber / Not published) must match the public source for those
   assets.
6. Check the flagged, amber, and not-published counts against a
   manual count of the sample.
7. Filter to Flagged only, and to Amber only. Only matching rows
   must remain.
8. Click a column heading. The table must sort correctly, including
   chronologically for EOS date.
9. Export the enhanced file. Open it in Excel. The input columns
   must be present, plus Source URL, Date Accessed, Quoted EOS
   Date, Phase, and Verified By, in that order, with no broken rows.
10. Block or skip the fetch. The parsed table and export must still
    work, and failures must be shown per asset, not as a crash.
11. Load a file larger than 100 MB, and a spreadsheet with more rows
    than the configured limit. Each must be rejected with a clear
    error naming the limit, and the page must not freeze or crash.
12. Check the browser Network tab while loading a file. No request
    must contain the asset file. Only the EOS fetches may appear,
    and they must carry only product name, brand, and version — no
    hostnames, descriptions, dates, or usage status.
13. Click Clear. The table and summary must empty.

## 6. Assumptions and questions to confirm

Assumptions I have made while writing this, so OpenCode can start:

- My office browser is Chrome or Edge on Windows 11.
- My office may or may not allow internet access to
  `endoflife.date`, `eosl.ai`, and the vendor sites in the mapping
  file. That is why parsing and the table must work offline, and
  fetching is per asset with failures shown, not a crash.
- I do not yet have vendor portal logins in the tool, and none will
  be stored in it.
- All nine requirements, with the clarifications in the ITAES
  Requirements Log, are mandatory for version 1.

Questions for me (Solo) to answer after the first test, not blockers
for building version 1:

- What columns does the mapping Excel file have? The spec proposes
  Brand, Product or name pattern, Source URL, Source type, Last
  known EOS date, Notes — is that right?
- How does extended support = Yes affect the flag status of an
  asset past End of Support?
- Is "within 3 months before EOS" calendar months?
- What date formats appear in the input files (purchased date,
  commissioned date)?
- Is the reading right that more than 3 months before EOS, an asset
  is not flagged?
- Does the office allow internet to `endoflife.date` and `eosl.ai`?
  If not, is there an approved offline feed I can use instead?
- How large is a typical asset list — roughly how many records?
- Does "Verified By" for manually entered dates mean the auditor's
  name or initials, typed in the tool?

## 7. Deliverable

One file: `itaes-tool.html`, built in the OpenCode trial folder,
plus a short note from OpenCode saying what it built, what it tested
with the sample files, and what still needs a real asset file in the
office to check.
