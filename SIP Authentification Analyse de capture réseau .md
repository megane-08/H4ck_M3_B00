
## Énoncé

**Objectif :** retrouver le mot de passe utilisé lors de l’authentification sur une infrastructure SIP.

Le fichier fourni contient des informations extraites d’une capture réseau :

```text
172.25.105.3"172.25.105.40"555"asterisk"REGISTER"sip:172.25.105.40"4787f7ce""""PLAIN"1234

172.25.105.3"172.25.105.40"555"asterisk"INVITE"sip:1000@172.25.105.40"70fbfdae""""MD5"aa533f6efa2b2abac675c1ee6cbde327

172.25.105.3"172.25.105.40"555"asterisk"BYE"sip:1000@172.25.105.40"70fbfdae""""MD5"0b306e9db1f819dd824acf3227b60e07
```

Extrait du fichier : ch4

# 1. Comprendre le protocole SIP

**SIP (Session Initiation Protocol)** est un protocole utilisé pour établir des communications VoIP :

* appels téléphoniques IP
* visioconférences
* communications audio

Quelques méthodes SIP fréquentes :

* **REGISTER** = enregistrement du client auprès du serveur SIP
* **INVITE** = démarrage d’un appel
* **BYE** = fin de l’appel

Dans notre capture on observe :

```text
REGISTER
INVITE
BYE
```

Le plus intéressant ici est **REGISTER**, car c’est généralement à ce moment que l’utilisateur s’authentifie.

# 2. Analyse de la ligne REGISTER

La ligne importante est :

```text
REGISTER ... PLAIN 1234
```

Décomposition :

```text
172.25.105.3
```

adresse IP du client

```text
172.25.105.40
```

serveur SIP

```text
555
```

identifiant utilisateur / extension

```text
asterisk
```

serveur téléphonique utilisé (Asterisk)

```text
REGISTER
```

demande d’authentification

Puis :

```text
PLAIN
```

Le mot **PLAIN** signifie que le mécanisme d’authentification est en **texte clair**.

Et juste après :

```text
1234
```

Le mot de passe apparaît directement.

Donc :

```text
Mot de passe = 1234
```

Preuve dans le fichier : ch4


# 3. Pourquoi les lignes suivantes ne servent pas ?

On trouve ensuite :

```text
INVITE ... MD5 aa533f6efa2b2abac675c1ee6cbde327
```

et

```text
BYE ... MD5 0b306e9db1f819dd824acf3227b60e07
```

Ici le mécanisme est :

```text
MD5
```

Cela signifie que les informations ne sont plus visibles directement mais transformées par une fonction de hachage.

Exemple :

```text
motdepasse
      ↓
MD5
      ↓
34819d7beeabb9260a5c854bc85b3e44
```

Dans ce cas il faudrait :

* casser le hash
* utiliser dictionnaires
* faire du bruteforce
* utiliser `hashcat` ou `john`

Mais ce n’est pas nécessaire ici car le mot de passe est déjà exposé dans **REGISTER**.

# 4. Faiblesse de sécurité observée

Le problème principal :

```text
PLAIN
```

Le mot de passe est transmis en clair :

```text
1234
```

Un attaquant qui capture le trafic réseau peut immédiatement récupérer les identifiants.

Une configuration plus sûre utiliserait :

* Digest Authentication
* SHA
* TLS
* SIP sécurisé

# Résultat final

Mot de passe retrouvé :

```text
1234
```

# Conclusion

Ce challenge montre qu’en analyse réseau il faut toujours regarder :

1. les phases d’authentification (`REGISTER`)
2. le type d’authentification (`PLAIN`, `MD5`, etc.)
3. les champs visibles avant de tenter du cassage de hash

Ici l’authentification était faite en **texte clair**, ce qui permet de récupérer immédiatement le mot de passe :

```text
FLAG : 1234
```

