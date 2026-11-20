# TP 7 – Notes de sécurité (Q3, Q4, Q5)

## Observations sur `actions/setup-node`

- **Créateur vérifié** : l'action est publiée par l'organisation `actions` (GitHub). Sur le Marketplace, elle porte le badge *Verified creator*.
- **Étoiles** : environ 5 000 étoiles et plus de 1 700 forks (relevé le 20/11/2026).
- **Code source** : le code TypeScript se trouve dans le dossier `src/`. Le dossier `dist/` contient la version compilée, celle qui est réellement exécutée (`action.yml` pointe vers `dist/setup/index.js`).

## Test `@main` vs `@v4`

Le workflow a été modifié temporairement pour utiliser `actions/checkout@main` (commit *test temporaire de actions/checkout@main*), puis remis sur `actions/checkout@v4` (commit *retour a actions/checkout@v4*). Les deux fonctionnent, mais `@main` exécute le code de la branche tel qu'il est au moment du run, et il peut donc changer d'un run à l'autre.

**Q3. Pourquoi faut-il épingler une version (`@v4`) plutôt que `@main` ?**
`@main` pointe vers le dernier commit de la branche principale de l'action. Chaque nouveau commit poussé par le mainteneur est exécuté automatiquement dans notre pipeline, sans relecture de notre part. Cela pose plusieurs problèmes :
- **Stabilité** : un changement incompatible ou un bug en cours de développement casse notre workflow du jour au lendemain, sans qu'on ait rien modifié.
- **Reproductibilité** : deux runs du même commit peuvent exécuter un code d'action différent.
- **Sécurité** : si le dépôt de l'action est compromis, le code malveillant est exécuté immédiatement, avec accès au `GITHUB_TOKEN` et aux secrets du job.

`@v4` limite les changements aux versions de la v4, qui restent compatibles entre elles. Pour une sécurité maximale, on épingle même le **SHA complet du commit** (`actions/checkout@<sha40>  # v4.x.x`). Un tag peut en effet être déplacé par le mainteneur ou par un attaquant, alors qu'un SHA est immuable.

**Q4. Le badge « créateur vérifié » garantit-il qu'une action est sûre ? Justifiez.**
Non. Le badge garantit seulement **l'identité** de l'éditeur : GitHub a vérifié que l'organisation est bien celle qu'elle prétend être, par exemple que le domaine lui appartient. Il ne dit rien sur le **contenu** de l'action :
- GitHub n'audite pas le code des actions, vérifiées ou non.
- Une organisation vérifiée peut publier une version boguée ou vulnérable.
- Son compte ou son dépôt peut être compromis. C'est arrivé avec `tj-actions/changed-files` en 2025 : des tags existants ont été redirigés vers du code qui affichait les secrets dans les logs.

Le badge est un indice de confiance, pas une preuve de sécurité.

**Q5. Citez deux vérifications à faire avant d'utiliser, en production, une action d'un fournisseur inconnu.**
1. **Lire le code source et `action.yml`** : vérifier ce que l'action exécute réellement (`dist/` compris), les commandes réseau (`curl`, envoi de données vers l'extérieur), l'usage des secrets et du token, et les dépendances qu'elle installe.
2. **Évaluer le dépôt et sa maintenance** : activité récente, nombre d'utilisateurs (étoiles, forks), issues et alertes de sécurité ouvertes, releases signées et versionnées, et présence d'une politique de sécurité (`SECURITY.md`).

Ensuite, en production : **épingler l'action sur un SHA complet** et donner au job des **permissions minimales** (`permissions: contents: read`), pour limiter les dégâts si l'action s'avérait malveillante. On peut aussi forker l'action dans sa propre organisation pour contrôler les mises à jour.
