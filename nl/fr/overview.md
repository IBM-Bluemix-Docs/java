---

copyright:
  years: 2019
lastupdated: "2019-06-10"

keywords: openjdk, open j9, download openjdk, java frameworks, adoptopenjdk, eclipse openj9, openj9 binaries, openjdk binaries, microprofile framework, jakarta

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

# Tutoriel d'initiation
{: #getting-started}

Les applications Java&trade; existent sous toutes les formes et dans toutes les tailles, dans les applications autonomes comme dans les applications Java&trade; EE/Jakarta EE, MicroProfile ou Spring.

## OpenJDK avec Open J9
{: #openjdk}

Le projet OpenJDK&trade; est l'implémentation de référence open source du langage Java&trade;. Il fournit une stabilité et une compatibilité des API aux programmes utilisateurs. La [plateforme Java&trade; SE](https://docs.oracle.com/javase/8/docs/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") se compose des API et de la machine virtuelle Java&trade;. La machine virtuelle Java est le moteur d'exécution responsable de l'exécution de l'application Java&trade;.

Le [projet Eclipse OpenJ9](https://www.eclipse.org/openj9/index.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") fournit une machine virtuelle Java alternative qui remplace la machine virtuelle Java Hotspot dans OpenJDK. La machine virtuelle Java OpenJ9 a été rendue open source par {{site.data.keyword.IBM}} en septembre 2017, bien que sa lignée remonte encore plus loin. En effet, OpenJ9 alimente plus de 3000 produits {{site.data.keyword.IBM}}, y compris {{site.data.keyword.appserver_short}}, pendant plus d'une décennie.

Ensemble, OpenJDK et Eclipse OpenJ9 offrent des performances et une facilité d'entretien améliorées pour les applications Java&trade; natives du cloud, bien au-delà des temps de traitement Java&trade; traditionnels basés sur Hotspot.

### Téléchargement d'OpenJDK avec OpenJ9
{: #downloads}

Les binaires OpenJDK avec OpenJ9 sont disponibles gratuitement sur le hub de la communauté Java&trade; sous [AdoptOpenJDK](https://adoptopenjdk.net/releases.html?variant=openjdk8&jvmVariant=openj9){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"). AdoptOpenJDK fournit des binaires OpenJDK pré-construits à partir d'un ensemble de scripts de compilation et d'infrastructures entièrement open source. Les images Docker sont également disponibles sur le [hub Docker](https://hub.docker.com/u/adoptopenjdk){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Infrastructures Java
{: #frameworks}

OpenJDK et OpenJ9 fournissent une base solide pour toute application Java&trade;, mais les structures d'application sont souvent utilisées pour résoudre des problèmes communs. Les applications d'entreprise, par exemple, sont généralement construites avec [Java&trade; EE (et Jakarta EE)](/docs/java?topic=java-jee-overview#jakarta-ee) ou avec [Spring](/docs/java?topic=java-spring-overview).  [MicroProfile](/docs/java?topic=java-jee-overview#microprofile) apporte un rythme de développement plus rapide et la prise en charge des concepts natifs cloud dans Java&trade; EE.

[{{site.data.keyword.runtime_liberty_full}}](/docs/java?topic=java-liberty) est un serveur d'application Java&trade; rapide, dynamique et facile à utiliser, basé sur le projet open source Open Liberty. Il fournit un temps d'exécution flexible, adapté aux besoins des entreprises pour les applications Java&trade; EE et Spring.
