<%*
/* ──────────────────────────────────────────────────────────────────────────
   CLIENT NEW PACK – MASTER TEMPLATER
   Creates a full client workspace with folders, hub notes, trackers, logs,
   and a canvas dashboard.
   ────────────────────────────────────────────────────────────────────────── */

/* Prompts */
const client      = await tp.system.prompt("Client / Business Name");
if (!client) { tR += "❗Client name is required."; return; }

const shortcode   = await tp.system.prompt("Shortcode / Initials (e.g., OBS, PE)", "");
const primaryName = await tp.system.prompt("Primary Contact Name", "");
const primaryEmail= await tp.system.prompt("Primary Contact Email", "");
const primaryPhone= await tp.system.prompt("Primary Contact Phone", "");
const website     = await tp.system.prompt("Company Website (e.g., https://...)", "");
const startDate   = await tp.system.prompt("Engagement Start Date (e.g., 2025-09-10)", tp.date.now("YYYY-MM-DD"));
const billing     = await tp.system.suggester(["Monthly Retainer","Fixed Project","Hourly","Mixed"], ["retainer","fixed","hourly","mixed"], false, "Billing Model");
const rate        = await tp.system.prompt("Default Hourly Rate (e.g., 95)", "95");
const currency    = await tp.system.suggester(["USD","EUR","GBP","CAD"], ["USD","EUR","GBP","CAD"], false, "Currency");
const timezone    = await tp.system.prompt("Timezone (IANA, e.g., America/Indiana/Indianapolis)", "America/Indiana/Indianapolis");
const ndaSigned   = await tp.system.suggester(["Yes","No"], ["Yes","No"], false, "NDA Signed?");
const repoLink    = await tp.system.prompt("Git/Repo Link (optional)", "");
const driveLink   = await tp.system.prompt("Cloud Storage Link (Google Drive/Dropbox/OneDrive) (optional)", "");

/* Paths */
const baseFolder  = `Clients/${client}`;
const folders = [
  `${baseFolder}/00-Admin/Contracts`,
  `${baseFolder}/00-Admin/Finance/Invoices`,
  `${baseFolder}/00-Admin/Finance/Estimates`,
  `${baseFolder}/00-Admin/Legal`,
  `${baseFolder}/00-Admin/Procurement`,
  `${baseFolder}/01-Discovery/Intake`,
  `${baseFolder}/01-Discovery/Research/Competitors`,
  `${baseFolder}/01-Discovery/Research/Personas`,
  `${baseFolder}/02-Brand/Logos`,
  `${baseFolder}/02-Brand/Fonts`,
  `${baseFolder}/02-Brand/Guidelines`,
  `${baseFolder}/03-Web/Brief`,
  `${baseFolder}/03-Web/Design`,
  `${baseFolder}/03-Web/Development`,
  `${baseFolder}/03-Web/Content`,
  `${baseFolder}/03-Web/QA`,
  `${baseFolder}/03-Web/Launch`,
  `${baseFolder}/04-Marketing/SEO`,
  `${baseFolder}/04-Marketing/Social`,
  `${baseFolder}/04-Marketing/Ads`,
  `${baseFolder}/04-Marketing/Email`,
  `${baseFolder}/05-Analytics/GA-GTM-GSC`,
  `${baseFolder}/05-Analytics/Reports`,
  `${baseFolder}/06-Meetings`,
  `${baseFolder}/07-Deliverables`,
  `${baseFolder}/08-Media/Raw`,
  `${baseFolder}/08-Media/Edited`,
  `${baseFolder}/Proposals/${tp.date.now("YYYY")}`,
  `${baseFolder}/SOW`,
  `${baseFolder}/Archive`
];

/* Create folders (idempotent) */
for (const f of folders) {
  try {
    const exists = await app.vault.adapter.exists(f);
    if (!exists) { await app.vault.createFolder(f); }
  } catch (e) {
    // Parent may not exist yet—attempt to create parents progressively
    const parts = f.split("/");
    let acc = "";
    for (const p of parts) {
      acc = acc ? `${acc}/${p}` : p;
      if (!await app.vault.adapter.exists(acc)) {
        try { await app.vault.createFolder(acc); } catch {}
      }
    }
  }
}

