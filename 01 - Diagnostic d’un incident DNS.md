
# Lab Support IT N1 — Diagnostic d’un incident DNS

## Objectif

Ce lab simule un incident courant en support IT N1 : un utilisateur est connecté au réseau, mais n’arrive plus à accéder aux sites web à cause d’un problème DNS.

L’objectif est de diagnostiquer la panne, confirmer que la connectivité réseau fonctionne, isoler le problème DNS, appliquer une correction temporaire et proposer une escalade propre si nécessaire.

---

## Contexte de l’incident

Un utilisateur contacte le support :

> “Je suis bien connecté au Wi-Fi, mais je n’arrive plus à ouvrir les sites web. Chrome affiche une erreur DNS.”

Symptômes observés :

- Le poste est connecté au réseau
- L’adresse IP est bien attribuée
- Internet fonctionne en IP directe
- Les noms de domaine ne se résolvent pas
- Le problème semble lié au DNS

---

## Environnement

- Poste utilisateur : Windows 10 / Windows 11
- Connexion : Wi-Fi
- Réseau local : 192.168.1.0/24
- Passerelle : 192.168.1.1
- DNS configuré : 192.168.1.254
- Outils utilisés : `ipconfig`, `ping`, `nslookup`, `PowerShell`

---

## Étape 1 — Vérification de la configuration IP

Commande utilisée :

~~~powershell
ipconfig /all
~~~

Résultat observé :

~~~text
Carte réseau sans fil Wi-Fi :

   Adresse IPv4. . . . . . . . . . . . .: 192.168.1.45
   Masque de sous-réseau . . . . . . . .: 255.255.255.0
   Passerelle par défaut . . . . . . . .: 192.168.1.1
   Serveurs DNS . . . . . . . . . . . . : 192.168.1.254
~~~

Analyse :

Le poste possède une adresse IP valide, un masque réseau, une passerelle par défaut et un serveur DNS configuré.

La machine est bien connectée au réseau local.  
Le problème ne vient donc pas d’une absence d’adresse IP ou d’une déconnexion réseau.

---

## Étape 2 — Test de connectivité vers Internet

Commande utilisée :

~~~powershell
ping 8.8.8.8
~~~

Résultat observé :

~~~text
Réponse de 8.8.8.8 : octets=32 temps=22 ms TTL=117
Réponse de 8.8.8.8 : octets=32 temps=21 ms TTL=117
Réponse de 8.8.8.8 : octets=32 temps=23 ms TTL=117
Réponse de 8.8.8.8 : octets=32 temps=22 ms TTL=117
~~~

Analyse :

Le ping vers une adresse IP publique fonctionne.

Cela confirme que le poste peut sortir vers Internet.  
La connectivité réseau est donc fonctionnelle.

---

## Étape 3 — Test de résolution DNS

Commande utilisée :

~~~powershell
ping google.com
~~~

Résultat observé :

~~~text
La requête Ping n’a pas pu trouver l’hôte google.com.
Vérifiez le nom et essayez à nouveau.
~~~

Analyse :

Le ping vers une adresse IP fonctionne, mais le ping vers un nom de domaine échoue.

Cela indique que le problème ne vient pas de la connexion Internet, mais probablement de la résolution DNS.  
Le poste n’arrive pas à convertir `google.com` en adresse IP.

---

## Étape 4 — Vérification du DNS configuré

Commande utilisée :

~~~powershell
nslookup google.com
~~~

Résultat observé :

~~~text
Serveur :  UnKnown
Address:  192.168.1.254

DNS request timed out.
timeout was 2 seconds.
DNS request timed out.
timeout was 2 seconds.
*** Le délai de la requête vers UnKnown est dépassé.
~~~

Analyse :

Le serveur DNS configuré sur le poste est `192.168.1.254`.

La commande `nslookup` montre que ce serveur ne répond pas aux requêtes DNS.  
Le poste ne peut donc pas résoudre les noms de domaine.

---

## Étape 5 — Test avec un DNS externe

Commande utilisée :

~~~powershell
nslookup google.com 8.8.8.8
~~~

Résultat observé :

~~~text
Serveur :  dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    google.com
Address: 142.250.74.206
~~~

Analyse :

La résolution DNS fonctionne avec le serveur DNS externe `8.8.8.8`.

Cela confirme que le problème vient du DNS actuellement configuré sur le poste ou du serveur DNS interne fourni par le réseau.

---

## Étape 6 — Correction temporaire

Commande utilisée :

~~~powershell
Set-DnsClientServerAddress -InterfaceAlias "Wi-Fi" -ServerAddresses 8.8.8.8,1.1.1.1
~~~

Vérification :

~~~powershell
ping google.com
~~~

Résultat observé :

~~~text
Envoi d’une requête 'ping' sur google.com [142.250.74.206] avec 32 octets de données :
Réponse de 142.250.74.206 : octets=32 temps=18 ms TTL=117
Réponse de 142.250.74.206 : octets=32 temps=19 ms TTL=117
Réponse de 142.250.74.206 : octets=32 temps=18 ms TTL=117
Réponse de 142.250.74.206 : octets=32 temps=20 ms TTL=117
~~~

Analyse :

Après modification temporaire du serveur DNS, le poste arrive de nouveau à résoudre `google.com`.

La navigation web peut reprendre.  
L’incident est temporairement résolu côté utilisateur.

---

## Étape 7 — Actions complémentaires

Commandes utiles :

~~~powershell
ipconfig /flushdns
ipconfig /release
ipconfig /renew
Restart-NetAdapter -Name "Wi-Fi"
~~~

Rôle des commandes :

~~~text
ipconfig /flushdns  : vide le cache DNS local du poste.
ipconfig /release   : libère l’adresse IP actuelle.
ipconfig /renew     : demande une nouvelle configuration IP au serveur DHCP.
Restart-NetAdapter  : redémarre la carte réseau.
~~~

---

## Étape 8 — Escalade possible

En environnement entreprise, l’utilisation d’un DNS public comme `8.8.8.8` ne doit pas forcément être conservée comme solution définitive.

Actions à réaliser avant escalade :

~~~text
- Vérifier si d’autres utilisateurs sont impactés
- Vérifier si le DNS interne répond depuis un autre poste
- Vérifier si le problème concerne uniquement le Wi-Fi ou tout le réseau
- Vider le cache DNS local
- Renouveler la configuration DHCP
- Documenter les tests réalisés
- Escalader vers le support N2 ou l’équipe réseau si le DNS interne reste indisponible
~~~

---

## Conclusion

L’incident était causé par une panne de résolution DNS.

Le poste disposait bien d’une adresse IP valide et d’un accès Internet en IP directe.  
Cependant, les noms de domaine ne pouvaient pas être résolus car le serveur DNS configuré ne répondait pas.

Le diagnostic a permis d’isoler le problème grâce aux commandes `ipconfig`, `ping` et `nslookup`.

Une correction temporaire a été appliquée en configurant un DNS externe, puis une escalade réseau peut être réalisée pour corriger le DNS interne.

---

## Compétences mises en pratique

- Diagnostic d’un incident réseau utilisateur
- Vérification de la configuration IP
- Test de connectivité avec `ping`
- Analyse de résolution DNS avec `nslookup`
- Modification temporaire d’un serveur DNS
- Vidage du cache DNS
- Renouvellement DHCP
- Documentation et escalade d’un incident support N1

---

