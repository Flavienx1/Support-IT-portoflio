# Lab Support IT N1 — Compte utilisateur bloqué / mot de passe expiré

## Objectif

Ce lab simule un incident courant en support IT N1 : un utilisateur n’arrive plus à se connecter à son poste Windows ou à Microsoft 365 car son compte est bloqué ou son mot de passe a expiré.

L’objectif est de diagnostiquer la cause du blocage, vérifier l’état du compte, appliquer une correction adaptée et documenter l’incident.

---

## Contexte de l’incident

Un utilisateur contacte le support :

> “Je n’arrive plus à me connecter à mon PC ni à Outlook. Le système me dit que mon mot de passe est incorrect.”

Symptômes observés :

- Connexion Windows impossible
- Connexion Microsoft 365 impossible
- Message d’erreur : mot de passe incorrect ou compte verrouillé
- Plusieurs tentatives de connexion échouées
- L’utilisateur pense avoir saisi le bon mot de passe

---

## Environnement

- Poste utilisateur : Windows 10 / Windows 11
- Domaine : entreprise.local
- Compte utilisateur : prenom.nom@entreprise.com
- Outils utilisés : Active Directory Users and Computers, Entra ID, Microsoft 365 Admin Center, PowerShell, portail Microsoft 365

---

## Étape 1 — Recueillir les informations utilisateur

Questions à poser :

~~~text
- Depuis quand le problème est-il présent ?
- Le problème concerne-t-il le PC, Outlook, Teams ou tous les services ?
- Le mot de passe a-t-il été modifié récemment ?
- Plusieurs tentatives de connexion ont-elles été faites ?
- L’utilisateur a-t-il reçu une notification MFA ?
- Le problème se produit-il sur un seul poste ou aussi sur le webmail ?
~~~

Analyse :

Ces questions permettent de savoir si le problème concerne uniquement le poste Windows, le compte Microsoft 365, le mot de passe, le MFA ou un verrouillage de compte.

---

## Étape 2 — Tester la connexion à Outlook Web

Action réalisée :

~~~text
Demander à l’utilisateur de tester la connexion à Outlook Web :
https://outlook.office.com
~~~

Résultat observé :

~~~text
La connexion échoue.
Message affiché :
Votre compte ou votre mot de passe est incorrect.
~~~

Analyse :

Le problème ne concerne pas uniquement le poste Windows.

Si l’utilisateur ne peut pas non plus se connecter à Outlook Web, le problème vient probablement du compte, du mot de passe, du MFA ou d’un verrouillage côté annuaire.

---

## Étape 3 — Vérifier l’état du compte dans Active Directory

Action réalisée :

~~~text
Ouvrir Active Directory Users and Computers.
Rechercher le compte utilisateur : prenom.nom
Ouvrir les propriétés du compte.
Vérifier l’onglet Account.
~~~

État observé :

~~~text
Account is locked out : activé
User must change password at next logon : non activé
Password never expires : non activé
Account is disabled : non activé
~~~

Analyse :

Le compte utilisateur est verrouillé.

Cela peut arriver après plusieurs tentatives de connexion avec un mauvais mot de passe, ou lorsqu’un ancien mot de passe est encore enregistré sur un appareil, une application ou une session.

---

## Étape 4 — Vérifier les événements de connexion échouée

Commande PowerShell possible sur un poste ou serveur autorisé :

~~~powershell
Get-EventLog -LogName Security -InstanceId 4625 -Newest 10
~~~

Exemple de résultat observé :

~~~text
EventID       : 4625
TimeGenerated : 2026-04-27 09:14:21
EntryType     : FailureAudit
Message       : An account failed to log on.

Account Name  : prenom.nom
Failure Reason: Unknown user name or bad password
Workstation   : PC-COMPTA-07
Source Network Address : 192.168.1.42
~~~

Analyse :

L’événement 4625 indique une tentative de connexion échouée.

La présence de plusieurs échecs de connexion peut expliquer le verrouillage du compte.  
Le poste `PC-COMPTA-07` peut être à l’origine des tentatives répétées.

---

## Étape 5 — Débloquer le compte utilisateur

Action réalisée dans Active Directory :

~~~text
Ouvrir le compte utilisateur.
Aller dans l’onglet Account.
Décocher "Unlock account".
Appliquer la modification.
~~~

Alternative PowerShell si autorisé :

~~~powershell
Unlock-ADAccount -Identity prenom.nom
~~~

Analyse :

Le compte est déverrouillé.

L’utilisateur peut à nouveau tenter de se connecter, à condition d’utiliser le bon mot de passe.

---

## Étape 6 — Réinitialiser le mot de passe si nécessaire

Si l’utilisateur ne connaît plus son mot de passe ou si le mot de passe a expiré, une réinitialisation peut être effectuée.

Action réalisée :

~~~text
Réinitialiser le mot de passe utilisateur dans Active Directory ou Microsoft 365 Admin Center.
Cocher "User must change password at next logon" si la politique interne le demande.
Communiquer le mot de passe temporaire via un canal sécurisé.
~~~