/* Helper: create a file if missing */
async function createFile(path, content) {
  const exists = await app.vault.adapter.exists(path);
  if (!exists) {
    await app.vault.create(path, content);
  } else {
    // Append unique suffix to avoid overwriting
    const alt = path.replace(/\.md$/, ` ${tp.date.now("HHmmss")}.md`);
    await app.vault.create(alt, content);
    return alt;
  }
  return path;
}

const todayLong = tp.date.now("MMMM D, YYYY");
const ymd       = tp.date.now("YYYY-MM-DD");
const hdr = (t)=> `# ${t}\n\n`;

/* Core files content */
const clientHub = `---
type: client_hub
client: "${client}"
shortcode: "${shortcode}"
timezone: "${timezone}"
start_date: "${startDate}"
billing_model: "${billing}"
hourly_rate: ${rate}
currency: ${currency}
nda_signed: ${ndaSigned === "Yes"}
website: "${website}"
repo: "${repoLink}"
drive: "${driveLink}"
---

# 🗂️ Client Hub – ${client}

**Primary Contact:** ${primaryName} · ${primaryEmail} · ${primaryPhone}  
**Website:** ${website}

## Quick Links
- [[Overview – ${client}]]
- [[Contacts – ${client}]]
- [[Credentials – ${client}]]
- [[Comms Log – ${client}]]
- [[Change Log – ${client}]]
- [[Decisions & Risks – ${client}]]
- [[Invoice Tracker – ${client}]]
- [[Content Calendar – ${client}]]

## Work Areas
- [[Brief – ${client}]] · [[Project Index – ${client}]]
- Brand: ${baseFolder}/02-Brand
- Web: ${baseFolder}/03-Web
- Marketing: ${baseFolder}/04-Marketing
- Analytics: ${baseFolder}/05-Analytics
- Meetings: ${baseFolder}/06-Meetings
- Deliverables: ${baseFolder}/07-Deliverables
- Media: ${baseFolder}/08-Media

## Recent Files (Dataview)
\`\`\`dataview
LIST FROM "${baseFolder}"
SORT file.mtime DESC
LIMIT 15
\`\`\`
`;

const overview = `---
type: overview
client: "${client}"
---

# Overview – ${client}

**Engagement start:** ${startDate}  
**Billing:** ${billing} · ${currency}  
**Default hourly rate:** ${rate}

## Objectives
- 

## Scope Summary
- 

## Constraints & Assumptions
- 

## Stakeholders
- Primary: ${primaryName} (${primaryEmail}, ${primaryPhone})
- 

## Important Links
- Repo: ${repoLink}
- Drive/Cloud: ${driveLink}
- GA/GTM/GSC: (link)
`;

const contacts = `---
type: contacts
client: "${client}"
---

# Contacts – ${client}

| Name | Role | Email | Phone | Notes |
|------|------|-------|-------|-------|
| ${primaryName} | Primary | ${primaryEmail} | ${primaryPhone} | |
`;

const credentials = `---
type: credentials
client: "${client}"
---

# Credentials – ${client}

> 🔐 **Security Reminder:** Store only what you must. Consider using a password manager (1Password/Bitwarden). If you store secrets here, encrypt the vault or keep this file outside sync.

## Web Hosting / DNS
- Registrar:
- DNS:
- Hosting:

## CMS / Website
- Platform:
- URL:
- Admin user:

## Integrations
- Email Service:
- Analytics:
- Ads:
- Tag Manager:

## Social
- Facebook:
- Instagram:
- LinkedIn:
`;

const commsLog = `---
type: comms_log
client: "${client}"
---

# Communications Log – ${client}

| Date | Channel | With | Summary | Next Action |
|------|---------|------|---------|-------------|
| ${ymd} | Email/Call | ${primaryName} | Kickoff | |
`;

const changeLog = `---
type: change_log
client: "${client}"
---

# Change Log – ${client}

| Date | Area | Change | Requested By | Impact | Status |
|------|------|--------|--------------|--------|--------|
| ${ymd} | | | | | Proposed |
`;

