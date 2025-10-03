# TP PROGRES TME1

Ce dépôt contient les solutions du TP1 du module **PROGRES**, portant sur la programmation de clients/serveurs UDP et TCP en Python.

les prompts utilisés dans ce TP: https://chatgpt.com/share/68dd969a-20f4-8011-b235-2e50fb6b5012. Le reste a été réalisé avec nos compétences personnelles.
---

## Structure du dépôt

### Exercice 1 – Client/Serveur UDP (Ping)
- `EXO1/udp_server.py` : serveur UDP qui reçoit et renvoie les messages du client.
- `EXO1/udp_client_without_expectation.py` : client UDP basique qui envoie un message (ping) et mesure le RTT.
- `EXO1/udp_client_with_expectation.py` : client UDP amélioré, capable de gérer les pertes de réponses.
- `EXO1/server_prob.py` : version du serveur qui oublie de répondre avec une probabilité de 0.5 (simule la perte de paquets).
- `EXO1/Multi_clients_Boucle.py` : script pour lancer **plusieurs clients UDP simultanément** et observer l’interleaving des messages et le RTT moyen par client.

### Exercice 2 – Client/Serveur TCP (Time server)
- `EXO2/server_tcp.py` : serveur TCP qui envoie l’heure locale au client.
- `EXO2/tcp_client.py` : client TCP qui interroge le serveur et calcule la différence d’horloge.
- `EXO2/multi_clients_tcp.py` : script pour lancer **plusieurs clients TCP simultanément**, permettant de tester la gestion de plusieurs connexions concurrentes par le serveur.

### Exercice 3 – Serveur Web
- `EXO3/server.py` : serveur Web minimaliste (réponses HTTP simples).
- `EXO3/index.html` : page de test à servir via le serveur.

---

## Exécution

> ⚠️ Avant d’exécuter les programmes, assurez-vous d’avoir **Python 3** installé.  
> - Sur Linux/Mac : `python3`  
> - Sur Windows : `python`  

### Exercice 1 – UDP Ping

Se placer dans le dossier du TP :  
```bash
cd ~/Download/TME01_harima_tireche
```

#### 1. Client/Serveur simple
Dans un premier terminal (serveur) :  
```bash
python3 EXO1/udp_server.py
```

Dans un second terminal (client) :  
```bash
python3 EXO1/udp_client_without_expectation.py
```

💡 **Sur différentes machines** : le code reste identique, seule l’adresse IP du serveur doit être changée dans le client.  

> Pour arrêter le serveur ou le client : **Ctrl+C**

---

#### 2. Plusieurs clients simultanés
- Terminal 1 (serveur) :  
  ```bash
  python3 EXO1/udp_server.py
  ```
- Terminal 2 (Multi UDP clients) :  
  ```bash
  python3 EXO1/Multi_clients_Boucle.py
  ```

---

#### 3. Gestion des pertes
- Lancer le serveur avec pertes :  
  ```bash
  python3 EXO1/server_prob.py
  ```
- Lancer le client classique :  
  ```bash
  python3 EXO1/udp_client_without_expectation.py
  ```
- Lancer le client tolérant aux pertes :  
  ```bash
  python3 EXO1/udp_client_with_expectation.py
  ```

---

### Exercice 2 – TCP Time
- Serveur (terminal 1) :  
  ```bash
  python3 EXO2/server_tcp.py
  ```
- Client (terminal 2) :  
  ```bash
  python3 EXO2/tcp_client.py
  ```
- Multi TCP Clients :  
  ```bash
  python3 EXO2/multi_clients_tcp.py
  ```
---

### Exercice 3 – Web Server
- Lancer le serveur Web :  
  ```bash
  python3 EXO3/server.py
  ```
- Ouvrir un navigateur et accéder à :  
  ```
  http://localhost:8080/ #OR http://localhost:8080/index.html (the server handle that)
  ```
- Pour tester plusieurs requêtes au serveur, il existe deux méthodes :  

  - Ouvrir plusieurs fenêtres ou onglets dans le navigateur web.  
  - Exécuter la commande suivante dans le terminal (MacOS/Linux) :
  ```bash
  for i in {1..100}; do echo "Request $i"; curl -s http://127.0.0.1:8080; echo; done
  ```
  - Cette commande permet de générer 100 requêtes automatiquement.  
- **Remarque :** assurez-vous que `curl` est installé et que le serveur est lancé avant de l’exécuter.
---

## Tests recommandés
- **Localhost** (même machine).  
- **Deux machines différentes** (modifier l’adresse IP du serveur dans le client).  

---

## Auteur
- *HARIMA Mourad Abdelmohcen*  
- *TIRECHE Lina*  
