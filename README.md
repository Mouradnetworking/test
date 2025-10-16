
# Compte rendu 

**Mini-Projet PROGRES 2025 – MP1**

*Auteur : [tireche lina ,HARIMA A. Mourad]*  
*Date : 16 octobre 2025*

---

# EXO 1 - Relai TCP Client-Serveur en Python ||| Prompt: https://chatgpt.com/share/68f17eaa-af30-8011-82f9-1015a6addc5a

## Description

Cet exercice implémente un **relai TCP** entre un client et un serveur en Python.  
Le relai agit comme un **serveur pour le client** et comme un **client pour le serveur**, en retransmettant toutes les données dans les deux sens.  

Le projet contient trois fichiers principaux :  

- `server.py` : serveur TCP qui reçoit et répond aux messages du client.  
- `client.py` : client TCP qui envoie des messages au serveur.  
- `relay.py` : relai TCP qui fait le lien entre le client et le serveur, gérant le transfert bidirectionnel et pouvant servir plusieurs clients simultanément.

---

## Prérequis

- Python 3.x  
- Pas de dépendances externes, seulement la bibliothèque standard (`socket`, `threading`, `sys`)  

---

## Exécution

### 1. Lancer le serveur TCP

```bash
python3 server.py
```
Le serveur écoute sur le port défini dans le code (ex : 5555).

2. Lancer le relai TCP
```bash
python3 relay.py <server_ip> <server_port>
```

Exemple pour un serveur local :

```bash
python3 relay.py 127.0.0.1 5555
```
- Le relai écoute sur le port 5555 (modifiable dans le code) et relaie toutes les connexions vers le serveur spécifié.

3. Lancer le client TCP
```bash
python3 client.py
```

Par défaut, le client se connecte au relai sur 127.0.0.1:5555.

Le client peut envoyer un message et reçoit la réponse du serveur via le relai.

## Fonctionnement général

* Le client envoie une requête au relai.
* Le relai se connecte au serveur et transmet la requête.
* Le serveur traite la requête et renvoie la réponse au relai.
* Le relai transmet la réponse au client.

La communication peut être simultanée et multi-clients, grâce à l’utilisation de threads.

### Test sur différentes machines (Question 2)

Pour tester le relai et les fichiers sur des machines différentes, nous avons utilisé un **conteneur Docker** pour exécuter le client.  

Le conteneur a été lancé avec la commande suivante afin de lui permettre d’accéder au réseau de l’hôte :

```bash
docker run -it --add-host=host.docker.internal:host-gateway ubuntu bash
```
Cela permet d’utiliser **host.docker.internal** comme adresse du relai depuis le conteneur, au lieu de l’adresse loopback (127.0.0.1).

Exemple d’exécution depuis le conteneur Docker :

```bash
root@c16769efe5ca:/# python3 client.py 
Connecté au serveur host.docker.internal:5555
Entrez un message: Hello From Docker container 
Réponse du serveur: HELLO FROM DOCKER CONTAINER 
 bye
```

Le client dans le conteneur Docker s’est connecté avec succès au relai et au serveur exécutés sur le PC hôte.

Cela a permis de simuler un environnement multi-machines tout en utilisant une seule machine physique.

# Test multi-clients simultanés
Pour tester plusieurs clients en même temps, il suffit de lancer plusieurs instances du client ou d’utiliser une boucle dans un seul script client.

Exemple avec une boucle Python :

```bash
for i in $(seq 1 10); do
    python3 client.py
done
```

Le relai crée un thread par client, permettant à tous les clients de communiquer simultanément avec le serveur.

Points importants

* Chaque client est géré dans un thread séparé pour permettre plusieurs connexions simultanées.
* Le relai assure un transfert bidirectionnel complet.
* Les sockets sont fermés proprement même en cas d’erreur ou de déconnexion.


---

# Compte rendu – Exercice 2 : Relais HTTP
##  Objectif

L'objectif de l'exercice 2 est de transformer le **relai TCP** de l'exercice 1 en un **relai HTTP** spécialisé, en implémentant trois fonctionnalités distinctes :

1. **Cache HTTP**
2. **Sniffeur HTTP (logger)**
3. **Censeur HTTP**

Chaque relai agit comme un proxy intermédiaire entre un **client HTTP** (navigateur ou `curl`) et un **serveur HTTP**.

Enfin, on doit **tester ces relais individuellement**, puis **enchaînés** dans une chaîne.

---

## 1 Cache HTTP

### Fonctionnalité

- Stocke en mémoire la **réponse du serveur** pour chaque URI `GET`.
- En cas de requête identique ultérieure → répond **directement depuis le cache**, sans contacter le serveur.

### Implémentation

- Dictionnaire `CACHE = {}` avec clé = URI, valeur = réponse de serveur.
- Analyse de la requête avec une expression régulière :

