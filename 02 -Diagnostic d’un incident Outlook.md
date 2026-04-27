# Lab Support IT N1 — Diagnostic d’un incident Outlook / Microsoft 365

## Objectif

Ce lab simule un incident courant en support IT N1 : un utilisateur n’arrive plus à recevoir ses mails dans Outlook.

L’objectif est de diagnostiquer si le problème vient de la connexion réseau, du client Outlook, du compte Microsoft 365, de la licence, du mot de passe, du MFA ou du service Exchange Online.

---

## Contexte de l’incident

Un utilisateur contacte le support :

> “Je ne reçois plus mes mails sur Outlook depuis ce matin. J’ai redémarré mon PC, mais ça ne fonctionne toujours pas.”

Symptômes observés :

- Outlook affiche “Déconnecté” ou “Tentative de connexion”
- Aucun nouveau mail n’arrive dans l’application Outlook
- L’utilisateur peut parfois accéder à Internet
- Le problème peut venir du poste, du profil Outlook, du compte Microsoft 365 ou du service Exchange

---

## Environnement

- Poste utilisateur : Windows 10 / Windows 11
- Application : Microsoft Outlook
- Messagerie : Microsoft 365 / Exchange Online
- Navigateur : Microsoft Edge / Chrome
- Outils utilisés : Outlook, Outlook Web, Microsoft 365 Admin Center, Entra ID, commandes réseau Windows

---

## Étape 1 — Vérifier la connexion Internet

Commande utilisée :

~~~powershell
ping 8.8.8.8
~~~

Résultat observé :

~~~text
Réponse de 8.8.8.8 : octets=32 temps=21 ms TTL=117
Réponse de 8.8.8.8 : octets=32 temps=22 ms TTL=117
Réponse de 8.8.8.8 : octets=32 temps=20 ms TTL=117
Réponse de 8.8.8.8 : octets=32 temps=21 ms TTL=117
~~~

Analyse :

Le poste arrive à joindre une adresse IP publique.

La connexion réseau fonctionne en IP directe.  
Le problème ne semble pas venir d’une coupure réseau complète.

---

## Étape 2 — Vérifier la résolution DNS

Commande utilisée :

~~~powershell
nslookup outlook.office.com
~~~

Résultat observé :

~~~text
Serveur :  box.local
Address:  192.168.1.1

Réponse ne faisant pas autorité :
Nom :    outlook.office365.com
Addresses:  52.98.150.34
            52.98.150.50
Aliases: outlook.office.com
~~~

Analyse :

La résolution DNS fonctionne.

Le poste arrive à résoudre le nom `outlook.office.com`.  
Le problème ne vient donc pas d’un DNS totalement indisponible.

---

## Étape 3 — Vérifier l’accès à Outlook Web

Action réalisée :

~~~text
Ouvrir le navigateur et accéder à Outlook Web :
https://outlook.office.com
~~~

Résultat observé :

~~~text
L’utilisateur arrive à se connecter à Outlook Web.
Les nouveaux mails sont visibles dans le navigateur.
~~~

Analyse :

La boîte mail fonctionne côté Microsoft 365.

Si les mails sont visibles sur Outlook Web mais pas dans l’application Outlook, le problème est probablement local au poste ou au client Outlook.

Hypothèses possibles :

~~~text
- Outlook est en mode hors connexion
- Le profil Outlook est corrompu
- Le cache Outlook pose problème
- L’application Outlook rencontre un problème d’authentification
- Le poste garde une ancienne session ou un ancien mot de passe
~~~

---

## Étape 4 — Vérifier l’état de connexion Outlook

Action réalisée :

~~~text
Dans Outlook :
Envoyer/Recevoir > Préférences > Vérifier que "Travailler hors connexion" n’est pas activé
~~~

Résultat observé :

~~~text
Le mode "Travailler hors connexion" est activé.
Outlook affiche "Déconnecté" en bas de la fenêtre.
~~~

Analyse :

Outlook est configuré en mode hors connexion.

Cela explique pourquoi les nouveaux mails n’arrivent plus dans l’application, alors qu’ils sont bien visibles dans Outlook Web.

---

## Étape 5 — Correction du mode hors connexion

Action réalisée :

~~~text
Cliquer sur :
Envoyer/Recevoir > Travailler hors connexion

Objectif :
Désactiver le mode hors connexion.
~~~

Résultat observé :

~~~text
Outlook passe de "Déconnecté" à "Connecté à Microsoft Exchange".
Les nouveaux mails commencent à se synchroniser.
~~~

Analyse :

Le problème était lié au mode hors connexion activé dans Outlook.

