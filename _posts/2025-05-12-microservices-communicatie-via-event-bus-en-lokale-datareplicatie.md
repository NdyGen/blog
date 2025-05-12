---
layout: post
title: "Microservices Communicatie via Event Bus en Lokale Datareplicatie"
date: 2025-05-12
categories: [software-architectuur, microservices]
tags: [microservices, event-driven, eventbus, datareplicatie, architectuur]
author: Andy van Dongen
excerpt: "Een designpatroon voor microservices die met een event bus en lokale data duplicatie onafhankelijk en schaalbaar samenwerken zonder directe service-naar-service communicatie."
---

# Microservices Communicatie via Event Bus en Lokale Datareplicatie

In de wereld van microservices is één van de grootste uitdagingen het efficiënt en betrouwbaar laten communiceren tussen services zonder dat ze elkaar direct nodig hebben. Te veel directe koppelingen leiden immers tot strakke afhankelijkheden en minder robuuste systemen. Vandaag vertel ik je het verhaal van een designpatroon waarin services via een event bus communiceren én lokaal data bijhouden - een elegante manier om de losse koppelingen te bewaren en toch razendsnelle, betrouwbare interactie te faciliteren.

> “Communiceren tussen microservices is soms als collega's die roepen over kantoor, tenzij ze via een centrale intercom praten en zelf aantekeningen maken voor later.” — Andy

---

## Context: Het Probleem van Inter-Service Communicatie

Stel je voor: je hebt een hele reeks kleine, zelfstandige microservices zoals een Bestelservice, Klantservice en Productcatalogus. Elk draait op zijn eigen tempo, technologie en database. Nu wilt de Bestelservice weten of een klant nog actief is voordat hij een bestelling accepteert. 

De klassieke aanpak: Bestelservice roept direct de Klantservice aan (sync REST aanroep). Maar oh oh, wat als de Klantservice traag is of offline? Daarna ontstaat “distributed coupling” en worden je services afhankelijk van elkaars beschikbaarheid. Debuggen voelt als speleologie zonder kaart.

---

## De Event Bus als Tussenpersoon

Het alternatief: een Event Driven Architectuur met een centrale Event Bus.

- **Wat is een Event Bus?**  
Een message broker (zoals Kafka, RabbitMQ of NATS) waar services events naar toe kunnen publiceren — berichten als “KlantGeregistreerd” of “BestellingAangemaakt”.

- **Hoe helpt het?**  
Services publiceren domain events naar de bus. Andere services kunnen zich abonneren om deze events asynchroon te ontvangen zonder direct contact te leggen.

---

## Het Designpatroon: Lokale Datareplicatie via Domain Events

In plaats van telkens live bij een andere service te informeren, houdt elke service zijn eigen lokale “kopie” van relevante data bij door deze events te consumeren.

### Hoe werkt dat praktisch?

1. **Klantservice publiceert een event:**  
Wanneer een klant wordt geregistreerd of gewijzigd, produceert de Klantservice een event zoals `KlantAangemaakt` of `KlantGewijzigd`.

2. **De Bestelservice luistert mee:**  
De Bestelservice abonneert zich op klantgerelateerde events. Bij ontvangst werkt hij zijn eigen lokale klantendatabase bij. Hierdoor heeft hij altijd snelle toegang tot actuele klantdata.

3. **Bestelservice is autonoom:**  
Voor elke bestelling heeft hij alle info lokaal. Geen netwerkverzoeken naar Klantservice nodig.

---

## Voordelen van deze aanpak

- **Losse koppeling:** Services communiceren alleen via de event bus en niet direct onderling.
- **Hoge beschikbaarheid:** Een uitvallende service blokkeert geen verzoeken.
- **Schaalbaarheid:** Elke service kan onafhankelijk opgeschaald worden.
- **Performance:** Lezen uit een lokale databank is sneller dan netwerkverkeer.
- **Consistentie op eventual basis:** Data is bijna realtime synchroon, met minimale vertraging.

---

## Mogelijke nadelen en oplossingen

| Uitdaging               | Toelichting                                    | Oplossing                                  |
|------------------------|-----------------------------------------------|--------------------------------------------|
| Data duplicatie         | Meerdere services houden eigen kopieën bij     | Consistentiegaranties via event sequencing |
| Event ordering          | Events kunnen niet altijd in vertrekvolgorde komen | Kafka-gebaseerde event streams met partitions |
| Complexiteit bij fouten | Event handlers kunnen missen of falen           | Dead-letter queues en retry-mechanismen     |
| Data consistentie       | Geen directe transactie tussen services         | Eventual consistency accepteren en compenserende acties |

> “Het is een beetje zoals een goed georganiseerde krant: iedereen die hem nodig heeft, leest dezelfde editie, maar dat betekent niet dat ze allemaal tegelijkertijd lezen.” — Andy

---

## Scenario: Een Voorbeeld Implementatie in de Praktijk

Laten we de Bestelservice en Klantservice als voorbeeld gebruiken. Hieronder een fragment van hoe je dat kunt doen met Kafka als event bus.

```java
// Voorbeeld event publicatie in Klantservice
public void registreerKlant(Klant klant) {
    klantRepository.save(klant);
    KlantAangemaaktEvent event = new KlantAangemaaktEvent(klant.getId(), klant.getNaam());
    kafkaProducer.produce("klant-events", event);
}

// Voorbeeld event consumer in Bestelservice
@KafkaListener(topics = "klant-events")
public void handleKlantEvents(KlantAangemaaktEvent event) {
    Klant klant = new Klant(event.getKlantId(), event.getNaam());
    klantRepository.save(klant);
}
```

Zodra de Klantservice een nieuwe klant aanmaakt, wordt automatisch de lokale klantendatabase van de Bestelservice bijgewerkt.

---

## Visuele Uitleg

![Event Bus Communicatie Diagram](/images/microservices-eventbus-communicatie.png)  
*Alt-text: Diagram dat toont hoe microservices berichten via een event bus publiceren en lokale databanken bijwerken.*

---

## Conclusie

Door communicatie tussen microservices te organiseren via een event bus en het bijhouden van lokale datareplicatie, bereik je een robuuste, schaalbare microservices-architectuur zonder ingewikkelde synchronisatieproblemen. Dit patroon voorkomt de klassieke valkuilen van directe, synchrone communicatie en maakt je systeem veerkrachtiger en onderhoudsvriendelijker.

Misschien klinkt het eerst wat ingewikkeld — maar "debuggen dat legacy systeem was vroeger alsof je spelendeot spelletje doolhof zonder kaart," — en nu met de event bus navigeren we een stuk eenvoudiger.

---

## Referenties

- [Martin Fowler over Event-driven Architectuur](https://martinfowler.com/articles/201701-event-driven.html)
- [Apache Kafka officiële documentatie](https://kafka.apache.org/documentation/)
- [Microservices patterns, Chris Richardson](https://microservices.io/patterns/data/event-sourcing.html)

---

#hashtags  
#Microservices #EventBus #EventDriven #DataReplicatie #SoftwareArchitectuur