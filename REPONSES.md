**Q1. Pourquoi la variable n'est-elle pas disponible dans la step qui l'a définie, mais seulement dans les suivantes ?**
`GITHUB_ENV` est un fichier local. Le runner lit ce fichier uniquement à la fin de l'étape pour injecter les variables dans l'environnement des étapes suivantes. La variable n'est donc pas encore dans le shell de l'étape active.

**Q2. Ajoutez un second job verify-env avec une step Print APP_NAME et tentez d'y afficher $APP_NAME. Que constatez-vous ? Quelle est donc la portée de GITHUB_ENV ?**
La variable `$APP_NAME` s'affiche vide dans le second job. La portée de `GITHUB_ENV` est locale au job courant. Chaque job s'exécutant sur une machine (runner) isolée, ils ne partagent pas leurs variables d'environnement.

**Q3. Que se passe-t-il si vous écrivez > au lieu de >> dans GITHUB_OUTPUT alors que deux outputs sont définis dans la même step ?**
L'opérateur `>` écrase le contenu du fichier. Si on l'utilise pour plusieurs outputs dans la même étape, seul le dernier output sera conservé (les précédents effaceront les autres). L'opérateur `>>` permet d'ajouter (append) à la suite du fichier sans perdre les données existantes.

**Q4. Pourquoi GITHUB_ENV ne suffit-il pas ici, alors que GITHUB_OUTPUT fonctionne entre jobs ?**
`GITHUB_ENV` a une portée strictement limitée au job en cours (et à son runner). Pour passer une donnée entre deux jobs isolés, il faut utiliser `GITHUB_OUTPUT` au niveau de l'étape, puis le déclarer dans la section `outputs:` du job source. GitHub Actions se charge alors de le transmettre au job cible via le contexte `needs`.

**Q5. Que se passe-t-il si vous retirez needs: build du job deploy ?**
Si on retire `needs: build`, le job `deploy` s'exécutera en parallèle du job `build` au lieu de l'attendre. Par conséquent, il ne pourra pas récupérer la valeur de l'output (la variable sera vide ou une erreur surviendra car la ressource ne sera pas prête).
