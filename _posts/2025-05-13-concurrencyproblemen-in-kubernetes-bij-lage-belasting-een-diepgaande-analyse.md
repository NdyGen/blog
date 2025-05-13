---
layout: post
title: "Concurrencyproblemen in Kubernetes bij lage belasting: een diepgaande analyse"
date: 2025-05-13
categories: [kubernetes, concurrency, spring-boot, devops]
tags: [kubernetes, concurrency, spring boot, resource management]
author: Andy van Dongen
excerpt: "Rapporten van onverwachte pending statusproblemen in Spring Boot API's op Kubernetes bij lage belasting benadrukken de noodzaak van betere resourcebeheerstrategieën."
cover-img: /images/kubernetes-concurrency-issues-low-load.png
---

# Concurrencyproblemen in Kubernetes bij lage belasting: een diepgaande analyse

Concurrencyproblemen verwacht je meestal onder hoge belasting, maar wat als jouw Spring Boot API's op Kubernetes juist bij _lage_ belasting onverwacht in een `Pending` status blijven hangen? Dit lijkt tegenstrijdig, maar het is een serieus issue dat recent aan het licht komt in productieomgevingen. 

In dit artikel analyseren we dit fenomeen: waarom deze concurrencyproblemen ontstaan, hoe Kubernetes en Spring Boot hier een rol in spelen, en hoe je dit succesvol kunt aanpakken met praktische tips. Zo ben je beter voorbereid om resource management effectief te managen, zelfs wanneer de request load laag is.

## Scenario: Spring Boot API’s in Kubernetes met lage belasting

Stel je voor: een organisatie draait een reeks microservices gebouwd met Spring Boot in een Kubernetes cluster. De API's ontvangen slechts een paar verzoeken per minuut. Ondanks die rustige situatie merkt het engineering team dat sommige pods of API calls langdurig in een "Pending" of "Waiting" status blijven, met verhoogde latentie en soms time-outs als gevolg.

### Wat betekent de "Pending" status?

In Kubernetes betekent `Pending` onder meer:

- Resource tekorten (CPU, geheugen)
- Niet beschikbaar zijn van volumes
- Problemen met scheduling door node beperkingen
- Concurrency limieten op applicatie- of infrastructuurniveau

Het opvallende hier is dat de resource situatie schijnbaar ruim voldoende is, waardoor andere oorzaken onderzocht moeten worden.

## Concurrency in Spring Boot en Kubernetes: een korte uitleg

Spring Boot verwerkt inkomende aanvragen via ingebouwde threadpools, bijvoorbeeld met Tomcat of Netty connectors. Deze hebben standaard limieten om oversturing te vermijden. Kubernetes beheert daarnaast pod resources en netwerkverkeer, waarbij pods draaien in containers met ingestelde resource requests en limits. Als deze niet goed zijn afgestemd – ook bij lage belasting – kan dit leiden tot verborgen blokkades die requests in wachtrijen plaatsen.

![Diagram concurrency in Kubernetes en Spring Boot](images/kubernetes-concurrency-issues-low-load.png "Schema van threadgebruik en pod scheduling in Kubernetes bij lage belasting")

## Mogelijke oorzaken van de problemen

### 1. Onjuiste resource requests en limits

De scheduler plaatst pods op nodes op basis van opgegeven CPU- en geheugenvraag. Bij te lage resource requests kan een pod op een node komen te staan waar toch schaarste ontstaat, wat leidt tot threadblokkering in de applicatie.

### 2. Default threadpool configuraties die niet passen bij het deployment scenario

Spring Boot’s standaard threadpools kunnen te beperkt zijn. Bij lage frequenties worden threads niet soepel vrijgegeven, of raken wachtrijen overvol, waardoor requests blijven hangen.

### 3. Sidecar containers of netwerk proxies

Sidecar proxies zoals Istio of Linkerd kunnen netwerkverkeer routeren op manieren die connecties onbeheerd laten hangen wanneer verkeerd geconfigureerd.

