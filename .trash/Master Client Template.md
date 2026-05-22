---
color: var(--mk-color-green)
---
<%*
/* ──────────────────────────────────────────────────────────────────────────
   CLIENT + PROJECT – GENERATOR (Templater, deluxe)
   - Creates full Client Pack (folders, hub, trackers, canvas, timesheet)
   - Optional Project + pages
   - Optional Proposal draft (platform-aware scope rows, add-ons)
   - Matches structure: Clients/<Client>/...
   ------------------------------------------------------------------------- */

/* ========== Helpers ========== */
function folderSafe(name){
  return (name || "")
    .replace(/[\\/:*?"<>|]/g, "-")
    .replace(/\s+/g, " ")
    .trim();
}

async function ensureFolder(path){
  if (!await app.vault.adapter.exists(path)) {
    const parts = path.split("/");
    let acc = "";
    for (const p of parts) {
      acc = acc ? `${acc}/${p}` : p;
      if (!await app.vault.adapter.exists(acc)) {
        try { await app.vault.createFolder(acc); } catch(e) {}
      }
    }
  }
}

async function createFile(path, content){
  const exists = await app.vault.adapter.exists(path);
  if (!exists) {
    await app.vault.create(path, content);
    return path;
  } else {
    const alt = path.replace(/\.md$/, ` ${tp.date.now("HHmmss")}.md`);
    await app.vault.create(alt, content);
    return alt;
  }
}

async function includeOrFallback(partialPath, fallback){
  const has = await app.vault.adapter.exists(partialPath);
  if (has) return await tp.file.include(partialPath);
  return fallback;
}

/* ========== Prompts (Client) ========== */
const clientRaw    = await tp.system.prompt("Client / Business Name");
if (!clientRaw) { tR = "❗Client name is required."; return; }
const client       = folderSafe(clientRaw);

const shortcode    = folderSafe(await tp.system.prompt("Shortcode / Initials (e.g., OBS, PE)", ""));
const primaryName  = await tp.system.prompt("Primary Contact Name", "");
const primaryEmail = await tp.system.prompt("Primary Contact Email", "");
const primaryPhone = await tp.system.prompt("Primary Contact Phone", "");
const website      = await tp.system.prompt("Company Website (https://…)", "");
const startDate    = await tp.system.prompt("Engagement Start Date", tp.date.now("YYYY-MM-DD"));
const billing      = await tp.system.suggester(["Monthly Retainer","Fixed Project","Hourly","Mixed"], ["retainer","fixed","hourly","mixed"], false, "Billing Model");
const rate         = Number(await tp.system.prompt("Default Hourly Rate", "95")) || 95;
const currency     = await tp.system.suggester(["USD","EUR","GBP","CAD"], ["USD","EUR","GBP","CAD"], false, "Currency");
const timezone     = await tp.system.prompt("Timezone (IANA)", "America/Indiana/Indianapolis");
const ndaSigned    = await tp.system.suggester(["Yes","No"], ["Yes","No"], false, "NDA Signed?");
const repoLink     = await tp.system.prompt("Git/Repo Link (optional)", "");
const driveLink    = await tp.system.prompt("Cloud Storage Link (Drive/Dropbox/OneDrive) (optional)", "");

/* ========== Base Paths ========== */
const baseFolder   = `Olive Branch Studio/Clients/${client}`;
const yearStr      = tp.date.now("YYYY");
const ymd          = tp.date.now("YYYY-MM-DD");
const todayLong    = tp.date.now("MMMM D, YYYY");

/* ========== Create Folder Tree ========== */
const folders = [
  `${baseFolder}/00-Admin/Contracts`,
  `${baseFolder}/00-Admin/Finance/Invoices`,
  `${baseFolder}/00-Admin/Finance/Estimates`,
  `${baseFolder}/00-Admin/Finance/Timesheets`,
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
  `${baseFolder}/Proposals/${yearStr}`,
  `${baseFolder}/SOW`,
  `${baseFolder}/Archive`
];
for (const f of folders) await ensureFolder(f);

/* ========== Core Client Files ========== */
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
- [[Timesheet – ${client}]]

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

> 🔐 **Security:** Prefer a password manager. Store only what’s necessary.

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

const timesheet = `---
type: timesheet
client: "${client}"
hourly_rate: ${rate}
currency: ${currency}
---

# Timesheet – ${client}

> **Log entries** below as lines: \`YYYY-MM-DD | 1.5 | Task description\`  
> Hours × rate are summed automatically.

## Entries
- ${ymd} | 0.5 | Setup client pack

## Totals
\`\`\`dataviewjs
// Parses "Entries" bullets in THIS file: "YYYY-MM-DD | hours | desc"
const rate = dv.current().hourly_rate ?? ${rate};
const lines = dv.current().file.lists
  .where(l => l.section && l.section.subpath === "## Entries")
  .map(l => l.text);
let hours = 0;
for (const t of lines) {
  const m = String(t).match(/\\|\\s*([0-9]*\\.?[0-9]+)\\s*\\|/);
  if (m) hours += Number(m[1]);
}
const amt = hours * rate;
dv.paragraph(\`**Hours:** \${hours.toFixed(2)}h · **Rate:** \${rate} · **Total:** ${currency} \${amt.toFixed(2)}\`);
\`\`\`
`;

const brief = `---
type: brief
client: "${client}"
---

# Brief – ${client}

Use this as a general client brief, separate from project-specific briefs.

## Background
- 

## Goals & KPIs
- 

## Deliverables
- 

## Milestones
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
- [ ] Canonicals verified
- [ ] Internal links added
- [ ] Schema (where applicable)
- [ ] Page speed improvements
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

| Event | Trigger | Parameters | Destination (GA4/GTM) | Status |
|------|---------|------------|------------------------|--------|
|      |         |            |                        |        |
`;

/* Create core files */
const hubPath     = await createFile(`${baseFolder}/_Client Hub.md`, clientHub);
const overviewPath= await createFile(`${baseFolder}/_Quick Links/Overview – ${client}.md`, overview);
const contactsPath= await createFile(`${baseFolder}/_Quick Links/Contacts – ${client}.md`, contacts);
const credsPath   = await createFile(`${baseFolder}/_Quick Links/Credentials – ${client}.md`, credentials);
const commsPath   = await createFile(`${baseFolder}/_Quick Links/Comms Log – ${client}.md`, commsLog);
const changePath  = await createFile(`${baseFolder}/_Quick Links/Change Log – ${client}.md`, changeLog);
const decRiskPath = await createFile(`${baseFolder}/_Quick Links/Decisions & Risks – ${client}.md`, decisionsRisks);
const invPath     = await createFile(`${baseFolder}/00-Admin/Finance/Invoice Tracker – ${client}.md`, invoiceTracker);
const tsPath      = await createFile(`${baseFolder}/00-Admin/Finance/Timesheets/Timesheet – ${client}.md`, timesheet);
const calPath     = await createFile(`${baseFolder}/04-Marketing/Content Calendar – ${client}.md`, contentCalendar);
const briefPath   = await createFile(`${baseFolder}/_Quick Links/Brief – ${client}.md`, brief);
const indexPath   = await createFile(`${baseFolder}/_Quick Links/Project Index – ${client}.md`, projectIndex);
const wCheckPath  = await createFile(`${baseFolder}/03-Web/Launch/Website Checklist – ${client}.md`, websiteChecklist);
const seoPath     = await createFile(`${baseFolder}/04-Marketing/SEO/SEO Checklist – ${client}.md`, seoChecklist);
const socialPath  = await createFile(`${baseFolder}/04-Marketing/Social/Social Plan – ${client}.md`, socialPlan);
const analyticsPth= await createFile(`${baseFolder}/05-Analytics/Analytics Plan – ${client}.md`, analyticsPlan);

/* Optional: Canvas dashboard */
const canvas = {
  "nodes":[
    {"id":"hub","type":"file","x":-200,"y":-120,"width":300,"height":100,"file":hubPath,"color":"1"},
    {"id":"ov","type":"file","x":200,"y":-120,"width":300,"height":100,"file":overviewPath,"color":"2"},
    {"id":"con","type":"file","x":200,"y":40,"width":300,"height":100,"file":contactsPath},
    {"id":"cred","type":"file","x":200,"y":200,"width":300,"height":100,"file":credsPath},
    {"id":"idx","type":"file","x":-200,"y":40,"width":300,"height":100,"file":indexPath},
    {"id":"com","type":"file","x":-560,"y":-40,"width":300,"height":100,"file":commsPath},
    {"id":"chg","type":"file","x":-560,"y":120,"width":300,"height":100,"file":changePath},
    {"id":"risk","type":"file","x":-560,"y":280,"width":300,"height":100,"file":decRiskPath},
    {"id":"inv","type":"file","x":-200,"y":200,"width":300,"height":100,"file":invPath},
    {"id":"cal","type":"file","x":-200,"y":360,"width":300,"height":100,"file":calPath},
    {"id":"ts","type":"file","x":-560,"y":-200,"width":300,"height":100,"file":tsPath}
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
    {"id":"e9","fromNode":"hub","fromSide":"bottom","toNode":"cal","toSide":"top"},
    {"id":"e10","fromNode":"hub","fromSide":"left","toNode":"ts","toSide":"right"}
  ],
  "scale":1,"x":0,"y":0
};
const canvasJSON = JSON.stringify(canvas, null, 2);
await createFile(`${baseFolder}/_Client Hub.canvas`, canvasJSON);

/* ========== Proposal (Optional) ========== */
const makeProposal = await tp.system.suggester(["Yes","No"], [true,false], false, "Create a Proposal draft now?");
if (makeProposal) {
  const business    = client; // reuse
  const services    = await tp.system.prompt("Primary Services (short, e.g., brand & landing page)", "==INSERT SERVICES==");
  const platformKey = await tp.system.suggester(["Squarespace","WordPress","Headless CMS","None"], ["squarespace","wordpress","headless","none"], false, "Landing Page Platform");
  const includeBrand       = await tp.system.suggester(["Yes","No"], [true,false], false, "Include Brand Design scope?");
  const includeMaintenance = await tp.system.suggester(["Yes","No"], [true,false], false, "Include Monthly Maintenance add-on?");
  const chosenPkg  = await tp.system.prompt("Chosen Package (e.g., '2B. WordPress' or '1 + 2A + Maintenance')", "");

  let rows = [];
  // 1. Brand
  rows.push(includeBrand
    ? `| **1. Brand Design** | • Discovery & positioning workshop (1–2 hours)  <br>• Logo (3 concepts + revisions)  <br>• Brand palette & typography  <br>• Basic brand guidelines (PDF) | 2–3 weeks | $2,200 |`
    : `| **1. Brand Design** | — | — | — |`);

  // 2. Landing
  rows.push(`| **2. Landing Page Development** |  |  |  |`);
  if (platformKey === "squarespace") {
    rows.push(`| **2A. Squarespace** | • Template selection & customization  <br>• Responsive design  <br>• On-page SEO basics | 1–2 weeks | $1,450 |`);
    rows.push(`| **2B. WordPress** | — | — | — |`);
    rows.push(`| **2C. Headless CMS** | — | — | — |`);
  } else if (platformKey === "wordpress") {
    rows.push(`| **2A. Squarespace** | — | — | — |`);
    rows.push(`| **2B. WordPress** | • Theme setup & customization  <br>• Responsive design  <br>• Better SEO Integration | 2–3 weeks | $1,950 |`);
    rows.push(`| **2C. Headless CMS** | — | — | — |`);
  } else if (platformKey === "headless") {
    rows.push(`| **2A. Squarespace** | — | — | — |`);
    rows.push(`| **2B. WordPress** | — | — | — |`);
    rows.push(`| **2C. Headless CMS** | • Coded landing page  <br>• Integration with headless CMS  <br>• Responsive CSS/JS  <br>• Best SEO Optimization  <br>• Deployment pipeline setup | 3–4 weeks | $2,850 |`);
  } else {
    rows.push(`| **2A. Squarespace** | — | — | — |`);
    rows.push(`| **2B. WordPress** | — | — | — |`);
    rows.push(`| **2C. Headless CMS** | — | — | — |`);
  }

  // 3. Add-ons
  rows.push(`| **3. Optional Add-Ons** |  |  |  |`);
  rows.push(includeMaintenance
    ? `| **• Monthly Maintenance & Updates** | • Ongoing content updates  <br>• Plugin / Theme Updates  <br>• Ongoing SEO optimization  <br>• Website backups | Monthly retainer | $220/month |`
    : `| **• Monthly Maintenance & Updates** | — | — | — |`);

  const propName = `${ymd} - Proposal - ${business}`;
  const proposalMD = `---
type: proposal
status: draft
client: "${business}"
date: "${todayLong}"
prepared_by: "Jackson Ingenito (Olive Branch Studio LLC)"
chosen_package: "${chosenPkg}"
---

# **Prepared for:** ${business}  
**Prepared by:** Jackson Ingenito (Olive Branch Studio LLC)  
**Date:** ${todayLong}

## Introduction
Thank you for inquiring with me about this opportunity to serve you through the creation of ${services}. I look forward to hearing back from you about this proposal.

Outlined are three tiers of service, all of which are designed to suit different levels of activity and business needs. If there are any questions or concerns, please reach out as soon as possible, and we will work to resolve them together.

## Service Scopes

| **Service** | **Scope & Deliverables** | **Timeline** | **Fee** |
| --- | --- | --- | --- |
${rows.join("\n")}

## Notes & Terms
- **Payment Schedule:** 30% deposit; 30% at beginning of site development; 40% upon completion.  
- **Turnaround:** Timelines assume your timely feedback; changes in scope may adjust dates.  
- Payments are made via PayPal Invoice.

## Next Steps
Let me know which package you’d like to proceed with (“**${chosenPkg}**”), and I’ll prepare the agreement so we can secure a start date. Thank you for your time; I look forward to helping **${business}** **${services}**.

**Best,**  
Jackson Ingenito  
[jacksoncingenito@gmail.com](mailto:jacksoncingenito@gmail.com)  
765.729.1159

## Signatures
| Servicer Name (Printed): _________________<br><br>Servicer Signature: ______________________<br><br>Date: ____________ | Chosen Package: _______________________<br><br>Client Name (Printed): ___________________<br><br>Client Signature: ________________________<br><br>Date: ____________ |
| ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
`;
  await createFile(`${baseFolder}/Proposals/${yearStr}/${propName}.md`, proposalMD);
}

/* ========== Project (Optional) ========== */
const makeProject = await tp.system.suggester(["Yes","No"], [true,false], false, "Create a project now?");
let created = [];
if (makeProject) {
  const projectRaw = await tp.system.prompt("Project Name");
  if (projectRaw) {
    const project = folderSafe(projectRaw);
    const root    = baseFolder;
    const webBase = `${root}/03-Web`;
    const mktBase = `${root}/04-Marketing`;

    for (const f of [ `${webBase}/Brief`, `${mktBase}/SEO`, `${root}/06-Meetings` ]) await ensureFolder(f);

    async function makeChild(path, partial, fallbackBody){
      const fm = `---\ntype: project_doc\nclient: "${client}"\nproject: "${project}"\ndate: "${ymd}"\n---\n\n`;
      const body = await includeOrFallback(partial, fallbackBody);
      return await createFile(path, fm + body);
    }

    const incs = await tp.system.suggester(
      ["Brief","Timeline","Decision Log","Meeting (Kickoff)","Campaign Brief","SEO Checklist","Content Calendar"],
      ["brief","timeline","decisions","meeting","campaign","seo","content"],
      true,
      "Select pages to create"
    );

    const want = new Set(incs || []);

    if (want.has("brief")) {
      created.push(await makeChild(
        `${webBase}/Brief/${ymd} - Brief – ${project}.md`,
        "Templates/partials/pages/Brief – Body.md",
        `# Project Brief – ${project}

## Background
- 

## Objectives & KPIs
- 

## Deliverables
- 

## Audience & Messaging
- 

## Timeline & Milestones
| Milestone | Due | Owner | Status |
|-----------|-----|-------|--------|
| Kickoff   |     |       |        |
| Design    |     |       |        |
| Build     |     |       |        |
| QA        |     |       |        |
| Launch    |     |       |        |

## Dependencies / Risks
- `
      ));
    }

    if (want.has("timeline")) {
      created.push(await makeChild(
        `${webBase}/${ymd} - Timeline – ${project}.md`,
        "Templates/partials/pages/Timeline – Body.md",
        `# Timeline – ${project}

| Phase     | Start | End | Owner | Notes |
|-----------|-------|-----|-------|-------|
| Discovery |       |     |       |       |
| Design    |       |     |       |       |
| Build     |       |     |       |       |
| QA        |       |     |       |       |
| Launch    |       |     |       |       |`
      ));
    }

    if (want.has("decisions")) {
      created.push(await makeChild(
        `${webBase}/${ymd} - Decision Log – ${project}.md`,
        "Templates/partials/pages/Decision Log – Body.md",
        `# Decision Log – ${project}

| Date | Decision | Rationale | Impact | Owner |
|------|----------|-----------|--------|-------|
| ${ymd} |  |  |  |  |`
      ));
    }

    if (want.has("meeting")) {
      const meetingTitle = "Kickoff";
      created.push(await makeChild(
        `${root}/06-Meetings/${ymd} - ${meetingTitle}.md`,
        "Templates/partials/pages/Meeting – Body.md",
        `# 📝 Meeting – ${client} / ${project} – ${todayLong}

## Agenda
- 

## Notes
- 

## Decisions
- 

## Action Items
- [ ] Owner — Task (due: )`
      ));
    }

    if (want.has("campaign")) {
      const campaign = "Campaign";
      created.push(await makeChild(
        `${mktBase}/${ymd} - Campaign – ${campaign}.md`,
        "Templates/partials/pages/Campaign – Body.md",
        `# Campaign – ${project}

## Objective
- 

## Audience
- 

## Channels & Tactics
- 

## Creative Requirements
- 

## KPIs & Measurement
- 

## Timeline
- `
      ));
    }

    if (want.has("seo")) {
      const page = "home";
      created.push(await makeChild(
        `${mktBase}/SEO/${ymd} - SEO – ${page}.md`,
        "Templates/partials/pages/SEO – Body.md",
        `# SEO Checklist – ${project}

- [ ] Keyword selected & mapped
- [ ] Title tag set
- [ ] Meta description set
- [ ] H1/H2 structure
- [ ] Alt text for images
- [ ] Internal links added
- [ ] Canonical verified
- [ ] Schema (if applicable)
- [ ] Page speed reviewed`
      ));
    }

    if (want.has("content")) {
      created.push(await makeChild(
        `${mktBase}/Content Calendar – ${client}.md`,
        "Templates/partials/pages/Content Calendar – Body.md",
        `# Content Calendar – ${client}

| Date | Channel | Topic/Asset | Owner | Status | Link |
|------|---------|-------------|-------|--------|------|
| ${ymd} | Blog |  |  | Idea |  |`
      ));
    }

    // Lightweight project README
    const projReadme = `---
type: project
client: "${client}"
project: "${project}"
created: "${ymd}"
---

# ${project} – Project Home

Open from Client Hub: [[_Client Hub - ${client}]]

## Shortlinks
- Briefs: ${baseFolder}/03-Web/Brief
- Marketing: ${baseFolder}/04-Marketing
- Meetings: ${baseFolder}/06-Meetings
`;
    await createFile(`${webBase}/${project} – README.md`, projReadme);
  }
}

/* Move THIS generator note into the client root as a stamped README */
await tp.file.move(`${baseFolder}/_README (generated ${ymd}).md`);

/* Final output */
tR = `# ✅ Client Pack${makeProposal ? " + Proposal" : ""}${makeProject ? " + Project" : ""} created for **${client}**

Open: [[${Client} Hub]] · Canvas: [[Olive Branch Studio/Clients/${Client} Hub.canvas]]

**Base:** ${baseFolder}

- Client folders, hub, contacts, credentials, logs, trackers
- Website & SEO checklists, Content Calendar, **Timesheet**
- ${makeProposal ? "Proposal draft created in Proposals/" + yearStr + "." : "No proposal created."}
- ${makeProject ? "Project docs created (see folders)." : "No project chosen right now."}

> If any child note shows an \`include\` line, run **Templater → Replace templates in active file** to render partials.`;
%>
