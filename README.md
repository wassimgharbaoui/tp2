# Audit de Sécurité Android — Environnement Rooté avec DIVA

## Présentation du projet

Ce rapport documente un audit de sécurité réalisé sur un émulateur Android en mode rooté. L'objectif principal est d'observer concrètement comment l'obtention des privilèges **UID 0** contourne les mécanismes de protection natifs d'Android — notamment le **Verified Boot** et l'isolation applicative — et d'en mesurer l'impact sur la confidentialité des données.

## Contexte technique

| Paramètre | Valeur |
|---|---|
| Application cible | DIVA (Damn Insecure and Vulnerable App) v1.0 Beta |
| Émulateur | AVD Pixel 6 — Android Studio |
| Image système | Android 15.0 (Google APIs) — API Level 35 |
| Auteur | Wassim Gharbaoui |
| Date | 13 Février 2026 |

---

## Phase 1 — Obtention des privilèges root sur l'AVD

La première phase consiste à élever les droits de l'émulateur jusqu'à l'accès super-utilisateur, ce qui permet ensuite d'accéder aux zones normalement protégées du système de fichiers.

Les commandes utilisées sont les suivantes :

```bash
adb devices
adb root
adb remount
adb shell id
adb shell getprop ro.boot.veritymode
adb shell getprop ro.boot.vbmeta.device_state
adb shell "su -c id"
```

### 📸 Image 1 — Affichage du UID après élévation
<img width="1780" height="93" alt="Screenshot 2026-02-13 234207" src="https://github.com/user-attachments/assets/2aec55bb-8bd9-4ca2-9305-9f24877ca60e" />


---

### 📸 Image 2 — Processus de rooting et remount
<img width="786" height="113" alt="Screenshot 2026-02-13 233925" src="https://github.com/user-attachments/assets/14d7d9e3-96b0-4269-a76e-9dbb2cfce9af" />

<img width="716" height="69" alt="Screenshot 2026-02-13 234033" src="https://github.com/user-attachments/assets/63388c2a-d059-4482-bfe5-7fc4c60aecfc" />

<img width="903" height="50" alt="Screenshot 2026-02-14 203946" src="https://github.com/user-attachments/assets/e80e3994-a936-4375-9a8f-8db0469170ae" />


### 📸 Image 3 — Vérification des indicateurs d'intégrité (1)
<img width="915" height="46" alt="Screenshot 2026-02-13 235317" src="https://github.com/user-attachments/assets/e71dc1e2-81b9-4174-9cc2-ba3b5afaec4f" />


---

### 📸 Image 4 — Vérification des indicateurs d'intégrité (2)
<img width="1131" height="545" alt="image" src="https://github.com/user-attachments/assets/a7226de1-f3a5-4336-a68a-5a1108bc7a56" />


---

### 📸 Image 5 — Analyse du fichier Logcat
> Insérer ici la capture du journal Logcat capturé pendant la phase de rooting.

---

## Phase 2 — Préparation d'un AVD vierge

Avant de procéder aux tests applicatifs, un AVD propre est démarré afin de partir d'un état système sain et d'assurer la reproductibilité des observations.

### 📸 Image 6 — État initial de l'AVD et connexion ADB
> Insérer ici la capture montrant le démarrage de l'AVD vierge et la connexion ADB établie.

---

## Phase 3 — Installation et lancement de DIVA

L'application DIVA est déployée sur l'émulateur via ADB :

```bash
adb install diva-beta.apk
```

### 📸 Image 7 — Confirmation de l'installation via ADB
> Insérer ici la capture montrant "Success" dans le terminal après l'installation.

---

### 📸 Image 8 — Interface principale de DIVA
> Insérer ici la capture montrant le menu d'accueil de l'application DIVA dans l'émulateur.

---

## Phase 4 — Scénarios de test applicatifs

Trois parcours utilisateur ont été définis pour couvrir des surfaces d'attaque différentes.

### Scénario A — Accueil de l'application

Lancement de DIVA et contrôle de l'affichage du menu principal, avec surveillance des journaux système en temps réel.

### 📸 Image 9 — Journaux système pendant le lancement de DIVA
> Insérer ici la capture du terminal Logcat actif pendant l'ouverture de l'application.

---

### Scénario B — Interaction avec un module de saisie

Sélection du module **"1. INSECURE LOGGING"** pour soumettre une entrée utilisateur et observer ce qui est journalisé.

### 📸 Image 10 — Module de saisie et recherche d'un item (2 captures)
> Insérer ici les deux captures : exploitation du module Input Validation et la saisie d'un terme de recherche.

---

### Scénario C — Consultation d'un détail applicatif

Navigation vers le module **"Insecure Data Storage - Part 1"** pour saisir des identifiants fictifs et inspecter ensuite leur stockage sur le disque.

### 📸 Image 11 — Saisie dans le module stockage et écran Settings (2 captures)
> Insérer ici les deux captures : formulaire de saisie et vue Settings.

---

### 📸 Image 12 — Inspection du système de fichiers privé via les droits root
> Insérer ici la capture montrant la lecture du répertoire /data/data/ avec les privilèges UID 0.

---

## Phase 5 — Analyse du Verified Boot

### Contexte

Le **Verified Boot** (ou AVB 2.0) garantit que chaque partition démarrée provient d'une source officielle non altérée. Il s'appuie sur une signature cryptographique granulaire de chaque partition et intègre une protection anti-rollback pour bloquer le retour vers des versions antérieures vulnérables.

Dans un environnement rooté, cet état passe à **orange**, signalant que la chaîne de confiance a été volontairement levée pour permettre l'accès administrateur. Ce n'est pas un état d'erreur en contexte de laboratoire, mais un indicateur que les garanties d'intégrité au démarrage ne sont plus actives.

