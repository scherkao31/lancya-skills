---
name: document-word
description: "Genere de vrais documents Word (.docx) telechargeables entierement cote navigateur, via un artifact HTML autonome (librairie docx). Le document est mis en forme, editable en apercu, et telechargeable en un clic, sans serveur (rien ne quitte la machine de l'utilisateur). Utiliser cette skill des que l'utilisateur veut CREER ou GENERER un document Word a telecharger : lettre, courrier, rapport, note, attestation, modele, mise en page de texte. Ne pas utiliser pour un tableur Excel (skill tableur-excel), une presentation (skill html-slides), ni pour analyser un document existant (autre skill)."
when-to-use: "L'utilisateur veut creer et telecharger un document Word (.docx) : lettre, courrier, rapport, note, attestation, modele mis en forme."
always-apply: false
---

# Documents Word (.docx) cote navigateur

Tu generes de **vrais documents Word telechargeables** sous forme d'un **artifact HTML autonome** (`text/html`). Le document est construit en JavaScript avec la librairie **docx**, affiche en apercu (editable), et telecharge au clic. Rien n'est envoye a un serveur : souverain par construction, et leger (ne charge pas l'infra).

## Regles

1. **Un seul fichier HTML autonome**, rendu en artifact `text/html`.
2. Librairie docx depuis le CDN : `https://cdn.jsdelivr.net/npm/docx@9.5.1/build/index.umd.js` (variable globale `docx`).
3. **Apercu editable** (zone `contenteditable`) + bouton **Telecharger en .docx**.
4. Mets le **vrai contenu** demande. Style sobre, professionnel, neutre (adapte a la marque de l'utilisateur s'il en donne).
5. Nom de fichier explicite.

## Modele complet (courrier editable, a adapter)

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
      const blocs = Array.from(document.getElementById("doc").querySelectorAll("p"));
      const children = blocs.map(p => new Paragraph({
        children: [ new TextRun({ text: p.textContent, bold: p.querySelector("strong") != null }) ]
      }));
      const d = new Document({ sections: [{ children }] });
      Packer.toBlob(d).then(blob => telecharger("courrier.docx", blob));
    }
  </script>
</body>
</html>
```

## Astuces docx

Titres (`heading: docx.HeadingLevel.HEADING_1`), gras/italique (`new TextRun({ text, bold:true, italics:true })`), listes (`bullet: { level: 0 }`), tableaux (`new docx.Table({ rows: [...] })`), saut de page (`new docx.Paragraph({ pageBreakBefore: true })`).

## Rappels

- Tout dans **un seul fichier HTML autonome** (librairie en CDN, styles en `<style>`).
- Toujours un **apercu editable** + un **bouton de telechargement** (declenche au clic).
- Remplis avec les **vraies donnees** de l'utilisateur. Style sobre et neutre.
