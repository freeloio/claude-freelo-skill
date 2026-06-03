# Creating an invoice is not supported via API

Load when the user asks to create / issue / vystavit an invoice (faktura) — invoice creation is not exposed through Freelo's REST API.

### Create invoice — **not supported via API**

There is no `POST /issued-invoice` (or similar) endpoint. Verified live against `api.freelo.io`: the OpenAPI surface exposes only:

| Operation | Endpoint | Reference |
|---|---|---|
| List issued invoices | `GET /issued-invoices` | [list-issued-invoices.md](list-issued-invoices.md) |
| Get invoice detail | `GET /issued-invoice/{id}` | [get-invoice-detail.md](get-invoice-detail.md) |
| Mark as invoiced | `POST /issued-invoice/{id}/mark-as-invoiced` | [mark-invoice-as-invoiced.md](mark-invoice-as-invoiced.md) |
| Download reports | `GET /issued-invoice/{id}/reports` and `…/reports-json` | (raw API only) |

Invoice creation happens in the Freelo web UI: **Project → Fakturace → Vystavit fakturu**.

### Suggested response pattern

If the user asks "vytvoř fakturu pro klienta X za únor", do this:

1. **Compute the totals first** so the user has the numbers ready when they open the UI. Use [list-work-reports.md](list-work-reports.md) with `date_reported_range[date_from]` / `date_reported_range[date_to]` filters, sum `minutes` per worker, multiply by hourly rate.

2. **Tell the user explicitly** that invoice creation isn't API-driven — link them to the Freelo UI:

   > Vystavit samotnou fakturu musíš ve Freelo UI: **Projekt → Fakturace → Vystavit fakturu**. Tady jsou podklady: 142 hodin za únor 2026 na projektu [Klient ABC](https://app.freelo.io/project/{id}/tasklists?layout=kanban) (rozpis viz tabulka výše).

3. **After the user creates the invoice in UI**, they can come back and use [mark-invoice-as-invoiced.md](mark-invoice-as-invoiced.md) to close the loop (accounting flag).

Do not attempt to guess an undocumented endpoint or POST to `/issued-invoices` — both will fail and confuse the user.