Alternative PowerShell si autorisé :

~~~powershell
Set-ADAccountPassword -Identity prenom.nom -Reset -NewPassword (ConvertTo-SecureString "MotDePasseTemporaire!" -AsPlainText -Force)
Set-ADUser -Identity prenom.nom -ChangePasswordAtLogon $true
~~~

Analyse :

La réinitialisation permet à l’utilisateur de récupérer l’accès.

Le changement du mot de passe à la prochaine connexion force l’utilisateur à définir un mot de passe personnel.

---

## Étape 7 — Vérifier le compte dans Entra ID / Microsoft 365

Action réalisée :

~~~text
Ouvrir Entra ID ou Microsoft 365 Admin Center.
Rechercher l’utilisateur : prenom.nom@entreprise.com.
Vérifier l’état du compte.
~~~

Points à contrôler :

~~~text
- Compte actif
- Licence Microsoft 365 attribuée
- MFA activé ou non
- Méthodes MFA configurées
- Aucun blocage de connexion
- Aucun risque utilisateur critique signalé
~~~

Analyse :

Si le compte est synchronisé entre Active Directory et Microsoft 365, il faut vérifier que le problème n’existe pas aussi côté cloud.

Un problème MFA, licence ou blocage Entra ID peut empêcher l’accès à Outlook, Teams ou OneDrive même si le compte Windows est débloqué.

---

## Étape 8 — Vérifier les causes possibles du verrouillage

Causes fréquentes :

~~~text
- Mot de passe saisi plusieurs fois incorrectement
- Ancien mot de passe enregistré dans Outlook
- Ancien mot de passe enregistré dans le Gestionnaire d’identifiants Windows
- Téléphone connecté à la messagerie avec un ancien mot de passe
- Session ouverte sur un autre poste
- Service ou lecteur réseau utilisant d’anciens identifiants
- Tentative de connexion suspecte
~~~

Actions possibles :

~~~text
- Demander à l’utilisateur de mettre à jour son mot de passe sur ses appareils
- Supprimer les anciens identifiants Windows enregistrés
- Redémarrer Outlook et Teams
- Déconnecter les anciennes sessions Microsoft 365 si nécessaire
- Vérifier si les échecs de connexion continuent après le déblocage
~~~

---

## Étape 9 — Tester la reconnexion utilisateur

Action réalisée :

~~~text
Demander à l’utilisateur de se reconnecter à son poste Windows.
Demander ensuite un test sur Outlook Web, Outlook et Teams.
~~~

Résultat observé :

~~~text
Connexion Windows réussie.
Connexion Outlook Web réussie.
Outlook se synchronise.
Teams se connecte correctement.
~~~

Analyse :

Le compte est de nouveau opérationnel.

Le test confirme que l’incident est résolu.

---

## Étape 10 — Escalade possible

Escalader vers le support N2, l’administrateur système ou l’équipe sécurité si :

~~~text
- Le compte se rebloque immédiatement après déverrouillage
- Les événements 4625 continuent en masse
- Les tentatives viennent d’une adresse IP inconnue
- L’utilisateur reçoit des demandes MFA qu’il n’a pas initiées
- Le compte est marqué à risque dans Entra ID
- Plusieurs utilisateurs sont impactés
- Le mot de passe semble compromis
~~~

Informations à transmettre lors de l’escalade :

~~~text
Utilisateur impacté : prenom.nom@entreprise.com
Poste concerné : PC-COMPTA-07
Heure du premier signalement : 09:10
Symptôme : compte verrouillé / mot de passe refusé
Tests réalisés : Outlook Web, connexion Windows, vérification AD, vérification Entra ID
Événements observés : Event ID 4625
Source des échecs : PC-COMPTA-07 / 192.168.1.42
Actions réalisées : déverrouillage du compte, réinitialisation du mot de passe, test de reconnexion
Résultat : résolu / à surveiller / escalade nécessaire
~~~

---

## Conclusion

L’incident était causé par un verrouillage du compte utilisateur après plusieurs tentatives de connexion échouées.

Le diagnostic a permis de confirmer que le problème ne venait pas uniquement du poste Windows, car la connexion à Outlook Web échouait également.

La vérification dans Active Directory a montré que le compte était verrouillé.  
Le compte a été débloqué, le mot de passe a été réinitialisé si nécessaire, puis l’utilisateur a pu se reconnecter à Windows, Outlook et Teams.

Une surveillance reste nécessaire si le compte se rebloque rapidement, car cela peut indiquer un ancien mot de passe enregistré, un appareil mal configuré ou une tentative de connexion suspecte.

---

## Compétences mises en pratique

- Diagnostic d’un incident de connexion utilisateur
- Vérification d’un compte dans Active Directory
- Déverrouillage d’un compte utilisateur
- Réinitialisation de mot de passe
- Vérification Microsoft 365 / Entra ID
- Identification d’événements de connexion échouée
- Analyse basique de l’Event ID 4625
- Vérification MFA
- Documentation et escalade d’un incident support N1

---

