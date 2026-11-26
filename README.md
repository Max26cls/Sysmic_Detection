<div align="center">

# Projet intégré STM32 – Étape 1

### Détection distribuée de secousses sismiques

![STM32](https://img.shields.io/badge/STM32-F767ZI-blue)
![FreeRTOS](https://img.shields.io/badge/FreeRTOS-green)
![Language](https://img.shields.io/badge/Language-C-orange)
![Status](https://img.shields.io/badge/Statut-En_cours-yellow)

</div>

## 1. Objectif
L'objectif de cette première étape est de mettre en place les fondations matérielles et logicielles du système embarqué sur la carte **NUCLEO-F767ZI**.

Cette étape consiste à développer une architecture multitâche basée sur **FreeRTOS**, permettant l'acquisition des données de l'accéléromètre, la supervision du fonctionnement du programme et l'établissement des premières communications réseau.

Une attention particulière sera portée à la **modularité du code**, à la réutilisation des fonctions et à la séparation des différentes responsabilités afin de faciliter l'intégration des fonctionnalités des étapes suivantes. 

---

## 2. Fonctionnalités à implémenter

- [ ] Configuration des périphériques dans STM32CubeIDE.
- [ ] Mise en place de FreeRTOS.
- [ ] Implémentation de la tâche maître.
- [ ] Acquisition des trois axes de l'accéléromètre ADXL335 par DMA.
- [ ] Échantillonnage à une fréquence de 100 Hz.
- [ ] Implémentation d'un heartbeat.
- [ ] Gestion centralisée des messages de debug via UART.*cette tache sera peut etre pas presente, une autre technique de debug est envisagée.
- [ ] Configuration de la communication Ethernet avec LwIP.
- [ ] Envoi d'un message UDP broadcast toutes les 30 secondes.
- [ ] Tests unitaires des différents modules.
- [ ] Intégration et validation globale de l'étape 1.

---

## 2. Description des différentes Task de l'étape 1

### 1. Tâche maître — MasterTask

La tâche maître sera responsable de la gestion et de la supervision générale du système.

**Responsabilités :**

- Détecter l'appui sur le bouton du mode "pause".
- Gérer l'état du système (actif ou arrêté).
- La LED Rouge indiquera si le systeme est en mode de fonctionnement.
- Autoriser le démarrage des différentes tâches.
- Assurer l'arrêt contrôlé des différentes fonctionnalités.
- Synchroniser le fonctionnement des tâches.
- Envoyer des messages de diagnostic à la tâche de Debug.

---

### 2. Tâche d'acquisition — AcquisitionTask

Cette tâche sera responsable de l'acquisition des données provenant de l'accéléromètre ADXL335.

**Responsabilités :**

- Démarrer et arrêter l'acquisition ADC avec DMA.
- Utiliser un timer pour déclencher les conversions à une fréquence de 100 Hz.
- Récupérer les valeurs des axes X, Y et Z.
- Attendre la disponibilité des nouvelles données à l'aide d'un sémaphore ou d'une notification.
- Mettre les données acquises à disposition des autres modules.
- Transmettre des informations de diagnostic à la tâche Debug.

> [!NOTE]
> Aucun traitement RMS ni filtrage des données ne sera effectué durant l'étape 1.

---

### 3. Tâche Heartbeat — HeartbeatTask

Cette tâche permettra de vérifier visuellement que le programme est en cours d'exécution grâce au clignotement périodique d'une LED.

**Responsabilités :**

- Initialiser la gestion de la LED heartbeat.
- Allumer, éteindre et inverser l'état de la LED.
- Générer un clignotement périodique.
- Utiliser les mécanismes de temporisation de FreeRTOS.
- Respecter les instructions de démarrage et d'arrêt de la tâche maître.

La période du heartbeat sera configurable afin de pouvoir être modifiée facilement.

---

### 4. Tâche Debug (DebugTask)

La tâche Debug aurait pour objectif de centraliser les messages
de diagnostic provenant des différentes tâches du système
et de les transmettre via UART.

Cependant, STM32CubeIDE propose des outils de diagnostic
spécifiquement adaptés à FreeRTOS.

Ces outils permettent notamment de :

- Visualiser les différentes tâches et leurs états.
- Vérifier les priorités des tâches.
- Analyser l'utilisation de la mémoire Stack.
- Analyser le temps d'exécution de chaque tâche.
- Observer les sémaphores.
- Observer les files de messages.
- Visualiser les timers FreeRTOS.

Nous étudierons la possibilité d'exploiter ces outils afin
de faciliter le développement et le diagnostic du système.


**Documentation :**

[How to enable FreeRTOS Run Time and Stack Usage view](https://community.st.com/stm32-mcus-60/how-to-enable-freertos-run-time-and-stack-usage-view-126326)

---

### 5. Tâche Broadcast (BroadcastTask)

La tâche Broadcast permettra à chaque carte NUCLEO de signaler périodiquement sa présence sur le réseau Ethernet.

La communication reposera sur le protocole UDP et sur l'utilisation de la bibliothèque LwIP.

**Fonctionnalités à implémenter :**

- Vérification de la disponibilité du réseau.
- Création d'un message de présence au format JSON.
- Identification de la carte NUCLEO.
- Envoi du message vers l'adresse broadcast du réseau.
- Répétition de l'envoi toutes les 30 secondes.
- Gestion des erreurs de transmission.
- Transmission des informations de diagnostic à la tâche Debug, si celle-ci est conservée.
- Démarrage et arrêt des envois sur instruction de la tâche maître.

**Configuration réseau :**

| Paramètre | Valeur |
|---|---|
| Réseau | `192.168.1.0/28` |
| Masque | `255.255.255.240` |
| Adresse IP | `192.168.1.7` |
| Adresse broadcast | `192.168.1.15` |
| Protocole | UDP |
| Port | `12345` |
| Périodicité | 30 secondes |
| Format des messages | JSON |

#### Format du message de présence

Le message envoyé périodiquement devra respecter la structure JSON définie dans l'énoncé du projet.

```json
{
    "type": "presence",
    "id": "nucleo-01",
    "ip": "192.168.1.2",
    "timestamp": "2025-10-02T08:20:00Z"
}
```

**Description des champs :**

| Champ | Type | Description |
|---|---|---|
| `type` | String | Type du message (`presence`) |
| `id` | String | Identifiant unique de la carte NUCLEO |
| `ip` | String | Adresse IP de la carte émettrice |
| `timestamp` | String | Date et heure d'émission du message au format ISO 8601 |

> [!NOTE]
> L'adresse IP `192.168.1.2` est utilisée à titre d'exemple afin de respecter le sous-réseau imposé `192.168.1.0/28`.
> L'identifiant et l'adresse IP devront être propres à chaque carte.
> L'horodatage présenté est un exemple. Sa gestion définitive sera réalisée lors de l'intégration de la RTC et de la synchronisation NTP.


---

## Configuration matérielle et logicielle — STM32CubeMX

### 1. Configuration des horloges (Clock Configuration)

### 2. Configuration des GPIO

### 3. Configuration de l'ADC

### 4. Configuration du DMA

### 5. Configuration du Timer et du TRGO

### 6. Configuration de l'UART

### 7. Configuration Ethernet

### 8. Configuration de LwIP

### 9. Configuration de FreeRTOS

#### 9.1. Création des tâches

#### 9.2. Priorités des tâches

#### 9.3. Taille des stacks

#### 9.4. Sémaphores et mécanismes de synchronisation

#### 9.5. Message Queue

### 10. Configuration des outils de debug FreeRTOS

## Fichiers

- [Projet STM32CubeIDE](firmware/ProjetIntegrerFreertos)
- [Configuration CubeMX](firmware/ProjetIntegrerFreertos/ProjetIntegrerFreertos.ioc)
- [Logigramme Draw.io](docs/Logigramme_reseau_sismique.drawio)
