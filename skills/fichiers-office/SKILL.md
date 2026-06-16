---
name: fichiers-office
description: "Genere de vrais fichiers Microsoft Office telechargeables (Excel .xlsx, Word .docx) entierement cote navigateur, via un artifact HTML autonome : tableaux, budgets, plannings, factures, rapports, courriers, comptes rendus mis en forme. Le fichier est construit en JavaScript (SheetJS pour Excel, la librairie docx pour Word), affiche en apercu, et telechargeable en un clic. Aucun serveur, rien ne quitte la machine de l'utilisateur. Utiliser cette skill des que l'utilisateur veut CREER ou GENERER un fichier Excel ou Word a telecharger (un .xlsx, un .docx, un tableur, un classeur, un document Word). Ne pas utiliser pour une simple reponse texte, pour analyser un document existant (autre skill), ni pour une presentation (utiliser html-slides)."
when-to-use: "L'utilisateur veut creer et telecharger un vrai fichier Excel (.xlsx) ou Word (.docx) : tableur, budget, planning, facture, rapport, courrier mis en forme."
always-apply: false
---

# Fichiers Office (Excel et Word) cote navigateur

Tu generes de **vrais fichiers Office telechargeables** (`.xlsx`, `.docx`) directement dans le navigateur, sous forme d'un **artifact HTML autonome** (`text/html`). Tu n'envoies rien a un serveur : le fichier est fabrique en JavaScript dans la page, affiche en apercu, et telecharge au clic. C'est souverain par nature (le fichier ne quitte jamais la machine de l'utilisateur).

## Regles de production

1. **Toujours un seul fichier HTML autonome**, rendu en artifact `text/html`.
2. Charge la librairie depuis le CDN : **SheetJS** pour Excel, **docx** pour Word.
3. **Affiche un apercu clair** du contenu (un tableau pour Excel, le document mis en forme pour Word) PLUS un bouton **Telecharger**.
4. Mets le **vrai contenu** demande par l'utilisateur (ses donnees, ses textes), pas un exemple generique.
5. **Style sobre et professionnel**, neutre. Adapte-toi a la marque ou au sujet de l'utilisateur s'il en donne ; sinon, reste neutre. Pas de branding impose.
6. Nom de fichier explicite (ex. `budget-2026.xlsx`, `courrier-resiliation.docx`).

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

## Excel (.xlsx) avec SheetJS

CDN : `https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js` (expose la variable globale `XLSX`).

Construction : `XLSX.utils.aoa_to_sheet(tableau2D)` ou `XLSX.utils.json_to_sheet(objets)`, puis `XLSX.utils.book_append_sheet`. Pour plusieurs feuilles, appelle `book_append_sheet` plusieurs fois. Telechargement via `XLSX.write(wb, {bookType:'xlsx', type:'array'})` + `telecharger(...)`.

