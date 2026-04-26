# Android Security Lab 5

## Objectif
Comprendre le rooting sur un environnement Android de laboratoire, observer son impact sur les controles d'integrite, et documenter une remise a zero propre de l'AVD.

## Fiche environnement

| Champ | Valeur |
|---|---|
| Date | 26/04/2026 |
| Auteur | CHARRAJ Mouad |
| Support | AVD Pixel 4a |
| Android | API 37 x86_64 |
| Application | FireStorm.apk |
| Donnees | Fictives uniquement |
| Reseau | Test |
| Objectif | Comprendre rooting et impacts |

## Definition du rooting
Le rooting consiste a obtenir les privileges super-utilisateur sur Android.  
Dans ce laboratoire, l'elevation de privileges a ete activee sur l'AVD via `adb root`.  
La commande `adb shell id` retourne `uid=0(root)`, ce qui confirme l'acces root.  
Le rooting est utile en environnement de test, mais il diminue la confiance accordee a l'integrite native du systeme.

## Verified Boot / AVB

Schema simple de la chaine de confiance :

```text
ROM de demarrage
    -> Bootloader
        -> Verification de l'image de boot
            -> Verification de l'integrite systeme
                -> Demarrage Android
```

Role general :
- Verified Boot verifie que le systeme demarre dans un etat attendu.
- AVB modernise ce controle et ajoute une protection contre le rollback.

Observation sur l'AVD teste :
- `ro.boot.veritymode` retourne `enforcing`
- `ro.boot.verifiedbootstate` n'est pas renseignee par cette image AVD
- `ro.boot.vbmeta.device_state` n'est pas renseignee par cette image AVD

## Risques identifies

| Risque | Resume |
|---|---|
| Integrite non garantie | Un environnement root peut biaiser les conclusions de securite. |
| Surface d'attaque accrue | Un appareil privilegie expose davantage de points d'entree. |
| Exposition de donnees | Des donnees sensibles seraient plus faciles a extraire si elles etaient presentes. |
| Instabilite systeme | Les modifications bas niveau peuvent rendre les tests non reproductibles. |
| Melange perso/test | L'usage d'un environnement non dedie peut provoquer des fuites d'information. |
| Mauvais nettoyage | Des traces peuvent rester entre deux seances si le reset n'est pas fait. |
| Reseau non isole | Un trafic de test mal controle peut affecter d'autres systemes. |
| Tracabilite insuffisante | Sans preuves ni journal, l'audit ne peut pas etre reproduit correctement. |

## Mesures defensives

| Mesure | Resume |
|---|---|
| Reseau isole | Limiter toute communication non controlee hors du labo. |
| Donnees fictives | Eliminer le risque de fuite de donnees reelles. |
| AVD dedie | Utiliser un support reserve aux tests de securite. |
| Wipe en fin de seance | Revenir a un etat propre avant toute reutilisation. |
| Journal technique | Conserver les commandes et observations essentielles. |
| Aucun compte personnel | Eviter tout melange avec des donnees privees. |
| Controle des APK | Installer uniquement les fichiers necessaires au test. |
| Captures horodatees | Garder des preuves simples et verifiables. |

## MASVS

- `STORAGE-1` : les donnees sensibles doivent etre stockees de facon securisee et ne pas reposer uniquement sur la protection du systeme.
- `NETWORK-1` : les communications reseau doivent utiliser TLS avec une verification correcte des certificats.

## MASTG

- Verifier si des informations sensibles apparaissent en clair dans les fichiers prives de l'application.
- Analyser `adb logcat` pour detecter d'eventuelles fuites d'informations pendant l'execution.

## Scenarios realises

1. Demarrage de l'AVD et verification de la connexion ADB.
2. Elevation de privileges sur l'emulateur avec `adb root`.
3. Installation de l'application de test et ouverture de l'application.

## Preuves

### 1. Detection de l'AVD
Commande utilisee : `adb devices`

![Detection de l'AVD](https://github.com/user-attachments/assets/9cec6146-5abe-42e2-9877-0ab271d35bbe)

*l'emulateur `emulator-5554` est bien detecte par ADB avec l'etat `device`, ce qui confirme que l'AVD est pret pour le test.*

### 2. Elevation de privileges et confirmation root
Commandes utilisees : `adb root` puis `adb shell id`

![Elevation de privileges et confirmation root](https://github.com/user-attachments/assets/0c8c089c-02fb-4454-80e9-8008cc9762e4)

*le service ADB a ete redemarre en mode root, puis la commande `adb shell id` retourne `uid=0(root)`, confirmant l'obtention des privileges super-utilisateur sur l'AVD.*

### 3. Verification des proprietes de boot
Commandes utilisees : `adb shell getprop ro.boot.verifiedbootstate`, `adb shell getprop ro.boot.veritymode`, `adb shell getprop ro.boot.vbmeta.device_state`

![Verification des proprietes de boot](https://github.com/user-attachments/assets/ef86f0f6-25d2-48b2-bf21-8ef41bd2547a)

*Lla propriete `ro.boot.veritymode` retourne `enforcing`, tandis que `ro.boot.verifiedbootstate` et `ro.boot.vbmeta.device_state` ne sont pas renseignees sur cette image d'emulateur.*

### 4. Installation de l'application
Commande utilisee : `adb install FireStorm.apk`

![Installation de l'application](https://github.com/user-attachments/assets/4c550e04-f2e2-42f9-a91b-f80b735f7907)

*l'installation de l'application de test `FireStorm.apk` s'est terminee avec succes sur l'AVD.*

### 5. Application ouverte

![Application ouverte](https://github.com/user-attachments/assets/fb4ea1a4-d14f-4afc-acb8-1d89ab24c7ee)

*l'application de test est bien lancee sur l'emulateur, ce qui confirme que l'installation et l'execution sont fonctionnelles dans l'environnement de laboratoire.*

### 6. Preparation de la remise a zero
Action utilisee : `Android Studio > Device Manager > Wipe Data`

![Preparation de la remise a zero](https://github.com/user-attachments/assets/583bbd5c-8009-476f-a706-a7cc7c78e0c0)

*l'option `Wipe Data` est disponible sur l'AVD Pixel 4a et constitue la methode retenue pour remettre l'environnement de test a zero en fin de seance.*


## Checklist de reset

- [x] AVD de laboratoire utilise
- [x] Aucun compte personnel engage
- [x] Root active uniquement dans le cadre du test
- [x] APK de test installee
- [x] Action `Wipe Data` preparee en fin de seance
- [x] Remise a zero prevue comme etape obligatoire

## Conclusion
Le test confirme qu'un AVD peut etre place en mode root de laboratoire via `adb root`, puis verifie avec `adb shell id`.  
L'environnement reste adapte a une demonstration controlee du rooting, de Verified Boot et des bonnes pratiques de tracabilite et de nettoyage.  
Le reset de l'AVD fait partie integrante du protocole afin de conserver un environnement jetable, propre et reproductible.
