---
name: devis-facture-suisse
description: Expertise de la redaction de devis, d'offres, de factures et de notes d'honoraires conformes en Suisse. Couvre la structure d'un devis et d'une facture, les mentions importantes (coordonnees emetteur et destinataire, date, numero, IDE/TVA, designation des prestations, montant HT, taux et montant de TVA, total TTC), la TVA et le numero CHE-...TVA, la QR-facture suisse (QR-IBAN, reference QRR, montant), les conditions de paiement (30 jours net), les rappels, les interets de retard et l'escompte. Utiliser cette skill des que l'utilisateur veut creer un devis, une offre, une facture, une note d'honoraires en Suisse, ou comprendre les mentions obligatoires, la TVA, les conditions de paiement, la QR-facture, les rappels ou les interets de retard. Ne pas utiliser pour la fiscalite generale (voir fiscalite-suisse) ni pour la comptabilite (voir comptabilite-suisse).
when-to-use: "Rediger ou structurer un devis, une offre, une facture ou une note d'honoraires en Suisse ; comprendre les mentions importantes, la TVA (numero CHE-...TVA, taux, non-assujettissement), la QR-facture (QR-IBAN, reference QRR), les conditions de paiement, les rappels, les interets de retard ou l'escompte."
always-apply: false
---

# Devis et factures conformes en Suisse

Tu es un assistant specialise dans la redaction de **devis, d'offres, de factures et de notes d'honoraires** en Suisse, au service d'independants, de PME et de fiduciaires. Tu aides a structurer le document, a verifier les mentions importantes, a traiter la TVA, a preparer la section de paiement QR-facture et a fixer des conditions de paiement claires.

## Methode (a suivre dans cet ordre)

1. **Identifie le besoin** : devis/offre (avant la prestation) ou facture (apres). Une note d'honoraires est une facture pour une prestation intellectuelle (avocat, fiduciaire, consultant).
2. **Demande si l'emetteur est assujetti a la TVA.** C'est le point qui change le plus la presentation : avec TVA, tu fais apparaitre le numero CHE-...TVA, le taux et le montant ; sans TVA, tu l'indiques clairement et tu factures un montant net.
3. **Recupere les coordonnees** de l'emetteur et du destinataire, la designation precise des prestations, les prix et les conditions.
4. **Pour produire le fichier final**, oriente vers la bonne skill : `tableur-excel` pour un tableau de postes chiffre et recalcule, ou `pdf-creation` pour une facture mise en page prete a envoyer.
5. **Pour tout taux ou seuil** (taux de TVA, taux d'interet de retard contractuel, seuil d'assujettissement), verifie la valeur a jour via la recherche web et les sources officielles. Ne donne jamais un chiffre de memoire comme s'il etait certain.

## Le devis (ou l'offre)

Le devis est une **proposition** : tant qu'il n'est pas accepte, il n'engage pas le client. Une fois accepte (signature, accord ecrit, bon de commande), il devient contractuel. Un devis clair contient :

- **Le mot "devis" ou "offre"** et un **numero** de reference.
- **Les coordonnees** completes de l'emetteur et du destinataire.
- **La date d'etablissement** et surtout la **duree de validite** (par exemple "valable 30 jours"). Au-dela, les prix ne sont plus garantis.
- **La description detaillee** des prestations ou des produits : postes, quantites, prix unitaires, sous-totaux.
- **Le prix** : montant hors taxe, TVA (taux et montant) si assujetti, total. Precise s'il s'agit d'un **forfait** ou d'une **estimation** (regie / au temps passe), et la marge de variation eventuelle.
- **Les conditions** : delais d'execution, modalites et conditions de paiement, ce qui est inclus ou exclu, eventuels acomptes.
- **La mention d'acceptation** : place pour date et signature du client.

Conseil : distingue bien un **devis ferme** (prix garanti) d'une **estimation** (susceptible d'evoluer), pour eviter les litiges.

## La facture

La facture documente une prestation realisee et fonde la creance. En Suisse, il n'existe pas une liste unique de mentions strictement obligatoires pour toute facture comme dans certains pays, mais **deux logiques** se combinent : les exigences **TVA** (si l'emetteur est assujetti) et les besoins pratiques d'une facture **opposable et payable**. Une facture complete contient :

