# 🏫 Vulniversity

{% embed url="https://tryhackme.com/room/vulnversity?ref=blog.tryhackme.com" %}

Site Gitbook: [https://olb.gitbook.io/thm-ctf-write-ups/vulniversity](https://olb.gitbook.io/thm-ctf-write-ups/vulniversity)

## Reconnaissance

```
export IP=10.10.107.33
```

```
nmap -sC -sV -oN nmap-report.txt $IP
```

<figure><img src=".gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Ports / services ouverts:

* ftp (22)
* ssh (23)
* samba (139/445)
* proxy Squid (3128)
* Serveur web Apache (3333)

Petit tour sur le site [http://10.10.107.33:3333/](http://10.10.107.33:3333/)

* code source : c'est un site sous Wordpress (pas d'info viible de version ...)
* pas d'accès à /wp-admin ou admin , pas accès au robots.txt ou sitemap.xml ..
* On lance un Brute Force de directory pour voir...

{% code overflow="wrap" %}
```
gobuster dir -u http://$IP:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x php,html,txt,htm,log
```
{% endcode %}

