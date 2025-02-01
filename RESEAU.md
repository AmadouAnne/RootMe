# Rapport d'Analyse de Réseau

## Introduction
Ce document présente une analyse approfondie des protocoles et services réseaux les plus courants, avec pour objectif de les comprendre et de les exploiter efficacement. Les prérequis pour cette analyse incluent :
- La maîtrise d'un analyseur réseau tel que Wireshark.
- La connaissance des protocoles applicatifs, réseaux et de transport les plus utilisés.

Ce rapport s'articule autour de 33 challenges répartis selon différents protocoles et techniques d'analyse.

---

## 1. Analyse de l'Authentification FTP
Après avoir téléchargé le fichier concerné, nous l'ouvrons avec Wireshark en utilisant la commande suivante :
```bash
sudo wireshark
```
Ensuite, en naviguant dans **Analyse** > **Suivre** > **Flux TCP**, nous obtenons le mot de passe en clair : **cdts3500**.

---

## 2. Analyse de l'Authentification TELNET
En utilisant `tshark`, nous pouvons extraire les données pertinentes du fichier de capture `ch2.pcap` :
```bash
tshark -r ch2.pcap -z "follow,tcp,ascii,0" | grep -i -A10 password
```
Le flag trouvé est : `user`.

---

## 3. Analyse d'une Trame Ethernet
Cette trame est codée en hexadécimal. En supprimant les espaces et en convertissant en ASCII, nous obtenons la requête HTTP suivante :
```
GET / HTTP/1.1
Authorization: Basic Y29uZmk6ZGVudGlhbA==
User-Agent: InsaneBrowser
Host: www.myipv6.org
Accept: */*
```
Après décodage en Base64, nous obtenons les identifiants suivants :
- **Login** : confi
- **Mot de passe** : dential

---

## 4. Analyse de l'Authentification Twitter
En ouvrant `ch3.pcap` dans Wireshark, nous identifions une requête GET :
```
GET /statuses/replies.wml HTTP/1.1
```
Dans les en-têtes HTTP, nous trouvons :
```
Authorization: Basic dXNlcnRlc3Q6cGFzc3dvcmQ=
```
Décodé en Base64, cela nous donne :
- **Login** : usertest
- **Mot de passe** : password

---

## 5. Analyse Bluetooth - Fichier Inconnu
Nous avons analysé le fichier `ch18.bin`, qui est un log de captures Bluetooth au format **BTSnoop**. Nous avons utilisé un script Python exploitant la bibliothèque `btsnoop` pour extraire les informations utiles :
```python
import btsnoop.btsnoop as bts
import binascii

filename = 'ch18.bin'
records = bts.parse(filename)
for record in records:
    rec_data = record[4]
    mac = binascii.hexlify(rec_data).decode().upper()
    print(f"Adresse MAC détectée : {mac}")
```
Le résultat nous donne :
- **Device** : GT-S7390G
- **Adresse MAC** : 0C:B3:19:B9:4F:C6
- **Flag** : c1d0349c153ed96fe2fadf44e880aef9e69c122b

---

## 6. Analyse des Mots de Passe Cisco
Nous avons identifié plusieurs mots de passe Cisco de type 7, facilement décryptables :
```
025017705B3907344E => 6sK0_hub
10181A325528130F010D24 => 6sK0_admin
124F163C42340B112F3830 => 6sK0_guest
144101205C3B29242A3B3C3927 => 6sK0_console
```
Nous avons ensuite analysé le mot de passe **enable** :
```
enable secret 5 $1$p8Y6$MCdRLBzuGlfOs9S.hXOp0.
```
Le préfixe `$1$` indique un hachage MD5 avec un salage `$p8Y6$`. En utilisant des outils de cassage de hash, nous pouvons tenter de retrouver le mot de passe.

---

## Conclusion
Ce rapport met en évidence l'importance de l'analyse des protocoles réseau pour identifier des vulnérabilités potentielles. L'utilisation d'outils comme Wireshark et Tshark permet de détecter des fuites d'informations sensibles, notamment les mots de passe transmis en clair. Pour renforcer la sécurité, il est essentiel d'adopter de bonnes pratiques, telles que l'utilisation de protocoles chiffrés (SSH, HTTPS) et la mise en place d'une politique de mots de passe robuste.

