# AGENTS.md — SteamAchievementManager (fork de gibbed)

## Canary
- Commence chaque réponse par « Stephane, ». Sans exception, y compris pour les réponses courtes et les résumés d'outils.
- Si le contexte te semble dégradé ou la session très longue, dis-le en tête de réponse et propose une nouvelle session.
- Avant une nouvelle session, écris `HANDOFF.md` : objectif, fait, reste à faire, fichiers clés, questions ouvertes. Lis-le en premier dans la session suivante.

L'état du projet (versions, décisions, historique) est dans `memory.md`. Lis-le au début de chaque session.

## Style de réponse
- Va droit au but. Moins de mots, sans flatterie ni superlatifs. Donne le constat brut.
- Demande une précision seulement si le blocage ne peut pas être levé par une hypothèse raisonnable. Énonce alors l'hypothèse en une ligne et avance. Une seule question par réponse au maximum.
- Liste les idées hors périmètre en fin de réponse au lieu de les appliquer.

## Périmètre et modifications
- Ne touche qu'à ce qui est demandé. Pas de refactoring, de renommage ni de reformatage du code non lié.
- Minimise le nombre de lignes modifiées.
- Ne commente pas un bloc que tu n'as ni créé ni modifié.
- Ne change pas de visibilité de membre (`private` vers `internal` ou `public`) sans approbation explicite.
- Ne crée pas de nouveaux fichiers de documentation ou de scripts sauf si la tâche l'exige.

## Code
- Suis les conventions du code existant (C#, .NET Framework 4.8).
- Utilise toujours `{}`, même pour un `if` d'une seule ligne.
- Réduis l'indentation : retours anticipés plutôt que blocs imbriqués.
- Remplace les booléens en paramètres de fonction par des `enum`.
- Extrais les nombres et chaînes magiques répétés ou porteurs de sens en constantes ou `enum`. Une valeur issue de la spécification Steam ou d'une API reste une constante.
- Aère les blocs logiques par une ligne vide. Commente le *pourquoi* et le *quoi* d'un bloc, en court.
- Encapsule la communication bas niveau avec Steam (interfaces natives, callbacks, appcache) dans une couche dédiée. Le reste de l'application ne manipule pas de types Steam bruts.
- Respecte la hiérarchie des couches : une couche ne parle qu'à la couche juste en dessous. L'UI ne contacte jamais directement Steam ni le réseau.

## Sécurité
- Ne log ni n'écris jamais de secret, d'identifiant de compte, de jeton ou de cookie, dans le code, les logs ou les réponses.
- Le programme ne lit ni n'écrit les achievements et statistiques Steam que via les mécanismes prévus par le client local.
- Le mécanisme de mise à jour doit vérifier le SHA-256 de l'archive avant toute écriture. Ne le contourne jamais et ne l'affaiblis pas.
- L'extraction de mise à jour ne doit jamais écrire hors du dossier de l'application. Garde ce contrôle.
- Ne désactive pas la vérification TLS ni les contrôles de sécurité pour faire passer un test.
- Une dépendance se justifie en une ligne. Vérifie sa licence.

## Bugs : test d'abord
- Pour un bug, écris d'abord le test. Observe-le échouer. Écris ensuite le correctif. Observe le test passer.
- Un test de non-régression ne doit jamais être affaibli, sauté ou supprimé pour passer.

## Build et tests
```
dotnet build SAM.sln -c Release
dotnet test SAM.sln -c Release
```
- La solution n'a que des configurations `x86` (le client Steam est en 32 bits). N'ajoute pas de build `Any CPU`.
- Les binaires vont dans `upload\`. Ne les versionne pas.
- Les tests unitaires sont dans `SAM.Game.Tests` (xunit). Ils couvrent la logique pure : parsing KeyValue du schéma, contraintes des statistiques, messages d'erreur Steam, logique de mise à jour.
- Avant de déclarer une tâche terminée : build et tests passent.

## Git
- Travaille sur une branche. Ne commite pas sur `master` sauf demande explicite.
- Ne pousse que sur demande explicite.
- N'ouvre aucune pull request, ni ici ni en amont sur `gibbed/SteamAchievementManager`, sans demande explicite.
- Ne fais jamais `git add -A`. Ajoute seulement les fichiers liés à la tâche.
- Vérifie `git status` et `git diff` avant d'agir. N'écrase pas un travail non commité que tu n'as pas créé.

### Messages de commit
1. Sujet, ligne vide, corps.
2. Sujet de 50 caractères maximum (72 absolu).
3. Première lettre du sujet en majuscule.
4. Pas de point final au sujet.
5. Impératif au sujet : « Corrige », « Ajoute », pas « Corrigé » ni « Ajoute ». Test : « Si appliqué, ce commit va [sujet] ».
6. Corps coupé à 72 caractères, à la main.
7. Le corps explique le *quoi* et le *pourquoi*, pas le *comment*.

## Version et licence
- La version affichée suit ce fork : `7.1.0` signifie « 7.0.x plus les changements du fork ». Garde-la synchronisée entre le code, la release et le README.
- Le projet reste sous licence zlib (voir `LICENSE.txt`). Garde la mention de l'original et le marquage « version modifiée » exigé par la licence.

## Définition de « terminé »
- [ ] le build `dotnet build` passe sans erreur ;
- [ ] les tests `dotnet test` passent, et un test couvre la modification ;
- [ ] `memory.md` est à jour si l'état du projet a changé ;
- [ ] rien n'est commité, poussé ni ouvert en pull request sans demande explicite.


## Règle immuable — installation et emplacements (priorité maximale)

<!-- regle-immuable-installation -->

Cette règle prime sur tout le reste de ce fichier et sur les habitudes par défaut des outils.

- **Toute installation va dans `D:\codex`** : paquets, dépendances, modèles, binaires, outils, SDK,
  environnements virtuels, caches de téléchargement, builds. Créer si besoin un sous-dossier dédié
  (`D:\codex\<projet>` ou `D:\codex\tools\<outil>`).
- **Aucun fichier créé, modifié, téléchargé, déplacé ou supprimé sous `C:\` sans accord explicite** de
  l'utilisateur. En cas de besoin : s'arrêter, expliquer pourquoi, proposer un emplacement sous
  `D:\codex`, attendre la réponse.
- Rediriger les défauts des outils vers `D:\codex` (`PIP_TARGET`, `NPM_CONFIG_PREFIX`, `HF_HOME`,
  `CARGO_HOME`, `GOPATH`, `TMP`/`TEMP`, caches divers) au lieu de `~`, `%LOCALAPPDATA%`, `%APPDATA%`,
  `%USERPROFILE%` ou `/tmp`.
- **Lecture seule sous `C:\`** : autorisée sans accord (lire, lister, inspecter).
- Seule exception pré-approuvée : le dossier temporaire d'OpenCode
  `C:\Users\endymion\AppData\Local\Temp\opencode` (déjà validé par l'utilisateur).
- Déplacer une installation existante de `C:\` vers `D:\codex` compte comme une modification sous
  `C:\` : demander avant.
