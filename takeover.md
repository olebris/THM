# ☢️ Takeover

{% embed url="https://tryhackme.com/room/takeover" %}

Lien Gitbook:   [https://olb.gitbook.io/thm-ctf-write-ups/takeover](https://olb.gitbook.io/thm-ctf-write-ups/takeover)



## Setup

<figure><img src=".gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

## Nmap

<figure><img src=".gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

### Résultats:

* Port 22 SSH Open&#x20;
* Port 80 http Open&#x20;
* Port 443 SSL open&#x20;
* OS Ubuntu&#x20;
* Apache/2.4.41&#x20;

&#x20;

On associe l'IP à `futurevera.thm` dans `/etc/hosts` comme indiqué dans la consigne du CTF.

Dès lors, l'URL [https://futurevera.thm](https://futurevera.thm) fonctionne mais le site ne donne accès a aucune info notable ...

## Gobuster  (directory)

&#x20;ne donne rien&#x20;

Il faut chercher des sous domaines:

## Gobuster (vhost)

&#x20;vhost et pas dns car ici il n'a pas de serveur DNS qui renvoie des infos sur ce domaine "local" futurevera.thm

{% code overflow="wrap" %}
```
gobuster vhost -u https://futurevera.thm -t 4 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain –k
```
{% endcode %}

* Ne pas oublier --`append-domain` : concaténer le sous domaine de la seclist au domaine&#x20;
* Ajouter `–k` pour ignorer le certificat TLS non reconnu par gobuster

<figure><img src=".gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

2 résultats !

On se rend sur [https://support.futurevera.thm/ ](https://support.futurevera.thm/) (après ajout de `support.futurevera.thm` à `/etc/hosts`)

L'analyse du certificat mentionne un autre domaine (DNS Name)

<figure><img src=".gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

[http://secrethelpdesk934752.support.futurevera.thm ](http://secrethelpdesk934752.support.futurevera.thm) (à ajouter au fichier host également) redirige vers un site qui donne le flag

<figure><img src=".gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>
