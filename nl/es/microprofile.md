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

# Visión general
{: #jee-overview}



## Java EE y Jakarta EE
{: #jakarta-ee}

Las tecnologías introducidas en Java EE7 y EE8 son adecuadas para la creación de microservicios en Java, con soporte para anotaciones, protocolos ligeros y patrones de interacción asíncrona. Los programas de utilidad de simultaneidad de Java EE7 proporcionan un mecanismo directo para utilizar bibliotecas reactivas como RxJava de forma sencilla para el contenedor.

En septiembre de 2017, Oracle, con el apoyo de IBM y Red Hat, anunció que Java EE iba a trasladarse a Eclipse Foundation, igual que Jakarta EE. Oracle sigue siendo propietario de todo lo relacionado con Java EE, hasta Java EE 8 incluido, pero todo el desarrollo futuro de esta plataforma Enterprise Java a partir de Java EE 8 se realizará en Eclipse Foundation bajo la dirección del grupo de trabajo de Jakarta EE. A principios de 2018, Oracle inició el gran esfuerzo de volver a otorgar licencias y de contribuir a las tecnologías de Java EE, incluidas API, RI, TCK y documentación de proyectos asociados al proyecto EE4J.

## MicroProfile
{: #microprofile}

Aunque Java EE proporciona una base sólida para crear microservicios, necesita tecnologías y modelos de programación para ajustarse mejor a las aplicaciones de microservicios. IBM y otras empresas trabajaron de forma conjunta para lanzar MicroProfile, una colaboración abierta entre desarrolladores, la comunidad y los proveedores.

La comunidad de MicroProfile se centra en la innovación rápida en torno a los microservicios y Enterprise Java. Esta comunidad crea e integra tecnologías que son más adecuadas para aplicaciones nativas de cloud que siguen patrones arquitectónicos de los microservicios. Cada release de MicroProfile contiene un conjunto de tecnologías. MicroProfile 1.0, publicado en 2016, contenía un subconjunto de especificaciones de Java EE 7 que proporcionan un conjunto de funciones básicas para microservicios ligeros: CDI 1.2, JAX-RS 2.0 y JSON-P 1.0. La plataforma MicroProfile incluye tecnologías para hacer frente a problemas de nube únicos, como la inyección de configuración, la tolerancia a errores, las comprobaciones de estado, las métricas y el rastreo abierto. También define estándares independientes del proveedor para trabajar con definiciones de OpenAPI y crear clientes REST que facilitan el consumo de API REST, y para utilizar la propagación de JWT para solucionar los problemas de seguridad de aplicaciones.

Para empezar a participar en el grupo de código abierto, visite [https://microprofile.io/](https://microprofile.io/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") o [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").
