# Invoice CSV Export

**Status:** ready
**Author:** tzarypau
**Date:** 2026-05-21

## 1. Problem

Finance managers and accountants use the invoices page daily to monitor outstanding and paid invoices. At the end of each reporting period they need to analyse invoice data in external tools such as Excel — reconciling totals, building pivot tables, or feeding data into accounting systems. The dashboard provides no way to extract this data, forcing users to transcribe values manually or abandon the analysis entirely.

Today, there is no mechanism to export invoice data from the dashboard; users either skip the analysis or re-enter data by hand.

## 2. Hypothesis

If we add a CSV export button to the invoices page that exports the currently filtered result set, then finance managers and accountants will be able to complete their monthly reporting workflow entirely within the dashboard, because the only missing step between viewing invoices and analysing them externally is a way to download the data.

## 3. Target users

Finance managers and accountants who prepare monthly reports. They use the invoices page daily for operational monitoring and need to export data once per month at period close to run reconciliations or reports in Excel or similar tools.

## 4. Success metrics

- Primary: ≥ 80% of authenticated users who visit `/dashboard/invoices` at least once per month trigger at least one CSV export within the first two months after release
- Secondary: None — single metric is sufficient

## 5. Solution overview

A download button labelled "Export CSV" is added to the invoices page toolbar, alongside the existing "Create Invoice" button. When clicked, the browser downloads a `.csv` file containing all invoices that match the current search filter — the same result set the user sees in the table, but without pagination.

The export honours the `query` search parameter present in the URL. If the user has typed a search term, only matching invoices are included in the export. If no filter is active, all invoices are exported. The exported columns are: invoice ID, customer name, customer email, date, amount (formatted as a USD currency string), and status.

The download is triggered by navigating to a Route Handler at `/api/invoices/export?query=<term>`. The server fetches the full unpaginated result set, serialises it to CSV, and responds with a `Content-Disposition: attachment; filename="invoices.csv"` header so the browser saves the file rather than displaying it. The route requires an active session; unauthenticated requests receive a 401 response.

On an empty result set (no invoices match the filter), the server still returns a valid CSV file containing only the header row, rather than an error.

## 6. Technical context

### Reuse
- `fetchFilteredInvoices` in `app/lib/data.ts` — query structure is reused; a new unpaginated variant `fetchAllFilteredInvoices` is derived from it by removing `LIMIT`/`OFFSET`
- `InvoicesTable` type in `app/lib/definitions.ts` — covers all required CSV columns (`id`, `name`, `email`, `date`, `amount`, `status`)
- `formatCurrency` in `app/lib/utils.ts` — formats the `amount` field (stored as a cents integer) for the CSV
- `formatDateToLocal` in `app/lib/utils.ts` — formats the `date` field consistently with the table display
- Button styling conventions from `app/ui/invoices/buttons.tsx` — `ExportInvoices` follows the same Tailwind classes and heroicons pattern

### Add
- `fetchAllFilteredInvoices(query: string)` in `app/lib/data.ts` — same SQL as `fetchFilteredInvoices` without `LIMIT`/`OFFSET`
- `app/api/invoices/export/route.ts` — Route Handler returning `text/csv` with `Content-Disposition: attachment` header; includes explicit `auth()` check
- `ExportInvoices` component in `app/ui/invoices/buttons.tsx` — link button pointing to `/api/invoices/export?query=<current query>`
- Integration of `ExportInvoices` in `app/dashboard/invoices/page.tsx` toolbar

### Constraints
- `/api/*` routes are not covered by the `auth.config.ts` middleware callback that protects `/dashboard/*`; the export route must call `auth()` directly and return 401 if no session is found
- No CSV library is available in `package.json`; serialisation must be pure string formatting with proper comma escaping for fields that may contain commas (customer names)
- `amount` is stored as an integer (cents); division by 100 and currency formatting must happen at serialisation time, not in SQL

## 7. Scope & constraints

### Out of scope
- Export to formats other than CSV (PDF, XLSX, ODS, etc.)
- Scheduled or automated exports (e-mailed reports, webhooks)
- Export of other entities (customers, revenue data)
- Column selection or custom field ordering in the export

### Business constraints
- None known

## 8. Open questions
- [x] What should the exported `amount` column header and format be — raw cents, a decimal number, or a formatted currency string like `$12.50`? → **decimal** (e.g. `12.50`)
- [x] Should the filename include the current date or filter term (e.g. `invoices-2026-05-21.csv`) for traceability? → **yes**, format `invoices-YYYY-MM-DD.csv` using the export date

## 9. Acceptance criteria

**Behavioral**
- [ ] Given a user is authenticated and on `/dashboard/invoices` with no active search filter, when they click "Export CSV", then all invoices are downloaded as a `.csv` file.
- [ ] Given a user is authenticated and has typed a search term, when they click "Export CSV", then only invoices matching that filter are included in the downloaded file.
- [ ] Given an unauthenticated request to `/api/invoices/export`, then the server responds with HTTP 401.
- [ ] Given a search filter that matches no invoices, when the user clicks "Export CSV", then a valid CSV file with only the header row is downloaded (no error is shown).

**Static**
- [ ] CSV header row: `id,customer,email,date,amount,status`
- [ ] `amount` column contains a decimal value (e.g. `12.50`), not raw cents and not a currency string
- [ ] Fields containing commas are wrapped in double quotes
- [ ] Response `Content-Type` header is `text/csv`
- [ ] Response `Content-Disposition` header is `attachment; filename="invoices-YYYY-MM-DD.csv"` where the date is the server-side export date