```python
re.match(rb"GET (\S+) HTTP/1\.[01]", request)
```
prompt qui nous a aider a realiser le code : Créer un programme Python a partir du relai tcp exo1 qui va fonctionne comme un proxy HTTP avec système de cache. Ce programme doit écouter sur le port 5555 et rediriger les requêtes vers un serveur spécifié en paramètre. Quand il reçoit une requête GET d'un client, il doit d'abord vérifier si la réponse est déjà stockée dans un cache en mémoire. Si c'est le cas, il renvoie directement la réponse mise en cache. Sinon, il forwarde la requête au vrai serveur, stocke la réponse dans le cache, et la renvoie au client. Le programme doit pouvoir gérer plusieurs clients simultanément avec des threads et extraire l'URI des requêtes GET pour utiliser comme clé de cache.
### Test

```bash
python3 cache.py localhost 8000
curl http://localhost:8080/test
curl http://localhost:8080/test  # 2ᵉ fois → pas de nouvelle requête au serveur
```

 **Résultat** : le vrai serveur ne reçoit qu'une seule requête pour la même URI pour gagner en latence.

---

## 2 Sniffeur HTTP (Logger)

### Fonctionnalité

- Archive dans `http.log` (format JSON) :
  - Adresse IP du client
  - URI demandée
- Fournit un outil de recherche : `search_log.py <partie_URI>`

### Implémentation

- Écriture atomique dans `http.log` à chaque requête `GET` avec réponse non vide.
- Programme de recherche qui filtre les entrées par URI.

*** outils ***
prompt qui a aider a generer la fonction handle_client :
Crée une fonction qui gère entièrement le processus de relais HTTP entre un client et un serveur distant : elle doit recevoir la requête du client, établir une connexion vers le serveur cible, transmettre la requête, collecter la réponse complète par morceaux tout en gérant les flux de données potentiellement fragmentés, journaliser l'adresse IP du client et l'URI demandée dans un fichier de logs au format JSON pour traçabilité, puis renvoyer la réponse au client, le tout en assurant une gestion robuste des erreurs réseau et une fermeture propre des sockets dans tous les scénarios pour éviter les fuites de ressources.
### Test

```bash
python3 sniff.py localhost 8000
curl http://localhost:8080/secret
python3 search_log.py secret
```

✅ **Résultat** : affiche `Client IP: 127.0.0.1 | URI: /secret`

---

## 3 Censeur HTTP

### Fonctionnalité

- Lit une liste de sites/URI interdits depuis `blocked_sites.txt`.
- Si une requête contient une URI interdite :
  - Répond `"Interdit"` au client (donc le relais agit comme un filtre ou bien firwall)
  - **Ne contacte pas** le serveur
  - Logge l'IP du client dans `blocked.log` pour garder la trace.

### Fichier de configuration (`blocked_sites.txt`)

```txt
/secret
/sorbonne
facebook.com
```

### Test

```bash
python3 censure.py localhost 8000
curl http://localhost:8080/secret
```

✅ **Résultat** : réponse = `"Interdit"`, et `blocked.log` contient l'IP du client.

---

##  Enchaînement des relais

### Chaîne utilisée

```
Client 
  → Cache (8080) 
    → sniff (8081) 
      → Censeur (8082) 
        → Serveur (8000)
```

### Lancement (4 terminaux)

```bash
# Terminal 1
python3 server.py

# Terminal 2
python3 censure.py localhost 8000        # écoute sur 8082

# Terminal 3
python3 sniff.py localhost 8082        # écoute sur 8081

# Terminal 4
python3 cache.py localhost 8081         # écoute sur 8080
```

### Tests de la chaîne

- `curl http://localhost:8080/test` → OK, loggé, mis en cache.
- `curl http://localhost:8080/sorbonne` → `"Interdit"`, loggé par le sniffeur, bloqué par le censeur, serveur non contacté.
- Rechargement de `/test` → servi depuis le cache (pas de log supplémentaire).

✅ **Toutes les fonctionnalités coexistent correctement**.

---

## 🛠️ Fichiers fournis

| Fichier | Rôle |
|--------|------|
| `server.py` | Serveur HTTP minimal (port 8000) |
| `cache.py` | Relai avec cache (port 8080) |
| `sniff.py` | Relai avec logging (port 8081) |
| `censure.py` | Relai avec censure (port 8082) |
| `blocked_sites.txt` | Liste des URI interdites |
| `search_log.py` | Outil de recherche dans les logs |
| `blocked.py` | Pour garder la tracabilité |

---

##  Conclusion

L'exercice 2 a permis de :

- Comprendre le fonctionnement des **proxies HTTP**
- Implémenter des **fonctionnalités réalistes** (cache, audit, censure)
- Tester des **architectures en chaîne** de relais
- Manipuler les **sockets brutes**, les **expressions régulières**, et la **gestion de fichiers**

Toutes les exigences de l'énoncé ont été respectées :

- Relais individuels fonctionnels
- Enchaînement réussi
- Bonne gestion des connexions simultanées (via threads)
- Logs et cache persistants et exploitables

---

*Fin du compte rendu.*
