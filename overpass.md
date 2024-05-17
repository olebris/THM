# ⚱️ Overpass

{% embed url="https://tryhackme.com/r/room/overpass" %}

Site Gitbook : [https://olb.gitbook.io/thm-ctf-write-ups/overpass](https://olb.gitbook.io/thm-ctf-write-ups/overpass)

## Reconnaissance

```
nmap -sC -sV -oN nmap-report.txt $IP 
```

Ce qu'on obtient comme info:

* OS LINUX&#x20;
* 22/tcp open  ssh&#x20;
* 80/tcp open

le site [http://10.10.232.151/](http://10.10.232.151/) est accessible

<figure><img src=".gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

&#x20;Après inspection manuelle on passe au directory brute force

```
gobuster dir -u $IP -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt 
```

on trouve rapidement&#x20;

`Found: /admin                (Status: 301) [Size: 42] [--> /admin/]`&#x20;

l'url [http://10.10.232.151/admin/](http://10.10.232.151/admin/) nous affiche un formulaire d'authentification

<figure><img src=".gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Le formulaire envoie le username et le password saisi a une API: `/api/login`\


<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

## Exploitation

Cette fois pas de user et pas de password, je tente un bruteforce Hydra de l'API `/api/login` sur les utilisateur ninja, pars, szymex etc. de cette page [http://10.10.232.151/aboutus/](http://10.10.232.151/aboutus/) mais sans succès. Un brute force user + password a partir des seclists habituelles n'aboutit pas non plus ...



En cherchant un peu, on trouve que la page login est générée par un script `login.js` qu'on peut analyser au debugger de Firefox

<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

La fonction `login()` est volontairement mal codée. Elle appelle l'API `/api/login` et si la fonction ne renvoie pas d'erreur alors elle crée un cookie de session appelé `SessionToken`

Bien qu'on ne sache pas si le contenu (`statusOrCookie`) de ce cookie sera vérifié, créons ce cookie `SessionToken`  avec des infos bidon pour simuler un retour positif de l'API de login. Pour cela dans la console du navigateur on a juste à entrer

```
Cookies.set("SessionToken", "blabla") 
```

<figure><img src=".gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>



On recharge [http://10.10.232.151/admin/](http://10.10.232.151/admin/) et yesss!  le cookie de session nous permet d'être connecté en mode "amin"

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

* on a un utilisateur nommé `james`
* on obtient sa clé RSA privée&#x20;

On copie la clé de james dans un fichier `jameskey` et on tente de se connecter en SSH au serveur avec cette clé (`ssh-i jameskey james@IP_cible)`

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

On s'en doutait la clé est protégée par une passphrase. On va la brute forcer avec johnTheRipper

```
ssh2john jameskey > jameskeypass.hash
```

```
john --wordlist=/usr/share/wordlist/rockyou.txt jameskeypass.hash
```

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

* la passphrase est `james13`

On se connecte a nouveau avec désormais la passphrase et cette fois c'est bon on se connecte et on peut récupérer le flag de james.

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

## Escalade de Privilège

* pas de `sudo -l`  possible
* pas de etc/password ou /etc/shadow vulnérables
* pas de permissions suid/guid exploitables
* on regarde la crontab:

```
cat /etc/crontab
```

<figure><img src=".gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

* Il ya un curl qui récupère un script shell et qui l'envoie à bash pour exécution (en root mode) toutes les minutes !
* le domaine du site est overpass.thm
* Il faudrait pouvoir pour exploiter ce cron remplacer le domaine overpass.thm par un domaine ou une ip qu'on maîtrise pour y placer un script custom ... Genre manipulation de DNS par exemple...

Regardons si par hasard on aurait accès au fichier hosts de la VM

```
cat /etc/hosts
```

<figure><img src=".gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

On y a surtout accès en écriture ce qui nous permet de remplacer `overpass.thm` par notre IP.

<figure><img src=".gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

Il ne reste plus qu'à créer l'arborescence du serveur initial sur notre Kali

<figure><img src=".gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

Et créer un script `buildscript.sh` custom comme par exemple un bon vieux reverseshell en bash:

<figure><img src=".gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

NB: ne pas oublier le shebang  au début du fichier : `#!/bin/bash`

Vous devez avoir cette arborescence

<figure><img src=".gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

A partir du même niveau d'arborescence que le répertoire downloads vous pouvez lancer un serveur python qui répondra à l’URL appelée par le cron de la VM cible

<figure><img src=".gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

On lance le netcat en écoute sur sa Kali et on attends que le cron passe ...

```
nc -lvnp 1234
```

<figure><img src=".gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

Si tout se passe bien on se retrouve avec un reverse shell en tant que root et on récupère le flag final

<figure><img src=".gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>