### 4. Liveness en readiness probes die niet correct zijn afgesteld

Foute probe-configuraties kunnen ervoor zorgen dat Kubernetes een pod onterecht als onready beschouwt, wat leidt tot verzoeken die in limbo blijven.

### 5. Concurrency limieten in de database of externe services

Externe services zoals databases kunnen bottlenecks hebben die niet direct zichtbaar zijn, maar wel verzoeken blokkeren en wachtrijen veroorzaken.

## Hoe herken je problemen als deze?

- Gebruik `kubectl get pods` om `Pending` pods te identificeren.
- Check Spring Boot logs op threadpool wachtrijen en time-outs.
- Monitor resourcegebruik via `kubectl top pod` en node metrics.
- Analyseer netwerkverkeer rondom sidecars en API-pods.
- Inspecteer readiness en liveness probe configuraties.

## Praktische oplossingen en aanbevelingen

### 1. Optimaliseer resource requests en limits

Voer performance profiling uit om realistische resource behoeften vast te stellen. Gebruik ook de [Vertical Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) voor adaptieve schaling van resources op pod-niveau.

### 2. Pas threadpools aan in Spring Boot

Hieronder volgt een voorbeeldconfiguratie in `application.yaml` om de Tomcat-connector meer threads en een hogere accept-count toe te wijzen, wat helpt om wachtrijen te voorkomen:

```yaml
server:
  tomcat:
    max-threads: 200
    accept-count: 100
```

Ook kan je met Java-configuratie de ThreadPoolTaskExecutor aanpassen om meer controle te hebben over poolgrootte en wachtrijcapaciteit:

```java
@Bean
public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(20);
    executor.setMaxPoolSize(50);
    executor.setQueueCapacity(100);
    executor.initialize();
    return executor;
}
```

### 3. Controleer en optimaliseer sidecar configuraties

Zorg ervoor dat proxies zoals Istio en Linkerd correct zijn geconfigureerd. Gebruik commando’s zoals `istioctl proxy-status` en analyseer logs om hangende connecties op te sporen.

### 4. Verfijn liveness en readiness probes

Pas tijdslimieten en frequenties aan zodat deze aansluiten bij de daadwerkelijke opstart- en responstijden van jouw applicatie, zodat Kubernetes ten onrechte niet requestverkeer blokkeert.

### 5. Monitor en beperk externe oorzaken

Implementeer uitgebreide monitoring op al je afhankelijkheden, zoals databases en externe API's. Gebruik patronen zoals circuit breakers (bijv. met [Resilience4j](https://resilience4j.readme.io/)) om te voorkomen dat falende services hele ketens blokkeren.

## Conclusie

Concurrencyproblemen bij lage belasting in Kubernetes zijn een uitdaging die om aandacht voor detail vraagt, vooral in resource- en threadpoolbeheer. Het is een waardevolle reminder voor engineers: lage load betekent niet automatisch zorgeloos draaien. Juist kleine configuratie-aanpassingen en goed inzicht in je deployment omgeving maken het verschil tussen gladde werking en onverwachte vertragingen.

❗ Wil je verder verdiepen in Kubernetes resource management en concurrency patterns? Bekijk dan de officiële Kubernetes documentatie over [resource requests en limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) en lees de uitgebreide [Spring Boot webserver configuratiehandleiding](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#webserver-configuration).

![Overzicht Kubernetes concurrency bij lage load](images/kubernetes-concurrency-issues-low-load-summary.png "Overzicht van concurrency en resource management in Kubernetes bij lage belasting")

---

**Referenties:**

- [Kubernetes Docs - Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)  
- [Spring Boot Docs - Thread Pool Configuration](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#webserver-configuration)  
- [Istio Documentation](https://istio.io/latest/docs/)  
- [Resilience4j Circuit Breakers](https://resilience4j.readme.io/)

---

*Door Andy van Dongen*  
*mei 2025*  

---