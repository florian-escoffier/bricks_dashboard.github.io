# Portefeuille Bricks

Suivi de portefeuille pour la plateforme Bricks.co, à partir de l'export des transactions
du wallet. Page unique, sans serveur : **le fichier importé est lu par le navigateur et
n'est transmis nulle part.**

## Mise en ligne sur GitHub Pages

### Par l'interface web de GitHub

1. Créez un dépôt, par exemple `portefeuille-bricks`.
2. **Add file → Upload files**, déposez les quatre fichiers de ce dossier
   (`index.html`, `manifest.webmanifest`, `icon-192.png`, `icon-512.png`), puis validez.
3. **Settings → Pages**. Sous *Build and deployment*, choisissez *Deploy from a branch*,
   branche `main`, dossier `/ (root)`. Enregistrez.
4. L'adresse `https://<votre-compte>.github.io/portefeuille-bricks/` est active au bout
   d'une à deux minutes.

### En ligne de commande

```sh
git init
git add .
git commit -m "Portefeuille Bricks"
git branch -M main
git remote add origin git@github.com:<votre-compte>/portefeuille-bricks.git
git push -u origin main
```
Puis l'étape 3 ci-dessus.

> GitHub Pages n'est disponible sur un dépôt privé qu'avec une formule payante. Un dépôt
> public convient ici : **le code ne contient aucune donnée personnelle**, et toute personne
> ouvrant l'adresse tombe sur une application vide.

## Où vont les données

| | |
|---|---|
| Le fichier importé | lu en mémoire par le navigateur, jamais téléversé |
| Les chiffres calculés | `localStorage`, sur l'appareil et dans ce navigateur |
| Le serveur GitHub | ne reçoit que la demande de la page, jamais vos données |

Conséquences à connaître :

- **Rien n'est synchronisé** entre vos appareils. Chaque navigateur a sa propre copie ;
  il faut importer l'export sur chacun.
- **Vider les données du site efface le portefeuille.** Il suffit de réimporter l'export.
- La navigation privée refuse souvent l'enregistrement : l'application le signale et
  fonctionne le temps de la session.
- `localStorage` plafonne à environ 5 Mo par site. Un export de 2 000 opérations occupe
  près de 400 Ko ; au-delà du plafond, l'application conserve les totaux et les projets
  et prévient que le détail des opérations n'a pas pu être gardé.

### Les trois requêtes sortantes

La page charge trois ressources depuis des CDN publics : les polices Google Fonts,
`d3` (graphiques) et `SheetJS` (lecture des fichiers `.xlsx`). Ces serveurs voient votre
adresse IP, jamais le contenu de votre portefeuille.

Pour supprimer toute requête tierce, téléchargez les deux bibliothèques dans le dépôt et
remplacez les `src` correspondants dans `index.html` :

```sh
curl -o d3.min.js https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js
curl -o xlsx.full.min.js https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js
```

```html
<script src="d3.min.js"></script>
<script src="xlsx.full.min.js"></script>
```

Les polices se retirent en supprimant la balise `<link>` vers `fonts.googleapis.com` :
la page bascule alors sur les polices système.

## Utilisation

- **Importer un export** accepte le `.xlsx` fourni par Bricks, et le `.csv`. Un nouvel
  import remplace le précédent.
- **Réinitialiser** efface les données de ce navigateur, après confirmation.
- Sur iPhone et iPad, *Partager → Sur l'écran d'accueil* installe l'application en plein
  écran, avec son icône.

Les colonnes de l'export sont reconnues par leur nom, pas par leur position : un
réordonnancement côté Bricks n'empêche pas l'import. Un type d'opération inconnu est
ignoré et signalé nommément.

## Durées et taux annoncés : `projets.json`

L'export Bricks ne contient ni la durée ni le taux promis d'un projet. L'onglet **Projets**
comporte une section où les renseigner, projet par projet. L'application en déduit
l'échéance, le retard éventuel, et l'écart entre le taux annoncé et le rendement réellement
servi.

Ces valeurs se lisent à deux endroits, dans cet ordre :

1. **`projets.json`**, à la racine du dépôt — facultatif. S'il existe, il sert de base
   commune à tous vos appareils.
2. **La saisie faite dans l'application**, conservée dans le navigateur, qui complète ou
   corrige le fichier.

Le bouton *Copier le fichier projets.json* produit le contenu à déposer dans le dépôt :

```json
{
  "Commerce Chaptal": { "duree": 12, "taux": 10 },
  "Promotion Fleurs de Cactus Cayenne": { "duree": 24, "taux": 11 }
}
```

`duree` est en mois, `taux` en pourcentage annuel brut. La clé est le nom du projet **tel
qu'il figure dans l'export**, au caractère près. Une fois le fichier commité, la saisie
locale devient superflue : vous pouvez l'effacer en vidant les données du site.

## Ce que l'application calcule

Capital placé, capital remboursé, capital investi restant, revenus bruts et nets, TRI
annualisé (XIRR) global et par projet, ventilation par type de contrat, retards de
versement mesurés sur le rythme propre à chaque projet, et une projection des revenus
futurs à hypothèses modifiables.

L'onglet **Méthode** détaille chaque calcul et ses limites. À lire avant de s'appuyer sur
les chiffres : l'export ne contient ni taux promis, ni échéance, ni durée de projet.

## Fichiers

```
index.html              l'application entière (structure, styles, calculs)
manifest.webmanifest    installation sur l'écran d'accueil
icon-192.png            icône
icon-512.png            icône
```
