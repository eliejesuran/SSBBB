# SSBBB — Fiche Technique

Application web single-file pour créer et exporter des fiches techniques sonores. Conçue pour le groupe **Super Sexy Big Boys Band**.

## Architecture

Un seul fichier : `index.html` (~1700 lignes). HTML + CSS + JS inline — aucun build, aucune dépendance locale.

> **Pourquoi pas de serveur ?** Une fiche technique est créée par une personne puis remise (en PDF) à l'ingé son. La collaboration temps réel (comme le projet *setlist*) serait surdimensionnée. Le partage se fait donc **sans serveur** : lien autoportant (état encodé dans l'URL), fichier `.html`/`.json` réouvrable, et autosave `localStorage`. Hébergement statique (GitHub Pages) suffit. Le pattern serveur WebSocket reste disponible dans `../setlist/server/` si la collaboration live devient nécessaire un jour.

**Dépendances CDN :**
- [jsPDF 2.5.1](https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js) — génération PDF
- [html2canvas 1.4.1](https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js) — rendu HTML→image pour le PDF
- Google Fonts : Bebas Neue, Space Mono, Inter

## Structure du code (`index.html`)

| Zone | Lignes | Rôle |
|---|---|---|
| CSS | 14–~335 | Styles complets, responsive, `html[data-theme="sepia"]` (mode clair), bouton bascule, media queries |
| HTML | ~338–440 | `#btn-theme-toggle` (haut droite), header, toolbar (7 boutons), tableau, palette, modal, `#file-input` |
| DATA | ~455 | `musicians[]`, `headerBadges[]`, `customBadges[]`, `LABELS` (couleurs des badges intégrés : classes CSS `.kick`/`.oh`… surchargées en sépia) |
| BADGE HELPERS | ~487 | `allTypes`, `labelOf`, `customColors`, `applyBadgeStyle` (classe CSS si intégré, style inline si perso) |
| HEADER BADGES | ~500 | Badges dynamiques en-tête (nombre musiciens, infos scène) |
| TABLE | ~570 | Rendu du tableau musiciens ; `renderTable()` appelle `saveToStorage()` en fin ; `moveRow()` = réordonnancement des lignes par glisser-déposer (poignée `.row-grip`) |
| PALETTE / MODAL | ~620 | Palette glissable + création badges perso (`addCustomBadge`/`promptNewBadge`/`deleteCustomBadge`) + bottom sheet mobile |
| THEME | ~812 | `applyMode`/`initTheme`/`systemMode` : sépia ↔ sombre, suit l'appareil, persiste `ssbbb_theme` |
| PDF EXPORT | ~865 | Pagination auto, rendu html2canvas, export jsPDF, fidèle au thème (`pdfBadgeColors` lit `getComputedStyle` d'un badge rendu), cadre `notesBlockHtml` placé sous la dernière page ou sur une page dédiée |
| SAVE HTML | ~1360 | Snapshot `.html` (état `let musicians`/inputs + blob `__ficheData` base64) |
| PORTABILITÉ | ~1430 | `captureState`, `loadData`, `applyState`, `encodeState`/`decodeState`, `newFiche`, `shareLink`, `importFile`, `exportJson` |
| INIT | ~1585 | `boot()` : `initTheme()` puis priorité lien `#d=` → localStorage → données d'exemple |

## Fonctionnalités

- **Tableau musiciens** : nom éditable, badges micro/DI par glisser-déposer (desktop) ou modal (mobile), colonne commentaires, réordonnancement drag & drop des lignes
- **Badges disponibles** : Kick, Overhead, DI, Ampli repiqué, SM58, XLR, Jack, ⚠ Sans phantom
- **Badges personnalisés** : bouton « + badge » dans la palette (ou « + badge perso » dans le modal mobile) → nom + couleur ; réutilisables, supprimables (×), intégrés au PDF / partage / sauvegarde (`customBadges[]`, types `c1`, `c2`…)
- **Badges d'en-tête** : informations scène (nombre de musiciens, dimensions, etc.), éditables et supprimables
- **Thème** : deux modes seulement — **sépia (clair)** et **sombre** — bascule via le bouton ☀️/🌙 en haut à droite. Suit `prefers-color-scheme` de l'appareil tant que l'utilisateur n'a pas choisi explicitement (choix mémorisé dans `localStorage` `ssbbb_theme`). Le thème est une **préférence d'appareil, pas une donnée de la fiche** (non inclus dans le partage / l'export). Implémenté via `html[data-theme="sepia"]` qui surcharge les variables CSS.
- **Notes générales** : cadre de texte libre (`#general-notes`) sous le tableau, imprimé sur le PDF (sous la dernière page si la place le permet, sinon page dédiée). Inclus dans l'état (`notes`) → partagé / exporté / sauvegardé comme le reste
- **Export PDF** : pagination automatique A4, **fidèle au thème actif** — fond/texte via les variables CSS (`--black`/`--white`) *et* badges via les couleurs réellement calculées du thème (`pdfBadgeColors`, donc suit les surcharges sépia) ; cadre de notes inclus ; nom de fichier avec l'année
- **Nouvelle fiche** (📄) : repart d'une fiche vierge (1 ligne, champs vides) — pour qu'un nouvel utilisateur crée la sienne sans effacer 12 lignes à la main
- **Partager** (🔗) : copie un lien autoportant (`#d=` + état base64 Unicode-safe) ; utilise `navigator.share` sur mobile. Aucun serveur — le lien contient toute la fiche
- **Ouvrir / importer** (📂) : recharge une fiche depuis un `.json` ou un `.html` sauvegardé (blob `__ficheData`, sinon parsing legacy `let musicians` + valeurs d'inputs via `DOMParser`) → pour modifier/transformer une fiche existante
- **Sauvegarde HTML** (💾) : snapshot complet avec données embarquées (blob `__ficheData`), réouvrable dans le navigateur
- **Export JSON** (`{ }`) : données seules (`captureState()`), légères à versionner/partager

## Lancer l'application

```bash
open index.html
# ou glisser index.html dans un navigateur
```

Aucun serveur requis. Fonctionne hors-ligne une fois ouvert (les polices Google nécessitent une connexion au premier chargement).

---

## Bugs connus & TODO

### Bugs résolus

- [x] **HTML invalide : `<meta>` avant `<html>`** — balises meta déplacées dans `<head>`
- [x] **Grain slider non fonctionnel** — `body::before` utilise désormais `opacity: var(--grain-val, 0.4)`
- [x] **Glow toujours en couleur or fixe** — la couleur est lue depuis `--gold` au moment de l'application
- [x] **Sauvegarde HTML : scripts `__savedTheme` cumulés** — remplacement regex si déjà présent
- [x] **Nom de fichier PDF/HTML codé en dur "SSBBB"** — `bandName` sanitizé utilisé pour les deux exports
- [x] **Badges dupliqués non bloqués** — vérification `includes()` dans le drop handler et le modal
- [x] **Variable CSS `--accent` inutilisée** — supprimée, remplacée par `--grain-val: 0.4`
- [x] **Aucune persistance localStorage** — autosave à chaque mutation (musiciens, badges, champs texte)
- [x] **`renderTable` en récursion infinie** (`Maximum call stack size exceeded`) — le wrapper d'autosave redéclarait `function renderTable` ; par hoisting, `const _origRenderTable = renderTable` capturait le wrapper lui-même → la table ne s'affichait plus du tout. Corrigé : `saveToStorage()` appelé directement en fin du vrai `renderTable`, wrapper supprimé.
- [x] **Réutilisable par tout le monde** — bouton « Nouvelle fiche », partage par lien autoportant, import `.json`/`.html`, export JSON (voir Fonctionnalités)
- [x] **Thème simplifié** — panneau (5 presets, pickers, polices, grain/glow) remplacé par 2 modes sépia/sombre suivant l'appareil, bascule ☀️/🌙 en haut à droite

### Améliorations à prévoir

- [x] ~~Ajouter un type de badge personnalisé (champ libre + couleur)~~ — fait (bouton « + badge » de la palette)
- [x] ~~Réordonner les lignes du tableau par glisser-déposer~~ — fait : poignée `⠿` (`.row-grip`) dans la colonne `#`, `moveRow(fromId, toId)` réordonne `musicians[]` (vers le bas = après la cible, vers le haut = avant) ; la ligne gère à la fois le dépôt de badge (`dragType`) et le réordonnancement (`dragRowId`)
- [x] ~~Export JSON des données seules~~ — fait (bouton `{ }`)
- [x] ~~État dans l'URL (Option 3)~~ — fait (bouton 🔗 Partager, état encodé dans `#d=`)
- [ ] Bouton "Dupliquer" une fiche pour créer des variantes (acoustique, électrique, etc.)

---

## Partage de la fiche — Options recommandées

### Option 1 — GitHub Pages (recommandée pour ce projet)
Le dépôt est déjà sur GitHub. Activer GitHub Pages sur la branche `main` rend la fiche accessible à une URL publique permanente, sans infrastructure supplémentaire.

```
Settings → Pages → Source: main / root → Save
→ https://<user>.github.io/SSBBB/
```

Chaque `git push` met à jour la fiche en ligne. Idéal pour partager avec un ingénieur du son avant un concert : un simple lien URL.

### Option 2 — Partage du fichier HTML sauvegardé
Cliquer "Sauvegarder HTML" génère un `.html` autonome avec toutes les données embarquées. Ce fichier peut être :
- Envoyé par email ou WeTransfer
- Partagé via Google Drive / Dropbox (prévisualisation dans le navigateur)
- Ouvert sur n'importe quel appareil sans installation

### Option 3 — État dans l'URL ✅ (implémenté)
Le bouton **🔗 Partager** encode tout l'état (`captureState()` → JSON → base64 Unicode-safe) dans le hash `#d=` de l'URL et le copie dans le presse-papier (ou `navigator.share` sur mobile). N'importe qui ouvrant ce lien voit la fiche préconfigurée, peut l'éditer et la ré-exporter — sans serveur ni commit. À l'ouverture, `boot()` charge le hash en priorité, nettoie l'URL (`history.replaceState`) puis persiste en localStorage. Idéal pour des fiches éphémères (guest, date unique). Taille typique du lien : ~2 Ko.

### Option 4 — Hébergement instantané
Glisser `index.html` (ou le HTML sauvegardé) sur [netlify.com/drop](https://netlify.com/drop) génère une URL publique en quelques secondes, sans compte requis.
