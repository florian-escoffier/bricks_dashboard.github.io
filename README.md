# Portefeuille Bricks

Suivi de portefeuille pour la plateforme Bricks.co, à partir de l'export des transactions
du wallet. Page unique, sans serveur : **le fichier importé est lu par le navigateur et
n'est transmis nulle part.**

## Mise en ligne sur GitHub Pages

### Par l'interface web de GitHub

1. Créez un dépôt, par exemple `portefeuille-bricks`.
2. **Add file → Upload files**, déposez les cinq fichiers de ce dossier
   (`index.html`, `projets.json`, `manifest.webmanifest`, `icon-192.png`, `icon-512.png`),
   puis validez.
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
- **Exporter**, dans la section *Tous les projets*, met le tableau au format A4 paysage et
  ouvre la fenêtre d'impression : choisissez-y *Enregistrer au format PDF*. Le document
  reprend les quatorze colonnes, les lignes telles que vos filtres et votre tri les
  laissent, une ligne de totaux, et rappelle en tête la date de l'export, le nombre de
  projets et le filtre en vigueur. `Cmd+P` donne exactement le même document.

  La découpe en pages est faite par l'application, pas par le navigateur : chacun répète
  l'en-tête d'un tableau et respecte les coupures de ligne à sa façon, quand il les
  respecte. Chaque page est donc un tableau complet, avec son propre en-tête et ses
  colonnes de largeur figée pour qu'elles se superposent d'une page à l'autre ; aucune
  ligne ne peut être coupée en deux puisqu'aucune page ne déborde.

  Le document est dessiné sur une surface de 265 × 165 mm, plus petite qu'une A4 paysage.
  Safari ignore la taille et les marges demandées et impose les siennes — environ 14 mm
  sur les côtés, 19 mm en haut —, ce qui ne laisse que 269 × 171 mm. Viser cette page-là
  plutôt que la pleine A4 laisse un peu de blanc sur les navigateurs plus généreux, mais
  le document tient partout. Si vous élargissez les marges dans la fenêtre d'impression,
  les pages se remplissent moins, sans rien perdre.
- Sur iPhone et iPad, *Partager → Sur l'écran d'accueil* installe l'application en plein
  écran, avec son icône.

Les colonnes de l'export sont reconnues par leur nom, pas par leur position : un
réordonnancement côté Bricks n'empêche pas l'import. Un type d'opération inconnu est
ignoré et signalé nommément.

## Durées et taux annoncés : `projets.json`

L'export Bricks ne contient ni la durée ni le taux promis d'un projet. Ces deux
informations viennent donc d'un fichier tenu à part, `projets.json`, placé à la racine du
dépôt à côté d'`index.html`. L'application le lit au chargement et remplit les colonnes
**Durée** et **Taux annoncé** du tableau *Tous les projets*.

Deux colonnes en sont déduites :

| | |
|---|---|
| **Revenus attendus** | capital placé × taux annoncé × durée. Le montant promis sur toute la vie du projet, à comparer à la colonne *Revenus*, qui donne l'encaissé réel. Un écart est normal tant que le projet court. |
| **Horizon** | les mois restants jusqu'à l'échéance, prise comme la date du premier achat plus la durée annoncée. Négatif, en rouge, quand cette échéance est passée sans que le capital soit revenu ; vide sur un projet soldé. |

Cette échéance est déduite, pas officielle : Bricks ne la communique pas dans l'export.

**Le fichier fourni dans ce dossier couvre déjà les 168 projets** de l'export : durées et
taux annoncés ont été repris du tableur `Bricks.numbers`. Déposez-le dans le dépôt en même
temps que le reste.

Le format est un objet, une entrée par projet :

```json
{
  "Commerce Chaptal": { "duree": 12, "taux": 7.72 },
  "Promotion Fleurs de Cactus Cayenne": { "duree": 24, "taux": 11 }
}
```

`duree` est en mois, `taux` en pourcentage annuel brut. La clé est le nom du projet **tel
qu'il figure dans l'export**, au caractère près : un nom qui ne correspond pas exactement
ne sera pas reconnu.

Un projet absent du fichier, ou dont une des deux valeurs manque, affiche **données
manquantes** dans la colonne concernée — jamais un zéro, qui se confondrait avec une vraie
valeur. Les colonnes *Revenus attendus* et *Horizon*, qui en dépendent, affichent alors un
simple tiret : le tiret signale une valeur sans objet ou incalculable, *données manquantes*
un trou dans le fichier. Taper `données manquantes` dans le filtre du tableau les rassemble. Pour les
compléter, ajoutez l'entrée au fichier dans le dépôt : c'est le seul endroit à modifier,
et la correction vaut pour tous vos appareils.

Si le fichier est absent, l'application fonctionne normalement : les deux colonnes sont
alors entièrement marquées *données manquantes*, et rien d'autre ne change.

## Remboursements

L'onglet **Vue d'ensemble** comporte une section qui récapitule le capital rendu et celui
à venir : nombre de projets totalement et partiellement remboursés, montant déjà rendu, puis
le capital attendu d'ici 3, 6, 12 et 24 mois.

Ces horizons sont **cumulatifs** — « d'ici 12 mois » comprend les six premiers — et
incluent le capital dont l'échéance est déjà passée, lequel est aussi donné sur sa propre
ligne. Le montant d'une ligne est l'encours des projets concernés, c'est-à-dire ce qu'il
leur reste à rendre, et non le capital placé au départ.

L'échéance d'un projet est déduite de son premier achat et de la durée lue dans
`projets.json` ; Bricks ne la communique pas. Un projet sans durée connue n'a donc pas
d'échéance : il est signalé à part sous le tableau plutôt que rangé arbitrairement, avec le
montant qu'il représente. La note finale indique aussi le capital qui revient au-delà de
24 mois, de sorte que l'ensemble des lignes et de la note couvre tout l'encours.

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
projets.json            durées et taux annoncés, 168 projets
manifest.webmanifest    installation sur l'écran d'accueil
icon-192.png            icône
icon-512.png            icône
```
