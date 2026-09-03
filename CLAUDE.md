# ComploScore — notes de projet pour Claude

Carte mondiale interactive de l'adhésion aux théories du complot, pays par pays, avec un grade A–E inspiré du Nutri-Score. Projet personnel de Tristan (betatristan@gmail.com). Site statique, sans backend. Dernière mise à jour de ce fichier : septembre 2026.

## État du projet

- **73 pays** couverts, chacun avec un grade A–E, un résumé, une justification du grade, un contexte et de 1 à 6 sondages sourcés (URL + année de terrain).
- Répartition des grades : A 8 · B 8 · C 16 · D 22 · E 19.
- Fraîcheur : 34 pays ont une donnée de 2024 ou plus récente ; 5 pays restent sur des données antérieures à 2022 (Kazakhstan 2017, Égypte 2020, Arabie Saoudite 2020, Kenya 2021, Ukraine 2021). Pour ces 5, aucune donnée récente exploitable n'existe : ce n'est pas un oubli.
- Fiabilité (indicateur calculé) : 20 pays solides, 31 moyens, 22 fragiles.
- Branche de travail : `claude/ecstatic-brahmagupta-7z3qwb`. Tout est poussé. `main` n'a pas encore été mis à jour (fusion à faire par Tristan).

## Architecture

Tout tient dans `index.html` (CSS, données, JS). Fichiers annexes :

- `world-data.js` : GeoJSON monde (`WORLD_GEOJSON`), noms de pays en anglais. Serbie = `Republic of Serbia`, Macédoine du Nord = `Macedonia`, Royaume-Uni = `England`, États-Unis = `USA`.
- `README.md` : présentation publique.
- `countries-data.js`, `world-simple.geojson`, `world-110m.json`, `index.backup.html`, `qa-*.png` : résidus de versions antérieures, non utilisés par la page. Ne pas s'y fier.

Dans `index.html`, l'ordre est : styles → HTML (en-tête, carte, panneau latéral, pied de page) → script. Le script contient dans cet ordre : `NS_CONFIG` (couleurs et libellés des grades), `COUNTRIES` (les données), `ISO2_TO_NAME` (code ISO → nom GeoJSON), `CENTROIDS` (position des badges), l'initialisation Leaflet, `showCountry`, le classement (`showRanking`), le filtre par grade, `confidenceOf`, le panneau méthode (`showMethod`), la recherche, le thème.

### Schéma d'un pays

```js
"FR": {
  name: "France", flag: "🇫🇷", nutriscore: "D", levelColor: "#c2610c",
  scoreMeta:   "Justification courte du grade, avec les chiffres clés et les sources. Finit par « Grade X. »",
  scoreDetail: "<strong>Pourquoi X et non Y ?</strong> Explication du cran choisi.",
  context:     "Paragraphe de lecture rapide.",
  surveys: [
    { source, url, year: "2024", topic, data: [{ label, val: 36 }, ...] },
  ]
}
```

Couleurs de grade (`levelColor`) : A `#15803d`, B `#4d9e38`, C `#ca8a04`, D `#c2610c`, E `#c0392b`.

### Conventions de données

- `year` est l'**année du terrain**, pas de publication. Si le terrain est inconnu, année de publication et préciser « publ. » dans `topic`.
- Un sondage par ligne, virgule finale sur chaque ligne (les virgules finales sont valides en JS).
- Les sondages sont **triés du plus récent au plus ancien** dans chaque pays. Après une insertion, relancer le tri (script Python en fin de fichier de ces notes).
- `val: null` est accepté pour une entrée qualitative (affichée « Analyse qualitative », sans barre).
- Pour ajouter un pays : bloc dans `COUNTRIES`, entrée dans `ISO2_TO_NAME` (vérifier le nom exact dans `world-data.js` avec `grep -o '"name":"Xxx"' world-data.js`), entrée dans `CENTROIDS`. Le compteur de pays se calcule tout seul ; mettre à jour les « 73 pays » statiques (repli sans JS) dans `index.html` et `README.md`.