const decisionsRisks = `---
type: decisions_risks
client: "${client}"
---

# Decisions & Risks – ${client}

## Key Decisions
- **[Date] Decision:** Rationale → Impact

## Risks
| Risk | Likelihood | Impact | Mitigation | Owner |
|------|------------|--------|------------|-------|
|      |            |        |            |       |
`;

const invoiceTracker = `---
type: invoice_tracker
client: "${client}"
currency: ${currency}
---

# Invoice Tracker – ${client}

| # | Date | Description | Amount (${currency}) | Status | Link |
|---|------|-------------|---------------------:|--------|------|
| 1 | ${ymd} | Deposit (30%) |  | Draft |  |
`;

const contentCalendar = `---
type: content_calendar
client: "${client}"
---

# Content Calendar – ${client}

| Date | Channel | Asset | Owner | Status | Link |
|------|---------|-------|-------|--------|------|
| ${ymd} | Blog |  |  | Idea |  |
`;

const brief = `---
type: brief
client: "${client}"
---

# Project Brief – ${client}

## Background
- 

## Goals & KPIs
- 

## Audience & Personas
- 

## Competitive Landscape
- 

## Deliverables
- 

## Timeline & Milestones
- 
`;

const projectIndex = `---
type: project_index
client: "${client}"
---

# Project Index – ${client}

## All Active Notes (Dataview)
\`\`\`dataview
TABLE file.ctime as "Created", file.mtime as "Updated"
FROM "${baseFolder}"
WHERE !contains(file.folder, "Archive")
SORT file.mtime DESC
\`\`\`

## By Area
- Discovery: "${baseFolder}/01-Discovery"
- Brand: "${baseFolder}/02-Brand"
- Web: "${baseFolder}/03-Web"
- Marketing: "${baseFolder}/04-Marketing"
- Analytics: "${baseFolder}/05-Analytics"
- Admin/Finance: "${baseFolder}/00-Admin"
`;

const websiteChecklist = `---
type: checklist
client: "${client}"
area: web
---

# Website Launch Checklist – ${client}

- [ ] Domain/DNS configured
- [ ] SSL active
- [ ] Redirects (301) set
- [ ] Meta titles/descriptions
- [ ] Sitemap.xml submitted
- [ ] Robots.txt reviewed
- [ ] GA4 / GSC / GTM installed
- [ ] Forms tested (success + notifications)
- [ ] Performance pass (LCP/CLS/INP)
- [ ] Accessibility pass
`;

const seoChecklist = `---
type: checklist
client: "${client}"
area: seo
---

# SEO Checklist – ${client}

- [ ] Keyword map by page
- [ ] H1/H2 structure
- [ ] Alt text for all images
- [ ] Internal links added
- [ ] Canonicals verified
- [ ] Schema (where applicable)
- [ ] Page speed improvements logged
`;

const socialPlan = `---
type: social_plan
client: "${client}"
---

# Social Media Plan – ${client}

## Channels
- 

## Cadence
- 

## Themes/Pillars
- 

## Assets Needed
- 
`;

const analyticsPlan = `---
type: analytics_plan
client: "${client}"
---

# Analytics Tracking Plan – ${client}

| Event | Trigger | Parameters | Destination (GA4/GTM etc.) | Status |
|------|---------|------------|-----------------------------|--------|
|      |         |            |                             |        |
`;

