# IT Asset EOS Scanner (ITAES)

A single-file HTML tool that helps an IT auditor tell whether an IT
asset has reached End of Life / End of Support. It runs in a browser
inside the office — no server, no install, no upload.

## How to use it

1. Double-click `itaes-tool.html` to open it in Chrome or Edge.
2. Load an asset list (Excel `.xlsx`/`.xls`, plain text, or CSV).
3. Load the vendor-to-source mapping file (Excel or CSV).
4. Run the check. The tool fetches End of Support dates live from
   public sources and shows a summary and a findings table.
5. Export the enhanced Excel file for workpapers.

Your asset file is never uploaded — it is parsed only in this
browser. Only product name, brand, and version are sent to public
sources to look up End of Support.

## Files

- `itaes-tool.html` — the tool (single file, inline CSS/JS).
- `ITAES-User-Requirements.md` — the user requirements the tool
  was built from.
- `sample-assets.csv` — 10 fictional assets for testing.
- `sample-mapping.csv` — sample vendor-to-source mapping.
- `sample-assets.xlsx` — the same 10 assets in Excel, two sheets
  with varying headers. Not in this repo (binary upload); it is
  kept with the build locally — ask Archie for it, or generate it
  from the CSV.

## Notes

- A finding is flagged on End of Support, not End of Life. Flagged
  (red) when EOS is exceeded; amber within 3 months before EOS;
  "Not published" when no EOS is published.
- Live sources: endoflife.date (software) and EOSL.ai (hardware)
  for triage, then the vendor's own page from the mapping file.
  Login-gated vendors fall back to the mapping-file date, then a
  manual check.
- Open questions: how extended support = Yes affects the flag, and
  the exact columns of the real mapping file. See the requirements
  doc, section 6.
