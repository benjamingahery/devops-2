# DevOps : exercice partie 2

La deuxième partie de notre exercice sur les pratiques DevOps !

Au programme de cette deuxième partie : l'installation d'un load-balancer pour avoir un point d'entrée unique entre nos différents noeuds du cluster et la mise en place d'un outil d'analyse de la qualité du code : SonarQube.

## Load-balancer : HAProxy

Nous allons installer un load-balancer (répartiteur de charge) sur notre infrastructure Play-With-Docker, afin d'avoir un point d'entrée unique pour nos visiteurs.

:question: Mais pourquoi décentraliser notre application sur plusieurs noeuds d'un cluster si c'est pour re-centraliser les connexions des visiteurs sur un seul point d'entrée ?

L'objectif est de pouvoir mettre en place un certificat HTTPS sur ce load-balancer, et ainsi chiffrer les échanges entre nos visiteurs et l'application.

La suite par [ici](README-haproxy.md).

## Qualité du code : SonarQube

Pour finir, nous allons mettre en place un outil d'analyse de la qualité de notre code source : SonarQube. Ça se passe par [ici](README-sonarqube.md).
