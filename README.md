# TP Réseaux – PROGRES TME1

Ce dépôt contient les solutions du TP1 du cours **PROGRES**, portant sur la programmation de clients/serveurs en Python avec UDP et TCP.

---

## Structure du dépôt

### Exercice 1 – Client/Serveur UDP (Ping)
- `EXO1/udp_server.py` : serveur UDP qui reçoit et renvoie les messages du client.
- `EXO1/udp_client_without_expectation.py` : client UDP basique qui envoie un message et mesure le RTT.
- `EXO1/udp_client_with_expectation.py` : client UDP amélioré, capable de gérer les pertes de réponses.
- `EXO1/server_prob.py` : version du serveur qui oublie de répondre avec une probabilité de 0.5 (simule la perte de paquets).

### Exercice 2 – Client/Serveur TCP (Time server)
- `EXO2/server_tcp.py` : serveur TCP qui envoie l’heure locale au client.
- `EXO2/tcp_client.py` : client TCP qui interroge le serveur et calcule la différence d’horloge.

### Exercice 3 – Serveur Web
- `EXO3/server.py` : serveur Web minimaliste (réponses HTTP simples).
- `EXO3/server_promp1.py` : variante du serveur Web, avec fonctionnalités supplémentaires.
- `EXO3/index.html` : page de test à servir via le serveur.

### Autres fichiers
- `client2.py` et `server2.py` : versions alternatives ou brouillons de tests.

---

## Exécution

> ⚠️ Avant d’exécuter les programmes, assurez-vous d’avoir **Python 3** installé.  
> Sur Linux/Mac : `python3`  
> Sur Windows : `python`

### Exercice 1 – UDP Ping
1. Lancer le serveur dans un terminal :
   ```bash
   python3 EXO1/udp_server.py
