<%*
/* ──────────────────────────────────────────────────────────────────────────
   PROMPTS
   ────────────────────────────────────────────────────────────────────────── */
const business = tp.file.title || await tp.system.prompt("Business Name");
const services   = await tp.system.prompt("Primary Services (short, e.g., 'brand & landing page')", "==INSERT SERVICES==");
const contact    = await tp.system.prompt("Client Contact Name", "");
const email      = await tp.system.prompt("Client Email", "");
const phone      = await tp.system.prompt("Client Phone", "");

const platformKey = await tp.system.suggester(
  ["Squarespace", "WordPress", "Headless CMS", "None"],
  ["squarespace", "wordpress", "headless", "none"],
  false,
  "Landing Page Platform"
);

const includeBrand       = await tp.system.suggester(["Yes","No"], [true,false], false, "Include Brand Design scope?");
const includeMaintenance = await tp.system.suggester(["Yes","No"], [true,false], false, "Include Monthly Maintenance add-on?");

const chosenPkg  = await tp.system.prompt("Chosen Package (e.g., '2B. WordPress' or '1 + 2A + Maintenance')", "");
const todayISO   = tp.date.now("YYYY-MM-DD");
const todayLong  = tp.date.now("MMMM D, YYYY");
const yearStr    = tp.date.now("YYYY");

/* ──────────────────────────────────────────────────────────────────────────
   OUTPUT FOLDERS & FILE NAMES
   ────────────────────────────────────────────────────────────────────────── */
const baseFolder = `Clients/${business}`;
const propFolder = `${baseFolder}/Proposals/${yearStr}`;
const propName   = `${todayISO} - Proposal - ${business}`;

/* Ensure folders exist and move this file */
await tp.file.move(`${propFolder}/${propName}.md`);

/* ──────────────────────────────────────────────────────────────────────────
   SCOPE ROW HELPERS
   (If you plan to use partial includes instead, see section 3)
   ────────────────────────────────────────────────────────────────────────── */
let rows = [];

// 1. Brand Design (optional)
if (includeBrand) {
  rows.push(`| **1. Brand Design** | • Discovery & positioning workshop (1–2 hours)  <br>• Logo (3 concepts + revisions)  <br>• Brand palette & typography  <br>• Basic brand guidelines (PDF) | 2–3 weeks | $2,200 |`);
} else {
  rows.push(`| **1. Brand Design** | — | — | — |`);
}

// 2. Landing Page Development header (always present)
rows.push(`| **2. Landing Page Development** |  |  |  |`);

// 2A/B/C platform specifics
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
  // None selected
  rows.push(`| **2A. Squarespace** | — | — | — |`);
  rows.push(`| **2B. WordPress** | — | — | — |`);
  rows.push(`| **2C. Headless CMS** | — | — | — |`);
}

// 3. Optional Add-Ons header
rows.push(`| **3. Optional Add-Ons** |  |  |  |`);

// Maintenance row (optional)
if (includeMaintenance) {
  rows.push(`| **• Monthly Maintenance & Updates** | • Ongoing content updates  <br>• Plugin / Theme Updates  <br>• Ongoing SEO optimization  <br>• Website backups | Monthly retainer | $220/month |`);
} else {
  rows.push(`| **• Monthly Maintenance & Updates** | — | — | — |`);
}

/* ──────────────────────────────────────────────────────────────────────────
   COMPANION DOCS CONTENT (auto-create files & link them)
   ────────────────────────────────────────────────────────────────────────── */
const notesTermsContent = `## Notes & Terms

- **Payment Schedule:** 30% deposit; 30% at beginning of site development; 40% upon completion.
- **Turnaround:** Timelines assume your timely feedback; changes in scope may adjust dates.
- Payments are made via PayPal Invoice.

**Prepared for:** ${business}  
**Date:** ${todayLong}
`;

const agreementContent = `---
type: agreement
status: draft
client: "${business}"
date: "${todayLong}"
---

# Service Agreement – ${business}

This Agreement ("Agreement") is between **Olive Branch Studio LLC** ("Provider") and **${business}** ("Client").

## 1. Scope
See the Proposal and Statement of Work (SOW) linked below.

## 2. Payment Terms
- 30% deposit; 30% at start of development; 40% at completion.
- Late payments may incur a fee after 10 days.

## 3. Timeline & Dependencies
Client will provide timely feedback and assets. Delays may shift target dates.

## 4. Ownership & IP
Upon full payment, final deliverables transfer to Client. Provider may showcase work in portfolio.

## 5. Change Requests
Out-of-scope requests will be quoted and approved before work continues.

**Signatures**  
Provider: ____________________  Date: __________  
Client (${business}): ____________________  Date: __________
`;