### 📸 Image 13 — État du Verified Boot (audit de clôture)
> Insérer ici la capture confirmant l'état "orange" du Verified Boot State.

---

## Phase 6 — Remise à zéro de l'AVD

Une fois l'audit terminé, l'émulateur est réinitialisé à son état d'usine pour éliminer toute trace des opérations réalisées et éviter toute contamination d'une future session.

### 📸 Image 14 — Sélection de l'option "Wipe Data"
<img width="511" height="433" alt="image" src="https://github.com/user-attachments/assets/a5661984-9aa3-4835-a73b-119ad7baf618" />


---

### 📸 Image 15 — Redémarrage post-réinitialisation
<img width="316" height="702" alt="image" src="https://github.com/user-attachments/assets/3f59dd84-d539-4f41-9a3b-f41a29980cf9" />



---

## Sécurité Android — Rappel des couches protégées

L'architecture de sécurité Android repose sur plusieurs piliers complémentaires :

**Sandboxing applicatif** — chaque application s'exécute dans un processus Linux isolé avec un UID unique, ce qui l'empêche d'accéder aux données des autres applications.

**Modèle de permissions** — tout accès à une ressource sensible (contacts, localisation, stockage) doit être explicitement accordé par l'utilisateur.

**Intégrité système (Verified Boot)** — garantit que le noyau et les partitions système n'ont pas été modifiés entre leur signature et leur exécution.

Le rooting contourne simultanément ces trois couches en accordant à son détenteur un contrôle total sur l'ensemble du système.

---

## Référentiel OWASP MASVS / MASTG

### Exigences MASVS applicables

**MASVS-STORAGE-1** — Toute donnée sensible (mot de passe, jeton d'accès, clé API) doit être chiffrée avant d'être persistée localement.

**MASVS-NETWORK-1** — L'ensemble des échanges réseau doit transiter via TLS avec vérification stricte des certificats.

### Application sur DIVA

L'accès root a permis de lire directement le contenu du répertoire `shared_prefs` de l'application et de constater que des identifiants (`Secret123!`) y sont stockés **en clair**, sans aucun chiffrement. Cela constitue un échec direct à l'exigence **MASVS-STORAGE-1**.

### Tests MASTG correspondants

- **Inspection statique** du répertoire `/data/data/[package]/shared_prefs/` via l'accès root pour identifier les données en clair.
- **Analyse dynamique** via `adb logcat` pour intercepter les flux de données à l'exécution et repérer des fuites dans les journaux système.

---

## Matrice des risques liés à un environnement rooté

| Risque | Description |
|---|---|
| Résultats biaisés | Un système modifié peut produire des comportements non représentatifs |
| Surface d'attaque élargie | L'appareil devient très exposé s'il quitte l'environnement contrôlé |
| Exposition des données | Sans sandbox actif, tout processus peut lire les fichiers privés |
| Instabilité système | Le root peut engendrer des comportements non reproductibles |
| Contamination entre sessions | Un mauvais nettoyage laisse des traces pour les utilisateurs suivants |
| Réseau non cloisonné | Des tests pourraient atteindre des systèmes extérieurs par accident |
| Traçabilité insuffisante | Sans journalisation, il devient impossible de rejouer ou de prouver un test |

---

## Mesures défensives appliquées

- **Réseau isolé** — aucune connexion externe autorisée pendant les tests.
- **Données fictives exclusivement** — aucune information personnelle réelle utilisée.
- **AVD dédié** — émulateur réservé à ce laboratoire, non partagé avec d'autres usages.
- **Wipe Data systématique** — réinitialisation complète effectuée en fin de session.
- **Journal de configuration** — chaque paramètre est noté pour garantir la reproductibilité.
- **Aucun compte personnel** — aucune connexion à des services tiers sur l'appareil de test.
- **Vérification des APK** — origine de chaque application installée contrôlée avant déploiement.
- **Traçabilité horodatée** — chaque étape capturée avec horodatage pour preuve d'audit.

---

## Fiche de traçabilité

| Champ | Valeur |
|---|---|
| Auteur | Wassim Gharbaoui |
| Date | 13 Février 2026 |
| Émulateur | AVD "emu64xa" — Pixel 6 (x86_64) |
| Système | Android 15 (VanillaIceCream) — API 35 |
| Application | DIVA v1.0 Beta |
| Accès root | ✅ uid=0 obtenu |
| État Verified Boot | 🟠 Orange (chaîne de confiance rompue) |
| Vulnérabilité confirmée | Identifiants (`Secret123!`) stockés en clair |
| Reset final | ✅ Wipe Data effectué |
| Limites | Tests sur émulateur uniquement — pas de TEE/StrongBox matériel |

### Preuves référencées

- **Preuve 1** — Lancement de l'application (Image 7)
- **Preuve 2** — Confirmation du mode root (Images 2 et 3)
- **Preuve 3** — État du Verified Boot (Image 13)
- **Preuve 4** — Lecture des données sensibles (Images 9, 10, 12)

---

## Conclusion

Cet audit démontre qu'un accès root sur un émulateur Android suffit à contourner l'ensemble des couches de sécurité natives et à exposer des données applicatives normalement inaccessibles. Dans le cas de DIVA, des identifiants sensibles sont stockés sans chiffrement dans les préférences partagées — une vulnérabilité directement observable grâce aux privilèges obtenus.

Ce type d'environnement de test est précieux pour comprendre les limites des protections applicatives, à condition de respecter un cadre d'utilisation strict : isolation réseau, données fictives, réinitialisation systématique et traçabilité complète de chaque action réalisée.