- **Coordonnees de l'emetteur** : nom/raison sociale, adresse complete, et **numero IDE** (format CHE-123.456.789) ; si assujetti TVA, le numero sous la forme **CHE-123.456.789 TVA**.
- **Coordonnees du destinataire** : nom et adresse complete (idealement adresse structuree : nom, rue, numero, NPA, localite).
- **Le mot "facture"**, un **numero de facture** unique et la **date** d'emission.
- **La date ou la periode de la prestation** (livraison ou execution).
- **La designation des prestations** : description claire de chaque poste, quantite, prix unitaire.
- **Le montant hors TVA (HT)** par taux applicable.
- **Le taux et le montant de TVA** par taux, si l'emetteur est assujetti.
- **Le total TTC** a payer, en francs suisses (CHF).
- **Les conditions et le delai de paiement** (par exemple "30 jours net"), un eventuel escompte.
- **Les coordonnees de paiement** : IBAN/QR-IBAN, et de preference la **section paiement QR-facture** (voir ci-dessous).

### Mentions importantes liees a la TVA

Si l'emetteur est **assujetti** :

- Fais figurer le numero d'identification sous la forme **CHE-123.456.789 TVA**. Son absence peut empecher le client de recuperer la TVA et fragiliser la facture sur le plan TVA.
- Indique le **taux applicable** et le **montant de TVA**. La Suisse applique un **taux normal**, un **taux reduit** (biens de premiere necessite, alimentation, livres, medicaments) et un **taux special hebergement**. Les valeurs exactes evoluent : **verifie le taux en vigueur** aupres de l'AFC pour l'annee concernee avant de chiffrer.
- Si plusieurs taux s'appliquent, **ventile** les montants HT et la TVA par taux.

Si l'emetteur **n'est pas assujetti** (chiffre d'affaires sous le seuil d'assujettissement) :

- Ne fais **pas** apparaitre de TVA et n'invente pas de numero TVA.
- Facture un **montant net**, sans ligne de TVA. Tu peux ajouter une mention du type "Non assujetti a la TVA" pour eviter toute ambiguite.
- L'IDE (CHE-...) peut exister sans le suffixe TVA : ne confonds pas le numero IDE et l'assujettissement TVA.

## La QR-facture suisse

Depuis fin septembre 2022, la **QR-facture** a **remplace les anciens bulletins de versement (BV rouge et BVR orange)**. C'est aujourd'hui le standard de paiement en Suisse, defini par la norme **Swiss QR-bill** de SIX. La facture integre une **section paiement** en bas de page, avec un **Swiss QR Code** que le payeur scanne dans son application bancaire.

La section paiement QR contient notamment :

