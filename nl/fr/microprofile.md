---

copyright:
  years: 2019
lastupdated: "2019-04-04"

keywords: java ee, jakarta ee, microprofile overview, cloud native java, cloud native microprofile

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

# Présentation
{: #jee-overview}

## Java EE et Jakarta EE
{: #jakarta-ee}

Les technologies introduites dans Java&trade; EE7 et EE8 sont particulièrement adaptées à la création de microservices dans Java&trade; grâce à une prise en charge des annotations, des protocoles légers et des modèles d'interaction asynchrones. Les utilitaires de concurrence dans Java&trade; EE7 fournissent un mécanisme simple pour utiliser des bibliothèques réactives comme RxJava dans un mode adapté aux conteneurs.

En septembre 2017, Oracle, soutenu par {{site.data.keyword.IBM}} et Red Hat, a annoncé que Java&trade; EE allait rejoindre la Fondation Eclipse, sous le nom de Jakarta EE. Oracle reste propriétaire de tout ce qui est associé à Java&trade; EE, jusqu'à Java&trade; EE 8 inclus. Néanmoins, tous les développements futurs de cette plateforme Java&trade; Enterprise après Java&trade; EE 8 sont réalisés par la Fondation Eclipse sous la direction du groupe de travail Jakarta EE. Début 2018, Oracle a commencé le grand effort de réattribution de licences et de contribution des technologies Java&trade; EE qui inclut les API, RI, TCK, ainsi que la documentation de projet associée au projet EE4J.

## MicroProfile
{: #microprofile}

Même si Java&trade; EE offrait une base solide pour créer des microservices, il avait besoin de technologies et de modèles de programmation pour mieux s'adapter aux applications de microservice. IBM et d'autres entreprises ont travaillé ensemble pour lancer MicroProfile, une collaboration ouverte entre les développeurs, la communauté et les fournisseurs.

La communauté MicroProfile se concentre sur l'innovation rapide autour des microservices et Enterprise Java&trade;. Cette communauté construit et intègre les technologies les mieux adaptées aux applications natives du cloud qui suivent les modèles architecturaux des microservices. Chaque version de MicroProfile contient un ensemble de technologies. MicroProfile 1.0, sorti en 2016, contenait un sous-ensemble de spécifications de Java&trade; EE 7 fournissant un ensemble de fonctionnalités de base pour les microservices légers : CDI 1.2, JAX-RS 2.0 et JSON-P 1.0. La plate-forme MicroProfile inclut des technologies qui permettent de répondre à des préoccupations spécifiques au cloud, telles que l'injection de configuration, la tolérance aux pannes, les diagnostics d'intégrité, les métriques et le traçage ouvert. Il définit également des normes agnostiques pour travailler avec les définitions OpenAPI et créer des clients REST qui facilitent la consommation d'API REST et, pour la propagation JWT, pour répondre aux problèmes de sécurité des applications.

Pour commencer à participer au groupe open source, visitez [https://microprofile.io/](https://microprofile.io/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") ou [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").
