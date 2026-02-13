# Documentation Complete - Projet `14.02`

## 1. Vue d'ensemble

Ce projet contient une page web interactive de type "Valentine game", basee sur:

- une interface HTML/CSS moderne
- des animations `anime.js`
- une logique JS de mini-jeu avec boutons `Oui` et `Non`
- un pipeline video avec retrait du fond vert (chroma key) dans un canvas
- une conversion runtime en `.webm` via `MediaRecorder`

Le fichier principal est `valentine.html`.

## 2. Objectif fonctionnel

La page pose une question ("veux-tu etre ma Valentine ?"), puis:

1. anime l'entree du titre, sous-titre et boutons
2. fait fuir le bouton `Non` quand la souris s'approche
3. apres plusieurs tentatives, declenche une soucoupe qui "enleve" le bouton `Non`
4. affiche une video WhatsApp seulement apres disparition du bouton `Non`
5. laisse cliquer `Oui` pour afficher une scene de victoire + confettis

## 3. Stack technique

- HTML5 (page unique)
- CSS3 (layout, glassmorphism, gradients, responsive)
- JavaScript vanilla
- Anime.js charge via CDN:
  - `https://cdnjs.cloudflare.com/ajax/libs/animejs/3.2.1/anime.min.js`

Dependance npm presente dans le repo:

- `animejs` `^4.3.5` dans `package.json`

Note: la page actuelle utilise le CDN v3.2.1, pas le package npm installe localement.

## 4. Structure des fichiers

```text
14.02/
  valentine.html                               # Page principale active
  valentinProposal.html                        # Ancienne variante de page
  essaye.html                                  # Fichier vide de test
  snaptik_7471747798052949279_v3 (1).mp4      # Video source pour la soucoupe/chroma key
  WhatsApp Video 2026-02-13 at 1.56.10 AM.mp4 # Video affichee apres disparition de Non
  vectorized.svg                               # Asset annexe
  vectorized.json                              # Asset annexe
  package.json
  package-lock.json
  node_modules/
```

## 5. Lancement du projet

### Option simple

Ouvrir `valentine.html` dans le navigateur.

### Option recommandee (serveur local)

Certaines API (canvas + video + MediaRecorder) sont plus stables en HTTP local.

Exemples:

```powershell
# Python
python -m http.server 8000
```

Puis ouvrir:

```text
http://localhost:8000/valentine.html
```

## 6. Architecture de `valentine.html`

### 6.1 Blocs HTML principaux

- `.scene` / `.content`: zone centrale du texte et boutons
- `#btnYes`, `#btnNo`: boutons interactifs
- `#dialogue`: texte dynamique selon les etapes
- `#hand`: conteneur de la soucoupe
- `#ufoCanvas`: rendu video traite (fond vert retire)
- `#ufoFallbackSvg`: fallback SVG si video indisponible
- `#afterNoVideo`: video WhatsApp affichee apres disparition de `Non`
- `#victory`: overlay de victoire

### 6.2 Organisation CSS

- Theme via variables `:root`
- Carte centrale glassmorphism (`.content`)
- Boutons stylises + transitions
- Overlay de victoire
- Styles dedies aux videos:
  - `.hand-grab.video-ready` (active le canvas)
  - `.hand-grab.above-text` (affichage au-dessus du texte pendant la capture)
  - `.after-no-video` / `.after-no-video.show` (video WhatsApp post-`Non`)

### 6.3 Etat JS principal

Variables importantes:

- `attempts`: nombre d'esquives du bouton `Non`
- `cooldown`: anti-spam de deplacement
- `handTriggered`: etat de declenchement de la soucoupe
- `followYesUntil`: fenetre temporelle de suivi du bouton `Oui`
- `ABDUCTION_ATTEMPTS` (par defaut `10`)
- `YES_FOLLOW_DURATION_MS` (par defaut `1200`)

Etat video/chroma:

- `chromaVideoReady`
- `chromaFrameHandle`
- `webmExported`
- `hasVisibleUfoFrame`

## 7. Flux d'interaction complet

### Etape 1 - Chargement

Au `window.load`:

1. reset de la position initiale de `Non`
2. reset de la video WhatsApp post-`Non`
3. init du pipeline video/chroma (`initChromaVideo`)
4. lancement des animations d'intro
5. particules + orbites + pulse du bouton `Oui`

### Etape 2 - Jeu avec bouton `Non`

`handleProximity(e)` calcule la distance curseur -> bouton `Non`.

