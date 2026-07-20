# United Services CRM — Project Overview & History

## What this is

"United Services CRM" is a custom, single-page web app used as a Microsoft Teams tab for tracking companies, contacts, documents, and events (EPCs, developers, contractors, etc. in the solar/renewables space). It has no separate backend or database — it reads and writes directly to Microsoft SharePoint via the Microsoft Graph API, using the signed-in user's own Microsoft 365 permissions.

- **Repo:** `natbej2007-create/united-crm` (public, hosted via GitHub Pages)
- **Files:** `index.html` (the app itself), `auth-start.html` (Teams sign-in helper popup)
- **Built by:** Natalie Bejarano
- **Now managed by:** German Torres (added as a GitHub collaborator)

## Architecture

**Frontend:** A single static HTML file with inline CSS and JavaScript — no build step, no framework. It runs either as a Teams tab or as a standalone page opened in a browser.

**Authentication:** Microsoft Authentication Library (MSAL) via Azure AD, requesting the `Sites.ReadWrite.All` delegated Graph scope.
- Inside Teams: the app opens `auth-start.html` in a Teams auth popup, which signs the user in and hands an access token back to the main app.
- Outside Teams: the app falls back to MSAL's normal redirect login flow.
- Azure AD app registration: client ID `08b5f379-bc54-423b-b9fd-72242cfe908b`, tenant `720752f2-761f-405d-866a-efdb633125d9`.

**Data storage:** Everything lives in SharePoint at `unitedservicesenergy.sharepoint.com/sites/UnitedServicesTeamHub`, in the form of SharePoint Lists and a document library:

| SharePoint object | Type | Holds |
|---|---|---|
| Companies | List (126 items) | Company records — name, relationship type, stage, tier, fit, industry, website, LinkedIn, location, owner, notes, custom fields |
| Contacts | List (91 items) | People at each company — name, role, email, phone, LinkedIn, last contact date, notes, **time zone (new)** |
| CRM Documents | Document library | Uploaded files (NDAs, MSAs, insurance/COI, etc.), tagged with doc type, status, date, and linked company |
| CRM Change Log | List (11 items) | Appears to track changes made to CRM records (not yet wired into the app's own change flow) |

On load, the app authenticates, resolves the SharePoint site, then pulls all Companies and Contacts via paginated Graph API calls and joins them in memory by company name. Saving a record (new or edited) writes straight back to the corresponding SharePoint list item over Graph.

**UI layout:** A left-hand company list (searchable, filterable by relationship type, tier, fit, and stage) and a right-hand detail panel with five tabs per company: Overview, Contacts, Documents, Events, and More (custom fields).

## Known limitations

- The **Events** tab and **custom fields** (More tab) are currently local-only — they live in the browser's in-memory state for the session but are not written back to SharePoint, so they don't persist across reloads. Worth flagging to Natalie/German if that data needs to be permanent.
- No offline mode: if Graph API calls fail (auth expired, network issue, missing SharePoint permissions), the relevant section simply won't load or save.

## Change history (recent)

- **Owner field becomes a dropdown** — a prior update (contributors: German Torres, Natalie Bejarano, Geronimo LeBaron) changed the company Owner field from free text to a fixed dropdown.
- **Time Zone field added to Contacts** (this session) — added a Time Zone dropdown (Eastern, Central, Mountain, Pacific, Alaska, Hawaii-Aleutian) to the Add/Edit Contact form and a matching column in the Contacts table. Required adding a new `TimeZone` column to the SharePoint "Contacts" list (done via the SharePoint UI) so the field can save and load correctly. Deployed by committing the updated `index.html` to the `main` branch, which GitHub Pages auto-redeploys.

## How to make a change going forward

1. Describe the change.
2. Edit `index.html` (data model, form fields, table columns, and the Graph `fields` mapping all need to stay in sync).
3. If the change needs a new piece of data, add a matching column to the relevant SharePoint list first — otherwise Graph will reject the save.
4. Upload the updated file to the GitHub repo (replacing the existing one) and commit to `main`.
5. GitHub Pages redeploys automatically; reload the Teams tab to see the change.