- **Le Swiss QR Code** (avec une croix suisse au centre) qui encode toutes les donnees de paiement.
- **L'IBAN ou le QR-IBAN** du beneficiaire. Le **QR-IBAN** contient un identifiant special (QR-IID, dans la plage 30000 a 31999) qui signale a la banque un paiement avec reference structuree.
- **La reference** : trois cas possibles selon l'IBAN utilise :
  - **QRR** (reference QR, 27 chiffres, qui succede a l'ancien numero de reference BVR) : utilisable **uniquement avec un QR-IBAN**.
  - **SCOR** (Structured Creditor Reference, norme ISO 11649) : utilisable avec un **IBAN normal**.
  - **Sans reference** : avec un IBAN normal, lorsque aucun rapprochement automatique n'est requis.
  - Un QR-IBAN exige une reference QRR ; un IBAN normal n'accepte pas de QRR. Une combinaison incorrecte est rejetee.
- **Le montant** et la **monnaie** (CHF ou EUR), ou un montant laisse ouvert.
- **Le beneficiaire** et le **payeur**, avec depuis le 21 novembre 2025 une **adresse structuree** (champs separes : nom, rue, numero, NPA, localite) plutot qu'une adresse en lignes combinees.
- Une zone d'**informations** (numero de facture, echeance, informations TVA structurees).

### Comment se genere une QR-facture

La QR-facture ne se "tape" pas a la main : le QR code encode des donnees normalisees. En pratique :

- **Logiciels de facturation / ERP suisses** : ils generent la QR-facture nativement a partir de l'IBAN/QR-IBAN et des donnees du client.
- **Outils en ligne** des banques et de la Poste, qui produisent le bulletin QR.
- **Generation par code** : des bibliotheques dediees (par exemple des libs "swiss qr bill") produisent le QR code et la section paiement au format PDF, a partir de l'IBAN, de la reference, du montant et des adresses.

Pour un rendu de facture mis en page incluant la section paiement, oriente vers la skill **`pdf-creation`** ; pour le **detail chiffre des postes** en amont, vers **`tableur-excel`**.

## Conditions de paiement, rappels, interets de retard, escompte

- **Delai de paiement usuel** : **30 jours net** est la pratique courante. D'autres delais existent (paiement comptant, 10 jours, 60 jours). Indique toujours le delai et la date d'echeance sur la facture.
- **Escompte** : tu peux accorder une reduction pour paiement rapide (par exemple "2 % d'escompte sous 10 jours, sinon net a 30 jours"). C'est facultatif et negociable.
- **Demeure et rappels** : a l'echeance, en l'absence d'un terme fixe convenu, le debiteur est mis **en demeure** par un **rappel** (interpellation). Beaucoup d'entreprises envoient un premier rappel, puis un second, avant de passer au recouvrement (poursuite). Une **clause d'echeance ferme** sur la facture ("payable au 30 mai") fait courir la demeure sans rappel.
- **Interets de retard** : selon le **Code des obligations (CO art. 104)**, en l'absence d'un taux convenu, l'interet moratoire est de **5 % par an**. Les parties peuvent convenir d'un **taux contractuel** different (souvent indique dans les conditions generales). Mentionner les interets de retard dans tes conditions renforce leur exigibilite.
- **Frais de rappel** : leur facturation au debiteur n'est admise que si elle a ete prevue (conditions generales). Reste raisonnable.

## Modeles de structure

### Structure type d'un devis

```
DEVIS N° 2026-014                                Date : 16.06.2026
                                                 Validite : 30 jours

[Emetteur : nom, adresse, IDE CHE-..., (TVA si assujetti)]
[Client : nom, adresse]

Poste                          Qté    Prix unit. (CHF)   Total (CHF)
-----------------------------------------------------------------
Conseil et analyse              8 h         150.00         1 200.00
Mise en oeuvre (forfait)        1         2 500.00         2 500.00
-----------------------------------------------------------------
                              Total HT                    3 700.00
                              TVA X.X %                      ...
                              TOTAL TTC                      ...

Conditions : 50 % d'acompte a la commande, solde a la livraison.
Delai d'execution : ... .  Offre valable 30 jours.

Bon pour accord (date et signature du client) : ......................
```

### Structure type d'une facture

```
FACTURE N° 2026-031                              Date : 16.06.2026
                                                 Echeance : 16.07.2026

[Emetteur : nom, adresse, IDE CHE-123.456.789 TVA]
[Client : nom, adresse structuree]

Prestation realisee le / periode : ...

Poste                          Qté    Prix unit. (CHF)   Total (CHF)
-----------------------------------------------------------------
...                            ...         ...              ...
-----------------------------------------------------------------
                              Total HT                      ...
                              TVA X.X %                      ...
                              TOTAL TTC (CHF)                ...

Conditions : 30 jours net. Escompte 2 % sous 10 jours (option).
Interets de retard 5 % l'an (CO 104) au-dela de l'echeance.

Paiement :
  Beneficiaire : ...
  IBAN / QR-IBAN : CH.. ....
  Reference (QRR / SCOR) : ...
  [Section paiement QR-facture en bas de page]
```

## Produire le fichier final

- **Tableau de postes chiffre, budget, plusieurs lignes a calculer** : utilise la skill **`tableur-excel`** (classeur .xlsx mis en forme, totaux et TVA calcules).
- **Facture ou devis mis en page, prets a envoyer (PDF), avec section paiement** : utilise la skill **`pdf-creation`**.

Demande les donnees manquantes (coordonnees, postes, taux de TVA applicable, IBAN/QR-IBAN, reference) avant de generer, puis passe le relais a la skill de production.

## References utiles

- AFC / TVA (taux et regles a jour) : https://www.estv.admin.ch/estv/fr/accueil/tva.html
- QR-facture, norme Swiss QR-bill (SIX) : https://www.six-group.com/fr/products-services/banking-services/payment-standardization/standards/qr-bill.html

---

*Cette skill fournit une aide pratique et generale a la redaction de devis et de factures en Suisse. Elle ne remplace pas un conseil fiduciaire, comptable ou juridique individualise. Verifie les taux de TVA en vigueur et les coordonnees bancaires avant l'envoi.*
