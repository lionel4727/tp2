# Cloud public - TP2

## Instructions
Durant ce TP, nous allons mettre en oeuvre le platform-as-a-service ainsi que les concepts de Serverless que nous avons vu durant le cours. Pour ce faire, nous allons déployer l'application PetClinic sur Elastic Beanstalk dans la première partie de ce TP, et sur AWS Lambda dans la seconde partie du TP.

Un bref rappel de l'architecture de l'application
![Architecture](https://spring-petclinic.github.io/images/petclinic-microservices-architecture.png "Architecture")

Le TP est découpé en deux parties : le fichier ```README.md``` suivant, ainsi qu'un fichier ```ÀNSWER.md``` qui contient 10 questions à répondre.

Le principe du TP est de répondre aux questions dans le fichier ```ANSWER.md``` et de le pousser sur votre repository. Le reste est expliqué au fur et à mesure de ce TP.

## 1 : Git
### 1.0 : Forker le repo du lab
Suivre le lien présent au tableau pour forker ce repository dans votre repository personnel GitHub Classroom. Si vous avez accès au repository, cela veut dire que la manipulation a bien fonctionné.

### 1.1 : Cloner le repository
Clonez le repo nouvellement copié sur votre ordinateur 
> A partir de maintenant, **vous ne travaillerez plus que dans votre copie**. Vous n'avez plus à revenir sur [le projet parent](https://github.com/cours-hei/tp1).

## 2 : Elastic Beanstalk
Dans un premier temps, nous allons déployer le projet PetClinic sur Elastic Beanstalk

### 2.1 : Déployer Elastic Beanstalk
Comme vu durant le cours, pour déployer un Elastic Beanstalk, aller sur le portail AWS et cliquer sur le menu Elastic Beanstalk (si vous ne trouvez pas le menu, il est possible aussi de passer par la recherche AWS).
Dans le menu Elastic Beanstalk, créer une application. Cette application doit être de type :
- Platform : Java
- Platform branch : Java 8 running on 64bit Amazon Linux
- Platform version : 2.10.11 (Recommended)

Afin de déployer l'application PetClinic, il est nécessaire d'uploader l'application compilée, grâce au menu : Upload your code. Il est donc surement nécessaire de recompiler l'application. L'application compilée se trouvera alors dans le dossier : ```target```

> ⚠️  **WARNING**: Créez une issue s'intitulant `2.1` ayant pour contenu les commandes que vous avez effectuées

### 2.2 : Vérifier que l'application fonctionne
Une fois l'application déployée, il faut maintenant vérifier que celle-ci fonctionne. Cliquer sur l'URL exposée de l'application.
Vous devriez alors avoir une erreur de proxy nginx. Cela est du au fait que par défaut, Elastic Beanstalk s'attend à ce que l'application écoute sur le port 5000. Il faut alors trouver un moyen de configurer le proxy.

Un peu d'aide se trouve par ici : https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-nginx.html

> ⚠️  **WARNING**: Créez une issue s'intitulant `2.2` ayant pour contenu les commandes que vous avez effectuées

### 2.3 : Se connecter à une base de données
Maintenant que l'application PetClinic fonctionne, il est nécessaire d'ajouter une base de données. Par chance, Elastic Beanstalk permet d'ajouter une base de données simplement. Dans la configuration d'Elastic Beanstalk, ajouter une base de données RDS en la configuration comme suit :
- Engine : mysql
- Engine version : 8.0.20
- Instance class : db.t2.micro
- Storage : 5
- Username : petclinic
- Password : petclinic
- Retention : Delete
- Availability : Low (one AZ)

Cette commande va avoir comme conséquence de créer une base de données managée RDS MySQL. Une fois la base de données créée, AWS va retourner le endpoint de la base RDS. Il faut alors redéployer l'application PetClinic avec une nouvelle configuration applicative prenant en compte cette base de données (changement de la configuration applicative dans le code source, recompilation de PetClinic, reupload du binaire compilé, déploiement sur l'environnement Elastic Beanstalk).

