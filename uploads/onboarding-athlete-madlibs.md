# Spec — Onboarding athlète « phrase à trous » (style Atoms)

> Statut : **active / draft** · Dernière édition : 2026-06-12 — par Benjamin + Claude
> Cible : athlète, **mobile-first** · Thème : **athlete-light**
> Destinée à être donnée à un agent de génération (Claude design / Figma, ou Lovable). Document **neutre** : les sections sont autoportantes, copiables une par une selon l'outil.

---

## 0. TL;DR

On refait l'onboarding athlète de Hustle Up en remplaçant le parcours **« 1 question = 1 écran »** du prototype actuel ([coach-matchmaker-buddy.lovable.app](https://coach-matchmaker-buddy.lovable.app)) par l'interaction signature de l'app **Atoms** : une **phrase narrative à trous**, où chaque trou est un choix fini que l'utilisateur tape pour ouvrir un sélecteur.

- **Même fond produit** que le proto (les 11 dimensions collectées restent les mêmes — elles alimentent le matching coach + la routine + le fittest).
- **Forme différente** : 11 écrans question-réponse → **4 phrases à trous** narratives.
- **Habillage DS** : on abandonne la palette bleu-marine du proto et on applique les **tokens Hustle Up athlete-light** (primary = lime `#DDF247`).

But métier : collecter assez de signal pour (a) **matcher un coach**, (b) proposer une **routine/première habitude**, (c) brancher sur le **fittest**.

---

## 1. Contexte produit

### 1.1 D'où ça vient
- Un prototype Lovable existe (`coach-matchmaker-buddy`). Il valide le **fond** (quelles données, quel objectif : « le coach qui te correspond, en 2 minutes ») mais **ne tient pas compte du DS** et **n'utilise pas le principe Atoms**.
- Côté conception, l'onboarding a été abordé en réunion (Miro de Florian, écrans Atoms montrés pour la collecte d'objectif). Une **réunion onboarding dédiée** est prévue avec un PO — voir §9 (questions ouvertes) pour ce qui reste à trancher avec eux.

### 1.2 Le principe Atoms qu'on veut reprendre
Atoms (habit-building, lignée BJ Fogg / Tiny Habits) collecte les préférences via une **phrase à compléter** : un texte fixe entrecoupé de **mots-trous tappables** à choix finis. C'est ludique, rapide, lisible, et ça réduit la charge cognitive vs un long formulaire. La formule d'ancrage d'habitude (« **Après [ancrage], je ferai [habitude]** ») en est le cœur — le proto la reprend déjà à sa dernière étape.

### 1.3 Cible & contraintes
- **Persona** : athlète qui s'inscrit (pas le coach, pas l'owner).
- **Device** : mobile (375 px de large, hauteur ~812). Tout doit fonctionner au pouce.
- **Thème** : `athlete-light` exclusivement pour ce parcours.
- **Durée perçue** : « 2 minutes » (promesse de la landing). Garder ça : 4 phrases courtes, progression visible.

---

## 2. Principe d'interaction (le cœur)

### 2.1 Anatomie d'une phrase à trous
Chaque écran présente **une phrase** composée de :
- **Texte statique** (style `Title/Large`, couleur `text/on-surface/default`) — le liant narratif.
- **Trous** = composants inline tappables (« chips ») insérés dans le flux du texte.

Un trou a **deux états** :

| État | Contenu | Remplissage | Bordure | Texte |
|---|---|---|---|---|
| **Vide** (placeholder) | `+ un objectif`, `+ un sport`… | `interactive/background/primary/faded` (ou transparent) | `interactive/border/primary/faded`, **dashed** 1.5px | `interactive/text/primary/default`, légèrement atténué |
| **Rempli** | la valeur choisie (`Devenir plus fort`) | `interactive/background/primary/faded` (plein) | `interactive/border/primary/faded`, **solid** | `interactive/text/primary/default` |

> Le contraste vide/rempli doit être net : un trou vide « appelle » le tap (style pointillé + signe `+`), un trou rempli est affirmé (plein, solide). Optionnellement, un trou rempli peut afficher un petit chevron `⌄` pour signaler qu'il reste éditable.

La phrase est un **conteneur auto-layout horizontal avec retour à la ligne** (`layoutWrap = WRAP`) : texte et chips s'enchaînent comme des mots et passent à la ligne naturellement. Interligne confortable (gap vertical = `spacing/8`).

