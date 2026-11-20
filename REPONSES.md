**Q1. Pourquoi la variable n'est-elle pas disponible dans la step qui l'a définie, mais seulement dans les suivantes ?**
`GITHUB_ENV` est un fichier local. Le runner lit ce fichier uniquement à la fin de l'étape pour injecter les variables dans l'environnement des étapes suivantes. La variable n'est donc pas encore dans le shell de l'étape active.

**Q2. Ajoutez un second job verify-env avec une step Print APP_NAME et tentez d'y afficher $APP_NAME. Que constatez-vous ? Quelle est donc la portée de GITHUB_ENV ?**
La variable `$APP_NAME` s'affiche vide dans le second job. La portée de `GITHUB_ENV` est locale au job courant. Chaque job s'exécutant sur une machine (runner) isolée, ils ne partagent pas leurs variables d'environnement.
