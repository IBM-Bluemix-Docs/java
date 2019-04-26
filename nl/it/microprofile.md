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

# Panoramica
{: #jee-overview}



## Java EE e Jakarta EE
{: #jakarta-ee}

Le tecnologie introdotte in Java EE7 e EE8 sono adatte per la creazione di microservizi in Java, con il supporto per le annotazioni, dei protocolli leggeri e dei modelli di interazione asincroni. I programmi di utilità di simultaneità in Java EE7 forniscono un semplice meccanismo per utilizzare librerie reattive come RxJava in un ambiente adattato ai contenitori.

Nel mese di settembre del 2017, Oracle, supportata da IBM e Red Hat, ha annunciato il passaggio di Java EE alla Eclipse Foundation, come Jakarta EE. Oracle continua a possedere tutto quanto è associato a Java EE, fino a Java EE 8 incluso, ma tutto lo sviluppo futuro di questa piattaforma Java Enterprise dopo Java EE 8 avrà luogo presso la Eclipse Foundation sotto la guida del gruppo di lavoro Jakarta EE. All'inizio del 2018, Oracle ha avviato l'ampio sforzo di concedere nuovamente le licenze e offrire come contributo le tecnologie Java EE, compresi API, RI, TCK e la documentazione del progetto associata al progetto EE4J.

## MicroProfile
{: #microprofile}

Pur potendo fornire una solida base per creare microservizi, Java EE aveva bisogno di tecnologie e modelli di programmazione per essere più adatto alle applicazioni di microservizi. IBM e altre società hanno lavorato insieme per lanciare MicroProfile, una collaborazione aperta che vede coinvolti sviluppatori, community e fornitori.

La community MicroProfile è focalizzata sulla rapida innovazione relativa ai microservizi e a Java Enterprise. Questa community sviluppa e integra tecnologie ideali per le applicazioni native del cloud che seguono gli schemi architetturali dei microservizi. Ogni release di MicroProfile contiene un insieme di tecnologie. MicroProfile 1.0, rilasciato nel 2016, conteneva un sottoinsieme di specifiche da Java EE 7 che fornisce un insieme di funzionalità di base per i microservizi leggeri: CDI 1.2, JAX-RS 2.0 e JSON-P 1.0. La piattaforma MicroProfile include delle tecnologie volte ad affrontare problematiche specifiche del cloud, come l'inserimento della configurazione, la tolleranza di errore, i controlli di integrità, le metriche e la traccia aperta. Definisce anche degli standard indipendenti dai fornitori per lavorare con le definizioni OpenAPI e creare dei client REST che semplificano l'utilizzo delle API REST e per la propagazione JWT per affrontare i problemi relativi alla sicurezza delle applicazioni.

Per iniziare a partecipare al gruppo open source, visita [https://microprofile.io/](https://microprofile.io/){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno") o [https://projects.eclipse.org/projects/technology.microprofile](https://projects.eclipse.org/projects/technology.microprofile){: new_window} ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno").
