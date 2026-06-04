# SSBBB — Fiche Technique

Application web single-file pour créer et exporter des fiches techniques sonores. Conçue pour le groupe **Super Sexy Big Boys Band**.

## Architecture

Un seul fichier : `index.html` (~1450 lignes). HTML + CSS + JS inline — aucun build, aucune dépendance locale.

**Dépendances CDN :**
- [jsPDF 2.5.1](https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js) — génération PDF
- [html2canvas 1.4.1](https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js) — rendu HTML→image pour le PDF
- Google Fonts : Bebas Neue, Space Mono, Inter

## Structure du code (`index.html`)

| Zone | Lignes | Rôle |
|---|---|---|
| CSS | 14–377 | Styles complets, responsive, animations, dark mode, media queries |
| HTML | 379–516 | Structure : header, toolbar, tableau, palette, modals |
| DATA | 522–563 | `musicians[]`, `headerBadges[]`, `LABELS`, `BADGE_COLORS` |
| HEADER BADGES | 570–628 | Badges dynamiques en-tête (nombre musiciens, infos scène) |
| TABLE | 630–688 | Rendu du tableau musiciens avec drag & drop |
| PALETTE | 691–700 | Palette de badges glissables (desktop uniquement) |
| MODAL | 703–740 | Bottom sheet mobile pour ajouter/retirer badges |
| THEME ENGINE | 762–922 | Presets couleur, sliders grain/glow, pickers CSS |
| PDF EXPORT | 928–1410 | Pagination auto, rendu html2canvas, export jsPDF |
| SAVE HTML | 1414–1441 | Snapshot de l'état courant dans un `.html` téléchargeable |
| INIT | 1447–1450 | Bootstrap : thème sauvegardé, rendu initial |

## Fonctionnalités

- **Tableau musiciens** : nom éditable, badges micro/DI par glisser-déposer (desktop) ou modal (mobile), colonne commentaires, réordonnancement drag & drop des lignes
- **Badges disponibles** : Kick, Overhead, DI, Ampli repiqué, SM58, XLR, Jack, ⚠ Sans phantom
- **Badges d'en-tête** : informations scène (nombre de musiciens, dimensions, etc.), éditables et supprimables
- **Thème** : 5 presets (Dark Gold, Ice Blue, Rouge, Vert, Blanc), couleurs entièrement personnalisables, choix de police, effet grain et glow
- **Export PDF** : pagination automatique A4, respect du thème actif, nom de fichier avec l'année
- **Sauvegarde HTML** : snapshot complet avec données et thème embarqués, réouvrable dans le navigateur

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

### Améliorations à prévoir

- [ ] Ajouter un type de badge personnalisé (champ libre + couleur)
- [ ] Réordonner les lignes du tableau par glisser-déposer (le CSS `drag-over` est prêt mais le réordonnement n'est pas implémenté)
- [ ] Export JSON des données seules (sans le HTML entier) pour versionner ou partager la liste
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

### Option 3 — État dans l'URL (à implémenter)
Encoder les données musiciens en JSON compressé dans le hash `#` de l'URL permettrait de partager un lien direct vers une fiche préconfigurée. Utile pour des fiches éphémères (guest, date unique) sans avoir à pousser un commit.

### Option 4 — Hébergement instantané
Glisser `index.html` (ou le HTML sauvegardé) sur [netlify.com/drop](https://netlify.com/drop) génère une URL publique en quelques secondes, sans compte requis.
