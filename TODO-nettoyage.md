# Canal 7 — TODO nettoyage / dette technique

> But de ce fichier : **ne pas compter sur la mémoire d'une conversation**. Tout ce qui est
> "à nettoyer plus tard" est écrit ici, vit dans le projet, voyage avec le code (zip + GitHub).
> Dans une future session : dire « regarde TODO-nettoyage.md » et tout est là.

Dernière mise à jour : 2026-07-11 · version app au moment de l'écriture : **v731**

---

## 1. Renommer des identifiants internes (cosmétique mais dette)

Le **texte affiché** a été changé sans toucher aux identifiants internes (pour ne pas casser).
Résultat : le nom interne ne correspond plus au nom à l'écran → source de confusion future.

- [ ] **`reperagesbis` → `fichedecor`** (l'onglet « Repérages » est affiché « Fiche décor » côté écran).
      À renommer partout d'un coup : `setView('reperagesbis')`, `id="view-reperagesbis"`,
      `id="nav-reperagesbis"`, `renderReperagesBis()`, et toute variable/clé liée.
- [ ] **`d2-incl-reperages`** (case « inclure les repérages » dans les exports) → nom cohérent
      (ex. `d2-incl-fichedecor`). Vérifier la clé dans le state sauvegardé si elle est persistée.
- [ ] **`d2-sousdecors-section`** → renommer clairement (ex. `d2-souslist-section`).
      ⚠️ PIÈGE CONNU : cet id contient "sousdecors" mais c'est la **NOUVELLE** liste de sous-décors
      (notes/photos/élec/héritage), PAS l'ancien module (qui, lui, a été supprimé en v681).
      Ne pas confondre lors du renommage.

- [ ] **Arrachage complet de l'onglet `reperagesbis` (« Fiche décor »)** — GROS morceau (~124 réfs).
      Contexte : l'onglet a été **désactivé** en v695 (retiré du sélecteur `TAB_DEFS` + forcé OFF dans
      `getCurrentTabs`). Il est donc invisible/inaccessible, mais tout son code reste (code mort inerte).
      À retirer méthodiquement plus tard : `nav-reperagesbis`, `view-reperagesbis` (page + sélecteur
      décor + grille), modales `modal-repbis-cfg` et `modal-repbis-pdf`, les fonctions `_repbis*`
      (`_repbisModuleMap`, `_repbisOrderedKeys`, `_repbisVisibleModules`, `renderReperagesBis`,
      `setRepBisDecor`, `openRepBisConfig`, `toggleAllRepbisModules`, `_repbisModuleFilled`, etc.),
      `REPBIS_MODULES`, `state.repbisOrder`, les règles CSS `#view-reperagesbis`/`#modal-repbis-cfg`,
      et les entrées `reperagesbis` dans les tableaux partagés (setView guard, boucle d'affichage,
      DEFAULT_TAB_ORDER, TOURNAGE_LABELS, defaultTabs).
      ⚠️ Le pop-up « Fiche décor » (grille de modules `modal-d2-modules`) et le détail décor
      (`modal-d2-detail`) sont une AUTRE chose → NE PAS les toucher, on les garde.

