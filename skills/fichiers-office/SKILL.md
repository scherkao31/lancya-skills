---
name: fichiers-office
description: "Genere de vrais fichiers Microsoft Office telechargeables (Excel .xlsx, Word .docx) entierement cote navigateur, via un artifact HTML autonome. Pour Excel : un TABLEUR EDITABLE dans le panneau (l'utilisateur modifie les cellules, ajoute des lignes, les totaux se recalculent en direct), puis export .xlsx. Pour Word : un document mis en forme, editable en apercu, exportable en .docx. Construit en JavaScript (SheetJS pour Excel, la librairie docx pour Word). Aucun serveur, rien ne quitte la machine. Utiliser cette skill des que l'utilisateur veut CREER ou GENERER un fichier Excel ou Word, un tableur, un budget, un planning, une facture, un rapport ou un courrier a telecharger. Ne pas utiliser pour analyser un document existant (autre skill) ni pour une presentation (utiliser html-slides)."
when-to-use: "L'utilisateur veut creer un fichier Excel (.xlsx) ou Word (.docx) : tableur editable, budget, planning, facture, rapport, courrier mis en forme."
always-apply: false
---

# Fichiers Office (Excel et Word) cote navigateur

Tu generes de **vrais fichiers Office** (`.xlsx`, `.docx`) directement dans le navigateur, sous forme d'un **artifact HTML autonome** (`text/html`). Le contenu est **interactif** : pour Excel, l'utilisateur edite les cellules dans le panneau et les totaux se recalculent ; pour Word, le texte est editable avant export. Rien n'est envoye a un serveur (souverain par construction).

## Regles de production

