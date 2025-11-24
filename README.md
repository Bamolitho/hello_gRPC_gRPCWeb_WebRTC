# gRPC en Python

Un service HelloService avec une méthode SayHello

L’objectif :
Un microservice HelloService expose une fonction SayHello(name)
Un client l’appelle via gRPC
Le serveur répond : “Hello, <name>”.

Strict. Simple. Parfait pour comprendre.

### Prérequis

```bash
pip install --upgrade pip && pip install grpcio grpcio-tools
```

1. ### Le fichier `hello.proto` (le contrat gRPC)

   C’est *la vérité absolue* :
    Le serveur doit implémenter ça,
    Le client doit appeler ça.

2. ### Génération du code gRPC

   ```lua
   python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. hello.proto
   ```

   Cela génère deux fichiers :

   - `hello_pb2.py` (messages)
   - `hello_pb2_grpc.py` (client + serveur stub)

   Tu ne les modifies jamais à la main.

3. ### Implémentation du serveur gRPC (`server.py`):

Ce code :

- démarre un serveur gRPC
- écoute sur le port 50051
- expose la méthode `SayHello`

------

4. ### Implémentation du client gRPC (`client.py` ):

Le client :

- ouvre une connexion gRPC
- appelle la fonction distante `SayHello()`
- reçoit la réponse `HelloResponse`

------

5. ### Exécution (sans Docker pour rester simple)

Dans un terminal :

```bash
python server.py
```

Dans un autre :

```bash
python client.py
```

Sortie attendue :

```less
Réponse du serveur : Hello, Bigo
```

------



# Ce que tu viens de comprendre

### 1. Le serveur n’est pas une API REST

Il expose *des fonctions*, pas des URL.

### 2. Le client appelle une *fonction distante*

`stub.SayHello()`
 comme si c’était local.

### 3. Protobuf remplace JSON

C’est binaire, compact, rapide.

### 4. HTTP/2 remplace HTTP/1.1

→ moins de latence
→ multiplexage
→ streaming natif

------



# Est-ce toujours aussi simple?

Dans **95 % des cas**, gRPC reste aussi simple.
Et c’est exactement pour ça qu’il est massivement utilisé dans les microservices modernes.

Mais laisse-moi t’expliquer **pourquoi ça reste simple**, et **dans quels cas ça devient plus complexe**.

------

### 1. Dans le cas simple (comme dans l'exemple "Hello")

c’est **toujours aussi simple** :

1. Tu écris un `hello.proto`
2. Tu génères le code (automatique)
3. Tu écris ton serveur → 10 lignes
4. Tu écris ton client → 10 lignes
5. Tu lances → ça marche

En vrai :
 **gRPC t’épargne 90 % du travail que REST demande normalement.**

Tu n’as pas à gérer :

- routes HTTP
- format JSON
- parsing
- validation
- transport
- codes HTTP
- latences liées à HTTP/1.1
- compatibilité des messages

Tu appelles juste des **fonctions** à distance.
 C’est pour ça que gRPC ressemble à une API locale.

------

### 2. Mais alors… quand gRPC devient plus complexe ?

Quand tu passes du petit exemple à **un vrai système à microservices**.

Voici ce qui ajoute de la complexité (**mais qui n’a rien à voir avec gRPC lui-même**) :

#### 1. Gestion des versions (`v1`, `v2`, `v3`)

Parce qu’en production, tu ne peux pas casser les clients existants.

#### 2. Gestion du load balancing

gRPC utilise HTTP/2 → parfois il faut configurer un load balancer spécial (Envoy, gRPC proxy).

#### 3. Sécurité / Authentification

Tu dois gérer TLS, authentification mutuelle (mTLS), jetons, certificats, etc.

#### 4. Observabilité

Logs + metrics + traces distribuées, comme avec n’importe quel microservice.

#### 5. Streamings (bidirectionnels)

Ultra puissant… mais ça demande un peu plus de code.

#### 6. Auto-génération du code dans plusieurs langages

Si tu as services en :

- Python
- Go
- Java
   → tu dois générer les stubs dans chaque langage.

