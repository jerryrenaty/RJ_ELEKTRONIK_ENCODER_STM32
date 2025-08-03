Description des codes pour le module encodeur RJ-ELEKTRONIK_STM32
Fonctionnalités principales
Ce module permet de gérer un encodeur rotatif avec bouton intégré sur des microcontrôleurs STM32 en utilisant les périphériques HAL. Il offre:

Lecture de la position de l'encodeur rotatif

Détection des appuis sur le bouton avec anti-rebond (debounce)

Configuration avancée du timer pour une lecture précise

Fichier d'en-tête (RJ-ELEKTRONIK_ENCODER_STM32.h)
Structure de données
Encoder_HandleTypeDef: Structure contenant:

htim: Pointeur vers le timer utilisé pour l'encodeur

btn_port et btn_pin: Configuration GPIO du bouton

position: Valeur accumulée de la position de l'encodeur

last_time: Dernier temps enregistré pour le debounce

btn_state: État actuel du bouton

Fonctions exposées
Encoder_Init(): Initialise l'encodeur et le bouton

Encoder_Read(): Lit la position cumulée de l'encodeur

Encoder_ButtonPressed(): Détecte les appuis sur le bouton avec debounce

Fichier d'implémentation (RJ-ELEKTRONIK_ENCODER_STM32.c)
Fonction Encoder_Init()
Initialise le timer en mode encodeur avec:

Mode TI12 (détection sur les deux canaux)

Filtre important (valeur 0xF) pour réduire le bruit

Pas de préscaler sur les entrées

Polarité sur front montant

Réinitialise le compteur et démarre le timer

Fonction Encoder_Read()
Lit la valeur courante du compteur du timer

Ajoute cette valeur (convertie en int16_t) à la position cumulée

Réinitialise le compteur à 0

Retourne la position totale

Fonction Encoder_ButtonPressed()
Implémente un mécanisme de debounce matériel/logiciel

Vérifie l'état du bouton avec un temps minimum entre les détections (debounce_time)

Met à jour le timestamp du dernier appui valide

Retourne 1 si un nouvel appui est détecté, 0 sinon

Points forts
Utilisation des périphériques matériels dédiés (timer en mode encodeur)

Gestion du débounce logiciel configurable

Accumulation de la position sur 32 bits (évite les débordements)

Configuration flexible des broches GPIO

Compatible avec la HAL STM32

Ce module est idéal pour intégrer des encodeurs rotatifs dans des projets STM32, offrant une interface simple et efficace pour lire la position et détecter les appuis sur le bouton.

#include "RJ-ELEKTRONIK_ENCODER_STM32.h"
#include "stm32f4xx_hal.h"
#include <stdio.h>

// Déclaration des handlers
extern TIM_HandleTypeDef htim3;  // Timer utilisé pour l'encodeur
extern UART_HandleTypeDef huart2; // Pour le debug (optionnel)

Encoder_HandleTypeDef encoder;

int main(void) {
    // Initialisation HAL
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_TIM3_Init();
    MX_USART2_UART_Init();
    
    // Initialisation de l'encodeur
    Encoder_Init(&encoder, &htim3, GPIOC, GPIO_PIN_13);
    
    // Valeur précédente pour détection de changement
    int32_t last_position = 0;
    uint8_t last_button_state = 0;
    
    while (1) {
        // Lecture de l'encodeur
        int32_t current_pos = Encoder_Read(&encoder);
        
        // Détection de changement de position
        if (current_pos != last_position) {
            last_position = current_pos;
            
            // Affichage de la position (optionnel - via UART)
            printf("Position: %ld\r\n", current_pos);
        }
        
        // Détection d'appui sur le bouton (debounce de 50ms)
        uint8_t btn_pressed = Encoder_ButtonPressed(&encoder, 50);
        if (btn_pressed && !last_button_state) {
            printf("Bouton appuyé!\r\n");
        }
        last_button_state = btn_pressed;
        
        // Petit délai pour réduire la charge CPU
        HAL_Delay(10);
    }
}

// Fonction printf redirigée vers UART (optionnel)
int __io_putchar(int ch) {
    HAL_UART_Transmit(&huart2, (uint8_t *)&ch, 1, HAL_MAX_DELAY);
    return ch;
}

Explications
Initialisation:

On initialise l'encodeur avec le timer TIM3 (qui doit être configuré en mode encodeur dans CubeMX)

Le bouton est connecté sur PC13 avec une résistance de pull-up

Boucle principale:

On lit régulièrement la position de l'encodeur

On détecte les changements de position pour éviter de traiter les mêmes valeurs

On gère les appuis sur le bouton avec un temps de debounce de 50ms

On affiche les informations via UART (optionnel)

Fonctionnalités démontrées:

Lecture de la position absolue de l'encodeur

Détection des rotations (incrément/décrément)

Gestion du bouton avec anti-rebond

Affichage des informations (pour debug)

Configuration CubeMX requise
Timer:

Sélectionner un timer (ex: TIM3)

Mode: Encoder Mode

Channel 1 et Channel 2 activés

Paramètres par défaut (pas de inversion, etc.)

GPIO:

Broche du bouton configurée en entrée avec pull-up

Broches du timer (PA6/PA7 pour TIM3) en mode Alternate Function

UART (optionnel pour le debug):

Configurer un UART en mode Asynchrone

Baud rate standard (ex: 115200)

Cet exemple montre comment intégrer facilement la bibliothèque dans un projet STM32 pour gérer un encodeur rotatif complet avec bouton intégré.
