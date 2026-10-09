# memory.md — Steam Achievement Manager (fork SAM)

Mémoire du fork : historique, décisions structurantes, leçons apprises. À lire en
complément de [`README.md`](README.md) (anglais) et [`README.fr.md`](README.fr.md).

## En bref

- **Quoi** : fork de [`gibbed/SteamAchievementManager`](https://github.com/gibbed/SteamAchievementManager)
  — SAM, outil Windows (.NET Framework 4.8, **x86**) qui gère les succès et statistiques
  Steam via le client local.
- **Chemin** : `D:\Codex\SteamAchievementManager` — remotes : `origin` = **amont (jamais
  poussé)**, `fork` = `https://github.com/Endymi0n74/SteamAchievementManager` (tout y est
  poussé ; **3 PR ouvertes sur l'amont** : **#643** build + correctifs, **#644** tests
  xunit, **#645** auto-update — ouvertes le 2026-10-09, préparées par Kumo, agent de
  vibecoding OpenCode, avec Endymi0n74 ; aucun suivi demandé).
- **Statut** : ⏸ **Clos le 2026-10-09** — sujet terminé pour l'instant, à **ne pas
  mélanger** avec les deux autres sujets du workspace : `D:\Codex\tsbak-gui` (**task-gui**)
  et l'outillage **REA** (`D:\Codex\tools\memory.md`). Aucune dépendance ni logique
  commune ; ne pas rouvrir sans demande explicite.
- **Dernière version** : **7.1.0** (2026-10-09) — release GitHub `7.1.0`
  (`SteamAchievementManager-7.1.0.zip` + `.zip.sha256`), tag = `master` = `455a0cd`.
  Amont : 7.0.x.
- **Grande nouveauté** : **mise à jour automatique** (`SAM.Picker/UpdateChecker.cs`) —
  contrôle silencieux au démarrage + bouton « Check for updates », vérification SHA-256
  avant extraction, helper PowerShell d'installation, relance automatique.
- **Tests** : `SAM.Game.Tests` (xunit, net48, x86) — **68 tests**, `dotnet test SAM.sln -c Release`.
- **État** : release publiée, testée de bout en bout (7.0.0 → 7.1.0), fork à jour (12
  commits devant l'amont).

## Dates clés

- **2026-10-08** — Correctifs de stabilité et de logique : build `dotnet` sans
  contournement MSBuild, robustesse du schéma (clés dupliquées, types inconnus) et des
  éditeurs, contraintes de statistiques du schéma (#380/#432), API/picker et courses
  d'exécution (liste, logos, recherche), résultat d'enregistrement par callback
  `UserStatsStored` (#405/#458/#429/#599), tri des succès (#495), langue d'affichage
  (#531), round-trip des valeurs de stats + drapeaux `AverageRate`, **projet de tests
  `SAM.Game.Tests`** (68 tests à la fin de la journée).
- **2026-10-09 (matin)** — **Auto-update** : `UpdateChecker.cs`, bouton dans le
  `Designer`, câblage `GamePicker`, `InternalsVisibleTo` + 17 tests, titres de fenêtres
  dérivés de la version d'assembly, bump **7.0.0 → 7.1.0** (3 `.csproj`).
- **2026-10-09** — Trois diagnostics ajoutés (commit `ecf2787`) : raison détaillée du
  schéma manquant (`DescribeSchemaFailure`, #593/#594/#625), téléchargeur d'icônes avec
  TLS 1.2 + proxy authentifié dans `SAM.Game` (#466), avertissement si `_Games.Count > 5000`
  (#468/#410/#581/#592).
- **2026-10-09** — **Release 7.1.0 publiée** (zip 219 459 o, SHA-256
  `F623AC123AFA3CDDC00F93AC25215D072612EFAC16E314D5EB5AD44E407E4A07`), notes de release
  rédigées, **test E2E de l'auto-update** : vieux client 7.0.0 → dialogue →
  téléchargement → SHA-256 vérifié → extraction → relance en 7.1.0, hash du fichier
  installé identique à celui du zip publié.
- **2026-10-09 (correctif de dernière minute)** — `455a0cd` : la vérification de la
  release a révélé un **bug de mon propre commit** — `SelectedItems` interrogeable en mode
  virtuel → `RefreshGames` plantait après le compteur, laissant un dialogue d'erreur à
  chaque démarrage, les boutons « Refresh Games »/« Add Game » désactivés et **l'avertissement
  > 5000 jeux jamais affiché**. Corrigé, tout re-testé, zip **et tag refaits**.

## Décisions structurantes (ne pas rouvrir sans raison)

- **Pas de PR sur l'amont.** Tout passe par le fork et ses releases ; `origin` ne reçoit
  jamais rien.
- **Numérotation du fork** : `7.1.0` = « 7.0.x + les changements du fork ». Les titres de
  fenêtre dérivent de `UpdateChecker.CurrentVersion` (plus de « 7.0 » codé en dur).
- **Convention de release** reprise d'amont : tag `X.Y.Z`, nom de release `X.Y.Z`, assets
  `SteamAchievementManager-X.Y.Z.zip` + `.zip.sha256`. Le `.sha256 est obligatoire : c'est
  ce que l'auto-update vérifie.
- **Auto-update** : contrôle silencieux à l'affichage (tout échec est ignoré sauf clic sur
  le bouton), `ReleaseRepository = "Endymi0n74/SteamAchievementManager"`, téléchargement
  des deux assets depuis la release, vérification SHA-256, script PowerShell qui attend la
  sortie de `SAM.Picker`/`SAM.Game`, refuse le zip-slip, relance l'outil.
- **Solution x86 uniquement** : `SAM.sln` ne garde que les configurations x86 (le jeu
  parle au client Steam 32 bits). Un `dotnet sln add` injecte `Any CPU`/x64 et **casse le
  build** — éditer le `.sln` à la main si besoin.
- **Tests x86** : `SAM.Game.Tests` référencé avec `AdditionalProperties="Platform=x86"` +
  `InternalsVisibleTo` depuis `SAM.Game` **et** `SAM.Picker`.
- **Sécurité** : jamais de mot de passe ni de secret journalisé ni sérialisé ; pas de
  `Debug` dérivé sur les types qui touchent un secret.
- **Choix écartés (ne pas re-proposer sans nouvel élément)** :
  - pas de wrapper `UpdateAvgRateStat` (risque de corrompre la moyenne côté Steam) —
    juste un drapeau `AverageRate` + message ;
  - pas de `SetDllDirectory("a;b")` (comportement non garanti) ;
  - #439 : indéterminé sans repro ;
  - #609 favoris : déjà couvert par la PR amont #613 ;
  - vtable `ISteamUserStats013` **verrouillée par commentaire** : ne pas réordonner les
    entrées.

## Leçons apprises

- **`ListView.SelectedItems` lève une `InvalidOperationException` en mode virtuel**
  (`VirtualMode = true`) : à remplacer par une décision portée par l'appelant
  (`RefreshGames(selectFirst:)`). `Items[0].Selected = true` et `SelectedIndices`
  fonctionnent, eux.
- **Les dialogues modaux sont invisibles dans une capture `PrintWindow` de la fenêtre
  principale** : toujours énumérer les fenêtres top-level du processus (classe `#32770`)
  avant de conclure « pas d'erreur ». C'est comme ça que le bug ci-dessus a été trouvé.
- **Lire l'UI** : les `ToolStripStatusLabel` n'ont pas de `HWND` — passer par UI Automation
  (`ControlType.Text`, nom de l'élément) plutôt que par `AutomationId` (introuvable).
- **Un binaire x86 ne se charge pas dans PowerShell 64 bits** (`BadImageFormatException`) :
  pour dissésembler, utiliser `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe`.
  Les chaînes UTF-16 de l'assembly sont à des offsets parfois impairs : une recherche de
  sous-chaîne dans les octets prouve peu, préférer la lecture des IL.
- **`gh` sous PowerShell** : les expressions `--jq` complexes échouent (« accepts 1 arg(s) »)
  → `Invoke-RestMethod` / `ConvertFrom-Json`, ou `--jq` simple.
- **`gh release create --target <sha>` renvoie « target_commitish is invalid »** : passer
  un nom de branche (`--target master`), pas un SHA. Pour re-taguer : `gh release delete`
  + `git push fork :refs/tags/X`, puis recréer.
- **Un hash SHA-256 différent ne prouve pas un binaire différent** : deux compilations
  identiques de la même source diffèrent (MVID/GUID PDB). Comparer un artefact à un
  artefact (zip publié ↔ fichier installé), jamais deux builds successifs.
- **Construire un « vieux client » pour un test E2E** sans toucher aux sources :
  `dotnet build SAM.Picker\SAM.Picker.csproj -c Release -p:Platform=x86 -p:Version=7.0.0
  -p:AssemblyVersion=7.0.0.0 -p:FileVersion=7.0.0.0`, copier `upload\*`, puis rebuilder
  la solution (les propriétés de ligne de commande l'emportent sur le `.csproj`).
- **Erreurs rencontrées** : `MSB3021` fichier verrouillé (SAM lancé → `Stop-Process` avant
  build), `MSB1009 SAM.sln` si la commande tourne sans `workdir`, Steam fermé = « failed to
  create pipe » (relancer `steam.exe -silent`).
- **Aucun clic synthétique sur le bureau** : captures via `PrintWindow` uniquement ; un
  `BM_CLICK` ciblé sur un dialogue (« Oui ») est acceptable pour piloter un test E2E.

## État actuel / à faire

- **Publié et vérifié** : release `7.1.0` (tag = `master` = `455a0cd`), 12 commits poussés
  sur `fork`, arbre propre, 68 tests verts, E2E auto-update OK, `README.md` (EN) +
  `README.fr.md` (FR) réécrits pour présenter le fork.
- **Ouvert / facultatif** :
  - CI GitHub Actions sur le fork (build + tests) — l'AppVeyor d'amont
    (`.appveyor.yml`) ne tourne pas ici ;
  - adoucir le libellé « > 5000 jeux » si la bibliothèque de la machine est légitime
    (6637 jeux remontés pendant les tests) ;
  - #439 reste indéterminé, #609 reste sur la PR amont #613 ;
  - la prochaine version = bump `<Version>` dans les 3 `.csproj`, build + tests, zip +
    `.sha256`, `gh release create` — les 7.1.0 sont prévenus automatiquement.