Mais encore une fois :
 **ce n’est pas gRPC qui est complexe, c’est l’architecture autour.**
 Sauf que c’est pareil pour REST, WebSockets ou MQTT.

------

## En résumé : 

**Pour des petits microservices ou des prototypes → toujours aussi simple.**

**Pour un système distribué complexe → ça demande une vraie discipline, mais pas plus que n’importe quelle API.**

gRPC reste dans tous les cas :

- plus rapide
- plus compact
- plus facile à maintenir
- plus strict
- plus efficace en microservices

------



# gRPC vs gRPC-Web

# gRPC

Le vrai gRPC, celui dont on parle depuis le début.

**Fonctionne en natif sur HTTP/2**, avec :

- Streaming bidirectionnel
- Multiplexage
- Compression binaire Protobuf
- Authentification avancée
- Très basse latence
- Appels RPC comme des fonctions locales

### Où il fonctionne ?

- Python
- Go
- Java
- C++
- Rust
- Node.js (serveur uniquement avec HTTP/2 natif)

**Mais pas dans un navigateur.**

### Pourquoi pas ?

Parce que :

- Le navigateur **n’autorise pas d’ouvrir des connexions HTTP/2 en mode gRPC pur**
- Le browser ne sait pas gérer les **trailers HTTP/2** utilisés par gRPC
- Le navigateur bloque certains headers obligatoires de gRPC

Résultat :
 **Un browser ne peut pas appeler un vrai service gRPC.**

------

# gRPC-Web

gRPC-Web est une version **adaptée pour les navigateurs**.

C’est gRPC… mais avec des modifications pour que Chrome/Firefox/Edge puissent l’utiliser.

### Qu’est-ce qui change ?

- Fonctionne sur **HTTP/1.1 ou HTTP/2**, mais encapsulé différemment
- Le streaming bidirectionnel **n’est pas supporté**
- Tu dois utiliser un **proxy** (Envoy, gRPC-Web proxy) entre le browser et ton backend
- Messages légèrement modifiés pour être compatibles navigateur

### Ce qui reste identique :

- Toujours Protobuf
- Toujours du RPC type-safe
- Toujours généré automatiquement

------

# Tableau synthèse

| Feature                         | gRPC                 | gRPC-Web                              |
| ------------------------------- | -------------------- | ------------------------------------- |
| Transport                       | HTTP/2 natif         | HTTP/1.1 ou HTTP/2 (adapté)           |
| Fonctionne dans un navigateur ? | Non                  | Oui                                   |
| Streaming bidirectionnel        | Oui                  | Non                                   |
| Streaming serveur → client      | Oui                  | Oui (limité)                          |
| Streaming client → serveur      | Oui                  | Non                                   |
| Nécessite un proxy ?            | Non                  | Oui (Envoy ou gRPC-Web proxy)         |
| Format                          | Binaire pur Protobuf | Protobuf encapsulé compatible browser |
| Performances                    | Maximales            | Très bonnes mais moins rapides        |
| Sécurité                        | mTLS complet         | TLS standard navigateur               |

------

# Diagramme d’architecture rapide

### gRPC

Client → (HTTP/2) → Serveur gRPC
 Aucun composant au milieu.

### gRPC-Web

Browser → (HTTP/1.1 ou HTTP/2) → Proxy Envoy → (HTTP/2 gRPC) → Serveur gRPC

------

# Quand utiliser quoi ?

## Utiliser **gRPC**

Quand la communication se fait entre services backend :

- microservices
- services interne à un cluster Kubernetes
- communication inter-datacenter
- Python ↔ Go ↔ Java etc.

## Utiliser **gRPC-Web**

Quand tu as :

- une application web React/Vue/Angular/etc.
- un frontend SPA qui doit appeler un backend gRPC
- du WebAssembly (Rust→wasm, Go→wasm)

Ton navigateur ne peut parler qu’en gRPC-Web.

------

# Résumé final

**gRPC = backend ↔ backend (HTTP/2 pur)**
 **gRPC-Web = browser ↔ backend (via proxy)**

Les deux utilisent **les mêmes .proto**,
 mais pas le même transport.

------

