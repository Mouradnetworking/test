
# Compte rendu 

**Mini-Projet PROGRES 2025 – MP1**

*Auteur : [tireche lina ,HARIMA A. Mourad]*  
*Date : 16 octobre 2025*

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

## Architecture générale

Tous les relais suivent le même principe :

- Écoute sur un **port local fixe** (codé en dur : 8080, 8081, 8082…)
- Prend **2 arguments en ligne de commande** :

```bash
python relai.py <serveur_distant> <port_serveur_distant>
```

- Forward les requêtes HTTP `GET` vers le serveur cible, sauf comportement spécifique.

Le **serveur HTTP de test** (`http_server.py`) répond à toute URI avec une page simple affichant le chemin demandé.

---

## 1️⃣ Cache HTTP

### Fonctionnalité

- Stocke en mémoire la **réponse du serveur** pour chaque URI `GET`.
- En cas de requête identique ultérieure → répond **directement depuis le cache**, sans contacter le serveur.

### Implémentation

- Dictionnaire `CACHE = {}` avec clé = URI, valeur = réponse HTTP brute (bytes).
- Analyse de la requête avec une expression régulière :

```python
re.match(rb"GET (\S+) HTTP/1\.[01]", request)
```

### Test

```bash
python cache.py localhost 8000
curl http://localhost:8080/test
curl http://localhost:8080/test  # 2ᵉ fois → pas de nouvelle requête au serveur
```

✅ **Résultat** : le serveur ne reçoit qu'une seule requête pour la même URI.

---

##  Sniffeur HTTP (Logger)

### Fonctionnalité

- Archive dans `http.log` (format JSON) :
  - Adresse IP du client
  - URI demandée
  - Taille de la réponse
  - Timestamp
- Fournit un outil de recherche : `search_log.py <partie_URI>`

### Implémentation

- Écriture atomique dans `http.log` à chaque requête `GET` avec réponse non vide.
- Programme de recherche qui filtre les entrées par URI.

### Test

```bash
python sniff.py localhost 8000
curl http://localhost:8080/secret
python search_log.py secret
```

✅ **Résultat** : affiche `Client IP: 127.0.0.1 | URI: /secret`

---

## Censeur HTTP

### Fonctionnalité

- Lit une liste de sites/URI interdits depuis `blocked_sites.txt`.
- Si une requête contient une URI interdite :
  - Répond `"Interdit"` au client
  - **Ne contacte pas** le serveur
  - Logge l'IP du client dans `blocked.log`

### Fichier de configuration (`blocked_sites.txt`)

```txt
/secret
/sorbonne
facebook.com
```

### Test

```bash
python censure.py localhost 8000
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
python server.py

# Terminal 2
python censure.py localhost 8000        # écoute sur 8082

# Terminal 3
python sniff.py localhost 8082        # écoute sur 8081

# Terminal 4
python ]cache.py localhost 8081         # écoute sur 8080
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
