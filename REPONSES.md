# TP 7 – GitHub Actions : actions tierces et actions locales

## Q1 – Pourquoi actions/checkout n'a-t-il pas besoin de with alors que setup-node en a besoin ?

`actions/checkout` récupère par défaut le dépôt courant ainsi que la référence appropriée pour l'événement déclencheur, donc son utilisation minimale ne nécessite pas de bloc `with`. `actions/setup-node` doit, quant à elle, recevoir une configuration explicite dans ce TP : `node-version: 20` indique la version de Node.js à installer. Les actions peuvent avoir des entrées facultatives ou obligatoires, selon leurs valeurs par défaut.

## Q2 – Pourquoi le job verify ne retrouve-t-il pas report.txt sans l'artefact ?

Chaque job GitHub Actions s'exécute dans son propre environnement et possède son propre système de fichiers. Le fichier créé dans `build` n'est donc pas automatiquement visible dans `verify`, même si les deux jobs sont liés par `needs`. L'artifact `build-report` sert à transférer explicitement `report.txt` entre ces deux environnements.

## Q3 – Pourquoi faut-il épingler une version (@v4) plutôt que @main ?

`@main` pointe vers une branche mobile : son contenu peut changer sans aucune modification du workflow. `@v4` désigne un tag de version majeure plus stable et prévisible, mais ce tag peut être déplacé. Pour une sécurité de production plus forte, il faut épingler une action à un SHA de commit complet, qui représente une révision immuable.

## Q4 – Le badge « créateur vérifié » garantit-il qu'une action est sûre ?

Non. Le badge confirme seulement que GitHub a vérifié l'identité du créateur ou de l'organisation associée à l'action. Il ne garantit pas que le code est exempt de vulnérabilités, que toutes ses dépendances sont sûres ou que l'action ne pourra jamais exposer de secrets. Une vérification d'identité ne remplace pas un audit de sécurité.

## Q5 – Citez deux vérifications à faire avant d'utiliser une action inconnue en production.

Il faut d'abord inspecter le code source, ses dépendances, les permissions demandées et la gestion des secrets afin de détecter un accès excessif ou une fuite potentielle. Il faut ensuite examiner l'activité du dépôt, les releases, les issues et les avis de sécurité, puis épingler l'action à un SHA vérifié pour éviter une mise à jour inattendue.

## Q6 – Pourquoi actions/checkout doit-il s'exécuter avant l'appel d'une action locale ?

Une action locale est située dans le dépôt, par exemple `./.github/actions/build-info`. GitHub Actions ne peut résoudre son chemin que si les fichiers du dépôt ont déjà été placés dans l'espace de travail du runner avec `actions/checkout`. Chaque job indépendant qui utilise une action locale doit effectuer son propre checkout.

## Q7 – Pourquoi shell: bash est-il obligatoire sur les steps run d'une action composite ?

Dans une action composite, chaque step qui contient `run` doit préciser explicitement l'interpréteur à utiliser. `shell: bash` indique que les commandes sont exécutées par Bash dans ce TP ; ce n'est pas le seul shell possible. Cette déclaration évite toute ambiguïté sur la façon dont les commandes seront interprétées sur le runner.
