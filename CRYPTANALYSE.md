# Cryptanalyse

Retrouvez les secrets protégés par des systèmes de chiffrement plus ou moins solides en réalisant des attaques cryptographiques.

Les épreuves de cryptologie vous permettent de mettre à l’épreuve votre cryptanalyse. Vous êtes face à une donnée codée, à vous de retrouver le type de codage ou de chiffrement pour parvenir à revenir au clair.

## Prérequis

- Connaître les formats d’encodage les plus communs ;
- Connaître les algorithmes de hash / de chiffrement les plus communs ;
- Maîtriser un langage de script.

---

## 1. Encodage - ASCII

Dans le terminal, après avoir téléchargé le fichier :

```bash
echo "4C6520666C6167206465206365206368616C6C656E6765206573743A203261633337363438316165353436636436383964356239313237356433323465" | xxd -r -p
```

Le flag de ce challenge est :

```
2ac376481ae546cd689d5b91275d324e
```

---

## 2. Encodage - UU

Le titre du challenge permet de deviner que le fichier est codé en "UUencode". Copier/coller le contenu du challenge dans un fichier texte (`code.txt`, par exemple) et utiliser `uudecode` pour le déchiffrer.

Commande à utiliser :

```bash
uudecode -o code_plus_simple.txt code.txt
```

La commande va créer un fichier `code_plus_simple.txt`. Il suffit ensuite de le lire pour voir apparaître le résultat du challenge :

```
Very simple ;)
PASS = ULTRASIMPLE
```

### Complément d’information

Si vous n’avez pas `uuencode/uudecode` d’installés, il faut installer le paquet `sharutils` :

```bash
sudo apt-get install sharutils
```

**Flag :** `ULTRASIMPLE`

---

## 6. Hash - Message Digest 5 (MD5)

Pour ce challenge, on utilise **Hashcat**. L’algorithme de chiffrement utilisé est **MD5**, dont la valeur-code pour Hashcat est `0`.

On peut utiliser un dictionnaire basique (`rockyou.txt`) pour bruteforcer le hash :

```bash
hashcat -a0 -m0 md5.txt rockyou.txt
```

### Résultat obtenu :

```
7ecc19e1a0be36ba2c6f05d06b5d3058:weak
```

**Le Flag est :** `weak`

---

## 7. Hash - SHA-2

On a un hash qu’on ne peut pas cracker dans sa forme initiale. Cependant, nous devons trouver le mot de passe qui est hashé en SHA-2 et le re-hasher en SHA-1 pour valider le challenge.

En utilisant l’outil Kali Linux `hashid` :

```bash
hashid 96719db60d8e3f498c98d94155e1296aac105ck4923290c89eeeb3ba26d3eef92
```

On obtient :

```
[+] Unknown hash
```

### Analyse du hash

- Il contient majoritairement des caractères hexadécimaux.
- On remarque la présence du caractère **"k"**, qui ne fait pas partie de l’alphabet hexadécimal.

On enlève le **"k"** et on décode le nouveau hash avec [md5decrypt.net](https://md5decrypt.net/Sha256/).

On obtient alors : **4dM1n**.

### Conversion en SHA-1

On utilise l’outil Kali Linux `hURL` :

```bash
hURL -2 4dM1n
```

**Résultat :**

```
SHA1 checksum   :: a7c9d5a37201c08c5b7b156173bea5ec2063edf9
```

**Le Flag est :** `a7c9d5...edf9`

