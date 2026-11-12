# Comparing a legacy screen with the new one

Go through the items that apply. For each one, record the legacy evidence (`file:line`) and the new evidence, then the verdict (Same / Intended / Defect / Unverified). If the screen was re-planned, compare data meaning only and treat layout differences as intended.

## Search

- Which fields exist, and their default values (including the default "search by" option and default date range)
- Radio vs checkbox, and whether an "All" option exists
- Input normalization: pasted values with hyphens or spaces (e.g. business numbers), trimming, case
- What an empty search returns: everything, nothing, or a validation message
- Whether the date range is required when another filter is set

## List

- Columns, their order, and their labels
- Formatting: numbers, currency, dates and times, ID formats with hyphens
- Code-to-label mapping, including codes the legacy page shows as blank
- How empty or `NULL` values appear
- Sort order (default and per column) and the order of history rows (e.g. oldest first)
- Page size, page numbering, and total count
- Row click destination, links, and popups (what opens, with which key)

## Detail

- Fields shown and where each value comes from
- Values that are derived on the page (concatenations, conditional labels)
- Child lists and their order

## Downloads

- Columns and their order, compared with the legacy file
- Whether the download uses the current search conditions or exports everything
- Value formatting in the file, and masking rules for files

## Save and actions

- Which fields are written, and whether unchanged values are written back (overwriting concurrent changes)
- Validation rules and their messages
- What happens after saving: stay, reload, or go back to the list with the search kept
- Side effects in legacy (history rows, notifications, other tables)

## Access and privacy

- Masking of personal data in lists and details. The new system may follow a newer policy on purpose; check the current policy before calling a difference a defect.
- Which roles can see or change each item

## Known intended differences

Keep a list of differences the user has already accepted, and do not report them again as defects.
