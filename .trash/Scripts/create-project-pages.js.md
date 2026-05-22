module.exports = async (params, settings) => {
  const { app, quickAddApi } = params;
  // Inputs: client, project
  let client  = await quickAddApi.inputPrompt("Client / Business Name");
  if (!client) return;
  let project = await quickAddApi.inputPrompt("Project Name");
  if (!project) return;

  const folderSafe = (s)=> (s||"").replace(/[\\/:*?"<>|]/g,"-").replace(/\s+/g," ").trim();
  client  = folderSafe(client);
  project = folderSafe(project);

  const root    = `Clients/${client}`;
  const webBase = `${root}/03-Web`;
  const mktBase = `${root}/04-Marketing`;
  const ymd = window.moment().format("YYYY-MM-DD");

  async function ensureFolder(path){
    const exists = await app.vault.adapter.exists(path);
    if (!exists) {
      const parts = path.split("/");
      let acc = "";
      for (const p of parts) {
        acc = acc ? `${acc}/${p}` : p;
        if (!await app.vault.adapter.exists(acc)) {
          try { await app.vault.createFolder(acc); } catch {}
        }
      }
    }
  }
  await ensureFolder(`${webBase}/Brief`);
  await ensureFolder(`${mktBase}/SEO`);
  await ensureFolder(`${root}/06-Meetings`);

  const choices = [
    "Brief","Timeline","Decision Log","Meeting (Kickoff)","Campaign Brief","SEO Checklist","Content Calendar"
  ];
  const picked = await quickAddApi.checkboxPrompt("Select pages", choices);
  if (!picked) return;

  async function createFile(path, content){
    const exists = await app.vault.adapter.exists(path);
    if (!exists) await app.vault.create(path, content);
    else await app.vault.create(path.replace(/\.md$/, ` ${Date.now()}.md`), content);
  }
  const fm = (type)=> `---\ntype: project_doc\nclient: "${client}"\nproject: "${project}"\ndate: "${ymd}"\n---\n\n`;

  for (const c of picked) {
    if (c === "Brief") {
      await createFile(`${webBase}/Brief/${ymd} - Brief – ${project}.md`, fm("brief") + "# Project Brief\n");
    }
    if (c === "Timeline") {
      await createFile(`${webBase}/${ymd} - Timeline – ${project}.md`, fm("timeline") + "# Timeline\n");
    }
    if (c === "Decision Log") {
      await createFile(`${webBase}/${ymd} - Decision Log – ${project}.md`, fm("decisions") + "# Decision Log\n");
    }
    if (c === "Meeting (Kickoff)") {
      await createFile(`${root}/06-Meetings/${ymd} - Kickoff.md`, fm("meeting") + "# Meeting – Kickoff\n");
    }
    if (c === "Campaign Brief") {
      await createFile(`${mktBase}/${ymd} - Campaign – Campaign.md`, fm("campaign") + "# Campaign\n");
    }
    if (c === "SEO Checklist") {
      await createFile(`${mktBase}/SEO/${ymd} - SEO – home.md`, fm("seo") + "# SEO Checklist\n");
    }
    if (c === "Content Calendar") {
      await createFile(`${mktBase}/Content Calendar – ${client}.md`, fm("content") + "# Content Calendar\n");
    }
  }
};
