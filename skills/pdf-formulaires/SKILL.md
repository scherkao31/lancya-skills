---
name: pdf-formulaires
description: Utiliser cette skill des que l'utilisateur veut remplir un formulaire PDF, completer des champs d'un PDF, pre-remplir un formulaire administratif (demande, inscription, declaration, contrat a signer), cocher des cases ou figer les valeurs saisies dans un PDF a champs interactifs (AcroForm). Ne pas utiliser pour extraire du texte (voir pdf-extraction), pour fusionner ou decouper des PDF (voir pdf-fusion-decoupe), ni pour creer un PDF de zero (voir pdf-creation).
when-to-use: L'utilisateur fournit un formulaire PDF deja construit avec des champs interactifs et veut le remplir par code, sans le rouvrir manuellement dans un lecteur PDF. Typiquement : un formulaire administratif suisse, une demande d'allocation, un bulletin d'inscription, un contrat type. La skill detecte les champs, les remplit, gere les cases a cocher et les boutons radio, puis aplatit le rendu si on veut figer les valeurs.
always-apply: false
---

# Remplir des formulaires PDF (champs AcroForm)

Cette skill remplit par code les champs interactifs d'un formulaire PDF (norme AcroForm) :
champs texte, cases a cocher, boutons radio, listes deroulantes. Tout se fait dans le
sandbox, sans rouvrir le fichier dans un lecteur PDF.

Bibliotheque principale : **pypdf** (version moderne). `PyPDF2` est l'ancien nom du meme
projet et fonctionne avec une API proche ; `pdfrw` et `reportlab` servent uniquement aux
cas limites (overlay sur un PDF sans champs).

## Quand l'utiliser, quand l'eviter

- A utiliser : remplir, pre-remplir ou cocher les champs d'un formulaire PDF interactif, puis
  eventuellement figer le rendu.
- A ne pas utiliser :
  - extraire le texte d'un PDF -> skill `pdf-extraction` ;
  - fusionner, decouper, faire pivoter des pages -> skill `pdf-fusion-decoupe` ;
  - construire un PDF a partir de rien -> skill `pdf-creation`.

## Environnement

- Code Python dans le sandbox `id='py'` : `pypdf` (ou `PyPDF2`), `pdfrw`, `reportlab`.
- `soffice` / LibreOffice disponible en BASH (utile seulement pour convertir un autre format
  vers PDF, pas pour remplir des champs).
- Dossier persistant : `/mnt/data`. Lire et ecrire les fichiers la.

## Workflow

1. **Inspecter** le formulaire : verifier qu'il contient bien des champs AcroForm, puis
   lister les noms, types et valeurs possibles.
2. **Construire le dictionnaire** `nom_du_champ -> valeur` a partir de ce que veut
   l'utilisateur. Pour les cases a cocher et les boutons radio, utiliser la valeur exacte de
   l'etat coche (souvent `/Oui`, `/On`, `/1`... jamais devine, toujours lu a l'etape 1).