const sowContent = `---
type: sow
status: draft
client: "${business}"
date: "${todayLong}"
---

# Statement of Work (SOW) – ${business}

## Overview
Project to deliver: **${services}**.

## Deliverables
- See detailed scope table in the Proposal.
- Any extras will be documented as Change Orders.

## Milestones
- Kickoff  
- Mockups/Design review  
- Development/Build  
- QA + Revisions  
- Launch

## Acceptance Criteria
- Deliverables meet described scope and pass QA.
- Client sign-off collected on milestone reviews.
`;

const invoiceContent = `---
type: invoice
status: draft
client: "${business}"
date: "${todayLong}"
prepared_by: "Jackson Ingenito (Olive Branch Studio LLC)"
---

# Invoice (Draft) – ${business}

**Bill To:**  
${business}  
${contact}  
${email} | ${phone}

**Items (example):**
| Item | Description | Qty | Rate | Amount |
|------|-------------|----:|-----:|-------:|
| Deposit | 30% project deposit | 1 | — | — |
| Milestone 2 | 30% at start of development | 1 | — | — |
| Final | 40% on completion | 1 | — | — |

> Adjust actual rates/amounts before sending.
`;

/* Create companion files if they don't exist already */
const createdNotesTerms = await tp.file.create_new(notesTermsContent, propFolder, `Notes & Terms - ${business}.md`);
const createdAgreement  = await tp.file.create_new(agreementContent,  propFolder, `Agreement (Draft) - ${business}.md`);
const createdSOW        = await tp.file.create_new(sowContent,        propFolder, `SOW (Draft) - ${business}.md`);
const createdInvoice    = await tp.file.create_new(invoiceContent,    propFolder, `Invoice (Draft) - ${business}.md`);

/* ──────────────────────────────────────────────────────────────────────────
   LINKS to companion docs
   ────────────────────────────────────────────────────────────────────────── */
const linkNotes = `[[${createdNotesTerms.basename}]]`;
const linkAgr   = `[[${createdAgreement.basename}]]`;
const linkSOW   = `[[${createdSOW.basename}]]`;
const linkInv   = `[[${createdInvoice.basename}]]`;
%>
---
type: proposal
status: draft
client: "<%= business %>"
date: "<%= todayLong %>"
prepared_by: "Jackson Ingenito (Olive Branch Studio LLC)"
contact:
  name: "<%= contact %>"
  email: "<%= email %>"
  phone: "<%= phone %>"
chosen_package: "<%= chosenPkg %>"
---

# **Prepared for:** <%= business %>  
**Prepared by:** Jackson Ingenito (Olive Branch Studio LLC)  
**Date:** <%= todayLong %>

---

## Introduction

Thank you for inquiring with me about this opportunity to serve you through the creation of <%= services %>.  
I look forward to hearing back from you about this proposal.

Outlined are three tiers of service, all of which are designed to suit different levels of activity and business needs.  
If there are any questions or concerns, please reach out as soon as possible, and we will work to resolve them together.

---

## Service Scopes

| **Service** | **Scope & Deliverables** | **Timeline** | **Fee** |
| --- | --- | --- | --- |
<%* tR += rows.join("\n"); %>

---

## Notes & Terms
See <%= linkNotes %>.

- **Payment Schedule:** 30% deposit; 30% at beginning of site development; 40% upon completion.  
- **Turnaround:** Timelines assume your timely feedback; changes in scope may adjust dates.  
- Payments are made via PayPal Invoice.

---

## Next Steps

Let me know which package you’d like to proceed with (“**<%= chosenPkg %>**”), and I’ll prepare the agreement so we can secure a start date.  
Thank you for your time; I look forward to helping **<%= business %>** **<%= services %>**.

**Best,**

Jackson Ingenito  
[jacksoncingenito@gmail.com](mailto:jacksoncingenito@gmail.com)  
765.729.1159

---

## Signatures

| Servicer Name (Printed): _________________<br><br>Servicer Signature: ______________________<br><br>Date: ____________ | Chosen Package: _______________________<br><br>Client Name (Printed): ___________________<br><br>Client Signature: ________________________<br><br>Date: ____________ |
| ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

---

### Linked Docs
- Agreement (Draft): <%= linkAgr %>  
- Statement of Work (Draft): <%= linkSOW %>  
- Invoice (Draft): <%= linkInv %>
