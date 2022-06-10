# SonarQube

## Qualité du code ?

Un code "de qualité", est un code qui va respecter des bonnes pratiques de développement, liées au langage de programmation utilisé. Il faut par exemple éviter de dupliquer du code, et factoriser autant que possible. Le code doit aussi respecter certains standards, comme les PSR en PHP par exemple.

La qualité du code peut être analysée automatiquement par des outils comme SonarQube, que nous allons installer.

## Installation

Démarrez votre [VM Serveur Kourou](https://kourou.oclock.io/ressources/vm-cloud/), et installez Docker avec les commandes suivantes :

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER
```

Une fois ces commandes lancées, déconnectez-vous avec la commande `exit` et reconnectez-vous.

SonarQube nécessite une configuration spécifique, lancez les commandes ci-dessous :

```bash
sudo su
sysctl -w vm.max_map_count=524288
sysctl -w fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
exit
```

On peut maintenant créer un dossier pour stocker SonarQube, et se déplacer dedans avec `mkdir sonarqube && cd sonarqube`.

Nous allons utiliser Docker Compose pour faire fonctionner SonarQube, créez donc un fichier YAML avec `nano compose.yaml`, et mettez le contenu suivant dedans :

```yaml
version: "3"

services:
  sonarqube:
    image: sonarqube:community
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
  db:
    image: postgres:12
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```

Enregistrez et quittez avec `Ctrl+X`, puis démarrez les services avec `docker compose up -d`.

Rendez-vous sur `http://<pseudo_github>-server.eddi.cloud`, connectez-vous avec les identifiants `admin/admin` et changez le mot de passe par `adminoclock`.

## Configuration

Cliquez sur le `A` en haut à droite pour accéder aux paramètres du compte admin, puis sur `My Account` & `Security`. **Générez un token et copiez-le**.

Rendez-vous sur les secrets de votre dépôt GitHub et ajoutez deux secrets : `SONARQUBE_TOKEN` (contenant le token généré) et `SONARQUBE_HOST` (contenant l'url de notre VM Serveur Kourou, avec le `http://`).

Depuis la page d'accueil de SonarQube, choisissez de créer un projet manuellement et suivez les instruction. Le code à ajouter à notre workflow YAML nous sera directement donné par SonarQube.

:warning: **Attention, la VM Serveur doit être rendue publique depuis Kourou.**

## Tester

Essayez de pousser votre code sur la branche `master`, et surveillez l'état du workflow. Si tout s'est bien passé, vous devriez pouvoir avoir le résultat de l'analyse du code sur SonarQube peu de temps après.

## Aller plus loin

Les outils comme SonarQube nous permettre de définir nos propres règles en terme de qualité de code. On peut donc imaginer créer nos propres règles, pour garantir que le code produit par une équipe de développeur respecte les standards imposés par l'entreprise.