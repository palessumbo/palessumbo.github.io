---
title: Publications
permalink: /publications/
layout: single
---

<div id="pubs-root">
  <p>Loading publications from HAL…</p>
</div>

<noscript>
  <p><strong>Enable JavaScript</strong> to load your publications automatically from HAL.</p>
</noscript>

<script>
(function() {
  const IDHAL = "alessandro-palumbo"; // your HAL id
  const ROOT = document.getElementById("pubs-root");

  // Escape helper
  const escapeHTML = (s) => (s||"")
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");

  // Venue
  function venueOf(d) {
    return d.journalTitle_s || d.bookTitle_s || d.conferenceTitle_s || d.serieTitle_s || "";
  }

  // Links
  function halUrl(d) {
    if (d.halId_s) return `https://hal.science/${encodeURIComponent(d.halId_s)}`;
    if (d.docid)   return `https://hal.science/hal-${encodeURIComponent(d.docid)}`;
    return null;
  }
  function pdfUrl(d) {
    if (d.fileMain_s) return d.fileMain_s.startsWith("http") ? d.fileMain_s : `https://hal.science/${d.fileMain_s}`;
    return null;
  }
  function doiUrl(d) {
    return d.doiId_s ? `https://doi.org/${encodeURIComponent(d.doiId_s)}` : null;
  }
  function bibtexUrl(d) {
    if (d.halId_s) return `https://hal.science/${encodeURIComponent(d.halId_s)}/bibtex`;
    if (d.docid)   return `https://hal.science/hal-${encodeURIComponent(d.docid)}/bibtex`;
    return null;
  }

  // Badge
  function typeBadge(t) {
    if (!t) return "";
    const map = { ART:"Journal", COMM:"Conference", POSTER:"Poster", THESE:"Thesis", HDR:"HDR",
                  OUV:"Book", COUV:"Chapter", REPORT:"Report", PATENT:"Patent", OTHER:"Other" };
    const label = map[t] || t;
    return `<span class="px-2 py-0.5 text-xs rounded bg-gray-200">${escapeHTML(label)}</span>`;
  }

  function itemHTML(d) {
    const title = escapeHTML(d.label_s || "(untitled)");
    const where = escapeHTML(venueOf(d));
    const year  = d.producedDateY_i || "";
    const links = [];
    const uHAL = halUrl(d); if (uHAL) links.push(`<a href="${uHAL}" target="_blank" rel="noopener">HAL</a>`);
    const uPDF = pdfUrl(d); if (uPDF) links.push(`<a href="${uPDF}" target="_blank" rel="noopener">PDF</a>`);
    const uDOI = doiUrl(d); if (uDOI) links.push(`<a href="${uDOI}" target="_blank" rel="noopener">DOI</a>`);
    const uBIB = bibtexUrl(d); if (uBIB) links.push(`<a href="${uBIB}" target="_blank" rel="noopener">BibTeX</a>`);

    const meta = [typeBadge(d.docType_s), year && `<span>${year}</span>`, where && `<em>${where}</em>`]
      .filter(Boolean).join(" · ");

    return `
      <li class="pub-item">
        <div class="pub-title">${title}</div>
        ${meta ? `<div class="pub-meta">${meta}</div>` : ""}
        ${links.length ? `<div class="pub-links">${links.join(" · ")}</div>` : ""}
      </li>`;
  }

  function render(docs) {
    const byYear = new Map();
    for (const d of docs) {
      const y = d.producedDateY_i || "No year";
      if (!byYear.has(y)) byYear.set(y, []);
      byYear.get(y).push(d);
    }
    const years = Array.from(byYear.keys()).sort((a,b) => String(b).localeCompare(String(a)));
    ROOT.innerHTML = years.map(y => `
      <section class="pub-year">
        <h2>${escapeHTML(String(y))}</h2>
        <ul class="pub-list">
          ${byYear.get(y).map(itemHTML).join("")}
        </ul>
      </section>
    `).join("");
  }

  async function fetchHAL() {
    // NOTE: use /search/index/ and quote the idHAL to be safe with hyphens
    const base = "https://api.archives-ouvertes.fr/search/index/";
    const fields = [
      "docid","halId_s","label_s","authFullName_s","producedDateY_i","docType_s",
      "journalTitle_s","bookTitle_s","conferenceTitle_s","serieTitle_s",
      "publicationDate_s","doiId_s","fileMain_s","linkExtUrl_s"
    ];
    const params = new URLSearchParams({
      q: `authIdHal_s:"${IDHAL}"`,
      fl: fields.join(","),
      wt: "json",
      rows: "500",
      sort: "producedDate_tdate desc"
    });
    const url = `${base}?${params.toString()}`;

    try {
      const res = await fetch(url, { headers: { "Accept": "application/json" } });
      if (!res.ok) throw new Error(`HAL error ${res.status}`);
      const data = await res.json();
      const docs = (data.response && data.response.docs) || [];
      if (!docs.length) {
        ROOT.innerHTML = `<p>No publications found on HAL for <code>${escapeHTML(IDHAL)}</code>.</p>`;
        return;
      }
      render(docs);
    } catch (err) {
      console.error(err);
      const profile = `https://hal.science/search/index/?q=authIdHal_s%3A${encodeURIComponent('"' + IDHAL + '"')}`;
      ROOT.innerHTML = `
        <p>Could not load publications from HAL right now.</p>
        <p>Test the API call directly here:<br>
           <a href="${url}" target="_blank" rel="noopener">${escapeHTML(url)}</a></p>
        <p>Your HAL profile search:<br>
           <a href="${profile}" target="_blank" rel="noopener">${escapeHTML(profile)}</a></p>
      `;
    }
  }

  // Minimal styles
  const style = document.createElement("style");
  style.textContent = `
    #pubs-root { line-height: 1.5; }
    .pub-year { margin: 1.5rem 0; }
    .pub-year h2 { margin-bottom: 0.5rem; border-bottom: 1px solid #eee; padding-bottom: 0.25rem; }
    .pub-list { list-style: none; padding-left: 0; }
    .pub-item { margin: 0.75rem 0; }
    .pub-title { font-weight: 600; }
    .pub-meta { color: #666; font-size: 0.95em; margin-top: 0.15rem; }
    .pub-links { margin-top: 0.25rem; font-size: 0.95em; }
    .pub-links a { text-decoration: none; }
  `;
  document.head.appendChild(style);

  fetchHAL();
})();
</script>
