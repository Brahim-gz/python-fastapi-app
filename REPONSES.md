# TP 6 – GitHub Actions : variables d'environnement, inputs et outputs

## Q1 – Pourquoi une variable n'est-elle pas disponible dans la step qui l'a définie, mais seulement dans les suivantes ?

Quand une step écrit dans `$GITHUB_ENV`, le runner GitHub Actions met à jour l'environnement après la fin de cette step. Le système traite donc le fichier à la fin de la step, puis fournit les nouvelles variables aux steps suivantes du même job. La step qui contient `echo "APP_NAME=my-app" >> "$GITHUB_ENV"` ne peut donc pas utiliser `APP_NAME` immédiatement avec `$APP_NAME`.

## Q2 – Quelle est la portée de GITHUB_ENV ?

`GITHUB_ENV` permet de transmettre une variable aux steps suivantes d'un même job. Elle ne crée pas une variable globale pour tous les jobs du workflow. Chaque job possède ses propres processus et son propre environnement d'exécution : même avec `needs: build`, le job `verify-env` ne reçoit pas `APP_NAME`, qui reste donc vide ou indisponible.

## Q3 – Que se passe-t-il si on utilise `>` au lieu de `>>` dans GITHUB_OUTPUT ?

L'opérateur `>` ouvre le fichier de sortie en écriture puis l'efface avant d'y écrire. Avec `>>`, le runner ajoute une nouvelle ligne à la fin du fichier sans supprimer les sorties déjà présentes. Si plusieurs sorties sont écrites avec `>` dans le même step, une sortie peut donc écraser les précédentes ; `>>` est nécessaire pour les conserver toutes.

## Q4 – Pourquoi GITHUB_ENV ne suffit-il pas pour communiquer entre jobs ?

`GITHUB_ENV` est limité à un job : ses variables ne sont pas automatiquement transférées aux autres jobs. Pour communiquer entre jobs, il faut écrire une valeur dans `$GITHUB_OUTPUT` depuis une step, déclarer cette step avec un `id`, puis exposer sa valeur au niveau du job avec `outputs`. Enfin, le job dépendant utilise le contexte `needs`, par exemple `${{ needs.build.outputs.full_version }}`, pour lire la sortie.

## Q5 – Que se passe-t-il si on retire `needs: build` du job deploy ?

Sans `needs: build`, `deploy` n'attend plus la fin de `build` et les deux jobs peuvent s'exécuter en parallèle. Comme `build` n'est plus une dépendance déclarée, l'expression `${{ needs.build.outputs.full_version }}` n'est plus disponible et la valeur affichée dans `deploy` sera vide ou « unavailable ». Le workflow perd aussi la relation permettant au job dépendant de récupérer cette sortie.