La table à utiliser dans la chaine de connexion est : ebdb

Il est aussi nécessaire de changer la configuration de Hibernate dans le projet en mettant la propriété suivante à jour : ```spring.jpa.hibernate.ddl-auto=create-drop```

> ⚠️  **WARNING**: Créez une issue s'intitulant `2.3` ayant pour contenu les commandes que vous avez effectuées

### 2.4 : Activer le blue/green deployment
Une fois l'application PetClinic modifiée afin d'utiliser la base de données RDS, nous allons modifier le code source afin d'apporter un changement visible sur l'application. Cependant, et afin de ne pas impacter les utilisateurs connectées, il faudra utiliser la technique du blue/green deployment.

Cloner l'environnement actuel (celui avec la base de données RDS) afin de créer une pré-production ressemblant en tout point à la production. Cet environnement nous servira d'environnement de test afin de tester les changements apportés.

Modifier le code afin d'apporter un changement visible. Par exemple, modifier le fichier ```welcome.html``` présent dans le dossier ```src/main/resources/templates/welcome.html```.

Recompiler l'application et la redéployer sur le nouvelle environnement. Vérifier avec la nouvelle URL que les changements apportés sont bien visible.

Une fois que les changements apportés sont satisfaisant, basculer l'URL de production sur la nouvelle version.

> ⚠️  **WARNING**: Créez une issue s'intitulant `2.4` ayant pour contenu les commandes que vous avez effectuées

## 3 : AWS S3 et AWS Lambda
Nous allons maintenant déployer l'application en utilisant AWS S3 et AWS Lambda

### 3.1 : Déployer le site web
Nous allons utiliser une nouvelle version de l'application web développer en Angular. Cette nouvelle version à l'avantage de pouvoir être compilé en site static et ainsi de pouvoir être déployer sur un hébergement de site static, comme AWS S3.

Cloner le nouveau repo suivant : ```https://github.com/spring-petclinic/spring-petclinic-angular```

Suivre la documentation d'installation et de build de l'application. A la fin de la compilation, vous devriez avoir un dossier ```dist``` contenant le site web compilé.

> ⚠️  **WARNING**: Créez une issue s'intitulant `3.1` ayant pour contenu les commandes que vous avez effectuées

### 3.2 : Création du bucket S3
Le site va devoir maintenant être hébergé sur un bucket S3 public.

Aller dans le menu S3 et cliquer sur le bouton **Create bucket**. Dans le nouveau menu, choisir un nom DNS unique, puis dans les permissions, décocher la case **Block *all* public access**, puis cocher la case *I acknowledge that the current settings may result in this bucket and the objects within becoming public*

Une fois le bucket créé, uploader **tout** le contenu du dossier *dist* (sous-dossier inclus) dans le bucket S3. Ensuite, sélectionner tous les fichiers et sous-dossiers présent dans le bucket, cliquer sur le bouton **Actions** et sélectionner **Make public**.

Il faut ensuite activer la solution d'hébergement static d'AWS S3. Pour ce faire, aller dans l'onglet Properties, cliquer sur le carré Static Web Hosting et activer l'option *Use this bucket to host a website*. Dans le champ, **Index document**, indiquer le fichier **index.html**.

Cliquer enfin sur **Save**.

Vous devrier pouvoir accéder au site web via une URL de la forme : ```http://<bucket-name>.s3-website-<region>.amazonaws.com```

Le site ne devrait pas fonctionner parfaitement, car il manque maintenant les calls API. Dans la suite du TP, nous allons déployer l'application sur AWS Lambda.

> ⚠️  **WARNING**: Créez une issue s'intitulant `3.2` ayant pour contenu les commandes que vous avez effectuées

