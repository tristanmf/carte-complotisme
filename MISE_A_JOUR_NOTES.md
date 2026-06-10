# Notes de mise à jour des sondages — ComploScore

> Document de travail interne (branche `claude/ecstatic-brahmagupta-7z3qwb`).
> Objectif : rafraîchir les sondages complotistes par pays, ajouter les pays absents,
> réviser les grades A–E si de nouvelles données le justifient.
> **À supprimer une fois la mise à jour terminée.**

## ⚠️ Contrainte rencontrée (juin 2026)
Dans la session web où ce travail a démarré, le pare-feu réseau n'autorisait que `github.com`.
`WebFetch` (lecture des pages sources) renvoyait 403 ; seule la **recherche web** fonctionnait.
→ Décision : **rouvrir le réseau de l'environnement** (politique réseau plus permissive) pour
pouvoir vérifier chaque chiffre à la source. Une fois fait, reprendre ce plan.

## Structure des données (dans `index.html`)
Chaque pays est une entrée d'un objet, clé = code ISO-2 (`"FR"`, `"US"`…). Schéma :
```js
"FR": {
  name, flag, nutriscore: "A"–"E", levelColor,
  scoreMeta:   "...",   // justification du grade + sources clés
  scoreDetail: "...",   // « Pourquoi X ? »
  context:     "...",   // paragraphe contextuel
  surveys: [
    { source, url, year, topic, data: [ { label, val } ] }
  ]
}
```
70 pays actuellement. Couleurs grade : A `#15803d`, B `#4d9e38`, C `#ca8a04`, D `#c2610c`, E `#c0392b`.

## Analyse de fraîcheur (année du sondage le plus récent par pays)

### Prioritaire — pays « minces & périmés » (1 sondage, ancien)
- **Géorgie** (D) — 1 sondage **2017**
- **Kazakhstan** (D) — 1 sondage **2017**
- **Arabie Saoudite** (D) — 1 sondage YouGov **2020**
- **Égypte** (D) — 1 sondage YouGov **2020**
- **Ukraine** (D) — 1 sondage **2021** (données KIIS 2023–2025 à chercher)
- **Kenya** (E) — 1 sondage **2021**
- **Thaïlande** (E) — 1 sondage **2021**

### Bloc « 2022 » à 1 seul sondage (Harvard HKS / YouGov) — à enrichir
Autriche (B), Suisse (B), Norvège (A), Pays-Bas (A), Islande (A), Estonie (C), Lettonie (C),
Lituanie (C), Irlande (C), Israël (B), Bulgarie (E), Croatie (D), Slovénie (D), Serbie (D),
Portugal (D), Macédoine du Nord (E), Russie (E), Inde (E), Guatemala (D), Chili (C).

### Données déjà fraîches (2025) — laisser tel quel sauf nouveauté
États-Unis, Royaume-Uni, Suède, Espagne, Italie, Pologne, République tchèque.

## Pays absents à investiguer (données potentielles via baromètres régionaux)
- **MENA** : Iran, Irak, Jordanie, Liban, Palestine, Émirats AU, Qatar, Koweït, Yémen, Libye, Soudan
- **Afrique** : Ghana, Sénégal, Tanzanie, Ouganda, Éthiopie, Zambie, Zimbabwe, Cameroun, Côte d'Ivoire
- **Amérique latine** : Venezuela, Équateur, Bolivie, Uruguay, Paraguay, Costa Rica, Rép. dominicaine, Panama
- **Asie / Caucase / Asie centrale** : Arménie, Azerbaïdjan, Ouzbékistan, Bangladesh, Sri Lanka, Népal, Myanmar, Cambodge, Mongolie, Moldavie, Biélorussie

## Sources candidates (à vérifier à la source une fois le réseau ouvert)
- **Multi-pays récents** : Special Eurobarometer « science » (n≈37 079, ~2024, données par pays UE) ;
  étude PLOS One « democratic quality as protective factor » (PMC12714214) ;
  ⚠️ l'étude « 20 pays / 26 416 répondants » a un terrain de **2020** (trop ancien).
- **USA 2025** : Change Research « Beyond the Fringe » (août 2025) ; Northeastern 50-états (oct. 2025) ;
  Carsey/UNH « Conspiracy vs Science » POLES 2025 ; PRRI QAnon.
- **MENA** : Arab Barometer Wave 8/9 (2023–2024).
- **Afrique** : Afrobarometer Round 9 (2021–2023) / Round 10 (2024–2025) — items désinformation/confiance.
- **Amérique latine** : Latinobarómetro 2023 & 2024 ; LAPOP/AmericasBarometer.
- **Caucase / Asie centrale** : Caucasus Barometer (CRRC) ; Central Asia Barometer.
- **Ukraine** : KIIS (Kyiv International Institute of Sociology) 2023–2025.
- **Transversal** : YouGov-Cambridge Globalism (maj éventuelle), Ipsos, Wellcome Global Monitor (vaccins).

## Domaines à autoriser dans la politique réseau (pour WebFetch)
```
yougov.com today.yougov.com ipsos.com arabbarometer.org afrobarometer.org
latinobarometro.org europa.eu europarl.europa.eu journals.plos.org
pmc.ncbi.nlm.nih.gov ncbi.nlm.nih.gov nature.com changeresearch.com
news.northeastern.edu carsey.unh.edu prri.org cam.ac.uk wellcome.org
pewresearch.org gallup.com caucasusbarometer.org kiis.com.ua
theconversation.com medrxiv.org
```
(Le plus simple reste une politique « egress complet » : les recherches mènent à des domaines variés.)

## Plan d'exécution (session à réseau ouvert)
1. Mineur d'abord les grandes enquêtes multi-pays (Eurobaromètre, Afrobarometer, Latinobarómetro,
   Arab Barometer) → rafraîchit plusieurs pays d'un coup.
2. Pour chaque pays cible : ouvrir la source, extraire 1–4 points (`label`/`val`), URL + année exactes.
3. Ajouter les `surveys` récents ; mettre à jour `scoreMeta`/`scoreDetail` ; réviser le grade si justifié.
4. Ajouter les pays absents pour lesquels une source fiable existe.
5. Mettre à jour le compteur de pays dans le README et l'intro.
6. QA visuel + commit/push sur la branche.