## Décisions prises

### Sourcing

- Sources primaires uniquement : instituts (Ipsos, YouGov, IFOP, VCIOM…), baromètres officiels (Eurobaromètre, ESS, Arab Barometer…), études académiques revues. Pas de presse sans lien vers l'étude, pas de chiffre inventé.
- Chaque chiffre doit être lisible dans la source ou dans un extrait vérifiable. Faute de vérification possible, on n'intègre pas.
- Les items « frontière » (« grand remplacement », « élections truquées ») sont conservés, mais explicitement traités comme moins probants qu'une théorie « dure » (groupe secret, vaccins, climat, virus de laboratoire).
- Les enquêtes à échantillon urbain (Ipsos le signale pour 13 pays) portent la mention « échant. + urbain » dans `topic`.
- Les baromètres régionaux (Afrobarometer, Latinobarómetro, Caucasus Barometer, Arab Opinion Index) mesurent la confiance ou la géopolitique, pas l'adhésion complotiste : ils ne servent pas à fabriquer un grade.

### Grades

- Le grade est une **synthèse éditoriale**, pas une mesure. Le panneau « Comment lire les grades » l'assume publiquement et donne l'échelle en ordre de grandeur : A < 15 %, B 15–25 %, C 25–35 %, D 35–50 %, E > 50 % sur une théorie centrale, ou tête de classement international.
- Un grade se justifie par la source la plus récente et la plus robuste, et chaque fiche explique pourquoi pas un cran au-dessus ou en dessous.
- Cohérence entre voisins : un pays encadré par deux pays E sur la même enquête ne reste pas en D (c'est ce qui a relevé Colombie, Chypre, Grèce, Croatie).
- Un grade ne repose pas sur un contexte politique sans mesure : la Russie est repassée de E à D pour cette raison (VCIOM 2025 : 40 %).
- Un pays absent n'est pas « sans complotisme », il est sans sondage exploitable. On n'ajoute un pays qu'avec une source, même fragile, à condition que la fiche le dise (Jordanie, Liban).

Révisions de grades effectuées en septembre 2026 : Hongrie, Pérou, Colombie, Chypre, Grèce, Croatie → E ; Japon A→B ; Suisse B→C ; Russie E→D ; Espagne C→D.

### Interface

- Indicateur de fiabilité par pays, calculé par `confidenceOf` : solide si ≥ 3 sources dont une de 2023+ et ≥ 4 données ; moyenne si ≥ 2 sources dont une de 2022+, ou ≥ 3 sources ; fragile sinon. Affiché dans la fiche et comme point coloré dans le classement.
- Badge « ⚠ Dernière vague AAAA » quand la donnée la plus récente est antérieure à 2022.
- Classement des pays par grade (E → A), accessible depuis l'accueil et le bouton « Voir le classement » ; clic sur un grade de la légende = filtre.
- Panneau « Comment lire les grades », accessible depuis l'intro, l'accueil, chaque fiche et le pied de page.
- Mobile : carte et panneau empilés sans chevauchement ; carte bornée au monde (pas de zone noire) ; infobulles de survol désactivées sur écran tactile.
- La mention « Created with Perplexity Computer » a été retirée partout : c'est le projet de Tristan.

## Ce qui reste à faire

Par ordre d'intérêt :

1. **Fusionner la branche dans `main`** et vérifier le rendu en production, surtout sur mobile (aucune capture n'a pu être prise depuis l'environnement Claude, qui bloque Leaflet et les polices).
2. **Enrichir les pays « minces » d'Europe** avec le jeu de données brut du Special Eurobarometer 557 (terrain 2024) : Autriche, pays baltes, Portugal, Slovaquie, Irlande, Malte, Roumanie, Serbie, Macédoine du Nord. Les valeurs par pays sont dans le rapport PDF (europa.eu/eurobarometer/surveys/detail/3227) ; seules 8 ont pu être récupérées par la presse.
3. **Sources nationales attendues** : Arab Barometer Wave IX (terrain 2025, publication en cours) pour Égypte, Jordanie, Liban, Maroc, Tunisie, Irak ; IFOP/Jean-Jaurès si une vague post-2022 sort ; CeMAS/FES Mitte-Studie 2024/25 (chiffres complotisme non repérés en ligne).
4. **Afrique** : aucune enquête publiant un pourcentage d'adhésion exploitable n'a été trouvée pour Ghana, Éthiopie, Ouganda, Tanzanie, Zambie, Malawi, Zimbabwe, Sénégal, Côte d'Ivoire, Cameroun, Mali, Burkina. Pistes : rapports Africa CDC par pays, PLOS Global Public Health (Ghana 2024, Togo), Afrobarometer si un item complotiste apparaît.
5. **Idée ouverte : un globe** au lieu de la carte plate (Globe.gl ou D3 orthographique). Réécriture de la couche carte seulement ; à faire sur une branche séparée si Tristan valide.
6. Nettoyage : supprimer les fichiers résiduels non utilisés (`countries-data.js`, `world-simple.geojson`, `world-110m.json`, `index.backup.html`, `qa-*.png`) après vérification.

## Comment travailler sur ce projet

### Vérifier avant de commiter

Syntaxe et test d'exécution simulé (Node est disponible) :

```bash
python3 - << 'EOF'
import re
html=open("index.html",encoding="utf-8").read()
s=re.findall(r'<script(?![^>]*src=)[^>]*>(.*?)</script>', html, flags=re.S)[-1]
open("/tmp/inline.js","w",encoding="utf-8").write(s)
EOF
node --check /tmp/inline.js && echo OK
```

Pour un test plus complet (fiches, classement, méthode), il faut un stub de `document` et de `L` ; voir l'historique de la session de septembre 2026 ou réécrire un stub minimal.

### Retrier les sondages après une insertion

```bash
python3 - << 'EOF'
import re
p="index.html"; lines=open(p,encoding="utf-8").read().split("\n"); out=[]; i=0; n=len(lines)
while i<n:
    l=lines[i]; out.append(l)
    if l.strip()=="surveys: [":
        block=[]; j=i+1
        while j<n and lines[j].strip()!="]": block.append(lines[j]); j+=1
        y=lambda s:int((re.search(r'year: "(\d{4})"',s) or [0,0])[1])
        for k in sorted(range(len(block)), key=lambda k:(-y(block[k]),k)):
            b=block[k].rstrip().rstrip(","); out.append(b+",")
        out.append(lines[j]); i=j+1; continue
    i+=1
open(p,"w",encoding="utf-8").write("\n".join(out))
EOF
```

### Pousser

Dans l'environnement Claude Code web, le proxy Git authentifié peut disparaître en cours de session (`could not read Username for 'https://github.com'`). Dans ce cas : commiter localement, envoyer `index.html` (et les autres fichiers modifiés) à Tristan avec `SendUserFile`, et lui donner les commandes à coller sur son Mac :

```bash
cp ~/Downloads/index.html ~/carte-complotisme/
cd ~/carte-complotisme
git add -A
git commit -m "message"
git push
```

Puis vérifier depuis l'environnement Claude que le distant correspond (`git fetch` fonctionne sans authentification, le dépôt est public) et réaligner : `git reset --hard origin/<branche>`.

### Réseau

L'environnement ne peut lire que github.com ; `WebFetch` est bloqué pour tout autre domaine, `WebSearch` fonctionne. La vérification à la source passe donc par les extraits de recherche, ou par un outil externe (Cowork) auquel on donne un prompt de sourcing strict. Le format de sortie qui s'intègre le mieux : un bloc par pays avec source, URL, année de terrain, données « libellé : valeur », note.
