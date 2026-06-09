# Projet Géomatique — Analyse Accessibilité Réseau RTC

## Contexte
Projet de démonstration géomatique préparé pour une entrevue au **Réseau de transport de la Capitale (RTC)** de Québec. L'objectif est de visualiser l'accessibilité piétonne du réseau de bus sur le territoire de la ville.

---

## Structure du projet

```
rtc_project/
├── .claude/
│   └── context.md          ← ce fichier
├── site/                   ← site web Leaflet (à compléter)
│   ├── index.html
│   ├── css/
│   ├── js/
│   └── data/               ← fichiers GeoJSON/JS des couches
│       ├── accessibilite_reseau_15m_1.js   ← isochrones 15 min à pied
│       ├── Parcours_2.js                   ← lignes de bus RTC (133 lignes)
│       ├── Arrets_4.js                     ← arrêts de bus (4 204 arrêts)
│       └── aVelo_3.js                      ← stations À Vélo (vide pour l'instant)
└── README.md
```

---

## Données

### Sources
| Jeu de données | Source | Format | Notes |
|---|---|---|---|
| Arrêts de bus | RTC — flux GTFS (`stops.txt`) | CSV → GeoJSON | 4 204 arrêts, EPSG:4326 |
| Parcours de bus | RTC | GeoJSON | 133 parcours, types Régulier/Express |
| Isochrones 15 min | Calculées via ORS local | GeoJSON/Polygone | Mode `foot-walking` |
| Données de recensement | StatCan 2021 | Jointure spatiale | Champ `TOTAL_POP` dans isochrones |

### Propriétés clés par couche

**Isochrones (`accessibilite_reseau_15m_1`)**
- `stop_id` — identifiant de l'arrêt RTC
- `AA_MINS` — durée (15 min)
- `AA_MODE` — mode de transport (`foot-walking`)
- `TOTAL_POP` — population totale dans la zone (recensement 2021, peut être null)
- `CENTER_LON` / `CENTER_LAT` — coordonnées du centroïde

**Arrêts (`Arrets_4`)**
- `ARRET` — numéro d'arrêt
- `ACCESSIBLE` — accessibilité PMR (`Oui` / `Non`)
- `ABRIBUS` — présence d'un abribus
- `AFFICHES` — code d'affichage

**Parcours (`Parcours_2`)**
- `Parcours` — ID numérique
- `Nom` — numéro de ligne
- `Type` — `Régulier` ou `Express`

---

## Infrastructure technique

### ORS Local (Docker)
Un serveur OpenRouteService tourne localement pour calculer les isochrones sans quota API.

```powershell
# Démarrer ORS
docker start ors-app

# Vérifier que c'est prêt
# http://localhost:8080/ors/v2/health  →  {"status":"ready"}
```

**Configuration ORS :**
- Port : `8080` (interne `8082`)
- Fichier OSM : `quebec-260607.osm.pbf` (1.15 GB, Province de Québec)
- Profils actifs : `foot-walking`, `driving-car`
- Graphes buildés depuis le Québec complet (~8 min de build)

> ⚠️ Le port 8000 est utilisé par WebODM. Utiliser `8081` ou autre pour le serveur de dev.

### Serveur de développement
```powershell
cd C:\chemin\vers\rtc_project\site
python -m http.server 8081
# → http://localhost:8081
```

### Stack technique
- **QGIS 3.38.1-Grenoble** — analyse spatiale, export qgis2web
- **OpenRouteService 9.9.0** — calcul d'isochrones local (Docker)
- **Leaflet.js** — carte interactive web
- **qgis2web** — export initial de la carte
- **Docker Desktop** — hébergement ORS

---

## Travail réalisé

### Analyse QGIS
- [x] Import des données GTFS RTC (`stops.txt`)
- [x] Calcul d'isochrones 15 min à pied depuis chaque arrêt via ORS local
- [x] Jointure spatiale avec données de recensement 2021 (`population 2021`)
- [x] Pondération par superficie (densité pop/km² via calculatrice de champs : `"population_2021" / ($area / 1000000)`)
- [x] Export qgis2web → Leaflet

### Site web
- [x] Version initiale générée (structure + panel latéral + toggles de couches)
- [ ] **À faire : structure multi-pages** (page d'accueil + carte)
- [ ] Serveur local fonctionnel (tuiles bloquées en `file://`)
- [ ] Améliorer l'UI/UX

---

## Problèmes connus / À faire

### Site web
- Les tuiles et données ne s'affichent pas en `file://` → nécessite un serveur HTTP local (`python -m http.server 8081`)
- Structure UI à revoir : trop dense sur une seule page
- Options envisagées :
  - Page d'accueil + carte séparée
  - Navbar/onglets (Carte / Méthodologie / Outils)
  - Carte plein écran + modal méthodologie

### Données
- `aVelo_3.js` est vide (aucune station À Vélo dans le jeu de données exporté)
- `TOTAL_POP` est `null` pour plusieurs isochrones (jointure partielle)

---

## Commandes utiles

```powershell
# Vérifier containers Docker actifs
docker ps

# Démarrer ORS
docker start ors-app

# Logs ORS (vérifier que le graphe Québec est chargé)
docker logs --tail 20 ors-app

# Tester ORS dans le navigateur
# http://localhost:8080/ors/v2/health

# Lancer serveur de dev site
python -m http.server 8081
```

---

## Notes pour l'entrevue RTC

- Maîtrise de la chaîne complète : données GTFS → analyse spatiale → visualisation web
- Utilisation d'outils open source sans dépendance à des APIs payantes (ORS local)
- Concepts géomatiques démontrés : isochrones, densité de population, jointure spatiale, SCR
- Stack moderne : QGIS + Docker + Leaflet

