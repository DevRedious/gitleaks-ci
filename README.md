# gitleaks-ci

Workflow GitHub Actions **réutilisable** qui recherche des secrets (clés d'API, tokens, mots de passe)
avec [Gitleaks](https://github.com/gitleaks/gitleaks) dans les dépôts de DevRedious et de ses
organisations.

## Fonctionnement

Chaque dépôt contient un petit fichier `.github/workflows/gitleaks.yml` (voir [`modele/`](modele/gitleaks.yml))
qui appelle `.github/workflows/gitleaks.yml@v1` de ce dépôt :

| Événement | Ce qui est analysé | Effet |
|---|---|---|
| `push` | les commits poussés | le run échoue si un secret est trouvé |
| `pull_request` | les commits de la PR | la PR affiche un échec |
| chaque lundi, ou lancement manuel | **tout l'historique** | détecte les fuites anciennes |

- Les valeurs sont **masquées** (`--redact`) dans les logs et le résumé du run.
- Le binaire Gitleaks est téléchargé depuis les releases officielles et **vérifié par sha256**.
- `.gitleaks.toml` et `.gitleaksignore` du dépôt analysé sont pris en compte.

## Déploiement automatique

Le fichier est ajouté automatiquement à chaque dépôt qui ne l'a pas (y compris les nouveaux) par
l'outil local `gitleaks-ci-sync`, lancé chaque jour par un timer systemd. Le supprimer ne sert à
rien : il sera remis.

## Quand le run échoue

1. Retirer le secret du code.
2. S'il a été poussé : le considérer comme **compromis** et le régénérer.
3. Faux positif avéré seulement : ajouter l'empreinte affichée dans le résumé du run à
   `.gitleaksignore`.

## Mettre à jour

Modifier `.github/workflows/gitleaks.yml` ici, puis déplacer l'étiquette `v1` : tous les dépôts en
profitent au run suivant.
