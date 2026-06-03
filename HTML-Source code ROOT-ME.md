Write-up 1 : HTML - Code source

Objectif
Trouver le mot de passe de validation qui a été laissé par erreur dans le code source de la page par le développeur.

Résolution

1. Accès à la cible : En cliquant sur "Démarrer le challenge", une page de connexion basique (Login v0.00001) s'affiche demandant un mot de passe.

2. Analyse du code source : Les navigateurs web permettent de lire le code HTML reçu. En faisant un clic droit sur la page puis "Afficher le code source de la page" (ou Ctrl + U), le code brut apparaît.

3. Extraction du flag : Dans le corps du HTML (souvent dans une balise <script> ou un commentaire ``), le mot de passe est écrit en clair sous la forme d'une variable ou d'une condition :

Validation : Il suffisait de copier ce mot de passe et de le soumettre dans le champ de validation sur la plateforme Root-Me.