### 2.2 Le sélecteur (picker)
Taper un trou ouvre un **bottom sheet** (surface `surface/background/bottom-sheet`, coins supérieurs arrondis `border-radius/2xl` = 16, overlay `surface/background/overlay` derrière) contenant :
- Un **titre court** rappelant la question (`Title/Small`).
- La **liste des options** (choix finis) sous forme de lignes sélectionnables (voir patterns §5.4).
- Sélection → le sheet se ferme, le trou se remplit, la phrase se met à jour. Sélection unique par défaut.

Variantes de picker selon la dimension :
- **Liste simple** (la majorité) : lignes full-width, radio implicite.
- **Grille** (lieu, objet connecté) : options en grille 2-3 colonnes avec emoji/icône.
- **Numérique** (jours/semaine) : **stepper** ou mini-slider dans le sheet (1→7).

### 2.3 Progression & validation
- **Indicateur de progression** en haut : 4 segments (un par phrase) — segment actif/rempli en `interactive/background/primary/default` (lime), à venir en `interactive/background/primary/faded`. (Le proto utilise un compteur `n/11` + barre ; ici on passe à **4 phrases** → 4 segments.)
- **CTA bas d'écran** : `Button` DS, `variant=primary, intent=primary, size=lg, isFullWidth=True`.
  - **Désactivé** (`state=disabled`) tant que **tous les trous de la phrase courante** ne sont pas remplis.
  - **Actif** (`state=default`) dès la phrase complète. Label : `Continuer` (et `Voir mes recommandations` sur la dernière phrase).
- Navigation **retour** possible (flèche `←` en haut à gauche) sans perdre les réponses.

### 2.4 Pourquoi c'est mieux que le proto actuel
| Proto actuel | Cette spec |
|---|---|
| 11 écrans, 1 question chacun | 4 écrans narratifs |
| Cards de sélection génériques | Phrase vivante + chips inline (signature Atoms) |
| Palette bleu-marine hors-DS | Tokens DS athlete-light (lime) |
| Sensation « formulaire » | Sensation « je raconte mon objectif » |

---

## 3. Modèle de données collectées

Les **11 dimensions** restent celles du proto (ne rien retirer — elles servent en aval). Type, options, et usage :

| # | Dimension | Clé suggérée | Type | Contextuel ? | Options | Usage aval |
|---|---|---|---|---|---|---|
| 1 | Objectif principal | `goal` | enum (1) | — | `strength` Devenir plus fort · `weight_loss` Perdre du poids · `energy` Avoir plus d'énergie · `endurance` Améliorer mon endurance · `sleep` Mieux dormir · `event` Préparer un événement · `move_more` Bouger plus, simplement | matching + routine |
| 2 | Limitations physiques | `limitation` | enum (1) | — | `none` Non, tout va bien · `injury_recovery` Je récupère d'une blessure · `chronic_pain` Douleur chronique · `mobility` Limitation de mobilité | sécurité programme + matching |
| 3 | Niveau | `level` | enum (1) | — | `zero` Je pars de zéro · `stopped` Je m'entraînais avant, j'ai arrêté · `occasional` De temps en temps · `regular` Régulièrement · `competitor` Très compétiteur | calibrage |
| 4 | Résultat mesurable | `target` | enum (1) | **oui → dépend de `goal`** | _(ex. `strength`)_ Faire 10 tractions en 12 semaines · Soulever 120 kg au deadlift · Grimper une voie en 6b | objectif SMART, suivi |
| 5 | Frein habituel | `blocker` | enum (1) | — | `time` Pas le temps · `motivation` Manque de motivation · `know_how` Je sais pas quoi faire · `consistency` Difficile de tenir dans la durée · `injury_fatigue` Blessures / fatigue | nudges, ton coaching |
| 6 | Fréquence actuelle | `current_freq` | enum (1) | — | `0` · `1-2` 1-2 fois · `3-4` 3-4 fois · `5+` 5+ fois | point de départ |
| 7 | Lieu d'entraînement | `place` | enum (1) | — | `home` À la maison · `gym` En salle · `outdoor` Dehors · `mixed` Un peu de tout | type de programme |
| 8 | Objet connecté | `device` | enum (1) | — | `none` Aucun · `apple_watch` · `garmin` · `suunto` · `whoop` · `polar` · `fitbit` · `oura` · `other` Autre | **branche fittest / santé** |
| 9 | Micro-habitude | `micro_habit` | enum (1) | **oui → dépend de `goal`** | _(ex. `strength`)_ 10 pompes · Suspension à la barre 20 sec · 15 squats au poids du corps | 1re habitude |
| 10 | Engagement hebdo | `weekly_commit` | int 1-7 | — | slider/stepper, défaut 3 | planning routine |
| 11 | Ancrage habitude | `habit_anchor` | enum (1) | — | `wake` Après le réveil · `coffee` Après le café · `work` Après le travail · `lunch` Après le déjeuner · `sleep` Avant de dormir | habit stacking |

