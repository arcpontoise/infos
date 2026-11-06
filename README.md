# Site d'informations — Compagnie d'archers de Pontoise

Génération du planning hebdomadaire des créneaux à partir d'une source unique
en YAML, publiée sur GitHub Pages à chaque poussée sur `main`.

## Principe

- `data/planning.yaml` et `data/tarifs.yaml` sont les **seules sources
  de vérité**. Aucun total tarifaire n'y est saisi : `build.py` les
  calcule (en centimes) à partir du détail des parts et des remises,
  ce qui rend les erreurs d'addition impossibles. Le document transmis
  chaque saison par le capitaine est transcrit ici, puis tout le reste est
  généré.
- `scripts/build.py` valide la structure (jours, horaires, publics) puis génère
  deux pages dans `dist/` depuis la même source :
  - `index.html` (`templates/liste.html.j2`) : vue liste par jour, adaptée aux
    téléphones ;
  - `paysage.html` (`templates/paysage.html.j2`) : grille horaire de la
    semaine, jours en colonnes, pour grand écran et impression. Les créneaux
    simultanés d'une même journée sont automatiquement placés côte à côte.
- Le workflow `.github/workflows/deploy.yml` lint (`yamllint`, `ruff`),
  construit et déploie sur GitHub Pages.

## Charte graphique

Couleurs, polices, blason et styles communs proviennent de la
[charte de la compagnie](https://github.com/arcpontoise/charte), chargée
directement depuis <https://charte.arcpontoise.fr/> par chaque page et par les PDF
générés. Les gabarits ne contiennent que les styles propres à ce site et
utilisent les variables de la charte (`--ink`, `--blue`, `--accent`…).

L'en-tête affiche le menu commun à tous les sites de la compagnie
(`<arc-menu actuel="inscrire">` dans `templates/base.html.j2`), chargé lui aussi
en direct depuis la charte : il se modifie dans `menu.yaml` du dépôt `charte`,
sans rebuild de ce site. La barre `nav.site` ne contient que les pages propres
à ce site. Le menu est masqué à l'impression et n'apparaît pas dans les PDF.

Pour tester une modification de la charte sur les pages web, servir la charte
en local et la désigner avec `CHARTE_URL` :

```bash
(cd ~/git/arcpontoise_charte && python3 -m http.server 8001) &
CHARTE_URL=http://localhost:8001 python scripts/build.py
(cd dist && python3 -m http.server 8002)
```

Le serveur de test de Python n'envoie pas d'en-tête CORS : la police Caveat
n'est donc pas chargée dans ce cas, contrairement à GitHub Pages.

Les PDF figent la charte au moment du build. Pour qu'ils restent à jour :

- chaque modification de la charte relance automatiquement le workflow de ce
  dépôt (`workflow_dispatch` déclenché par le dépôt `charte`, avec le commit de
  la charte en paramètre `charte_ref`) ;
- le workflow clone la charte à ce commit dans `_charte/` et `build.py` sert
  les URL `https://charte.arcpontoise.fr/…` depuis cette copie
  (`CHARTE_LOCAL`) pendant la génération des PDF. Le PDF correspond ainsi
  exactement au commit de la charte, sans dépendre du cache de GitHub Pages.
  Le HTML publié garde les URL normales.

En local, sans `CHARTE_LOCAL`, les PDF utilisent la charte en ligne. Pour
reproduire le build de la CI ou tester une modification de la charte :

```bash
CHARTE_LOCAL=$HOME/git/arcpontoise_charte python scripts/build.py
```

Un rebuild manuel avec une version précise de la charte se lance depuis
l'onglet *Actions* (« Run workflow », champ `charte_ref`).

## Mise à jour d'une saison

1. Créer une branche.
2. Modifier `data/planning.yaml` (créneaux, encadrants, tir libre).
3. Vérifier en local :

   ```bash
   pip install -r requirements.txt
   python scripts/build.py --check
   python scripts/build.py
   ```

4. Ouvrir `dist/index.html` dans un navigateur pour contrôle visuel.
5. Fusionner : le déploiement est automatique.

## Structure des données

Chaque créneau comporte : `jour`, `debut`, `fin` (format `HH:MM`), `intitule`,
`public` (`jeunes`, `adultes` ou `tous`), `encadrants` (liste, éventuellement
vide) et un `detail` facultatif. Le tir libre est décrit par une phrase par
jour dans la table `tir_libre`.

Toute entrée invalide fait échouer la construction : le planning publié ne peut
pas être structurellement incohérent.

## Impression

La vue paysage comporte un mode impression A4 paysage : `Ctrl+P` depuis
`paysage.html` produit la version à afficher en salle d'armes.

## Activation de GitHub Pages

Dans les réglages du dépôt : *Settings → Pages → Build and deployment*, choisir
**GitHub Actions** comme source. Le premier déploiement crée l'environnement
`github-pages`.
