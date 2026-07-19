# app-config

Manifestes publics de version pour les apps JIXTER. Source de vérité de la **gate de mise à jour forcée au démarrage** (cf doctrine CLAUDE.md / Obsidian `Référence/Doctrine - Mise à jour forcée au démarrage`).

Chaque app lit `https://raw.githubusercontent.com/JIXTER-ZONIEF/app-config/main/<app>.json` au boot et compare son `buildNumber` (`package_info_plus`) à `minSupportedBuild`. Si `buildNumber < minSupportedBuild`, l'app affiche un écran non-dismissible « Mise à jour requise ».

## Champs

| Champ | Type | Rôle |
|-------|------|------|
| `minSupportedBuild` | int | Seuil de blocage. En dessous = bloqué. **Mettre à 1 = inerte** (personne bloqué). |
| `latestBuild` | int | Dernier build publié (info). |
| `androidStoreUrl` / `iosStoreUrl` | string\|null | Liens store (bouton de l'écran de blocage). |
| `forceUpdate` | bool | Kill-switch : force la MAJ même au-dessus du seuil tant que `buildNumber < latestBuild`. **⚠️ iOS : voir garde-fou ops#765 ci-dessous avant de remettre à `true`.** |
| `message` | {fr,en,es} | Message affiché (fallback intégré côté app). |

## Pour forcer une montée de version

Relever `minSupportedBuild` au `buildNumber` minimal acceptable, commit + push. Effet en quelques minutes (cache CDN GitHub ~5 min). **Fail-open** : si le manifeste est injoignable, l'app ne bloque pas.

## ⚠️ Garde-fou `forceUpdate` (ops#765)

**Ne jamais remettre `forceUpdate: true` sur une app dont le build iOS en prod ne porte pas l'exemption iOS du code (`shouldBlock(..., isIOS: true)`).**

Raison : `latestBuild` est bumpé par le workflow **Android** sur un manifeste **partagé** par les deux plateformes. iOS étant structurellement en retard (deploy-ios n'atteint les users qu'après un submit manuel via `submit-ios-for-review.yml`), `forceUpdate: true` forçait iOS vers un build indisponible → écran de blocage non-dismissible → app inutilisable (6 apps brickées ~4 j, ops#765).

Le fix code (exemption iOS dans `UpdateGateService.shouldBlock`) est sur `develop` pour les 10 apps. Avant de ré-armer `forceUpdate: true` sur une app :
1. vérifier que l'exemption iOS est **sur `master`** de cette app **et** servie en prod iOS (build soumis + READY_FOR_SALE) ;
2. sinon, laisser `forceUpdate: false` et s'appuyer uniquement sur `minSupportedBuild` (plancher dur, précis, sans risque de brick) pour forcer une montée.

Le nudge `forceUpdate`/`latestBuild` n'a de valeur que côté **Android** (iOS ne reçoit pas le dernier build de façon garantie). En pratique, `minSupportedBuild` suffit pour forcer une montée de version.
