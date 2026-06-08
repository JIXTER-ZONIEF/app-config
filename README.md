# app-config

Manifestes publics de version pour les apps JIXTER. Source de vérité de la **gate de mise à jour forcée au démarrage** (cf doctrine CLAUDE.md / Obsidian `Référence/Doctrine - Mise à jour forcée au démarrage`).

Chaque app lit `https://raw.githubusercontent.com/JIXTER-ZONIEF/app-config/main/<app>.json` au boot et compare son `buildNumber` (`package_info_plus`) à `minSupportedBuild`. Si `buildNumber < minSupportedBuild`, l'app affiche un écran non-dismissible « Mise à jour requise ».

## Champs

| Champ | Type | Rôle |
|-------|------|------|
| `minSupportedBuild` | int | Seuil de blocage. En dessous = bloqué. **Mettre à 1 = inerte** (personne bloqué). |
| `latestBuild` | int | Dernier build publié (info). |
| `androidStoreUrl` / `iosStoreUrl` | string\|null | Liens store (bouton de l'écran de blocage). |
| `forceUpdate` | bool | Kill-switch : force la MAJ même au-dessus du seuil tant que `buildNumber < latestBuild`. |
| `message` | {fr,en,es} | Message affiché (fallback intégré côté app). |

## Pour forcer une montée de version

Relever `minSupportedBuild` au `buildNumber` minimal acceptable, commit + push. Effet en quelques minutes (cache CDN GitHub ~5 min). **Fail-open** : si le manifeste est injoignable, l'app ne bloque pas.
