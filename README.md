# Détection sismique sur STM32

Projet de cours sur Nucleo-F767ZI avec un accéléromètre ADXL335. Le firmware tourne sous FreeRTOS (CMSIS-RTOS v2). Le projet porte sur la détection des secousses et l’échange de mesures entre cartes via Ethernet.

## Étape 1

- La LED verte LD1 change d’état toutes les 500 ms pour le heartbeat.
- Le bouton B1 bascule entre marche et arrêt. La LED rouge LD3 s’allume en mode marche.
- ADC1, le DMA et TIM2 sont configurés pour les trois axes à 100 Hz. La tâche d’acquisition attend le signal de marche. Le lancement des mesures et le broadcast UDP restent à faire.

## Fichiers

- [Projet STM32CubeIDE](firmware/ProjetIntegrerFreertos)
- [Configuration CubeMX](firmware/ProjetIntegrerFreertos/ProjetIntegrerFreertos.ioc)
- [Logigramme Draw.io](docs/Logigramme_reseau_sismique.drawio)
