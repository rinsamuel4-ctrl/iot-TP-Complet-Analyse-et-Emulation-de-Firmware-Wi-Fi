# Rapport de Vulnérabilités - Firmware Wi-Fi

**Auteur :** Manus AI
**Module :** Sécurité IoT
**Date :** 1er Avril 2026

## Introduction
Ce rapport présente les vulnérabilités identifiées lors de l'analyse statique et dynamique d'un firmware de routeur Wi-Fi simulé. L'objectif est de détailler les faiblesses découvertes et de proposer des recommandations pour les atténuer.

## Vulnérabilités Identifiées

### 1. Identifiants par Défaut / Faibles

*   **Description :** Le firmware contient des comptes utilisateurs (`root`, `admin`) avec des mots de passe par défaut ou des hashs faibles dans le fichier `/etc/shadow`. Ces identifiants sont souvent la première cible des attaquants.
*   **Localisation :** `/etc/passwd`, `/etc/shadow` dans le système de fichiers extrait.
*   **Impact :** Accès non autorisé au système, prise de contrôle du routeur, modification de la configuration, installation de logiciels malveillants.
*   **Preuve :**
    ```text
    root:$1$v9pB8.vD$v/U1y0.5.5.5.5.5.5.5.5:18000:0:99999:7:::
    ```
*   **Recommandations :**
    *   Forcer les utilisateurs à changer les mots de passe par défaut lors de la première connexion.
    *   Implémenter une politique de mots de passe forts (longueur minimale, caractères spéciaux, chiffres).
    *   Utiliser des algorithmes de hachage de mots de passe plus robustes (ex: bcrypt, scrypt).

### 2. Services Exposés Inutiles ou Non Sécurisés

*   **Description :** Le scan Nmap simulé a révélé la présence de plusieurs ports ouverts, indiquant des services potentiellement exposés. Bien que simulés, ces services peuvent représenter des points d'entrée pour des attaques s'ils ne sont pas correctement sécurisés ou s'ils sont inutiles pour le fonctionnement normal du routeur.
*   **Localisation :** Ports réseau ouverts (ex: 22, 80, 5900, 5901, 8333).
*   **Impact :** Augmentation de la surface d'attaque, exposition à des vulnérabilités spécifiques aux services (ex: buffer overflow, failles d'authentification).
*   **Preuve (simulation Nmap) :**
    ```text
    Discovered open port 5900/tcp on 127.0.0.1
    Discovered open port 22/tcp on 127.0.0.1
    ...
    ```
*   **Recommandations :**
    *   Désactiver tous les services non essentiels au fonctionnement du routeur.
    *   Restreindre l'accès aux services d'administration (SSH, interface web) aux adresses IP de confiance ou via un VPN.
    *   Mettre en œuvre un pare-feu pour filtrer le trafic entrant et sortant.
    *   Assurer que tous les services exposés sont à jour et configurés de manière sécurisée.

### 3. Binaire `httpd` Simple (Shell Script)

*   **Description :** Le binaire `httpd` analysé est un simple script shell. Bien que cela puisse être une simplification pour le TP, dans un firmware réel, un serveur web implémenté en shell script pourrait être plus sujet aux injections de commandes si les entrées utilisateur ne sont pas correctement validées.
*   **Localisation :** `/bin/httpd` dans le système de fichiers extrait.
*   **Impact :** Vulnérabilités potentielles aux injections de commandes, exécution de code arbitraire si des entrées non filtrées sont traitées par le script.
*   **Preuve :**
    ```text
    #!/bin/sh
    echo "Server running..."
    ```
*   **Recommandations :**
    *   Utiliser des serveurs web robustes et éprouvés (ex: Lighttpd, Nginx) pour les interfaces d'administration.
    *   Implémenter une validation stricte de toutes les entrées utilisateur pour prévenir les injections.
    *   Éviter l'exécution de commandes système (`system()`, `execve()`) avec des paramètres contrôlés par l'utilisateur.

## Conclusion
Les vulnérabilités identifiées soulignent l'importance d'une analyse approfondie des firmwares IoT. Des mesures préventives telles que des politiques de mots de passe robustes, la désactivation des services inutiles et une implémentation sécurisée des applications web sont cruciales pour protéger ces dispositifs contre les cyberattaques. Le patching défensif est une étape essentielle pour corriger ces faiblesses avant le déploiement. 
