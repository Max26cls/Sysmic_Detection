# Project_-C_Sysmic_Detection_FreeRTOS_F767ZI

Détection distribuée de secousses sismiques sur NUCLEO-F767ZI. Le nœud utilise FreeRTOS avec l'interface CMSIS-RTOS v2 et communique sur Ethernet.

## Arborescence

```text
.
├── README.md
├── .gitignore
├── docs/
│   └── Logigramme_reseau_sismique.drawio
└── firmware/
    └── ProjetIntegrerFreertos/
        ├── ProjetIntegrerFreertos.ioc
        ├── Core/          # Code de l'application et démarrage du STM32
        ├── Drivers/       # HAL, pilotes de la carte et en-têtes CMSIS
        ├── LWIP/          # Configuration et adaptation Ethernet
        └── Middlewares/   # FreeRTOS, CMSIS-RTOS v2 et LwIP
```

## Ouvrir le firmware

1. Dans STM32CubeIDE, importe un projet existant depuis `firmware/ProjetIntegrerFreertos/`.
2. Sélectionne la configuration `Debug` et compile le projet.
3. Ouvre `ProjetIntegrerFreertos.ioc` avec STM32CubeMX pour modifier les périphériques. Le projet utilise le pack STM32Cube FW F7 V1.17.4.

Le dossier `Drivers/CMSIS` est exclu des sources à compiler dans `.cproject`. CubeIDE utilise ses en-têtes et compile l'adaptateur CMSIS-RTOS v2 depuis `Middlewares/Third_Party/FreeRTOS`. Vérifie cette exclusion après une régénération du projet.

## Branches et versions

Les branches d'étape partent de `main`. Après validation, l'équipe ouvre une pull request vers `main` puis crée le tag sur le commit fusionné. Pour ouvrir l'étape suivante, mets d'abord `main` à jour avec `git pull --ff-only`.

| Branche | Périmètre | Tag après validation et fusion |
| --- | --- | --- |
| `etape-1` | Tâche maître, heartbeat, acquisition ADC/DMA à 100 Hz, UART de debug et broadcast de présence | `v0.1.0` |
| `etape-2` | Moyenne glissante, RMS sur 1 s et échanges client/serveur | `v0.2.0` |
| `etape-3` | FRAM, RTC et synchronisation, validation collective et alarme | `v1.0.0` |

Fais un commit par changement que l'équipe peut relire ou tester. Par exemple, `feat(acquisition): démarrer l'ADC et le DMA à 100 Hz`.

### État du firmware à l'import

- Le heartbeat sur LD1 et le changement d'état via B1 avec LD3 ont été testés sur la carte.
- La tâche d'acquisition attend la commande de marche. Le démarrage ADC/DMA et le contrôle à 100 Hz restent à réaliser.
- Le broadcast de présence et la sortie de debug restent à réaliser.