> **Contextualité (#4 et #9)** : les options dépendent de `goal`. La spec ne fige que la branche `strength` (observée dans le proto). **Les autres branches restent à définir** (§9). Prévoir la structure de données comme `Record<goal, Option[]>`.

---

## 4. Découpage en 4 phrases

On regroupe les 11 dimensions en 4 écrans-phrases thématiques. Texte fixe en **gras non-souligné**, trous entre `[crochets]`.

### Phrase 1 — Ton ambition
> « Je veux **[objectif #1]**. D'ici **[délai]**, mon premier objectif c'est **[résultat #4]**. »

- Trous : `objectif` (#1), `résultat` (#4, contextuel), `délai`.
- Note : le proto fusionne résultat + délai (« …en 12 semaines »). Ici on **peut** soit garder un seul trou « résultat » qui inclut le délai, soit l'éclater en deux trous (plus « Atoms »). **Recommandé : un seul trou `résultat`** pour rester fidèle au proto et limiter la dépendance ; garder `délai` séparé seulement si on veut un objectif vraiment SMART. _À trancher §9._
- `résultat` (#4) ne devient sélectionnable qu'une fois `objectif` (#1) choisi (dépendance). Tant que `objectif` est vide, le trou `résultat` est désactivé visuellement.

### Phrase 2 — Où tu en es aujourd'hui
> « Aujourd'hui je m'entraîne **[fréquence #6]** par semaine, surtout **[lieu #7]**, et je dirais que **[niveau #3]**. »

- Trous : `fréquence` (#6), `lieu` (#7, picker grille 2×2 + emoji), `niveau` (#3).

### Phrase 3 — Ce qui compte pour bien t'aider
> « Ce qui me freine souvent, c'est **[frein #5]**. Côté corps, **[limitation #2]**. Je suis mes données avec **[objet connecté #8]**. »

- Trous : `frein` (#5), `limitation` (#2), `objet connecté` (#8, picker grille 3×3).
- `device` alimente la bascule **fittest** en fin de parcours (un athlète équipé d'une montre a un fittest enrichi).

### Phrase 4 — Ta première habitude
> « Je m'engage sur **[jours #10]** par semaine. Pour démarrer en douceur, je ferai **[micro-habitude #9]**, **[ancrage #11]**. »

- Trous : `jours` (#10, picker numérique 1-7), `micro-habitude` (#9, contextuel à `goal`), `ancrage` (#11).
- C'est la formule Atoms canonique d'habit stacking (« je ferai X, après Y »).
- CTA de cette phrase : **`Voir mes recommandations`**.

→ puis **Écran Recap** (§7).

---

## 5. Spécifications visuelles — DS Hustle Up (athlete-light)

> **Règle d'or** : aucune valeur en dur. Lier aux **variables/text-styles** du DS Hustle Up. Les hex ci-dessous sont indiqués **à titre de repère visuel** ; la source de vérité est le token (mode Athlete Light, modeId `7531:0`, collection Theme `VariableCollectionId:4901d53dc1c5ba855481e2766bade8acdcbf9b85/45877:9`). **Ne pas reprendre le bleu-marine du proto.**

### 5.1 Couleurs (tokens Theme · mode Athlete Light)
| Rôle | Token | Clé | Repère |
|---|---|---|---|
| Fond page | `surface/background/default` | `56009e7ab5438b07fc5bea9067b8e3d223978895` | `#FFFFFF` |
| Carte / surface secondaire | `surface/background/secondary` | `16503482f665fd138afce8dd211fe5a70573e031` | `#F2F2F2` |
| Surface bottom-sheet | `surface/background/bottom-sheet` | - | `#FFFFFF` |
| Overlay (scrim) | `surface/background/overlay` | - | #121212 alpha 48% |
| Texte principal | `text/on-surface/default` | `2b71117ce8e25b227913cd46eed400ac9d049c98` | `#1C1C1C` |
| Texte atténué (sous-titres) | `text/on-surface/muted` | `8bc4bb37d6302a209e37982bbb064d7fbbd1c870` | `#454643`|
| Accent primaire (lime) | `interactive/background/primary/default` | `05eaed582cb732c166accb33aee6bbae7fc6c0e5` | `#DDF247` |
| Accent primaire atténué (fond chip) | `interactive/background/primary/faded` | `bb599682ccb2b8a4d4fd697ac68e19e7a2ef5f5c` | `#F3FAC3` |
| Bordure chip | `interactive/border/primary/faded` | `04596e3cce48e12e6bbbfcd19921b1a388b53766` | - | `#F3FAC3`
| Bordure chip rempli (forte) | `interactive/border/primary/default` | `172b00ab82e933dc9f09681f6f85d7f4e7edcf8c` | — |
| Texte sur chip | `interactive/text/primary/default` | `3c45db14781754c67cc913c79d0904e4d658fd88` | `#245453` |
| Texte sur bouton primaire | `text/on-surface-primary/default` | `9c89cf6767cecbfa75b9b44d0f920c303b098342` | `#245453` |

### 5.2 Typographie (text-styles DS)
| Usage | Style | Clé | Police |
|---|---|---|---|
| Titre de phrase / page | `Headline/small` | `e594308b9baeea222881dde83ccbc3304c405ea8` | **Sora** Medium 24/36 |
| (alt grand titre) | `Headline/Medium` | `a8fb9e61fab93e10713ef1f194213bdbd8b35eaf` | **Sora** SemiBold 28/39 |
| Texte de la phrase à trous | `Title/Large` | `8dad8c5432089d14dd316665c93f7383ebdba0dd` | **Trenda** Semibold 22/30 |
| Sous-titre / consigne | `Body/Medium` | `1825d6f8a1d624770b44420422e934c0e6db9472` | **Trenda** Regular 16/24 |
| Texte dans un chip | `Label/Large` | `2de873488eca238ba5913dee5bd010626dd9c226` | **Trenda** Semibold 16/20 |
| Titre du bottom-sheet | `Title/Small` | `4a07991cb491c29a3104928358108554623f1774` | **Trenda** Semibold 18/24 |
| Méta / micro-labels | `Label/Medium` | `388fe4c34d6962ab9cb2872a9545613070025d3c` | **Trenda** Semibold 14/18 |

> **Polices** : Sora (Display/Headline) + Trenda (Title/Body/Label). **Jamais** d'Inter/Roboto de substitution.

### 5.3 Rayons & espacements
| Usage | Token | Valeur |
|---|---|---|
| Chip | `border-radius/lg` | 8 |
| Carte phrase | `border-radius/xl` | 12 |
| Bottom-sheet (haut) | `border-radius/2xl` | 16 |
| Segment de progression (pilule) | `border-radius/4xl` | 32 (effet pleine-pilule) |
| Padding écran | `spacing/20` | 20 |
| Padding carte | `spacing/16` | 16 |
| Padding chip H / V | `spacing/12` / `spacing/6` | 12 / 6 |
| Gap mots ↔ chips | `spacing/8` | 8 |

### 5.4 Composants DS à réutiliser
| Élément | Composant DS | Clé set | Notes |
|---|---|---|---|
| CTA bas d'écran | **Button** | `036d726d870b01f48badfcad3f2218cb15002b0a` | variant=`primary`, intent=`primary`, size=`lg`, isFullWidth=`True`, state=`default`/`disabled`. Le label éditable est le TEXT dans le frame `label`. |
| Trou rempli (option) | **Chip** | `eed881e67a1583a601310b754b61954265d3ea44` | À vérifier : variants du Chip (taille, état sélectionné). Le trou **vide** (placeholder pointillé `+ …`) n'existe pas tel quel dans le DS → **élément custom** stylé avec les tokens (ou variante Chip à créer). _Signaler à l'équipe DS, cf §9._ |
| Picker / sélecteur | **Popup** | `130a731508df058108333247e576037ac86d4d3e` | Base possible pour le bottom-sheet. Pas de composant « BottomSheet » dédié dans le DS → pattern à confirmer/créer. |
| Sélection numérique (#10) | **SliderFilter** | `b85112e619a820dedc36d11b8b9827f3b8414f79` | ou stepper. À valider. |
| Options grille (#7, #8) | **Tags** / **RadioGroup** | `4295e8822d7cac9876568a071de01995520db8f0` / `fda96d8a50047667b12b2649cbb93e0d7e0da48a` | choisir selon le rendu voulu. |

---

## 6. Comportement & logique

1. **Entrée** : depuis la landing (« Commencer »). _Le proto passe par `/auth` ; ici l'auth est hors-scope de cette spec — supposer l'utilisateur déjà identifié ou auth traitée en amont._
2. **État** : les réponses sont conservées en mémoire pendant tout le parcours ; retour arrière non destructif.
3. **Validation par phrase** : CTA actif ssi tous les trous **requis** de la phrase sont remplis. Tous requis par défaut.
4. **Dépendances contextuelles** :
   - `target` (#4) dépend de `goal` (#1) → trou désactivé tant que `goal` vide ; ses options se chargent selon `goal`.
   - `micro_habit` (#9) dépend de `goal` (#1), idem.
5. **Persistance finale** : à la validation de la phrase 4, l'objet complet (11 champs) est soumis → calcul matching + génération routine.
6. **Routage** : URL par étape **non fiable** dans le proto (état interne). Pour la version finale, prévoir un routing par phrase robuste (deep-link repris en cas de reprise).

---

## 7. Écran de fin (Recap)

Reprend l'écran « Ton profil est prêt » du proto, ré-habillé DS :

- Icône lime ronde (accent `interactive/background/primary/default`), pictogramme « sparkles ».
- Titre `Headline/Medium` : **« Ton profil est prêt. »**
- Sous-titre `Body/Medium` muted : « On a tout ce qu'il faut pour te proposer le bon coach et la bonne routine. »
- **Deux cartes d'action** :
  1. **Mes coachs recommandés** (carte primaire mise en avant) — « Découvre les coachs qui matchent avec ton profil. » → écran matching.
  2. **Faire le fittest** (carte secondaire) — « Quelques tests pour mesurer ton point de départ. » → branche **fittest** (feature prioritaire de l'équipe ; le `device` collecté en #8 enrichit ce test).
- Lien tertiaire : « Retour à l'accueil ».

> Le matching coach lui-même est **hors-scope** ici (le proto affiche un stub « On finalise le matching… »). Cette spec produit l'**entrée** du matching, pas son rendu.

---

## 8. Accessibilité

- **Cibles tactiles** ≥ 44×44 px (chips inclus — prévoir padding suffisant même si le texte est court).
- **Trous** : rôle bouton, `aria-label` explicite (« Choisir un objectif », et après remplissage « Objectif : Devenir plus fort, modifier »).
- **Bottom-sheet** : focus trap, fermeture au scrim + au geste, `Échap`/retour Android.
- **Contraste** : vérifier `interactive/text/primary/default` (#245453) sur `interactive/background/primary/faded` (#F3FAC3) — ratio AA visé. Le texte muted sur blanc doit rester ≥ 4.5:1.
- **Progression** : annoncer « Étape 2 sur 4 » aux lecteurs d'écran.
- **Indépendance couleur** : l'état vide d'un trou ne repose pas que sur la couleur (pointillé + `+`).

---

## 9. Questions ouvertes (pour la réunion onboarding avec le PO)

1. **Découpage 4 phrases** : valider le regroupement (§4). Alternative : 3 phrases (fusionner 2+3) ou 5 (éclater l'ambition).
2. **Branches contextuelles #4 et #9** : définir les options `target` / `micro_habit` pour **chaque** `goal` (le proto ne couvre que `strength`). C'est le gros morceau de contenu manquant.
3. **Trou `délai`** dans la phrase 1 : trou séparé (objectif SMART) ou inclus dans `résultat` ?
4. **Composants DS manquants** : trancher avec l'équipe DS — créer `Chip placeholder pointillé` + `BottomSheet`, ou composer.
5. **Slider vs stepper** pour `weekly_commit` (#10) à l'intérieur d'un sheet.
6. **Auth** : avant ou après l'onboarding ? (le proto met `/auth` avant « Commencer »).
7. **Lien fittest** : profondeur de l'intégration `device` (#8) → fittest.

---