Si proche:

- `Non` se deplace (`moveNoButton`)
- un message contextuel est affiche dans `#dialogue`
- `attempts` augmente

### Etape 3 - Enlevement du bouton `Non`

Quand `attempts >= ABDUCTION_ATTEMPTS`:

- `triggerHand()` lance la sequence
- la soucoupe se positionne au-dessus de la zone titre
- `Non` est "capture" puis sort de l'ecran
- `btnNo.style.display = "none"`

### Etape 4 - Affichage de la video WhatsApp

Immediatement apres la disparition de `Non`:

- `showAfterNoVideo()` est appelee
- `#afterNoVideo` passe en classe `.show`
- la video `WhatsApp Video 2026-02-13 at 1.56.10 AM.mp4` demarre

### Etape 5 - Validation

Quand l'utilisateur clique `Oui`:

- `showVictory()` affiche l'overlay
- texte de victoire anime
- double vague de confettis

## 8. Pipeline video: retrait fond vert + `.webm`

### 8.1 Source soucoupe

La video source de la soucoupe est:

`snaptik_7471747798052949279_v3 (1).mp4`

### 8.2 Chroma key

Fonction: `keyOutGreen(imageData)`

Principe:

- lecture pixel RGBA du canvas buffer
- detection d'un vert dominant
- reduction progressive de l'alpha selon la dominance du vert

Regles actuelles:

- seuil vert: `g > 78`
- dominance: `g - max(r, b) > 22`

### 8.3 Rendu

Fonctions:

- `drawUfoFrameOnce()`: dessine une frame (utile meme si autoplay bloque)
- `renderUfoFrame()`: boucle `requestAnimationFrame`
- `startUfoChromaLoop()` / `stopUfoChromaLoop()`

Fallback integre:

- si lecture pixel bloquee, la frame video brute est dessinee

### 8.4 Conversion `.webm`

Fonction: `exportChromaWebm()`

Pipeline:

1. `ufoCanvas.captureStream(30)`
2. `MediaRecorder` en `video/webm` (vp9/vp8/fallback)
3. enregistrement de quelques secondes
4. creation d'un `Blob` `.webm`
5. URL stockee dans `window.generatedUfoWebm`

Note: le lien de telechargement UI a ete retire, mais l'URL generee existe toujours en memoire JS.

## 9. Parametres a personnaliser rapidement

Dans le script:

- `ABDUCTION_ATTEMPTS`:
  - plus petit = soucoupe plus rapide
  - plus grand = bouton `Non` reste plus longtemps
- `YES_FOLLOW_DURATION_MS`:
  - duree pendant laquelle `Oui` suit le curseur

Dans le CSS:

- `.after-no-video`:
  - position/taille de la video WhatsApp affichee apres `Non`
- `.hand-grab.above-text`:
  - niveau de superposition au-dessus du texte

## 10. Debug / resolution de problemes

### La video ne s'affiche pas

- verifier les noms exacts des fichiers video
- servir le projet en HTTP local (recommande)
- verifier la console navigateur (erreurs media/canvas)

### L'autoplay est bloque

Comportement prevu:

- un premier `pointerdown` relance la lecture video

### Le `.webm` ne se genere pas

- verifier support `MediaRecorder` dans le navigateur
- verifier `MediaRecorder.isTypeSupported(...)`
- tester sur Chrome/Edge recents

### Le bouton `Non` disparait visuellement

Le code actuel deplace `Non` dans `document.body` en mode flottant pour eviter le clipping par la carte.

## 11. Notes de maintenance

- `valentinProposal.html` peut servir de reference de design alternatif.
- `essaye.html` est vide et peut etre supprime ou reutilise.
- `vectorized.svg` / `vectorized.json` ne sont pas critiques pour le flow principal.
- une harmonisation anime.js (CDN v3 vs npm v4) peut etre planifiee.

## 12. Evolution recommandee

1. Ajouter un bouton "Rejouer" qui reset tous les etats JS.
2. Sauvegarder le `.webm` automatiquement via bouton optionnel.
3. Externaliser CSS et JS dans des fichiers dedies (`styles.css`, `app.js`).
4. Ajouter des tests visuels de regression (Playwright).

## 13. Resume

Le projet est une page romantique interactive avancee avec:

- animations UI riches
- logique de mini-jeu
- video traitee en chroma key
- conversion runtime en `.webm`
- sequence narrative complete jusqu'a la victoire

Le fichier central a maintenir est `valentine.html`.
