# ComploScore 🗺️

**Carte mondiale interactive des théories du complot**  
Un projet de [Tristan](https://tristan.pro)

ComploScore est une carte interactive qui visualise les niveaux d'adhésion aux théories complotistes dans le monde, à partir des sondages et études académiques les plus récents.

## Fonctionnalités

- Carte mondiale cliquable avec 71 pays couverts
- Système de notation A→E inspiré du Nutri-Score
- Fiches détaillées par pays avec sources académiques citées
- Mode clair / mode sombre
- Tooltips interactifs sur les scores

## Sources

Les données proviennent de sources académiques et institutionnelles : IFOP, Ipsos, YouGov-Cambridge Globalism Project, ESS Round 10, Harvard HKS, PONARS, USIP, Nature, Arab Barometer, Afrobarometer, et d'autres.

## Stack technique

- HTML/CSS/JS vanilla (aucune dépendance backend)
- [Leaflet.js](https://leafletjs.com/) pour la carte interactive
- GeoJSON inline pour les polygones pays

## Live

→ [comploscore.tristan.pro](https://tristan.pro)

## Lire les grades

Le grade A–E est une **synthèse éditoriale**, pas une mesure unique : les sondages diffèrent par leur question, leur échantillon et leur année. Chaque fiche pays affiche un indicateur de fiabilité (fragile / moyenne / solide) selon le nombre de sources, leur fraîcheur et le nombre de données comparables. Le panneau « Comment lire les grades ? » détaille la méthode et ses limites.