1. **Toujours un seul fichier HTML autonome**, rendu en artifact `text/html`.
2. Librairie depuis le CDN : **SheetJS** pour Excel, **docx** pour Word.
3. **Excel = tableur EDITABLE** (cellules `contenteditable`, bouton « ajouter une ligne », totaux recalcules en direct) PLUS un bouton **Telecharger en .xlsx**. Pas un simple apercu fige.
4. **Word = document mis en forme et editable** (zone `contenteditable`) PLUS un bouton **Telecharger en .docx**.
5. Mets le **vrai contenu** demande par l'utilisateur (ses donnees, ses textes).
6. **Style sobre, lisible, neutre** (look proche d'un tableur). Pas de branding impose.
7. Nom de fichier explicite.

## Fonction de telechargement (a reutiliser)

```html
<script>
  function telecharger(nomFichier, blob) {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = nomFichier;
    document.body.appendChild(a); a.click();
    setTimeout(() => { URL.revokeObjectURL(url); a.remove(); }, 1000);
  }
</script>
```

## Excel (.xlsx) : tableur editable avec SheetJS

CDN : `https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js` (variable globale `XLSX`).

Principe : une `<table>` dont les cellules de donnees sont `contenteditable`. Un ecouteur `input` recalcule les totaux. A l'export, on lit l'etat ACTUEL du tableau, on reconstruit un classeur, et on ajoute une ligne de totaux par **formules SUM** (le fichier reste vivant dans Excel).

### Modele complet editable (budget, a adapter)

```html
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js"></script>
  <style>
    body { font-family: Arial, Helvetica, sans-serif; margin: 24px; color: #1a1a1a; }
    h1 { font-size: 1.3em; }
    .barre { margin: 12px 0; display: flex; gap: 8px; flex-wrap: wrap; }
    button { background:#111; color:#fff; border:0; padding:8px 14px; border-radius:8px; cursor:pointer; font-size:.95em; }
    button.secondaire { background:#fff; color:#111; border:1px solid #ccc; }
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #d0d0d0; padding: 6px 10px; }
    th { background:#f3f4f6; }
    td[contenteditable]:focus { outline: none; background: #eef4ff; }
    td.num { text-align: right; font-variant-numeric: tabular-nums; }
    tr.total { background:#fafafa; font-weight:600; }
    .hint { color:#666; font-size:.85em; margin-top:10px; }
  </style>
</head>
<body>
  <h1>Budget previsionnel sur 3 mois</h1>
  <div class="barre">
    <button class="secondaire" onclick="ajouterLigne()">+ Ajouter une ligne</button>
    <button onclick="telechargerXlsx()">Telecharger en Excel (.xlsx)</button>
  </div>
  <table id="grille">
    <thead><tr><th>Poste</th><th>Juillet</th><th>Aout</th><th>Septembre</th></tr></thead>
    <tbody id="corps"></tbody>
    <tfoot><tr class="total"><td>TOTAL</td><td class="num" id="t0">0</td><td class="num" id="t1">0</td><td class="num" id="t2">0</td></tr></tfoot>
  </table>
  <p class="hint">Clique dans une cellule pour la modifier. Les totaux se recalculent automatiquement. Le bouton exporte l'etat actuel en .xlsx (avec formules).</p>

  <script>
    // Donnees initiales (a remplacer par celles de l'utilisateur)
    const lignesInit = [
      ["Loyer", 2500, 2500, 2500],
      ["Salaires nets", 12000, 12000, 12500],
      ["Marketing digital", 1500, 800, 2200],
    ];
    const corps = document.getElementById("corps");

    function cellule(val, num) {
      const td = document.createElement("td");
      td.contentEditable = "true";
      if (num) td.className = "num";
      td.textContent = val;
      td.addEventListener("input", recalc);
      return td;
    }
    function ajouterLigne(vals) {
      const v = Array.isArray(vals) ? vals : ["Nouveau poste", 0, 0, 0];
      const tr = document.createElement("tr");
      tr.appendChild(cellule(v[0], false));
      for (let i = 1; i < 4; i++) tr.appendChild(cellule(v[i], true));
      corps.appendChild(tr);
      recalc();
    }
    function nombre(t) {
      const n = parseFloat(String(t).replace(/[^0-9.,-]/g, "").replace(",", "."));
      return isNaN(n) ? 0 : n;
    }
    function recalc() {
      const tot = [0, 0, 0];
      for (const tr of corps.rows) for (let c = 1; c < 4; c++) tot[c - 1] += nombre(tr.cells[c].textContent);
      for (let c = 0; c < 3; c++) document.getElementById("t" + c).textContent = tot[c].toLocaleString("fr-CH");
    }
    function telecharger(nom, blob) {
      const u = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = u; a.download = nom;
      document.body.appendChild(a); a.click();
      setTimeout(() => { URL.revokeObjectURL(u); a.remove(); }, 1000);
    }
    function telechargerXlsx() {
      const aoa = [["Poste", "Juillet", "Aout", "Septembre"]];
      for (const tr of corps.rows) {
        aoa.push([tr.cells[0].textContent, nombre(tr.cells[1].textContent), nombre(tr.cells[2].textContent), nombre(tr.cells[3].textContent)]);
      }
      const n = aoa.length; // les donnees occupent les lignes 2..n
      aoa.push(["TOTAL", { t: "n", f: `SUM(B2:B${n})` }, { t: "n", f: `SUM(C2:C${n})` }, { t: "n", f: `SUM(D2:D${n})` }]);
      const wb = XLSX.utils.book_new();
      const ws = XLSX.utils.aoa_to_sheet(aoa);
      ws["!cols"] = [{ wch: 28 }, { wch: 12 }, { wch: 12 }, { wch: 12 }];
      XLSX.utils.book_append_sheet(wb, ws, "Budget");
      const out = XLSX.write(wb, { bookType: "xlsx", type: "array" });
      telecharger("budget.xlsx", new Blob([out], { type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" }));
    }

    lignesInit.forEach(ajouterLigne);
    recalc();
  </script>
</body>
</html>
```

Astuces : plusieurs feuilles en rappelant `book_append_sheet` ; pour un planning, memes principes avec des colonnes de dates ; garde les cellules de la 1re colonne (libelles) editables aussi.

## Word (.docx) : document editable avec la librairie docx

CDN : `https://cdn.jsdelivr.net/npm/docx@9.5.1/build/index.umd.js` (variable globale `docx`).

Principe : une zone `contenteditable` qui montre le document mis en forme (l'utilisateur peut retoucher le texte), et un bouton qui construit le `.docx` a partir du contenu et le telecharge via `docx.Packer.toBlob`.

### Modele complet (courrier editable, a adapter)

```html
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdn.jsdelivr.net/npm/docx@9.5.1/build/index.umd.js"></script>
  <style>
    body { font-family: Arial, Helvetica, sans-serif; margin: 24px; color: #1a1a1a; }
    button { background:#111; color:#fff; border:0; padding:8px 14px; border-radius:8px; cursor:pointer; font-size:.95em; }
    .doc { max-width: 640px; border: 1px solid #e0e0e0; padding: 32px; margin-top: 12px; line-height: 1.55; }
    .doc:focus { outline: 2px solid #b8c8ff; }
    .hint { color:#666; font-size:.85em; margin-top:10px; }
  </style>
</head>
<body>
  <button onclick="telechargerDocx()">Telecharger en Word (.docx)</button>
  <div class="doc" id="doc" contenteditable="true">
    <p>Geneve, le 16 juin 2026</p>
    <p><strong>Objet : resiliation de bail</strong></p>
    <p>Madame, Monsieur,</p>
    <p>Par la presente, je vous informe de la resiliation de mon bail concernant le logement sis...</p>
    <p>Veuillez agreer, Madame, Monsieur, mes salutations distinguees.</p>
  </div>
  <p class="hint">Tu peux modifier le texte ci-dessus avant de telecharger.</p>

  <script>
    function telecharger(nom, blob) {
      const u = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = u; a.download = nom;
      document.body.appendChild(a); a.click();
      setTimeout(() => { URL.revokeObjectURL(u); a.remove(); }, 1000);
    }
    function telechargerDocx() {
      const { Document, Packer, Paragraph, TextRun } = docx;
      // On lit chaque <p> de la zone editable et on le transforme en paragraphe Word
      const blocs = Array.from(document.getElementById("doc").querySelectorAll("p"));
      const children = blocs.map(p => {
        const gras = p.querySelector("strong") != null;
        return new Paragraph({ children: [ new TextRun({ text: p.textContent, bold: gras }) ] });
      });
      const doc = new Document({ sections: [{ children }] });
      Packer.toBlob(doc).then(blob => telecharger("courrier.docx", blob));
    }
  </script>
</body>
</html>
```

Astuces Word : titres (`heading: docx.HeadingLevel.HEADING_1`), gras/italique (`new TextRun({ text, bold, italics })`), listes (`bullet: { level: 0 }`), tableaux (`docx.Table`).

## Cas particuliers

- **Presentations** : utiliser `html-slides`, pas cette skill.
- **PowerPoint .pptx** explicitement demande : possible avec `pptxgenjs` (CDN `https://cdn.jsdelivr.net/npm/pptxgenjs@3.12.0/dist/pptxgen.bundle.js`, global `PptxGenJS`, export via `pptx.write({ outputType:'blob' }).then(...)`).

## Rappels

- Tout dans **un seul fichier HTML autonome** (librairie en CDN, styles en `<style>`).
- Excel **editable** (cellules modifiables + totaux en direct), Word **editable** en apercu. Toujours un bouton de telechargement (au clic de l'utilisateur).
- L'export lit l'**etat actuel** (modifie) du contenu, pas seulement les donnees initiales.
- Remplis avec les **vraies donnees** de l'utilisateur. Style sobre et neutre.