### Modele complet (budget, a adapter)

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
    table { border-collapse: collapse; margin: 16px 0; width: 100%; }
    th, td { border: 1px solid #d0d0d0; padding: 6px 10px; text-align: left; }
    th { background: #f3f4f6; }
    td.num { text-align: right; }
    button { background: #111; color: #fff; border: 0; padding: 10px 16px; border-radius: 8px; font-size: 1em; cursor: pointer; }
  </style>
</head>
<body>
  <h1>Budget mensuel 2026</h1>
  <p>Apercu du tableur. Clique pour telecharger le fichier Excel.</p>
  <button onclick="genererEtTelecharger()">Telecharger le fichier Excel (.xlsx)</button>
  <div id="apercu"></div>

  <script>
    // 1) Les donnees (a remplacer par celles de l'utilisateur)
    const enTetes = ["Poste", "Janvier", "Fevrier", "Mars"];
    const lignes = [
      ["Loyer", 1500, 1500, 1500],
      ["Salaires", 8000, 8000, 8200],
      ["Marketing", 1200, 900, 1100],
    ];

    // 2) Apercu HTML
    (function apercu() {
      let html = "<table><thead><tr>" + enTetes.map(h => `<th>${h}</th>`).join("") + "</tr></thead><tbody>";
      for (const l of lignes) {
        html += "<tr>" + l.map((c, i) => i === 0 ? `<td>${c}</td>` : `<td class="num">${c.toLocaleString('fr-CH')}</td>`).join("") + "</tr>";
      }
      html += "</tbody></table>";
      document.getElementById("apercu").innerHTML = html;
    })();

    // 3) Generation + telechargement
    function telecharger(nomFichier, blob) {
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = nomFichier;
      document.body.appendChild(a); a.click();
      setTimeout(() => { URL.revokeObjectURL(url); a.remove(); }, 1000);
    }
    function genererEtTelecharger() {
      const wb = XLSX.utils.book_new();
      const ws = XLSX.utils.aoa_to_sheet([enTetes, ...lignes]);
      XLSX.utils.book_append_sheet(wb, ws, "Budget");
      const out = XLSX.write(wb, { bookType: "xlsx", type: "array" });
      telecharger("budget-2026.xlsx", new Blob([out], { type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" }));
    }
  </script>
</body>
</html>
```

Astuces Excel : formules via une cellule `{ f: "SUM(B2:B4)" }` (utiliser `aoa_to_sheet` avec des objets cellule) ; largeurs de colonnes via `ws['!cols'] = [{wch:20},{wch:12}]` ; plusieurs feuilles en rappelant `book_append_sheet`.

## Word (.docx) avec la librairie docx

CDN : `https://cdn.jsdelivr.net/npm/docx@9.5.1/build/index.umd.js` (expose la variable globale `docx`).

Construction : `new docx.Document({ sections: [{ children: [...] }] })` avec des `docx.Paragraph`, `docx.TextRun`, titres via `heading: docx.HeadingLevel.HEADING_1`, tableaux via `docx.Table`. Telechargement via `docx.Packer.toBlob(doc).then(blob => telecharger(...))`.

### Modele complet (courrier, a adapter)

```html
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdn.jsdelivr.net/npm/docx@9.5.1/build/index.umd.js"></script>
  <style>
    body { font-family: Arial, Helvetica, sans-serif; margin: 24px; color: #1a1a1a; }
    .apercu { max-width: 640px; border: 1px solid #e0e0e0; padding: 32px; margin-top: 16px; line-height: 1.5; }
    button { background: #111; color: #fff; border: 0; padding: 10px 16px; border-radius: 8px; font-size: 1em; cursor: pointer; }
  </style>
</head>
<body>
  <button onclick="genererEtTelecharger()">Telecharger le document Word (.docx)</button>
  <div class="apercu" id="apercu">
    <p>Geneve, le 16 juin 2026</p>
    <p><strong>Objet : resiliation de bail</strong></p>
    <p>Madame, Monsieur,</p>
    <p>Par la presente, je vous informe de la resiliation de mon bail concernant le logement sis...</p>
    <p>Veuillez agreer, Madame, Monsieur, mes salutations distinguees.</p>
  </div>

  <script>
    function telecharger(nomFichier, blob) {
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = nomFichier;
      document.body.appendChild(a); a.click();
      setTimeout(() => { URL.revokeObjectURL(url); a.remove(); }, 1000);
    }
    function genererEtTelecharger() {
      const { Document, Packer, Paragraph, TextRun, HeadingLevel } = docx;
      const doc = new Document({
        sections: [{
          children: [
            new Paragraph("Geneve, le 16 juin 2026"),
            new Paragraph({ children: [ new TextRun({ text: "Objet : resiliation de bail", bold: true }) ] }),
            new Paragraph("Madame, Monsieur,"),
            new Paragraph("Par la presente, je vous informe de la resiliation de mon bail concernant le logement sis..."),
            new Paragraph("Veuillez agreer, Madame, Monsieur, mes salutations distinguees."),
          ],
        }],
      });
      Packer.toBlob(doc).then(blob => telecharger("courrier-resiliation.docx", blob));
    }
  </script>
</body>
</html>
```

Astuces Word : titres (`heading: HeadingLevel.HEADING_1/2`), gras/italique (`new TextRun({ text, bold:true, italics:true })`), listes (`bullet: { level: 0 }`), tableaux (`new docx.Table({ rows: [...] })`), sauts de page (`new docx.Paragraph({ pageBreakBefore: true })`).

## Cas particuliers

- **Presentations** : ne pas utiliser cette skill, utiliser `html-slides` (reveal.js).
- **PowerPoint .pptx** reellement demande : possible avec `pptxgenjs` (CDN `https://cdn.jsdelivr.net/npm/pptxgenjs@3.12.0/dist/pptxgen.bundle.js`, global `PptxGenJS`, telechargement via `pptx.write({ outputType:'blob' }).then(...)`), meme principe (apercu + bouton).

## Rappels

- Tout dans **un seul fichier HTML autonome** (librairie en CDN, styles en `<style>`).
- Toujours un **apercu visible** + un **bouton de telechargement** (le telechargement se declenche au clic de l'utilisateur).
- Remplis avec les **vraies donnees** de l'utilisateur. Style sobre, lisible, neutre.
