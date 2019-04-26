---

copyright:
  years: 2019
lastupdated: "2019-03-15"

keywords: open liberty java, websphere liberty java, jakarta, webpshere docker, liberty docker, liberty docker images, installutility, microprofile java, dual layer docker, develop microservices

subcollection: java

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Open Liberty et WebSphere Liberty
{: #liberty}

[Open Liberty](https://openliberty.io/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est un environnement d'exécution Java open source léger construit à l'aide de *fonctions* modulaires. [WebSphere Liberty](https://developer.ibm.com/wasdev/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est une version commerciale d'Open Liberty. Les fonctionnalités de Liberty prennent en charge les structures d'application suivantes :

* [MicroProfile](https://microprofile.io/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), un projet open source définissant de nouvelles normes et API pour accélérer et simplifier la création de microservices.
* [Jakarta EE](https://jakarta.ee){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") et [Java EE](https://www.oracle.com/technetwork/java/javaee/overview/index.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), qui incluent des fonctions pour des spécifications individuelles, comme JNDI ou JAX-RS.
* [Spring Framework et Spring Boot](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_dep_springboot.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), qui incluent des mécanismes pour fabriquer des conteneurs compacts à partir des fichiers JAR volumineux de Spring Boot.

Vous trouverez un grand nombre de guides de développement pratiques pour créer des microservices et des applications natives du cloud avec Liberty à l'adresse [https://openliberty.io/guides/](https://openliberty.io/guides/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Optimisé pour Docker
{: #liberty-optimized}

Lorsque des systèmes automatisés comme Kubernetes manipulent des images de conteneurs, la taille des images commence à avoir de l'importance. Pour résoudre ce problème, les calques des images Docker sont mis en cache. Grâce à son architecture modulaire, Liberty offre un pipeline d'emballage efficace pour les conteneurs Docker, ce qui en fait une solution opérationnelle idéale pour les environnements cloud. Une seule image de base peut être utilisée pour prendre en charge de nombreuses charges de travail, tout en permettant à l'empreinte des ressources des conteneurs en cours d'exécution de varier en fonction des exigences du service.

Liberty fournit des outils et un support optimisé pour convertir les fichiers JAR volumineux de Spring Boot en conteneurs Docker compacts et optimisés qui tirent parti des calques des images Docker en cache pour améliorer les durées de cycle et les temps de publication. En poussant vers le bas les dépendances de la bibliothèque (qui changent rarement) dans un calque séparé, et en ne conservant que les classes d'application dans le calque supérieur, les reconstructions et les redéploiements itératifs seront beaucoup plus rapides. Pour en savoir plus, lisez l'article [Creating Dual Layer Docker images for Spring Boot apps](https://openliberty.io/blog/2018/07/02/creating-dual-layer-docker-images-for-spring-boot-apps.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Images Docker Liberty
{: #liberty-images}

Open Liberty et WebSphere Liberty fournissent une variété d'images de base que vous pouvez utiliser comme points de départ pour vos microservices basés sur Java. Certaines images utilisent des petites empreintes ou des machines virtuelles OpenJ9 avec différentes fonctions présélectionnées. Par exemple :

1. open-liberty:microProfile2 ou websphere-liberty:microProfile2
2. open-liberty:javaee8 ou websphere-liberty:javaee8
3. open-liberty:springBoot2 ou websphere-liberty:springBoot2

Consultez [websphere-liberty](https://hub.docker.com/_/websphere-liberty/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") ou [open-liberty](https://hub.docker.com/_/open-liberty/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") sur le hub Docker pour obtenir une liste actualisée des images de base.

### Open Liberty et Docker
{: #openliberty-docker}

Choisissez l'image la plus appropriée pour votre application dans la [liste des images disponible sur le hub Docker](https://hub.docker.com/_/open-liberty/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lie externe") (par exemple `open-liberty:microProfile`) et créez un fichier Docker pour votre service avec `FROM`. Ajoutez les commandes pour copier votre application et la configuration de votre serveur dans la nouvelle image. Exemple :

```docker
FROM open-liberty:microProfile\
COPY server.xml /config/\
ADD Sample1.war /config/dropins/
```
{: codeblock}

Pour obtenir d'autres exemples et le code de travail, consultez les guides Open Liberty suivants :

* [Using Docker containers to develop microservices](https://openliberty.io/guides/docker.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [Running an application in a Docker container](https://openliberty.io/guides/getting-started.html#running-the-application-in-a-docker-container){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")

### WebSphere Liberty et Docker
{: #wlp-docker}

Il existe quelques différences entre Open Liberty et sa version commerciale, WebSphere Liberty. L'une des plus importantes pour la création d'images Docker est que la commande `installUtility` n'est pas disponible dans Open Liberty.

WebSphere Liberty prend en charge les mêmes modèles de personnalisation de base qu'OpenLiberty pour les images Docker, mais la conception modulaire inhérente à Liberty simplifie la création d'une image personnalisée contenant une application et l'ensemble de fonctionnalités spécifique qu'elle nécessite. WebSphere Liberty possède une image pour un noyau sans fonctionnalité, [websphere-liberty:kernel](https://github.com/WASdev/ci.docker/blob/9d28dfba4d20596f89b393bc9e3ae8295feec469/ga/developer/kernel/Dockerfile){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), qui constitue une base solide pour une image vraiment personnalisée.

La [documentation sur l'image websphere-liberty](https://hub.docker.com/_/websphere-liberty/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") décrit le fichier Docker simple suivant de 3 lignes nécessaire pour créer une image personnalisée :

```docker
FROM websphere-liberty:kernel
COPY server.xml /config/
RUN installUtility install --acceptLicense defaultServer
```
{: codeblock}

L'outil `installUtility` recherche toutes les fonctionnalités dont vous avez besoin dans votre fichier `server.xml` qui ne sont pas déjà disponibles dans votre image Liberty. Il téléchargera ensuite ces fonctions et les installera dans votre image Docker. Cette approche créera une image minimale, mais la liste des fonctionnalités implicites dans `server.xml` ne fonctionne pas bien avec les calques Docker en cache. Toute modification de `server.xml` invalide le calque avec les fonctionnalités installées, nécessitant que les fonctionnalités manquantes soient à nouveau installées la prochaine fois que l'image est construite.

Une meilleure façon de créer des images personnalisées est d'appeler `installUtility` avec une liste explicite de fonctions. Cela créera quand même une image minimale, mais se comportera beaucoup mieux au moment de la compilation, car le calque de l'image peut être mis en cache et réutilisé indépendamment des changements de configuration du serveur. Par exemple, les éléments suivants créeront une image personnalisée à l'aide d'un sous-ensemble de fonctions pour une application qui utilise JAX-RS, JNDI et WebSockets :

```docker
FROM websphere-liberty:kernel
RUN /opt/ibm/wlp/bin/installUtility install --acceptLicense \
  cdi-1.2 \
  concurrent-1.0 \
  jaxrs-2.0 \
  jndi-1.0 \
  ssl-1.0 \
  websocket-1.1
COPY server.xml /config/
```
{: codeblock}

La manière dont vous structurez votre image Docker dépend de quelques facteurs :

1. Dans quelle mesure voulez-vous que votre image de base soit réutilisable ou personnalisée ?
    Ces étapes produisent la plus petite image possible, potentiellement au détriment de la réutilisation si les caractéristiques de chaque application sont différentes. Cependant, le fait d'avoir de très petites images personnalisées signifie que l'application peut être rapidement déployée sur les hôtes Docker. Si vous n'êtes pas sûr, commencez par l'une des images prégénérées du hub Docker pour une meilleure réutilisation.
2. A quelle fréquence mettez-vous à jour cette image ?
    Si l'application est mise à jour très fréquemment, par exemple dans un pipeline CI/CD, il est avantageux de structurer l'image de manière à ce que le calque le plus fréquemment modifié (souvent vos classes d'application ou binaires applicatifs) soit en haut. Cela réduit le nombre de calques qui doivent être reconstruits lorsque l'application change et lorsque l'image est reconstruite, ce qui accélère les temps de construction et de déploiement.

En cas de doute, suivez le [guide Docker Open Liberty](https://openliberty.io/guides/docker.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour commencer.
