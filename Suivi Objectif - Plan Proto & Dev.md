# Suivi de l'objectif : plan prototype + dev

**But** : transformer le brainstorm [Exploitation Onboarding - Suivi Objectif.md](Exploitation%20Onboarding%20-%20Suivi%20Objectif.md) en un plan actionnable. Deux livrables : la **spec d'un prototype** (mock HTML interactif, comme l'onboarding `.dc.html`) et un **plan de dev** en lots.

**Décisions de cadrage** :

- Prototype = **mock HTML interactif** (réutilise l'approche `Onboarding Athlete.dc.html`, testable sur téléphone, état simulé en JS, pas de backend).
- Périmètre du 1er lot (proto + v1 dev) = **cœur suivi d'objectif** : carte objectif (Homepage) + écran « Mon objectif » + journal + check-in. C'est la demande initiale, le minimum qui prouve la valeur.

---

# Partie A — Spec du prototype (mock HTML)

## A.1 But du prototype (hypothèses à valider)

Le proto sert à **tester l'expérience avant le dev**, pas à être complet. Il doit répondre à :

1. La **carte objectif + le journal** donnent-ils un vrai **sentiment d'avancement** (vs un journal d'humeur) ?
2. Le **check-in** (« Faire mon point ») est-il assez **rapide** (geste de 10 s) ?
3. Les **3 types d'objectif** (chiffré / événement / qualitatif) se comprennent-ils sans explication ?
4. La **barre valeur/cible** (objectif chiffré) est-elle plus parlante que le ressenti seul ?
5. La cohabitation **objectif (hero) + test fitness (condensé) + rituel** sur la Homepage tient-elle visuellement ?

## A.2 Périmètre (cœur uniquement)

**Inclus** : carte objectif Homepage, écran canonique « Mon objectif », journal (timeline, pas de graphe), check-in (bottom sheet), barre valeur/cible / compte à rebours / barre de temps selon le type, empty states, « Marquer comme atteint ».

**Exclus du proto** (simulés en dur ou absents) : onboarding progressif, jauges de complétion, relance douce, cycle de fin complet, historique, célébration, notifications, graphe ressenti.

## A.3 Écrans à produire

| Écran | Contenu | États à montrer |
|---|---|---|
| **Homepage** | Carte objectif hero + ligne test fitness condensée + bloc « Mes séances du jour » avec ligne rituel | objectif chiffré en cours ; (variante) pas d'objectif → CTA « Définir mon objectif » |
| **Mon objectif** (canonique) | Barre valeur/cible (ou compte à rebours / barre de temps) + journal timeline + actions (Modifier, Voir mon parcours, Marquer comme atteint) | chiffré 18/30 ; événement (compte à rebours) ; qualitatif (ressenti seul) |
| **Check-in** (bottom sheet) | Valeur (si chiffré, avec cible affichée) + ressenti 4 états + note | saisie + ajout à la timeline |
| **Empty / 1er relevé** | « Tu pars de combien ? » au 1er point (baseline in-app) | objectif chiffré sans relevé |

## A.4 État simulé (le modèle du mock)

Un objet JS `state` en mémoire, modifiable par les interactions, sans persistance :

```js
state = {
  objectif: {
    phrase: "Faire 30 tractions",
    type: "force",            // force | poids | endurance | evenement | bouger
    kind: "chiffre",          // chiffre | evenement | qualitatif
    cible: 30, unite: "tractions",
    echeance: "2026-12-31",
    statut: "actif",          // actif | atteint
    depart: 8,                // baseline, fixée au 1er relevé
  },
  releves: [
    { date: "2026-06-15", valeur: 8,  ressenti: "loin",   note: "", depart: true },
    { date: "2026-06-22", valeur: 15, ressenti: "loin",   note: "" },
    { date: "2026-06-29", valeur: 18, ressenti: "en_voie", note: "ça vient" },
  ],
  habitude: { geste: "10 pompes", ancrage: "réveil", joursSemaine: 4 },
  testFitness: { score: 72 },
}
```

Prévoir un **sélecteur de scénario** caché (ex. `?scenario=chiffre|evenement|qualitatif|vide`) pour basculer l'état de démo.

## A.5 Scénarios cliquables

1. **Suivi chiffré** : Homepage → tap carte → écran « Mon objectif » (barre 18/30 = 60%) → « Faire mon point » → saisir 22 + « en bonne voie » → la barre passe à 73%, nouvelle ligne en haut de la timeline.
2. **Atteinte** : saisir une valeur = cible (30) → popup « Tu as atteint ton objectif ? » → « Oui » → écran de félicitation simple (statique).
3. **Événement** : objectif « Terminer un marathon » → écran avec **compte à rebours** (« J-180 ») + bouton « C'est fait ? », pas de barre valeur/cible.
4. **Empty** : pas d'objectif → carte « Définir mon objectif » ; 1er relevé → « Tu pars de combien ? ».

## A.6 Fidélité visuelle

