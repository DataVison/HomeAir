# HomeAir

**HomeAir** est une configuration ESPHome complète destinée à intégrer un échangeur d’air compatible Broan, NuTone, Venmar ou vanEE à Home Assistant à l’aide de son interface de communication RS485.

Le projet s’appuie sur un ESP32-S3 pour communiquer avec l’échangeur d’air, exposer ses données dans Home Assistant et ajouter plusieurs fonctions avancées adaptées à une installation résidentielle.

HomeAir permet notamment de contrôler les modes de ventilation, lancer des périodes de ventilation temporisées, mesurer l’humidité et la température dans plusieurs zones de la maison, transmettre l’humidité moyenne à l’échangeur et estimer le rendement thermique du système.

> HomeAir est un projet communautaire indépendant. Il n’est ni affilié, ni commandité, ni approuvé par Broan, NuTone, Venmar, vanEE ou leurs sociétés apparentées.

---

## Origine du projet

HomeAir est basé sur le composant ESPHome open source original :

[`nspitko/broan_erv_uart`](https://github.com/nspitko/broan_erv_uart)

Le projet original a été développé par **nspitko** afin de permettre la communication entre un ESP32 et certains échangeurs d’air Broan, NuTone, Venmar et vanEE par leur interface série RS485.

Le protocole de communication a été analysé et documenté par l’auteur original dans l’article suivant :

[Reverse Engineering an ERV](https://spitko.net/2025/08/08/Reverse-Engineering-an-ERV/)

Une grande partie du fonctionnement RS485 utilisé par HomeAir repose sur le travail de rétro-ingénierie, l’analyse des paquets et le composant ESPHome publiés dans ce projet original.

HomeAir ne prétend pas être l’auteur du protocole, du composant Broan ou du travail de rétro-ingénierie original.

---

## Objectif de HomeAir

HomeAir transforme le composant RS485 original en une configuration ESPHome complète adaptée à une utilisation quotidienne dans Home Assistant.

Le projet ajoute notamment :

* une configuration prête pour ESP32-S3;
* une interface Home Assistant en français;
* des modes de ventilation simplifiés;
* des modes temporisés Turbo, Maximum et OVR;
* un retour automatique au mode Économie;
* un capteur local de température et d’humidité;
* l’intégration de plusieurs capteurs Home Assistant;
* le calcul d’une humidité moyenne;
* le calcul d’une température intérieure moyenne;
* l’envoi de l’humidité moyenne au contrôleur Broan;
* un fonctionnement de secours avec le capteur local;
* un calcul du gain thermique;
* une estimation du rendement de récupération;
* un indicateur virtuel pour les périodes de pointe;
* des journaux adaptés à une utilisation stable sur RS485;
* des entités de diagnostic pour surveiller le fonctionnement de l’échangeur.

---

# Fonctions principales

HomeAir permet de :

* contrôler les principaux modes de ventilation;
* surveiller le mode réel rapporté par l’échangeur;
* lancer une ventilation temporaire;
* interrompre manuellement un mode temporaire;
* retourner automatiquement au mode Économie;
* régler la vitesse en mode manuel;
* régler la durée de fonctionnement du mode intermittent;
* contrôler la consigne d’humidité;
* calculer l’humidité moyenne de plusieurs zones;
* calculer la température moyenne de plusieurs zones;
* surveiller la consommation électrique;
* surveiller les débits de soufflage et d’extraction;
* surveiller la vitesse des ventilateurs;
* surveiller la durée de vie du filtre;
* réinitialiser la durée de vie du filtre;
* calculer le gain thermique;
* estimer le rendement de récupération;
* conserver un fonctionnement local même si Home Assistant est indisponible.

---

# Matériel requis

La configuration actuelle de HomeAir est prévue pour le matériel suivant :

* ESP32-S3 DevKitC-1;
* framework ESP-IDF;
* convertisseur TTL vers RS485;
* capteur AM2302 ou DHT22;
* échangeur d’air compatible avec le protocole Broan RS485;
* installation Home Assistant;
* ESPHome;
* alimentation adaptée pour l’ESP32;
* accès Wi-Fi.

---

## Carte ESP32

La configuration actuelle utilise :

```yaml
esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf
```

Le framework ESP-IDF est utilisé pour assurer une meilleure stabilité avec l’ESP32-S3 et la communication UART.

---

## Broches utilisées

| Fonction                  | Broche ESP32 |
| ------------------------- | -----------: |
| Transmission RS485        |       GPIO17 |
| Réception RS485           |       GPIO18 |
| Données du capteur AM2302 |        GPIO4 |

---

## Paramètres de communication RS485

La liaison série est configurée avec les paramètres suivants :

| Paramètre           |       Valeur |
| ------------------- | -----------: |
| Vitesse             | 38 400 bauds |
| Bits de données     |            8 |
| Parité              |       Aucune |
| Bits d’arrêt        |            1 |
| Taille du tampon RX |  4096 octets |

Configuration correspondante :

```yaml
uart:
  id: rs485
  tx_pin: GPIO17
  rx_pin: GPIO18
  baud_rate: 38400
  data_bits: 8
  parity: NONE
  stop_bits: 1
  rx_buffer_size: 4096
```

---

# Branchement RS485

Repère sur l’échangeur d’air le bornier de commande contenant les connexions de communication.

Les bornes peuvent notamment être identifiées comme suit :

* `12V`;
* `GND`;
* `D+`;
* `D-`;
* `LED`;
* `OVR`.

Le branchement RS485 général est le suivant :

| Échangeur d’air | Convertisseur RS485 |
| --------------- | ------------------- |
| D+              | B ou D+             |
| D-              | A ou D-             |
| GND             | GND                 |

Selon le fabricant du convertisseur RS485 :

* `A` peut correspondre à `D-`;
* `B` peut correspondre à `D+`.

L’identification des bornes RS485 n’est pas toujours uniforme. Vérifie toujours la documentation du module avant d’inverser les fils.

---

## Branchement du convertisseur à l’ESP32

Le branchement côté ESP32 dépend du convertisseur utilisé.

Pour un convertisseur TTL vers RS485 classique :

| Convertisseur | ESP32-S3                   |
| ------------- | -------------------------- |
| TX ou RO      | GPIO18                     |
| RX ou DI      | GPIO17                     |
| GND           | GND                        |
| VCC           | Selon la tension du module |

Certains modules possèdent également des broches :

* `DE`;
* `RE`;
* `RTS`;
* `FLOW`;
* `EN`.

Ces broches servent à contrôler le sens de communication.

La configuration actuelle suppose l’utilisation d’un convertisseur qui gère automatiquement la direction de transmission ou qui ne nécessite pas de broche supplémentaire dans ESPHome.

---

## Masse commune

Une masse commune entre l’échangeur d’air, le convertisseur RS485 et l’ESP32 est recommandée.

En cas de communication instable, vérifie en priorité :

* la présence du fil GND;
* la continuité de la masse;
* la polarité D+ et D-;
* la qualité des connexions;
* la longueur du câblage;
* la proximité de sources de parasites électriques.

---

## Résistance de terminaison

Certains convertisseurs RS485 possèdent une résistance de terminaison de 120 ohms.

Cette résistance peut être nécessaire ou non selon :

* la longueur du câble;
* le type de transceiver;
* la topologie du bus;
* la terminaison déjà présente dans l’échangeur;
* la qualité du signal.

Si tu observes des erreurs de communication, essaie :

1. sans résistance de terminaison;
2. avec la résistance activée;
3. en vérifiant la masse;
4. en inversant D+ et D- seulement après avoir vérifié la documentation.

---

# Alimentation

La sortie de commande de l’échangeur peut fournir du 12 V.

Ne branche jamais directement cette tension sur une entrée 5 V ou 3,3 V d’un ESP32 standard.

Utilise plutôt :

* un convertisseur abaisseur 12 V vers 5 V;
* une carte ESP32 conçue pour accepter du 12 V;
* une alimentation USB indépendante;
* un module d’alimentation isolé adapté.

L’ESP32, le convertisseur RS485 et le capteur doivent être alimentés selon leurs spécifications respectives.

---

# Installation ESPHome

## Composant externe

HomeAir charge le composant Broan depuis le dépôt GitHub suivant :

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/DataVison/HomeAir
      ref: main
    components:
      - broan
```

Le composant est ensuite déclaré ainsi :

```yaml
broan:
  id: broan_erv
  uart_id: rs485
```

---

## Configuration des secrets

Ajoute les valeurs suivantes dans le fichier `secrets.yaml` d’ESPHome :

```yaml
wifi_ssid: "NOM_DU_RESEAU"
wifi_password: "MOT_DE_PASSE_WIFI"

evr_api_encryption_key: "CLE_DE_CHIFFREMENT_API"
evr_ota_password: "MOT_DE_PASSE_OTA"
evr_fallback_password: "MOT_DE_PASSE_POINT_ACCES"
```

La clé API peut être générée par ESPHome lors de la création de l’appareil.

---

## Configuration Wi-Fi

HomeAir utilise une connexion Wi-Fi normale ainsi qu’un point d’accès de secours.

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  reboot_timeout: 0s

  ap:
    ssid: "EVR Fallback Hotspot"
    password: !secret evr_fallback_password

captive_portal:
```

Le paramètre suivant empêche l’ESP32 de redémarrer automatiquement lors d’une perte du Wi-Fi :

```yaml
reboot_timeout: 0s
```

Cela permet au contrôle RS485 local de continuer même lorsque le réseau est temporairement indisponible.

---

## API Home Assistant

La connexion avec Home Assistant est chiffrée :

```yaml
api:
  encryption:
    key: !secret evr_api_encryption_key

  reboot_timeout: 0s
```

Le redémarrage automatique en cas de perte de Home Assistant est également désactivé.

Ainsi, l’échangeur peut continuer à fonctionner même si Home Assistant redémarre ou devient temporairement indisponible.

---

## Mise à jour OTA

Les mises à jour à distance sont activées :

```yaml
ota:
  - platform: esphome
    password: !secret evr_ota_password
```

---

# Comportement au démarrage

Après le démarrage, HomeAir attend 30 secondes avant de remettre l’échangeur dans son mode par défaut.

```yaml
esphome:
  name: evr
  friendly_name: "Échangeur d'air"

  on_boot:
    priority: -100
    then:
      - delay: 30s
      - script.execute: retour_mode_defaut
```

Le mode par défaut est :

```text
Économie
```

Ce mode correspond à la valeur brute :

```text
int
```

Ce délai permet :

* au Wi-Fi de démarrer;
* à l’API ESPHome de s’initialiser;
* à l’interface RS485 de devenir disponible;
* au composant Broan de terminer son échange initial avec le VRE.

---

# Modes de ventilation

HomeAir expose deux niveaux de contrôle :

1. un sélecteur simplifié destiné à l’utilisateur;
2. un sélecteur brut destiné au diagnostic.

---

## Mode simplifié

L’entité principale est nommée :

```text
Mode de ventilation
```

Elle contient les options suivantes :

| Nom affiché   | Valeur Broan  |
| ------------- | ------------- |
| Arrêt         | `off`         |
| Doux          | `min`         |
| Économie      | `int`         |
| Maximum       | `max`         |
| Manuel        | `manual`      |
| Humidité      | `humidity`    |
| Recirculation | `recirculate` |

Ces modes sont permanents.

Ils restent actifs jusqu’à ce qu’un autre mode soit sélectionné.

---

## Mode brut

L’entité suivante affiche le mode réel retourné par l’échangeur :

```text
Mode de ventilation (brut)
```

Cette entité est classée dans la catégorie diagnostic.

Elle peut notamment afficher :

```text
off
min
max
int
manual
humidity
recirculate
turbo
ovr
```

Les modes `turbo` et `ovr` peuvent être visibles même s’ils ne sont pas proposés directement dans le sélecteur permanent simplifié.

---

# Modes temporisés

HomeAir ajoute trois commandes temporisées :

* Turbo;
* Ventilation maximale;
* Surpassement OVR.

Un seul mode temporisé peut être actif à la fois.

Le lancement d’un nouveau mode temporisé arrête automatiquement celui qui était déjà en cours.

À la fin de la durée sélectionnée, HomeAir revient automatiquement au mode Économie.

---

## Turbo

L’entité `Turbo` permet de choisir parmi les durées suivantes :

* Arrêt;
* 30 minutes;
* 1 heure;
* 2 heures;
* 4 heures;
* 6 heures;
* 8 heures.

Le mode brut envoyé à l’échangeur est :

```text
turbo
```

---

## Ventilation maximale

L’entité `Ventilation maximale` permet de choisir parmi les durées suivantes :

* Arrêt;
* 30 minutes;
* 1 heure;
* 2 heures;
* 4 heures;
* 6 heures;
* 8 heures.

Le mode brut envoyé à l’échangeur est :

```text
max
```

---

## Surpassement OVR

L’entité `Surpassement (OVR)` permet de choisir :

* Arrêt;
* 20 minutes;
* 40 minutes;
* 60 minutes.

Le mode brut envoyé à l’échangeur est :

```text
ovr
```

Ce mode correspond au surpassement utilisé par les commandes auxiliaires de salle de bain.

---

## Arrêt manuel d’un mode temporisé

Lorsque l’option `Arrêt` est choisie dans un sélecteur temporisé :

1. le script actif est arrêté;
2. l’état interne de la minuterie est effacé;
3. le libellé temporaire est supprimé;
4. le temps de fin est remis à zéro;
5. l’échangeur revient au mode Économie.

---

# État de la minuterie

L’entité suivante affiche le mode temporisé actif :

```text
Mode temporisé
```

Lorsqu’aucun mode n’est actif :

```text
Aucun
```

Exemples lorsqu’un mode est actif :

```text
Turbo — 29 min restantes
```

```text
Ventilation maximale — 2h15 restant
```

```text
Surpassement — 14 min restantes
```

Cette entité est actualisée toutes les 30 secondes.

---

# Capteur local AM2302

HomeAir utilise un capteur AM2302, également connu sous le nom DHT22.

Le capteur est connecté sur GPIO4.

```yaml
- platform: dht
  pin: GPIO4
  model: AM2302
  update_interval: 60s
```

Branchement :

| AM2302 | ESP32 |
| ------ | ----- |
| VCC    | 3,3 V |
| DATA   | GPIO4 |
| GND    | GND   |

Les entités créées sont :

```text
Température locale VRE
Humidité locale VRE
```

La lecture est effectuée toutes les 60 secondes.

---

# Capteurs Home Assistant utilisés

HomeAir récupère plusieurs capteurs provenant de Home Assistant.

Ces identifiants sont spécifiques à l’installation actuelle et doivent être remplacés pour une autre installation.

---

## Capteurs d’humidité

### Sous-sol

```yaml
entity_id: sensor.salle_familiale_temperature_humidite_sous_sol_humidite
```

### Rez-de-chaussée

```yaml
entity_id: sensor.salle_a_manger_temperature_humidite_rez_de_chaussee_humidite
```

### Étage

```yaml
entity_id: sensor.hall_etage_temperature_humidite_etage_humidite
```

---

## Capteurs de température

### Sous-sol

```yaml
entity_id: sensor.salle_familiale_temperature_humidite_sous_sol_temperature
```

### Rez-de-chaussée

```yaml
entity_id: sensor.salle_a_manger_temperature_humidite_rez_de_chaussee_temperature
```

### Étage

```yaml
entity_id: sensor.hall_etage_temperature_humidite_etage_temperature
```

### Température extérieure

```yaml
entity_id: sensor.bernieres_temperature
```

---

# Calcul de l’humidité moyenne

HomeAir calcule une moyenne à partir de quatre sources possibles :

1. capteur local AM2302;
2. capteur du sous-sol;
3. capteur du rez-de-chaussée;
4. capteur de l’étage.

Le résultat est publié dans l’entité :

```text
Humidité mesurée
```

---

## Validation des valeurs

Les valeurs sont acceptées seulement lorsqu’elles sont :

* disponibles;
* numériques;
* différentes de `NaN`;
* comprises dans une plage valide;
* inférieures ou égales à 100 %;
* supérieures ou égales à 0 % pour les capteurs Home Assistant.

Pour le capteur local, la valeur doit être strictement supérieure à 0 % et inférieure à 100 %.

---

## Capteurs indisponibles

Si un capteur est indisponible, il est simplement exclu du calcul.

La moyenne est calculée uniquement avec les capteurs valides disponibles.

Exemple :

* capteur local : 42 %;
* sous-sol : 46 %;
* rez-de-chaussée indisponible;
* étage : 40 %.

Calcul :

```text
(42 + 46 + 40) ÷ 3 = 42,7 %
```

---

## Repli sur le capteur local

Si Home Assistant devient indisponible, le capteur AM2302 continue de fonctionner localement.

HomeAir peut alors calculer l’humidité à partir du seul capteur local.

Un avertissement est enregistré :

```text
Repli sur le capteur local seul
```

Cela permet au contrôle de l’humidité de continuer même sans accès aux capteurs Home Assistant.

---

## Aucun capteur valide

Si aucun capteur d’humidité valide n’est disponible :

* l’entité `Humidité mesurée` retourne une valeur inconnue;
* aucune valeur n’est envoyée au contrôleur;
* un avertissement est ajouté aux journaux.

Message correspondant :

```text
Aucun capteur d'humidité valide disponible
```

---

# Contrôle automatique de l’humidité

HomeAir expose l’interrupteur :

```text
Contrôle d'humidité
```

Lorsque cet interrupteur est activé, l’humidité moyenne calculée est envoyée au composant Broan avec :

```cpp
id(broan_erv).setCurrentHumidity(moyenne);
```

L’échangeur peut alors utiliser cette valeur pour gérer le mode d’humidité.

---

## Consigne d’humidité

L’entité suivante règle la cible d’humidité :

```text
Consigne d'humidité
```

Pour utiliser cette fonction :

1. régler la consigne souhaitée;
2. activer `Contrôle d'humidité`;
3. sélectionner le mode `Humidité` lorsque nécessaire.

---

## Fréquence d’envoi

L’humidité est recalculée :

* lorsqu’une nouvelle mesure locale est reçue;
* lorsqu’une mesure Home Assistant est reçue;
* lorsque le contrôle d’humidité est activé;
* toutes les cinq minutes.

---

# Calcul de la température intérieure moyenne

HomeAir calcule une température intérieure moyenne à partir de :

1. la température locale du VRE;
2. la température du sous-sol;
3. la température du rez-de-chaussée;
4. la température de l’étage.

Le résultat est publié dans :

```text
Température mesurée
```

---

## Validation des températures

Les valeurs sont considérées valides lorsqu’elles sont :

* disponibles;
* numériques;
* supérieures à -50 °C;
* inférieures à 80 °C.

Les valeurs invalides sont ignorées.

---

## Repli local

Si Home Assistant est indisponible, HomeAir utilise uniquement la température locale du capteur AM2302.

Un avertissement est ajouté aux journaux :

```text
Repli sur le capteur local seul
```

---

## Fréquence de calcul

La température est recalculée :

* lors d’une nouvelle valeur du capteur local;
* lors d’une nouvelle valeur provenant de Home Assistant;
* toutes les cinq minutes.

---

# Capteurs natifs de l’échangeur

HomeAir expose plusieurs valeurs directement rapportées par le composant Broan.

---

## Consommation électrique

Entité :

```text
Consommation électrique
```

La valeur est rapportée en watts.

Elle représente la consommation indiquée par l’échangeur.

---

## Température de l’air entrant

Entité :

```text
Température air entrant
```

Cette valeur provient directement de l’échangeur.

Selon la conception du système, elle peut représenter une température interne ou une température mesurée dans un conduit.

---

## Température de l’air sortant

Entité :

```text
Température air sortant
```

Certains modèles ne fournissent pas cette valeur.

Dans ce cas, l’entité peut retourner :

```text
nan
```

Elle doit donc être surveillée avant d’être utilisée dans une automatisation importante.

---

## Durée de vie du filtre

Entité :

```text
Durée de vie du filtre
```

La valeur représente la durée de vie restante du filtre rapportée par l’échangeur.

---

## Réinitialisation du filtre

Bouton :

```text
Réinitialiser le filtre
```

Ce bouton remet la durée de vie du filtre à la valeur par défaut prévue par le composant.

---

## Débit de soufflage

Entité :

```text
Débit soufflage
```

La valeur est rapportée en CFM.

---

## Débit d’extraction

Entité :

```text
Débit extraction
```

La valeur est rapportée en CFM.

---

## Vitesse du ventilateur de soufflage

Entité :

```text
Vitesse ventilateur soufflage
```

La valeur est rapportée en tours par minute.

Cette entité est classée comme diagnostic.

---

## Vitesse du ventilateur d’extraction

Entité :

```text
Vitesse ventilateur extraction
```

La valeur est rapportée en tours par minute.

Cette entité est également classée comme diagnostic.

---

# Calcul du gain thermique

HomeAir calcule le gain thermique à l’aide de l’entité :

```text
Gain de l'échangeur
```

Formule :

```text
Température air entrant du VRE
− Température extérieure
```

Exemple :

```text
Température air entrant : 12 °C
Température extérieure : -8 °C

Gain = 12 - (-8)
Gain = 20 °C
```

Cette mesure indique l’augmentation de température observée entre l’air extérieur et la température rapportée par l’échangeur.

Elle ne représente pas directement le rendement officiel du fabricant.

---

# Rendement de récupération estimé

HomeAir calcule une estimation du rendement avec l’entité :

```text
Rendement de récupération
```

Formule utilisée :

```text
(
  Température air entrant du VRE
  − Température extérieure
)
÷
(
  Température intérieure moyenne
  − Température extérieure
)
× 100
```

Exemple :

```text
Température extérieure : -10 °C
Température intérieure moyenne : 20 °C
Température air entrant : 14 °C
```

Calcul :

```text
(14 - (-10)) ÷ (20 - (-10)) × 100
24 ÷ 30 × 100
80 %
```

---

## Conditions de validation

Le rendement n’est pas calculé lorsque l’écart entre la température intérieure et la température extérieure est inférieur à 5 °C.

Cette protection évite les valeurs instables lorsque les températures intérieure et extérieure sont trop proches.

Les résultats suivants sont également rejetés :

* rendement inférieur à 0 %;
* rendement supérieur à 100 %;
* valeur manquante;
* température invalide;
* capteur indisponible.

---

## Limite de cette estimation

Le rendement affiché par HomeAir est une estimation domotique.

Il dépend :

* de la précision des capteurs;
* de leur emplacement;
* de la température rapportée par le VRE;
* de la température extérieure utilisée;
* de la température moyenne calculée dans la maison;
* du débit réel de ventilation;
* du mode actif;
* des cycles de dégivrage;
* du fonctionnement du système de chauffage.

Cette mesure ne remplace pas une mesure professionnelle effectuée avec des sondes calibrées dans chacun des conduits.

---

# Mode intermittent

Le composant Broan utilise une période intermittente exprimée en secondes.

HomeAir expose cette valeur de manière plus intuitive avec l’entité :

```text
Marche par heure
```

Plage disponible :

```text
1 à 60 minutes
```

---

## Conversion

La conversion appliquée est :

```text
Minutes × 60 = Secondes
```

Exemples :

| Marche par heure | Valeur envoyée |
| ---------------: | -------------: |
|       10 minutes |   600 secondes |
|       20 minutes |  1200 secondes |
|       30 minutes |  1800 secondes |
|       40 minutes |  2400 secondes |
|       60 minutes |  3600 secondes |

---

## Exemple

Une valeur de 20 minutes signifie que l’échangeur fonctionne environ :

```text
20 minutes en marche
40 minutes à l’arrêt
```

par période d’une heure, lorsque le mode Économie ou intermittent est utilisé.

---

# Vitesse manuelle

L’entité suivante permet de régler la vitesse :

```text
Vitesse du ventilateur
```

Cette valeur est principalement utilisée lorsque le mode est réglé à :

```text
Manuel
```

La valeur agit comme un rapport entre la vitesse minimale et maximale prise en charge par l’échangeur.

---

# Heure de pointe

HomeAir expose un interrupteur virtuel :

```text
Heure de pointe
```

Cet interrupteur ne modifie pas directement le mode du VRE.

Il est conçu pour être utilisé dans les automatisations Home Assistant.

---

## Exemples d’utilisation

L’interrupteur peut servir à :

* réduire la ventilation pendant une pointe électrique;
* désactiver temporairement le mode Turbo;
* basculer en mode Doux;
* désactiver le chauffage auxiliaire associé au système;
* revenir en mode Économie après la période de pointe;
* empêcher certaines automatisations de ventilation.

---

## Comportement au redémarrage

L’interrupteur utilise :

```yaml
restore_mode: ALWAYS_OFF
```

Il revient donc toujours à l’état désactivé après un redémarrage de l’ESP32.

---

# Horloge locale

HomeAir utilise une horloge SNTP.

Fuseau horaire :

```text
America/Montreal
```

Serveurs utilisés :

```text
0.ca.pool.ntp.org
time.google.com
pool.ntp.org
```

L’horloge peut servir aux journaux, aux automatisations et aux futures fonctions temporelles.

---

# Actions périodiques

Toutes les cinq minutes, HomeAir exécute :

* la mise à jour de l’humidité;
* la mise à jour de la température.

Configuration :

```yaml
interval:
  - interval: 5min
    then:
      - script.execute: maj_humidite
      - script.execute: maj_temperature
```

Cette vérification périodique assure que les valeurs restent à jour même lorsqu’un capteur n’envoie pas immédiatement un nouvel état.

---

# Journalisation

Le niveau général des journaux est réglé à :

```yaml
logger:
  level: INFO
```

Les niveaux spécifiques sont :

```yaml
logs:
  broan: WARN
  uart: NONE
  uart_debug: NONE
  broan_humidite: INFO
  broan_temperature: INFO
```

---

## Pourquoi limiter les journaux RS485

Les journaux très détaillés peuvent :

* saturer la boucle principale;
* ralentir le traitement;
* provoquer une perte d’octets;
* perturber le protocole RS485;
* générer des erreurs de communication.

En fonctionnement normal, il est recommandé de conserver les niveaux actuels.

---

# Capture des trames RS485

Pour capturer les communications, décommente temporairement le bloc de débogage UART :

```yaml
uart:
  id: rs485
  tx_pin: GPIO17
  rx_pin: GPIO18
  baud_rate: 38400
  data_bits: 8
  parity: NONE
  stop_bits: 1
  rx_buffer_size: 4096

  debug:
    direction: BOTH
    dummy_receiver: false
    sequence:
      - lambda: |-
          UARTDebug::log_hex(direction, bytes, ' ');
```

---

## Capture passive

Pour effectuer une capture passive destinée à l’analyse du protocole, commente temporairement :

```yaml
broan:
  id: broan_erv
  uart_id: rs485
```

Cela permet à l’ESP32 d’écouter le bus sans participer activement aux échanges.

Une fois la capture terminée :

1. désactive le débogage UART;
2. réactive le bloc `broan`;
3. recompile;
4. réinstalle le firmware.

---

# Entités Home Assistant

## Commandes principales

HomeAir crée les commandes suivantes :

* Mode de ventilation;
* Turbo;
* Ventilation maximale;
* Surpassement OVR;
* Vitesse du ventilateur;
* Marche par heure;
* Consigne d’humidité;
* Contrôle d’humidité;
* Heure de pointe;
* Réinitialiser le filtre.

---

## États et mesures

HomeAir expose :

* Mode temporisé;
* Température locale VRE;
* Humidité locale VRE;
* Température mesurée;
* Humidité mesurée;
* Consommation électrique;
* Température air entrant;
* Température air sortant;
* Durée de vie du filtre;
* Débit soufflage;
* Débit extraction;
* Vitesse ventilateur soufflage;
* Vitesse ventilateur extraction;
* Gain de l’échangeur;
* Rendement de récupération.

---

## Entités de diagnostic

Les principales entités de diagnostic sont :

* Mode de ventilation brut;
* Gain de l’échangeur;
* Rendement de récupération;
* Vitesse ventilateur soufflage;
* Vitesse ventilateur extraction.

---

# Fonctionnement sans Home Assistant

Une partie importante de HomeAir continue de fonctionner sans Home Assistant.

Le capteur local AM2302 reste disponible.

Les fonctions locales suivantes peuvent continuer :

* lecture de la température locale;
* lecture de l’humidité locale;
* calcul local avec les capteurs disponibles;
* communication RS485;
* maintien du mode actif;
* exécution d’une minuterie déjà lancée;
* retour automatique au mode Économie;
* transmission de l’humidité locale si le contrôle est actif.

Les capteurs Home Assistant deviennent simplement indisponibles jusqu’au retour de la connexion.

---

# Limites actuelles

La configuration actuelle comporte certaines limites.

## Un seul mode temporisé

Un seul mode temporisé peut être actif à la fois.

Le lancement d’un nouveau mode remplace le précédent.

---

## Retour fixe au mode Économie

À la fin d’une minuterie, HomeAir revient toujours au mode :

```text
Économie
```

Le mode précédent n’est pas mémorisé.

---

## Sélecteurs temporisés

À la fin d’une minuterie, le sélecteur utilisé peut rester visuellement sur la durée précédente dans Home Assistant.

Le fonctionnement réel du VRE revient toutefois au mode Économie.

---

## Température de sortie

Le capteur `Température air sortant` peut retourner `nan` selon le modèle d’échangeur.

---

## Rendement estimé

Le calcul du rendement est indicatif et dépend fortement de la position des capteurs.

---

## Capteurs spécifiques à l’installation

Les entités Home Assistant inscrites dans la configuration sont propres à l’installation actuelle.

Elles doivent être remplacées pour une autre maison.

---

## Compatibilité des modes

Certains modes peuvent ne pas être disponibles sur tous les modèles.

La compatibilité dépend des fonctions exposées par le contrôleur de l’échangeur.

---

# Dépannage

## Aucun échange avec le VRE

Vérifie :

* les broches GPIO17 et GPIO18;
* la vitesse de 38 400 bauds;
* la masse commune;
* D+ et D-;
* l’alimentation du convertisseur;
* le type de convertisseur;
* la présence d’une commande murale série concurrente;
* la direction automatique du module RS485.

---

## Délais d’attente

Les délais d’attente peuvent être causés par :

* une mauvaise configuration UART;
* un mauvais contrôle DE/RE;
* une inversion TX/RX;
* une commande murale toujours connectée;
* un ESP32 alimenté séparément sans masse commune;
* un convertisseur incompatible.

---

## Commandes inconnues ou paquets désalignés

Vérifie :

* D+ et D-;
* la résistance de terminaison;
* le GND;
* le blindage du câble;
* la longueur du câble;
* le niveau de journalisation;
* la qualité du convertisseur.

---

## Valeur `nan`

Une valeur `nan` signifie généralement que :

* le modèle ne fournit pas la donnée;
* la donnée n’est pas encore reçue;
* le capteur n’est pas pris en charge;
* la valeur est invalide;
* la communication n’est pas encore stabilisée.

---

## Le contrôle de l’humidité ne fonctionne pas

Vérifie que :

* `Contrôle d'humidité` est activé;
* une humidité valide est disponible;
* `Humidité mesurée` affiche une valeur;
* la consigne d’humidité est configurée;
* le mode Humidité est actif;
* le modèle prend en charge cette fonction.

---

# Structure recommandée du dépôt

```text
HomeAir/
├── README.md
├── LICENSE
├── components/
│   └── broan/
├── examples/
│   └── homeair.yaml
└── docs/
    ├── installation.md
    ├── wiring.md
    └── troubleshooting.md
```

Le fichier principal de configuration peut être placé dans :

```text
examples/homeair.yaml
```

---

# Crédits

## Projet original

HomeAir est basé sur le projet open source :

[`nspitko/broan_erv_uart`](https://github.com/nspitko/broan_erv_uart)

Auteur original :

**nspitko**

Documentation du protocole :

[Reverse Engineering an ERV](https://spitko.net/2025/08/08/Reverse-Engineering-an-ERV/)

Le composant Broan, le protocole RS485 documenté et une partie essentielle de la logique de communication proviennent du projet original.

Merci à **nspitko** pour :

* le travail de rétro-ingénierie;
* l’analyse du protocole;
* la publication du composant ESPHome;
* la documentation technique;
* la mise à disposition du projet à la communauté.

---

## Adaptation HomeAir

HomeAir est maintenu par **DataVison**.

Les ajouts propres à HomeAir comprennent notamment :

* la configuration ESP32-S3;
* l’interface française;
* les modes simplifiés;
* les modes temporisés;
* le retour automatique au mode Économie;
* l’intégration du capteur AM2302;
* l’agrégation des capteurs Home Assistant;
* le calcul de l’humidité moyenne;
* le calcul de la température moyenne;
* le fonctionnement de secours local;
* le gain thermique;
* l’estimation du rendement;
* l’indicateur Heure de pointe;
* la structure des entités Home Assistant;
* les adaptations pour une utilisation résidentielle.

---

# Marques de commerce

Les noms suivants peuvent être des marques appartenant à leurs propriétaires respectifs :

* Broan;
* NuTone;
* Venmar;
* vanEE;
* ESPHome;
* Home Assistant.

L’utilisation de ces noms sert uniquement à décrire la compatibilité technique du projet.

HomeAir n’est pas un produit officiel de ces fabricants.

---

# Licence

HomeAir doit respecter la licence du projet original.

Avant de publier, modifier ou redistribuer le projet :

1. consulte le fichier `LICENSE` du dépôt original;
2. conserve les mentions de droits d’auteur;
3. conserve l’attribution à l’auteur initial;
4. ajoute un fichier `LICENSE` dans HomeAir;
5. indique clairement les modifications apportées;
6. ne supprime pas les notices présentes dans le code source original.

Dépôt original :

```text
https://github.com/nspitko/broan_erv_uart
```

---

# Sécurité

Toute intervention sur un échangeur d’air doit être effectuée avec prudence.

Avant le branchement :

* coupe l’alimentation de l’échangeur;
* vérifie l’absence de tension;
* identifie correctement les bornes;
* protège les conducteurs;
* utilise une alimentation adaptée;
* ne branche jamais le 12 V directement sur le 3,3 V;
* ne laisse pas le montage exposé;
* installe l’ESP32 et le convertisseur dans un boîtier;
* évite les courts-circuits;
* respecte les normes électriques locales.

Les bornes de commande basse tension ne garantissent pas que l’ensemble de l’appareil soit sans danger.

Certaines parties internes de l’échangeur peuvent être alimentées par la tension secteur.

---

# Avertissement

HomeAir est fourni sans garantie.

L’auteur de HomeAir et l’auteur du projet original ne peuvent être tenus responsables :

* d’un dommage matériel;
* d’une panne de l’échangeur;
* d’une perte de ventilation;
* d’une mauvaise qualité de l’air;
* d’un dommage électrique;
* d’une mauvaise installation;
* d’une perte de données;
* d’une incompatibilité avec un modèle particulier.

L’utilisation, l’installation et la modification du projet sont effectuées aux risques de l’utilisateur.

---

# Contribution

Les contributions sont les bienvenues.

Tu peux contribuer en fournissant :

* des tests sur de nouveaux modèles;
* des captures de trames RS485;
* des configurations pour de nouvelles cartes;
* des corrections de bogues;
* des améliorations de documentation;
* de nouvelles fonctions;
* des traductions;
* des exemples Home Assistant;
* des schémas de branchement;
* des rapports de compatibilité.

Lors de l’ouverture d’une issue, indique idéalement :

* la marque de l’échangeur;
* le modèle exact;
* le type VRC ou VRE;
* la commande murale originale;
* la carte ESP32 utilisée;
* le convertisseur RS485;
* les broches UART;
* la configuration ESPHome;
* les journaux;
* les trames pertinentes;
* la fonction attendue;
* le comportement observé.

---

# Résumé

HomeAir fournit une intégration avancée entre un échangeur d’air compatible RS485, ESPHome et Home Assistant.

Le projet combine :

* le composant RS485 original de nspitko;
* un ESP32-S3;
* des capteurs locaux;
* des capteurs Home Assistant;
* des modes temporisés;
* une gestion simplifiée;
* des diagnostics;
* une estimation du rendement;
* un fonctionnement local résilient.

HomeAir vise à rendre le contrôle d’un échangeur d’air plus accessible, automatisable et adapté à une maison connectée, tout en conservant une attribution claire au projet open source qui a rendu cette intégration possible.
