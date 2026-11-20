# TP 7 – Réponses (Q1, Q2, Q6, Q7)

**Q1. Pourquoi `actions/checkout` n'a-t-il pas besoin de `with` alors que `setup-node` en a besoin ?**
Tous les inputs de `actions/checkout` ont une valeur par défaut adaptée au cas courant : il clone le dépôt courant (`repository: ${{ github.repository }}`), sur le commit qui a déclenché le workflow (`ref`), avec le `GITHUB_TOKEN`. `with` ne sert qu'à changer ce comportement. Pour `setup-node`, aucune version de Node.js n'est choisie par défaut : si on ne précise pas `node-version`, l'action n'installe rien et on garde la version déjà présente sur le runner. Pour obtenir une version précise et reproductible, il faut la passer avec `with`.

**Q2. Pourquoi le job `verify` ne retrouve-t-il pas `report.txt` sans l'artefact, alors que les deux jobs partagent le même dépôt ?**
Chaque job tourne sur une machine virtuelle neuve et isolée. `report.txt` est généré pendant l'exécution du job `build` : il n'est pas commité, il n'existe que sur le disque du runner de `build`, qui est détruit à la fin du job. Le job `verify` ne partage que le dépôt Git, et il ne fait même pas de checkout. Pour transmettre un fichier entre jobs, il faut le stocker côté GitHub avec `actions/upload-artifact`, puis le récupérer avec `actions/download-artifact`. `needs: build` garantit que l'artefact existe déjà quand `verify` le télécharge.

**Q6. Pourquoi `actions/checkout` doit-il s'exécuter avant l'appel d'une action locale ?**
Une action locale (`uses: ./.github/actions/build-info`) est référencée par un chemin dans le dépôt. Au démarrage du job, le runner est vide : le fichier `action.yml` n'existe sur le disque qu'une fois le dépôt cloné par `actions/checkout`. Sans checkout, le runner ne trouve pas l'action et le job échoue (`Can't find 'action.yml'...`). Les actions publiées (`actions/setup-node@v4`, etc.) n'ont pas ce problème, car GitHub les télécharge lui-même depuis leur dépôt.

**Q7. Pourquoi `shell: bash` est-il obligatoire sur les steps `run` d'une action composite ?**
Dans un workflow, le shell par défaut est déduit de l'OS du runner, ou défini par `defaults.run.shell`. Une action composite est faite pour être réutilisée dans n'importe quel workflow, sur n'importe quel runner (Linux, Windows, macOS). Elle n'hérite pas de ces valeurs par défaut. GitHub impose donc de déclarer explicitement le shell de chaque step `run`, pour que l'action se comporte de façon identique partout. Sans `shell`, le workflow est refusé dès la lecture de l'action.

## Partie 4 – Action Docker vs action composite

Mesures relevées sur le run #1 (commit `79ff67d`) :

| | Step de l'action | Job complet |
|---|---|---|
| **Action composite** (`Run build info action`) | **0 s** | – (fait partie de `build`, 9 s, dont 4 s pour `Setup Node.js`) |
| **Action Docker** (`Run docker hello action`) | **2 s** | **6 s** pour `docker-hello`, alors que le job ne contient qu'un checkout et l'action |

- **Action composite** : les steps s'exécutent directement dans le shell du runner, sans rien préparer.
- **Action Docker** : au démarrage du job, le runner doit d'abord construire l'image à partir du `Dockerfile`, ce qui inclut le téléchargement de `alpine:3.20` (*Build container for action use*). Ensuite, à chaque appel, il démarre un conteneur. Un simple `echo` prend donc plusieurs secondes, contre un temps quasi nul en composite.

Conclusion : une action Docker fournit un environnement totalement maîtrisé, quels que soient les outils installés sur le runner. En contrepartie, elle est plus lente et ne fonctionne que sur des runners Linux. Une action composite est quasi instantanée et portable, mais elle dépend des outils déjà présents sur le runner.

Remarque : dans `action.yml`, `args` est écrit en liste YAML (`- ${{ inputs.who }}`) plutôt qu'en `[${{ inputs.who }}]`. Dans une liste entre crochets, l'accolade `{` serait lue par YAML comme le début d'un dictionnaire.
