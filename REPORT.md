# Rapport TP3 : CI/CD avec GitHub Actions et Render

Nour Kidoudi, Master DevOps & Cloud Computing, ISET Tozeur

- Dépôt : https://github.com/nourkidoudi/cicd-lab
- Production : https://cicd-lab-248i.onrender.com

## 1. La pipeline avec mes mots

J'ai cloné une petite application Express/TypeScript, puis je l'ai poussée sur mon propre dépôt GitHub. Elle a un script `npm run verify` qui enchaîne trois contrôles : vérification des types, tests et build. Si l'un échoue, les suivants ne s'exécutent pas.

Ensuite, j'ai déployé l'application sur Render (Web Service, plan Free), relié à la branche `main`. Au début, Render déployait à chaque commit, même cassé.

Puis j'ai ajouté un workflow GitHub Actions (`.github/workflows/ci.yml`). À chaque pull request vers `main` et à chaque push sur `main`, GitHub lance le job **Verify** : installation de Node 24, `npm ci`, puis `npm run verify`.

Deux protections complètent la pipeline :

- Dans Render, **Auto-Deploy** est passé de *On Commit* à *After CI Checks Pass* : Render attend que la CI soit verte avant de déployer.
- Dans GitHub, le ruleset **Protect main** impose une pull request et le check **Verify** avant tout merge. Le workflow signale un résultat, mais c'est le ruleset qui bloque.

Le flux final : branche `feature/*`, pull request, Verify vert, merge, nouveau Verify sur `main`, déploiement automatique sur Render.

J'ai testé trois cas :

- **Changement normal** : un message ajouté sur la page d'accueil est passé par la PR puis a été déployé.
- **Test qui échoue** : j'ai ajouté un test pour `/api/version` avant la route. Verify est devenu rouge et le merge était bloqué. Après l'ajout de la route, la même PR est devenue verte et a pu être mergée.
- **Preview** : Render a créé une copie temporaire de l'application pour la pull request, avec sa propre URL, pendant que la production affichait encore l'ancienne version.

## 2. Comparaison avec le TP2 (Coolify webhook vs GitHub Actions + Render)

| Critère | TP2 : Coolify + Forgejo (webhook) | TP3 : GitHub Actions + Render |
|---|---|---|
| Déclencheur | Webhook envoyé à chaque push | Pull request et push sur `main` |
| Vérification avant déploiement | Aucune : le code est déployé tel quel | Typecheck, tests et build avant tout déploiement |
| Protection de `main` | Non | Ruleset : PR et Verify obligatoires |
| Infrastructure | VM locale à gérer moi-même | Service cloud géré (GitHub et Render) |
| Environnement de test | Pas de preview | Preview par pull request |
| Contrôle et coût | Total, tout est chez moi | Moins de contrôle, limites du plan Free (instance qui s'endort) |

Avec le webhook, le déploiement est rapide à mettre en place, mais un commit cassé part directement en production. Avec GitHub Actions et Render, la CI joue le rôle de garde-fou : une erreur est arrêtée avant d'atteindre la production. En contrepartie, je dépends de services externes et de leurs limites.

## 3. Difficultés rencontrées

- **Render ne voyait pas mon dépôt.** Il fallait d'abord installer l'application Render sur mon compte GitHub et lui donner accès au repo `cicd-lab`.
- **Valeurs préremplies incorrectes dans Render.** Il proposait `npm install` et `node index.js`, alors que mon application TypeScript doit utiliser `npm ci && npm run build` et `npm start`. J'ai aussi dû vérifier que le plan **Free** était sélectionné et non le plan payant.
- **Commandes Linux sous Windows.** Les commandes du sujet comme `rm -rf` et `mkdir -p` ne fonctionnent pas telles quelles dans PowerShell, il a fallu les adapter.
- **Premières visites lentes.** L'instance Free de Render s'endort après une période d'inactivité, donc la première requête est plus longue.
- **Comprendre la différence entre workflow et ruleset.** Le workflow produit un résultat, mais c'est le ruleset qui empêche réellement le merge.
