# Steam Achievement Manager

**Ceci est le [fork d'Endymi0n74](https://github.com/Endymi0n74/SteamAchievementManager)
de [gibbed/SteamAchievementManager](https://github.com/gibbed/SteamAchievementManager).**

📦 **[Télécharger la dernière release](https://github.com/Endymi0n74/SteamAchievementManager/releases/latest)**
· 🇬🇧 **[README in English](README.md)**

Steam Achievement Manager (SAM) est une application légère et portable servie à gérer les
succès et les statistiques de la plateforme de jeux PC Steam. Elle exige le
[client Steam](https://store.steampowered.com/about/), un compte Steam et un accès réseau.
Steam doit être lancé et l'utilisateur connecté.

Release actuelle : **7.1.0** (les releases open-source d'amont s'arrêtent à 7.0.x).

## Pourquoi un fork ?

L'amont est en congé de maintenance : pull requests et correctifs s'accumulent sans être
mergés. Ce fork rassemble le travail de stabilité nécessaire pour garder SAM utilisable
aujourd'hui, le publie dans une release GitHub classique, et ajoute la **mise à jour
automatique** pour que vous n'ayez pas à surveiller le dépôt.

SAM ne parle toujours qu'à votre **client Steam local**. La seule chose que le fork
télécharge depuis GitHub est son propre paquet de mise à jour.

## Nouveautés de 7.1.0 (par rapport à l'amont 7.0.x)

### Mise à jour automatique

`SAM.Picker` vérifie les releases de ce dépôt au démarrage et propose d'installer ce qui
est plus récent :

- la version et l'archive viennent des releases GitHub de ce dépôt ;
- l'archive est vérifiée avec le **SHA-256 publié avant qu'on ne touche à quoi que ce soit** ;
- un processus auxiliaire attend la sortie de `SAM.Picker` (et de `SAM.Game`, s'il est
  ouvert), extrait par-dessus le dossier de l'application — en refusant d'écrire en dehors —
  puis relance l'outil ;
- un échec affiche ce qui s'est passé au lieu de laisser une installation à moitié
  mise à jour.

Un bouton **Check for updates** dans la barre d'outils relance le même contrôle à la
demande. La mise à jour ne remplace que des fichiers locaux : rien dans Steam (succès ou
statistiques) n'est lu ni écrit par elle.

### Correctifs

- **#593 / #594 / #625 — « la moitié des jeux n'a aucun succès »** : le message
  « Failed to load schema. » s'explique désormais (schéma absent de l'appcache de Steam et
  comment le faire récupérer, fichier illisible, ou aucune statistique pour cette app).
- **#466 — icônes derrière un proxy** : `SAM.Game` construisait son téléchargeur d'icônes
  sans le TLS 1.2 ni les identifiants de proxy authentifié utilisés par le sélecteur, si
  bien que les icônes de succès ne se téléchargeaient jamais. Il construit maintenant le
  téléchargeur exactement comme le sélecteur.
- **#468 / #410 / #581 / #592 — « tout est possédé »** : quand un outil de falsification
  de licences fait dire à Steam que vous possédez plus de 5000 jeux, la barre de statut
  indique que la liste n'a aucun sens.
- **#405 / #458 / #429 / #599 — résultat de l'enregistrement** : le succès ou l'échec
  vient du callback `UserStatsStored` de Steam (délai 15 s) au lieu d'être supposé, et un
  enregistrement échoué conserve vos modifications au lieu de les jeter.
- **#380 / #432 — contraintes de statistiques** : les règles minimum/maximum,
  « incrémente seulement » et delta maximum du schéma Steam sont appliquées et expliquées,
  et les statistiques de type taux moyen sont signalées (Steam n'accepte pour elles que
  des mises à jour de session).
- **#495 — tri** : cliquer un en-tête de colonne trie les succès par nom, état de
  déverrouillage ou date.
- **#531 — langue d'affichage** : un sélecteur de langue dans la barre d'outils lit les
  succès et les statistiques dans n'importe la langue que le jeu fournit réellement (la
  langue par jeu de Steam est la valeur par défaut, pas la langue du client Steam), et la
  barre de statut indique la langue utilisée.
- **#491 / #435 / #424 / #579 — échecs de démarrage silencieux** : interfaces Steam
  manquantes, échecs de callbacks et erreurs non traitées affichent désormais quelle étape
  a échoué au lieu de fermer sans un mot.
- **#601 / #466 / #618 — téléchargement de la liste de jeux** : TLS 1.2 forcé, identifiants
  de proxy respectés, erreurs de téléchargement lisibles, et courses d'exécution
  liste/logos/recherche corrigés.
- **#421 — « Invalid value » sur des lignes intactes** : les valeurs de statistiques
  renvoyées en nombre sont à nouveau acceptées, et les statistiques non prises en charge
  indiquent laquelle pose problème au lieu de planter.
- Les fichiers de schéma avec des clés dupliquées ou des types de statistiques inconnus
  n'interrompent plus le chargement, les modifications en attente survivent à un
  enregistrement échoué, et la liste de jeux ne s'arrête plus en cours de rafraîchissement
  (dialogue d'erreur à chaque démarrage, boutons « Refresh Games »/« Add Game » bloqués).
- La compilation de la solution entière fonctionne avec un simple
  `dotnet build SAM.sln -c Release` (sans contournement MSBuild).

## Téléchargement et installation

1. Téléchargez `SteamAchievementManager-7.1.0.zip` depuis la page des
   [releases](https://github.com/Endymi0n74/SteamAchievementManager/releases/latest).
2. Extrayez-le là où vous gardez SAM.
3. Lancez `SAM.Picker.exe`.

Le fichier `.sha256` publié à côté de l'archive est celui que le programme de mise à jour
vérifie : gardez-le à côté du zip si vous voulez que le chemin automatique fonctionne.
Rien d'autre à installer : SAM est portable (.NET Framework 4.8 fait partie de Windows
10/11).

## Compilation

```powershell
dotnet build SAM.sln -c Release
```

La solution ne propose que des configurations `x86` (le jeu communique avec le client
Steam 32 bits), il n'y a donc pas de build `Any CPU`. Les binaires sont écrits dans
`upload\`.

## Tests

Les tests unitaires vivent dans `SAM.Game.Tests` (xunit) et couvrent la logique pure :
analyse du schéma (KeyValue), contraintes de statistiques, messages d'erreur de Steam et
logique de mise à jour.

```powershell
dotnet test SAM.sln -c Release
```

## Périmètre de ce fork

- Les correctifs et fonctionnalités sont développés et publiés **ici** ; rien n'est proposé
  à l'amont (aucune pull request sur `gibbed/SteamAchievementManager`).
- La numérotation suit le fork : **7.1.0** signifie « 7.0.x plus les changements ci-dessus ».
- Les titres des fenêtres dérivent du version d'assembly : ils ne peuvent plus dériver.

## Crédits et licence

- Projet original : [gibbed/SteamAchievementManager](https://github.com/gibbed/SteamAchievementManager)
  de Rick (gibbed), publié sous licence [zlib](LICENSE.txt). Ce fork conserve cette licence
  et se déclare version altérée, comme la licence l'exige.
- La plupart (si ce n'est toutes) des icônes viennent de la collection
  [Fugue Icons](https://p.yusukekamiyamane.com/).