L’incident est résolu côté utilisateur.

---

## Étape 6 — Vérification de la synchronisation

Action réalisée :

~~~text
Envoyer un mail de test à l’utilisateur.
Demander à l’utilisateur de confirmer la réception dans Outlook.
~~~

Résultat observé :

~~~text
Le mail de test est reçu dans Outlook.
Les anciens mails se synchronisent progressivement.
L’utilisateur peut envoyer et recevoir des mails.
~~~

Analyse :

La messagerie fonctionne de nouveau dans l’application Outlook.

Le test confirme que l’incident est corrigé.

---

## Étape 7 — Cas alternatif : Outlook Web fonctionne mais Outlook reste bloqué

Si Outlook Web fonctionne mais que l’application Outlook reste bloquée malgré la désactivation du mode hors connexion, les actions suivantes peuvent être réalisées.

### Redémarrer Outlook

Action :

~~~text
Fermer Outlook complètement.
Vérifier dans le Gestionnaire des tâches qu’Outlook n’est plus actif.
Relancer Outlook.
~~~

Analyse :

Un redémarrage complet peut résoudre un blocage temporaire de synchronisation ou d’authentification.

---

### Vérifier les identifiants enregistrés

Action :

~~~text
Ouvrir le Gestionnaire d’identifiants Windows.
Supprimer les anciens identifiants liés à Office, Outlook ou Microsoft 365.
Relancer Outlook.
Se reconnecter avec le compte utilisateur.
~~~

Analyse :

Des identifiants expirés ou incorrects peuvent empêcher Outlook de se reconnecter correctement à Microsoft 365.

---

### Créer un nouveau profil Outlook

Action :

~~~text
Ouvrir le Panneau de configuration.
Aller dans Courrier / Mail.
Cliquer sur Afficher les profils.
Créer un nouveau profil Outlook.
Ajouter le compte Microsoft 365.
Définir le nouveau profil comme profil par défaut.
Relancer Outlook.
~~~

Analyse :

Si le profil Outlook est corrompu, la création d’un nouveau profil permet souvent de rétablir la synchronisation.

---

## Étape 8 — Vérification côté Microsoft 365

Si le problème n’est pas local au poste, il faut vérifier le compte Microsoft 365.

Points à contrôler :

~~~text
- Le compte utilisateur est actif
- Le mot de passe n’est pas expiré
- Le compte n’est pas bloqué
- Une licence Microsoft 365 est bien attribuée
- La boîte Exchange Online est bien active
- Le MFA ne bloque pas la connexion
- Aucun incident Microsoft 365 global n’est en cours
~~~

Analyse :

Si Outlook Web ne fonctionne pas non plus, le problème peut venir du compte, de la licence, du MFA ou du service Microsoft 365.

---

## Étape 9 — Escalade possible

Escalader vers le support N2 ou l’administrateur Microsoft 365 si :

~~~text
- Outlook Web ne fonctionne pas
- Le compte utilisateur est bloqué
- La licence Microsoft 365 est absente
- Le MFA empêche la connexion
- La boîte Exchange Online semble désactivée
- Plusieurs utilisateurs sont impactés
- Un incident Microsoft 365 global est suspecté
- Le profil Outlook recréé ne résout pas le problème
~~~

Informations à transmettre lors de l’escalade :

~~~text
Utilisateur impacté : prénom.nom@entreprise.com
Poste concerné : Windows 10 / Windows 11
Application impactée : Outlook
Outlook Web testé : oui / non
Connexion Internet : fonctionnelle
DNS : fonctionnel
Message d’erreur observé : Outlook déconnecté / tentative de connexion
Actions réalisées : test réseau, test Outlook Web, vérification mode hors connexion, redémarrage Outlook
Résultat : incident persistant / partiellement résolu / résolu
~~~

---

## Conclusion

L’incident était causé par l’activation du mode hors connexion dans Outlook.

La connexion réseau fonctionnait, la résolution DNS était opérationnelle et la boîte mail était accessible depuis Outlook Web.

Le diagnostic a permis de confirmer que le problème venait du client Outlook local et non du service Microsoft 365.

La désactivation du mode hors connexion a permis de rétablir la synchronisation des mails.

---

## Compétences mises en pratique

- Diagnostic d’un incident Outlook
- Vérification de la connexion réseau
- Vérification DNS avec `nslookup`
- Test Outlook Web
- Distinction entre problème local et problème Microsoft 365
- Vérification du mode hors connexion Outlook
- Redémarrage d’application
- Notions de profil Outlook
- Notions de licence Microsoft 365
- Documentation et escalade d’un incident support N1

---