**Méthode obligatoire** (un identifiant = plus risqué qu'un texte) :
1. Inventaire complet (`grep`) de TOUTES les occurrences.
2. Changement de toutes d'un coup (HTML + JS + CSS + éventuelles clés de state).
3. `node --check` sur le JS extrait + test à l'écran.
4. Backup + commit dédié avant de commencer.

---

## 2. Vers le natif iOS/Android + multi-métiers (vision long terme)

Contexte (discuté le 2026-07-10) : Canal 7 = l'app **chef élec**. Vision = décliner le même
patron pour **machino**, **caméra**, etc., puis une app **DP** qui agrège tout.

- [ ] **Séparer proprement le SOCLE COMMUN des MODULES MÉTIER.**
      Socle commun à tous les métiers : film, décors, séquences, PDT/planning, dates, GPS, photos,
      sous-décors, sauvegarde/export.
      Modules spécifiques élec : Énergie/branchement, liste lumière, bijoute, camion/parking…
      (machino aurait : grill, barres, déports… ; caméra : optiques, mouvements…).
      → Ne PAS mélanger davantage socle et modules ; à terme, isoler les modules métier.

- [ ] **Décision structurante à trancher AVANT le natif : le partage entre métiers.**
      Aujourd'hui : tout est local (iPhone, hors-ligne). L'agrégation DP nécessite un moment où
      les données se rejoignent → **serveur + synchronisation**.
      Contrainte non négociable : ça doit **continuer à marcher hors-ligne sur le plateau**.
      Modèle cible = "offline-first + sync quand réseau dispo" (PAS "serveur à la place du local").
      ⚠️ Ne pas construire le serveur trop tôt : finir/durcir l'app élec d'abord.

- [ ] **"cf. régie / cf. prod / cf. DP / cf. mach"** (renvois du livre blanc) : idée de marquer un
      item comme « à confirmer avec [X] » + une liste « ce qu'il me reste à valider ».
      Prend tout son sens dans l'écosystème multi-métiers (points de jonction entre les apps).

---

## 3. Idées fonctionnelles en attente (discutées, pas décidées)

- [ ] **Bilan de puissance** : liste lumière d'un décor vs abonnement Kva → voyant vert/rouge
      « ça tient / ça passe pas » + aide répartition mono/tri par phase.
- [ ] **Calcul d'équilibrage triphasé** (repissage dans le neutre) — logique validée le 2026-07-10
      (méthode "somme des écarts au min", prudente/sécuritaire). Reporté (pas prioritaire).
- [ ] **Heures de soleil** (lever/coucher/golden hour) à partir du GPS décor + date → calculable
      HORS-LIGNE. Intéresse mais reporté.
- [ ] **Section de câble / chute de tension** à partir de distance décor + ampérage.
- [ ] **Bloc "Intentions / discussions DP"** par décor/séquence (le pan "ARTISTIQUE" du livre blanc,
      absent de l'app aujourd'hui : le *pourquoi* de la lumière, pas juste le *quoi*).
- [ ] **Récap matériel agrégé** sur tout le film (somme des listes lumière → besoin global).
- [ ] **Dupliquer un décor** (cloner une fiche pour gagner de la saisie).
- [ ] Capturer explicitement : **distance ENTRE sous-décors** et **temps de déplacement** (pas juste
      la distance) — mentionnés dans le livre blanc, pas dans l'app.

---

## Repères techniques utiles (pour reprendre vite)

- Fichier unique `index.html` (~18k lignes, HTML+CSS+JS inline), pas de build.
- À chaque livraison : bump du cache dans `sw.js` (`const CACHE_NAME = 'canal7-vNNN';`).
- Après chaque modif JS : extraire les gros `<script>` + `node --check`.
- Jamais `prompt()` (PWA installée) → modales custom.
- Photos : normalisées en JPEG à l'import (canvas), stockées en IndexedDB (jetons `idb:` en local).
- Ancien module "sousdecors" (texte nom/plan/lieu) = **supprimé** en v681. Ne pas le réintroduire.
- Onglets visibles : Liste · Décors · Ponctuels · PDT · Équipe · Camion
  (secondaires masqués : Repérages/Fiche décor, M.e.s., Bijoute, Fichiers).

---

## 4. Édition inline du détail décor (mode « ✏️ Modifier ») — LARGEMENT FAIT

Objectif : le bouton « Modifier » rend la fiche éditable **sur place** (rester au même endroit,
champs texte modifiables, ex-boutons → dropdowns discrets). But final : retirer l'ancienne
grosse modale d'édition (`modal-d2` / `openD2Modal`) → supprime le doublon consultation/édition.

### Mécanique en place (v705 → v723)
- `_d2DetailEdit` (bool) + bouton « ✏️ Modifier » (rouge clair) / « ✓ Terminé » (vert) en haut du
  détail. Remis à `false` à chaque `openD2Detail()` (on ouvre toujours en lecture).
- Helper `section()` : une ligne devient éditable SI on lui passe `key` + `module`
  (+ `raw`, `type` : select/number/date, `options`, `reRender:true` pour les toggles conditionnels).
- Sauvegarde : `setD2ModuleField(module,key,value)` (texte = `onblur`, dropdown = `onchange`) ;
  `setD2ModuleFieldR(...)` = idem + re-render (pour les toggles qui déplient des sous-champs).
- Style « rien ne bouge » : inputs sans bordure + trait pointillé ; dropdowns sans flèche
  (`appearance:none`). Le contact est affiché en lecture dans TOUS les modes (sinon ça « saute »).
- ⚠️ Les dropdowns **préservent une valeur hors-liste** (ancienne donnée type « Sud-Ouest ») en
  l'ajoutant comme option sélectionnée → aucune perte de données. NE PAS casser ce comportement.
- Valeurs des dropdowns élec **alignées sur la modale** (`_ampOptions` 10/16/20…, `_abonOptions`,
  `_kvaOptions`, `_distOptions` 5 m→+500 m, `_durOptions` par 30 min).

### FAIT (éditable inline)
- Cœur : **contact** (nom/tel/email) + **effets** (chips en texte, gris = non sélectionné).
- **Soleil** (orientation = dropdown N/NE/E/SE/S/SO/O/NO), **Stock**, **Machinerie**, **Départements**.
- **Accès** : Étages, Notes, Marches/Rampe (oui-non), **Ascenseur** et **Monte-charge**
  (oui-non → déplient 4 dimensions H/L/P/poids chacun).
- **Élec** : Ampérage, Phase (mono/tri), Abonnement, Terre, Distance, Notes, + les composites
  **Branchement** (prise → type/phase ; épanoui → forme), **Groupe** (oui → mode → type → puissance),
  **Batterie** (oui → type → puissance).
- **Camion** : nom parking, distance, contraintes, notes (+ bouton « 📍 Carte / placer le camion »).
- **Temps** : Prépa/Remballe (dropdowns 30 min), **Prélight** (oui → date + heure), **Renforts**
  (oui → liste éditable : 📌 chef, nom avec datalist alimenté par l'onglet Équipe (poste « Renfort »),
  × retirer, + ajouter). Notes rendues EN DERNIER, après la liste des renforts.
- **Photos** : en mode édition, le visualiseur plein écran a une **corbeille 🗑** (à côté d'Annoter) ;
  tous les boutons du visualiseur sont en **icônes seules** (✏️ 🗑 ↺ 📌).
- Boutons d'accès aux éditeurs non-inlinables : **📍 Carte camion** (`openCamionFromDetail` → ferme
  le détail, ouvre `openD2Module('camion')`, et **revient au détail** après enregistrement, scroll
  sur l'ancre `d2-camion-anchor`) et **📐 Plan de feux** (`openPlanFeuxFromDetail`).

### RESTE À FAIRE
- [ ] Cœur : **séquences** et **sous-décors** (listes) en édition inline. Éditer une ligne = OK,
      mais ajout/suppression fera bouger la page (physique, inévitable).
- [ ] **Nom / adresse du décor** : ils sont dans l'EN-TÊTE de la modale → à éditer autrement.
- [ ] **Carte/GPS/parking** et **plan de feux** : resteront des boutons (incompressible : ça ne
      s'édite pas au clavier).
- [ ] Machinerie : confirmer si grill/grue/nacelle/… doivent être des dropdowns oui/non
      (aujourd'hui traités en texte libre, car affichés en texte brut).
- [ ] **Objectif final** : une fois séquences + sous-décors inline → retirer `openD2Modal` /
      `modal-d2` (l'ancienne modale d'édition).

---

## 5. Bugs de fond corrigés le 2026-07-11 (⚠️ NE PAS RÉINTRODUIRE)

### 5.1 Service worker : il corrompait `index.html` (grave)
L'ancien handler traitait TOUTE navigation comme du HTML et faisait
`cache.put('./index.html', réponse)` → ouvrir **plan-feux.html écrasait `index.html` dans le cache**
avec le contenu du plan de feux. Conséquences : plantages de l'app, page blanche, « bordel
d'affichage », et hors-ligne le plan de feux renvoyait `index.html`.
**Correctif (v725+)** : les pages HTML sont **précachées à l'install** (`ASSETS`) et **ne sont
JAMAIS réécrites depuis une navigation**. Réseau d'abord, cache en secours, avec la BONNE clé
(`./plan-feux.html` vs `./index.html`).
→ **Règle : ne jamais remettre un `cache.put` sur une réponse de navigation.**

### 5.2 Rendu Chromium cassé : 54 overlays floutés en permanence (grave)
`.modal-overlay` (×54), `.confirm-overlay` et `#toast-back` appliquaient un
`backdrop-filter: blur()` **même fermés** (ils étaient seulement `opacity:0`). Chromium devait
composer 54 calques de flou plein écran en continu → **le contenu n'était plus peint** (page vide,
qui « revenait » quand on redimensionnait la fenêtre). Firefox et Safari : OK. Brave/Responsively
(Chromium) : cassés.
**Correctif (v727)** : le `backdrop-filter` n'est appliqué QUE sur `.open` / `.show`, et les
overlays fermés passent en `visibility: hidden`.
→ **Règle : jamais de `backdrop-filter` sur un overlay fermé.** (Toutes les modales s'ouvrent via
la classe `open` — vérifié : aucune via `style.display`.)

### 5.3 Contenu invisible tant qu'on ne redimensionnait pas
Ajout de `_forceRelayout()` (appelé en fin de `setView()` et à l'affichage de `app-screen`) :
force le recalcul de mise en page après rendu au lieu d'attendre un resize.
NB : correctif de symptôme ; la cause profonde était surtout 5.2.

### 5.4 Flash du bouton Export au chargement
`_fitExportBar()` affichait la barre à sa position par défaut puis la recalait 60 ms plus tard
(→ flash sous « Extraire »). **Correctif (v728)** : la barre est mise à `opacity:0` pendant le
calage et révélée une fois posée. Vaut pour toutes les pages qui utilisent cette barre.

---

## 6. Workflow de dev (validé le 2026-07-11)

- **Ne PAS tester en `file://`** (double-clic sur index.html) : le navigateur y bloque des choses
  → faux bugs, page blanche. Ça a fait perdre du temps.
- **Serveur local** : `cd <dossier>` puis `python3 -m http.server 8000` → `http://localhost:8000`.
  Mêmes conditions que GitHub Pages (service worker, offline), rechargement instantané.
- **Responsively App** pour voir iPhone SE / iPad / desktop en même temps (activer « Live reload »
  et « Disable cache »).
- **Le cache/service worker ment** : plusieurs fois cette session, une modif était bien en place mais
  invisible à cause d'une version en cache. En cas de doute : navigation privée ou `Cmd+Shift+R`.
- Ne pousser sur GitHub que les versions **validées**.


### 5.5 Import JSON : il ÉCRASAIT tous les films (grave — perte de données)
`triggerImport()` faisait `state = parsed` → importer un fichier **effaçait tous les films existants**.
**Correctif (v729)** : l'import **FUSIONNE** par `id`, collection par collection
(`films`, `ponctuels`, `liste`, `decors`, `d2Items`) : même id → mis à jour ; nouvel id → ajouté ;
**rien n'est jamais effacé**. Le film courant reste sélectionné s'il existe encore.
→ **Règle : ne jamais remettre `state = parsed` dans l'import.**
NB : conséquence assumée — un décor supprimé chez soi mais encore présent dans le fichier
**réapparaît** à l'import (on ne supprime jamais tout seul).

### 5.6 Export JSON : il exportait TOUT, sans choix du film
**Correctif (v730)** : `exportData()` ouvre la modale `modal-export-film` (sélecteur de film) ;
`doExportFilm()` exporte **UN film avec tout ce qui lui est rattaché** (filtre `filmId` sur
ponctuels/liste/d2Items/decors/equipe/camions + photos via `_stateWithImages()`).
Option « Tous les films » conservée pour la sauvegarde complète.
Usage cible validé avec l'utilisateur : **partager un film** / **mettre à jour un film existant**.
(`savedMembers` est global → volontairement NON exporté.)

### 5.7 Service worker désactivé en LOCAL (dev)
Le SW servait une version en cache et masquait les modifs. Responsively **n'a aucun réglage pour
vider son cache** (il faut quitter/relancer l'app) → piège permanent.
**Correctif (v731)** : sur `localhost`/`127.0.0.1`, le SW **n'est pas enregistré**, celui déjà
présent est **désinstallé** et les caches vidés. En production (GitHub Pages) : inchangé,
l'offline plateau reste intact.
⚠️ Conséquence : **le mode hors-ligne ne peut plus être testé en local** → le tester sur GitHub
Pages ou sur le téléphone.