/* Create core files */
const hubPath     = await createFile(`${baseFolder}/_Client Hub.md`, clientHub);
const overviewPath= await createFile(`${baseFolder}/Overview – ${client}.md`, overview);
const contactsPath= await createFile(`${baseFolder}/Contacts – ${client}.md`, contacts);
const credsPath   = await createFile(`${baseFolder}/Credentials – ${client}.md`, credentials);
const commsPath   = await createFile(`${baseFolder}/Comms Log – ${client}.md`, commsLog);
const changePath  = await createFile(`${baseFolder}/Change Log – ${client}.md`, changeLog);
const decRiskPath = await createFile(`${baseFolder}/Decisions & Risks – ${client}.md`, decisionsRisks);
const invPath     = await createFile(`${baseFolder}/00-Admin/Finance/Invoice Tracker – ${client}.md`, invoiceTracker);
const calPath     = await createFile(`${baseFolder}/04-Marketing/Content Calendar – ${client}.md`, contentCalendar);
const briefPath   = await createFile(`${baseFolder}/03-Web/Brief/Brief – ${client}.md`, brief);
const indexPath   = await createFile(`${baseFolder}/Project Index – ${client}.md`, projectIndex);
const wCheckPath  = await createFile(`${baseFolder}/03-Web/Launch/Website Checklist – ${client}.md`, websiteChecklist);
const seoPath     = await createFile(`${baseFolder}/04-Marketing/SEO/SEO Checklist – ${client}.md`, seoChecklist);
const socialPath  = await createFile(`${baseFolder}/04-Marketing/Social/Social Plan – ${client}.md`, socialPlan);
const analyticsPth= await createFile(`${baseFolder}/05-Analytics/Analytics Plan – ${client}.md`, analyticsPlan);

/* Optional: Client Canvas Hub */
const canvas = {
  "nodes":[
    {"id":"hub","type":"file","x":-200,"y":-120,"width":300,"height":100,"file":app.metadataCache.getFirstLinkpathDest(`_Client Hub`, baseFolder)?.path || `${baseFolder}/_Client Hub.md`,"color":"1"},
    {"id":"ov","type":"file","x":200,"y":-120,"width":300,"height":100,"file":overviewPath,"color":"2"},
    {"id":"con","type":"file","x":200,"y":40,"width":300,"height":100,"file":contactsPath},
    {"id":"cred","type":"file","x":200,"y":200,"width":300,"height":100,"file":credsPath},
    {"id":"idx","type":"file","x":-200,"y":40,"width":300,"height":100,"file":indexPath},
    {"id":"com","type":"file","x":-560,"y":-40,"width":300,"height":100,"file":commsPath},
    {"id":"chg","type":"file","x":-560,"y":120,"width":300,"height":100,"file":changePath},
    {"id":"risk","type":"file","x":-560,"y":280,"width":300,"height":100,"file":decRiskPath},
    {"id":"inv","type":"file","x":-200,"y":200,"width":300,"height":100,"file":invPath},
    {"id":"cal","type":"file","x":-200,"y":360,"width":300,"height":100,"file":calPath}
  ],
  "edges":[
    {"id":"e1","fromNode":"hub","fromSide":"right","toNode":"ov","toSide":"left"},
    {"id":"e2","fromNode":"hub","fromSide":"right","toNode":"con","toSide":"left"},
    {"id":"e3","fromNode":"hub","fromSide":"right","toNode":"cred","toSide":"left"},
    {"id":"e4","fromNode":"hub","fromSide":"left","toNode":"idx","toSide":"right"},
    {"id":"e5","fromNode":"idx","fromSide":"left","toNode":"com","toSide":"right"},
    {"id":"e6","fromNode":"idx","fromSide":"left","toNode":"chg","toSide":"right"},
    {"id":"e7","fromNode":"idx","fromSide":"left","toNode":"risk","toSide":"right"},
    {"id":"e8","fromNode":"hub","fromSide":"bottom","toNode":"inv","toSide":"top"},
    {"id":"e9","fromNode":"hub","fromSide":"bottom","toNode":"cal","toSide":"top"}
  ],
  "scale":1,"x":0,"y":0
};
const canvasJSON = JSON.stringify(canvas, null, 2);
const canvasPath = await createFile(`${baseFolder}/Client Hub.canvas`, canvasJSON);

/* Move this template note into the client root as a readme */
await tp.file.move(`${baseFolder}/_README (generated ${ymd}).md`);

/* Final output in the created readme */
tR = `# ✅ Client pack created for **${client}**

Open: [[_Client Hub]]  ·  Canvas: [[Client Hub.canvas]]

**Base:** ${baseFolder}

- Overview, contacts, credentials, logs & trackers
- Full folder tree for Admin, Discovery, Brand, Web, Marketing, Analytics, Meetings, Deliverables, Media, Proposals, SOW, Archive
- Project Index with Dataview
- Website & SEO checklists
- Content Calendar & Analytics Plan

Happy building! 🚀`;
%>