- Réutiliser le **langage Hustle Up** des screenshots : vert citron + vert sombre, logo « Hustle Up », bottom nav (Aujourd'hui / Explorer / Training / Box / Profil), polices du repo (`fonts/`).
- Cadre **mobile** (comme les recaps `screenshots/recap*.png`).
- Haute fidélité visuelle, mais **données factices** et **pas de vraie logique** au-delà des scénarios ci-dessus.

## A.7 Interactions clés (le strict nécessaire)

- Tap carte Homepage → écran « Mon objectif ».
- « Faire mon point » → bottom sheet → « Ajouter à mon journal » → maj barre + timeline.
- Valeur = cible → popup de confirmation d'atteinte.
- Bascule de scénario (paramètre d'URL) pour la démo.

## A.8 Critères de « done » du proto

- Les 3 types d'objectif sont visibles et distincts.
- Le flux check-in complet fonctionne (sheet → maj timeline + barre).
- La Homepage montre la hiérarchie objectif / test fitness / rituel sans surcharge.
- Testable sur un téléphone (un seul fichier `.dc.html` autonome).

---

# Partie B — Plan de dev

## B.1 Principe de séquençage

On construit **les fondations d'abord**, puis le cœur (= le périmètre proto), puis la complétion, puis le parcours. La règle d'or du concept : **l'onboarding est une collection d'étapes rejouables**, pas un tunnel (brique « étape isolée »).

## B.2 Lot 0 — Fondations (pré-requis transverses)

- **Modèle de données** objectif / relevé / habitude / compartiment (voir B.7).
- **Brique « lancer une étape isolée »** de l'onboarding (section 9 du concept) : socle de l'onboarding progressif, de l'édition, de l'objectif suivant, du re-test.
- **Promotion des trous `[combien]` / `[unité]`** de l'étape 3 en **saisies structurées** (nombre + unité), source unique de la cible.

## B.3 Lot 1 — Cœur suivi (v1) — *périmètre du proto*

- **Carte objectif** Homepage (hiérarchie + repli ; test fitness condensé sur 1 ligne ; rituel dans « Mes séances du jour »).
- **Écran canonique « Mon objectif »** (barre valeur/cible | compte à rebours | barre de temps selon le type ; timeline ; actions).
- **Check-in manuel** (bottom sheet « Faire mon point » → entrée de journal).
- **Baseline in-app** au 1er relevé (« tu pars de combien ? »).
- **Empty states** (jamais d'objectif ; objectif sans relevé).
- **Marquer comme atteint** + déclenchement par valeur = cible.
- **Édition d'objectif** avec compartimentage (petite retouche / changement de cible estampillé / nouvel objectif).

## B.4 Lot 2 — Complétion & relances

- **Onboarding progressif** (bottom sheet, priorité contexte d'abord, plafonds).
- **Jauge « Mon plan sportif » (X/5)** contextuelle (pas une 2e jauge d'en-tête).
- **Relance douce unifiée** = nudge « pose un relevé » qui s'espace avec le silence.
- **Budget de notifications** (coordinateur priorité + plafonds, in-app vs push).

## B.5 Lot 3 — Parcours & cycle

- **Cycle de fin** : célébration, échéance dépassée (bottom sheet à la prochaine ouverture, 4 issues), objectif suivant.
- **Historique « Mon parcours »** (compartiments chronologiques).
- **Graphe ressenti** (v2 du journal, quand assez de relevés).

## B.6 Différé (section 13 du concept)

Auto-alimentation Records / montre, photo de progression, multi-objectifs, suggestion intelligente de l'objectif suivant, suivi d'habitude détaillé (streak / anneau).

## B.7 Modèle de données (cible dev)

```
Objectif
  id, phrase, type(force|poids|endurance|evenement|bouger),
  kind(chiffre|evenement|qualitatif), cible?, unite?, echeance,
  depart?(baseline), statut(actif|atteint|non_atteint|en_pause|archive),
  cree_le, compartiment_id

Entree (journal)
  id, objectif_id, date, valeur?, ressenti(loin|en_voie|presque|jy_suis),
  note?, est_depart(bool), annotation?(ex: "cible 30 -> 20")

Habitude            (découplée du cycle objectif)
  id, geste, ancrage, jours_semaine, heure_rappel?, streak

Parcours/Historique
  liste de compartiments (1 par objectif), chacun = objectif + ses entrées
```

Règles clés : valeurs brutes des relevés **jamais rescalées** ; un changement de cible recalcule le % et **estampille** la timeline ; un changement de nature crée un **nouveau compartiment**.

## B.8 Dépendances & risques

- **Dépendance dure** : Lot 1 suppose le modèle de données (Lot 0) et la cible structurée. Lot 2 suppose la brique « étape isolée » (Lot 0).
- **Seuils à chiffrer** avant dev : relance douce (« ~10-14 j », « ~30 j »), onboarding progressif (« N refus », « 48-72 h »). Laissés ouverts dans le concept, à calibrer.
- **Risque produit n°1** : rétention du journaling manuel. Le proto (A.1) doit valider que le suivi « donne envie de revenir » avant d'investir Lots 2-3.

---

# Partie C — Pont proto → dev

Le mock (Partie A) valide les hypothèses A.1. Selon les retours :

- **Si le cœur convainc** → on lance Lot 0 puis Lot 1 tels quels.
- **Si le sentiment d'avancement est faible** → arbitrer avant dev : renforcer la barre valeur/cible, revoir le ressenti, ou ajouter des paliers visuels.
- **Si le check-in est jugé lourd** → simplifier l'entrée (ressenti seul par défaut, valeur en option) avant de coder le Lot 1.

Le proto est volontairement **jetable** : son rôle est d'informer le Lot 1, pas d'en être la base de code.