### 3.3 : Déploiement de l'application sur AWS Lambda
Nous allons maintenant déployer l'application PetClinic sur AWS Lambda. Pour pouvoir utiliser PetClinic sur AWS Lambda, nous allons utiliser un nouveau projet exposant des API REST.

Cloner le repo : ```https://github.com/spring-petclinic/spring-petclinic-rest```. Avant de pouvoir déployer l'application sur AWS Lambda, il est nécessaire d'apporter quelques modifications afin qu'AWS Lambda accepte l'application. Toutes les explications sont disponible ici : https://github.com/awslabs/aws-serverless-java-container/wiki/Quick-start---Spring-Boot2#manual-setup--converting-existing-projects

Lors de l'étape **3. Packaging the application**, il est aussi nécessaire d'enlever la partie suivante du ```pom.xml```
```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <!-- Spring Boot Actuator displays build-related information
                if a META-INF/build-info.properties file is present -->
            <goals>
                <goal>build-info</goal>
            </goals>
            <configuration>
                <additionalProperties>
                    <encoding.source>${project.build.sourceEncoding}</encoding.source>
                    <encoding.reporting>${project.reporting.outputEncoding}</encoding.reporting>
                    <java.source>${maven.compiler.source}</java.source>
                    <java.target>${maven.compiler.target}</java.target>
                </additionalProperties>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Une fois toutes les manipulations effectuées, il faut alors recompiler l'application et déployer le ```.jar``` présent dans le dossier ```target``` dans un nouveau AWS S3 Bucket. Ce nouveau bucket servira à déployer l'application sur AWS Lambda.

> ⚠️  **WARNING**: Créez une issue s'intitulant `3.3` ayant pour contenu les commandes que vous avez effectuées

### 3.4 : AWS Lambda et API Gateway
Nous allons maintenant créer une AWS Lambda afin de récupérer les diverses informations de l'application PetClinic. Aller dans le menu AWS Lambda, et cliquer sur le bouton Create function. Donner un nom à votre fonction, choisisser le runtime **Java 8** et cliquer sur Create function.

Dans la nouvelle page, dans la section **Function code**, sélectionner **Actions** et **Upload a file from Amazon S3**. Copier/coller alors le chemin du bucket S3 contenant le binaire PetClinic.

Une fois le bucket S3 configuré, il faut maintenant changer le handler qui va répondre à nos requêtes. Aller dans la section Basic Settings plus bas dans la page. Changer alors le champ handler en mettant votre handler (chez moi ```org.springframework.samples.petclinic.StreamLambdaHandler::handleRequest```). Changer aussi le timeout à 2 minutes et la mémoire allouée à 1536 MB.

Une fois ces changements fait, retourner en haut de la page et cliquer sur Add trigger. Dans la nouvelle page, choisisser **API Gateway**, puis **Create a new API**, puis **HTTP API** et enfin **Open** dans le menu **Security**. Cliquer sur **Add**. Votre Lambda sera alors exposée via l'URL ```https://<random>.execute-api.<region>.amazonaws.com/default/<lambda-name>```.

> ⚠️  **WARNING**: Créez une issue s'intitulant `3.4` ayant pour contenu les commandes que vous avez effectuées

### 3.5 : Changement de l'URL d'API
Maintenant que la Lambda est exposée sur Internet, il est nécessaire de changer l'URL de call API dans le site web, et de redéployer le site web.

Aller dans le fichier ```src/environment.ts``` du projet ```spring-petclinic-angular``` et changer l'URL **REST_API_URL** avec la nouvelle URL de l'API Gateway (de la forme ```https://<random>.execute-api.<region>.amazonaws.com/default/<lambda-name>```)

Recompiler alors l'application ```spring-petclinic-angular``` et redéployer la sur le bucket S3.

La nouvelle page PetClinic est alors maintenant pleinement opérationnelle.

> ⚠️  **WARNING**: Créez une issue s'intitulant `3.5` ayant pour contenu les commandes que vous avez effectuées