3. **Remplir** avec `update_page_form_field_values`.
4. **Figer si besoin** : soit `NeedAppearances` (le lecteur regenere l'apparence a
   l'ouverture), soit aplatissement (`flatten=True`) qui transforme les valeurs en contenu
   fixe non modifiable.
5. **Sauvegarder** dans `/mnt/data` et confirmer.

## Etape 1 : inspecter et lister les champs

```python
from pypdf import PdfReader

CHEMIN = "/mnt/data/formulaire.pdf"
reader = PdfReader(CHEMIN)

fields = reader.get_fields()  # None si le PDF n'a aucun champ AcroForm

if not fields:
    print("Aucun champ AcroForm detecte. Voir la section 'Formulaire sans champs'.")
else:
    # Correspondance des codes de type vers un libelle lisible
    libelle_type = {
        "/Tx": "texte",
        "/Btn": "case a cocher / bouton radio",
        "/Ch": "liste (deroulante ou a choix)",
        "/Sig": "signature",
    }
    for nom, champ in fields.items():
        type_brut = champ.get("/FT")
        type_lisible = libelle_type.get(type_brut, type_brut)
        valeur = champ.get("/V")
        # Etats possibles pour une case a cocher ou un bouton radio
        etats = champ.get("/_States_")
        # Options possibles pour une liste
        options = champ.get("/Opt")
        print(f"- {nom!r} | type: {type_lisible} | valeur actuelle: {valeur!r}")
        if etats:
            print(f"    etats possibles (cocher/radio): {etats}")
        if options:
            print(f"    options (liste): {options}")
```

Points importants :

- `get_fields()` renvoie `None` quand le PDF n'a pas de couche AcroForm (voir plus bas).
- `/_States_` donne les valeurs valides d'une case a cocher ou d'un bouton radio, par exemple
  `['/Oui', '/Off']` ou `['/1', '/2', '/Off']`. `/Off` est l'etat decoche. La valeur a
  envoyer pour cocher est l'autre etat, **exactement** tel qu'affiche (le `/` initial est un
  artefact d'affichage ; au remplissage on passe en general la chaine sans le `/`, voir
  l'etape suivante).
- Pour ne lister que les champs texte avec leurs valeurs :
  `reader.get_form_text_fields()` renvoie un dictionnaire `{nom: valeur}`.

## Etape 2 et 3 : remplir les champs

On ouvre le formulaire, on l'ajoute a un writer, on remplit page par page, on ecrit le
resultat.

```python
from pypdf import PdfReader, PdfWriter

ENTREE = "/mnt/data/formulaire.pdf"
SORTIE = "/mnt/data/formulaire_rempli.pdf"

reader = PdfReader(ENTREE)
writer = PdfWriter()
writer.append(reader)  # copie pages + structure AcroForm dans le writer

# nom_du_champ -> valeur
valeurs = {
    "nom": "Cherkaoui",
    "prenom": "Salim",
    "date_naissance": "31.10.1990",
    "adresse": "Rue de l'Exemple 12, 1200 Geneve",
    # Case a cocher : passer la valeur de l'etat coche lue a l'etape 1.
    # Si /_States_ = ['/Oui', '/Off'], on coche avec '/Oui'.
    "accepte_conditions": "/Oui",
    # Bouton radio : meme principe, la valeur choisie parmi les etats.
    "civilite": "/M",
    # Liste deroulante : une des valeurs de /Opt.
    "canton": "Geneve",
}

# Remplir chaque page (les champs sont rattaches a une page precise).
for page in writer.pages:
    writer.update_page_form_field_values(
        page,
        valeurs,
        auto_regenerate=False,  # ne PAS forcer NeedAppearances ici (voir plus bas)
    )

with open(SORTIE, "wb") as f:
    writer.write(f)

print(f"Formulaire rempli : {SORTIE}")
```

Notes sur l'API :

- `auto_regenerate` controle le drapeau `/NeedAppearances`. Le mettre a `False` est le choix
  par defaut recommande : on evite que certains lecteurs affichent une boite de dialogue ou
  laissent les champs vides. On l'active seulement si le texte saisi n'apparait pas (voir
  section suivante).
- Si un nom de `valeurs` n'existe pas dans la page, il est simplement ignore pour cette page,
  d'ou la boucle sur toutes les pages.
- Les valeurs sont des chaines de caracteres, y compris pour les nombres et les dates.

### Cases a cocher : gerer les etats `On` / `Off`

Le piege classique : la valeur a envoyer pour cocher n'est pas `True` ni `"X"`, mais le nom
exact de l'etat coche, propre a chaque champ (`/Oui`, `/On`, `/Yes`, `/1`...). Toujours le
lire a l'etape 1 via `/_States_`.

```python
# Lire les etats d'une case precise avant de la cocher
reader = PdfReader(ENTREE)
etats = reader.get_fields()["accepte_conditions"].get("/_States_")
print(etats)  # ex: ['/Oui', '/Off']

# Etat coche = celui qui n'est pas '/Off'
etat_coche = next(e for e in etats if e != "/Off")
valeurs = {"accepte_conditions": etat_coche}  # ex: '/Oui'
```

Si la case reste decochee apres remplissage, essayer la valeur sans le `/` initial
(`"Oui"`), ou activer `auto_regenerate=True` pour cette passe.

## Etape 4 : figer le rendu

Deux strategies selon le besoin.

### Option A : NeedAppearances (champs encore modifiables)

Le drapeau demande au lecteur PDF de regenerer l'apparence visuelle des champs a
l'ouverture. Utile quand le texte saisi ne s'affiche pas dans certains lecteurs. Les champs
restent editables.

```python
from pypdf import PdfReader, PdfWriter
from pypdf.generic import NameObject, BooleanObject

reader = PdfReader(ENTREE)
writer = PdfWriter()
writer.append(reader)

for page in writer.pages:
    writer.update_page_form_field_values(page, valeurs, auto_regenerate=False)

# Forcer NeedAppearances sur l'AcroForm du document
writer.set_need_appearances_writer(True)

with open("/mnt/data/formulaire_rempli_appearances.pdf", "wb") as f:
    writer.write(f)
```

`writer.set_need_appearances_writer(True)` est la methode dediee de pypdf. En equivalent
manuel (utile si la methode n'existe pas dans une vieille version) :

```python
writer._root_object["/AcroForm"][NameObject("/NeedAppearances")] = BooleanObject(True)
```

### Option B : aplatir le formulaire (valeurs figees, non modifiables)

L'aplatissement transforme les valeurs saisies en contenu graphique fixe et supprime les
champs interactifs. C'est le bon choix pour un document final a transmettre ou archiver :
plus personne ne peut modifier les champs.

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader(ENTREE)
writer = PdfWriter()
writer.append(reader)

for page in writer.pages:
    writer.update_page_form_field_values(
        page,
        valeurs,
        auto_regenerate=False,
        flatten=True,  # fige les valeurs et retire les champs interactifs
    )

with open("/mnt/data/formulaire_aplati.pdf", "wb") as f:
    writer.write(f)
```

Conseil pratique : produire deux fichiers quand c'est pertinent, un version editable
(option A) et une version figee (option B), et laisser l'utilisateur choisir.

## Cas limite : formulaire sans champs (XFA ou scanne)

Quand `reader.get_fields()` renvoie `None`, le PDF n'a pas de couche AcroForm remplissable.
Trois cas typiques :

1. **PDF scanne ou image** : le formulaire est une image, sans champ. Aucun champ a remplir
   par code.
2. **Formulaire XFA** (souvent genere par d'anciens outils Adobe LiveCycle) : les champs sont
   decrits dans une couche XML XFA que pypdf ne sait pas remplir de maniere fiable. Souvent il
   existe une couche AcroForm de secours ; sinon, traiter comme un PDF sans champs.
3. **Faux formulaire** : les cases et lignes sont juste dessinees, sans champ interactif.

Verification rapide de la presence d'XFA :

```python
reader = PdfReader(ENTREE)
acro = reader.trailer["/Root"].get("/AcroForm")
a_xfa = bool(acro and "/XFA" in acro)
print("Couche XFA presente :", a_xfa)
print("Champs AcroForm :", "oui" if reader.get_fields() else "non")
```

### Alternative 1 : overlay avec reportlab (PDF non scanne, positions connues)

Si le PDF est un vrai document (texte vectoriel) mais sans champs, on superpose le texte aux
bons endroits. Il faut connaitre les coordonnees (origine en bas a gauche, en points ;
1 cm = 28.35 points).

```python
import io
from pypdf import PdfReader, PdfWriter
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import A4

FOND = "/mnt/data/formulaire.pdf"
SORTIE = "/mnt/data/formulaire_overlay.pdf"

# 1. Dessiner le calque de texte
buffer = io.BytesIO()
c = canvas.Canvas(buffer, pagesize=A4)  # adapter au format reel des pages
c.setFont("Helvetica", 11)
c.drawString(150, 700, "Cherkaoui")          # x, y en points
c.drawString(150, 680, "Salim")
c.drawString(150, 660, "31.10.1990")
c.drawString(95, 500, "X")                   # cocher une case dessinee
c.save()
buffer.seek(0)

# 2. Fusionner le calque sur la premiere page du fond
calque = PdfReader(buffer)
fond = PdfReader(FOND)
writer = PdfWriter()

for i, page in enumerate(fond.pages):
    if i == 0:
        page.merge_page(calque.pages[0])
    writer.add_page(page)

with open(SORTIE, "wb") as f:
    writer.write(f)
```

Methode pour trouver les coordonnees : afficher la taille de page
(`fond.pages[0].mediabox`), faire un premier essai approximatif, puis ajuster x/y par
iterations en regardant le rendu.

### Alternative 2 : OCR (PDF scanne)

Si le PDF est un scan (image), il n'y a ni champ ni texte a positionner proprement. Options :

- expliquer a l'utilisateur que le document n'est pas un formulaire remplissable par code ;
- proposer un OCR (skill `pdf-extraction`) pour recuperer le texte, puis recreer un vrai
  formulaire ou remplir via overlay si les zones sont identifiables ;
- ou recommander de remplir a la main dans un lecteur PDF.

Dans tous les cas, etre transparent sur la limite : un scan ou un XFA ne se remplit pas de
la meme maniere qu'un AcroForm, et le resultat par overlay depend de la precision des
coordonnees.

## Verification finale

Apres remplissage, relire le fichier produit pour controler les valeurs (sauf s'il a ete
aplati, auquel cas les champs n'existent plus) :

```python
verif = PdfReader(SORTIE)
print(verif.get_form_text_fields())  # {nom: valeur} pour les champs texte
```

## Reference

- Documentation pypdf sur les formulaires : https://pypdf.readthedocs.io/en/stable/user/forms.html
