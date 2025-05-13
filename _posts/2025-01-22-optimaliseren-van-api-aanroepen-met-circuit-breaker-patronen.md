---
layout: post
title: "Optimaliseren van API-aanroepen met Circuit Breaker-patronen"
date: 2025-05-22
categories: [software-architectuur, microservices]
tags: [circuit breaker, api calls, microservices, foutafhandeling]
author: Andy van Dongen
cover-img: /images/circuit-breaker-pattern-diagram.png
excerpt: "Circuit breaker-patronen zorgen voor robuustere API-aanroepen binnen microservices door falen af te handelen en systeemstabiliteit te waarborgen."
---

# Inleiding
In microservices is communicatie via API-aanroepen de ruggengraat van je systeem. Maar wat doe je als die cruciale dienst ineens traag wordt of niet meer reageert? Zonder bescherming kan dat een kettingreactie veroorzaken die je hele applicatie lamlegt.

Het **circuit breaker-patroon** helpt je om dit soort drama's te voorkomen. Zie het als een slimme stroomonderbreker voor je software die problemen detecteert en tijdelijk uitzet, zodat je systeem heel blijft.

In dit artikel leer je:

- Wat het circuit breaker-patroon inhoudt en hoe het werkt.
- Waarom het onmisbaar is voor stabiele API-aanroepen in microservices.
- Hoe je een circuit breaker praktisch implementeert in Java met Resilience4j.
- Best practices om het patroon effectief toe te passen in jouw projecten.

---

# Wat is het Circuit Breaker-patroon?
Het circuit breaker-patroon is gebaseerd op de elektrische schakelaars die de stroom onderbreken bij fouten of overbelasting om schade te voorkomen. In software werkt het concept hetzelfde: het voorkomt dat falende functies je hele keten meenemen in hun val.

In microservices betekent dit dat zodra een dienst tekenen van falen vertoont, een circuit breaker de aanroepen naar die dienst tijdelijk blokkeert. Verzoeken falen direct zonder er continue op te wachten, wat systeemresources spaart.

De breaker kent drie hoofdtoestanden:

- **Closed (Gesloten)**: Alles functioneert normaal, calls gaan door.
- **Open (Open)**: Na een drempel aan fouten opent de breaker en worden aanvragen direct geweigerd.
- **Half-open (Half-open)**: Een testfase waar een beperkt aantal verzoeken wordt toegelaten om te checken of de dienst weer gezond is.

---

# Waarom is het zo belangrijk bij API-aanroepen?
API-aanroepen vormen het zenuwstelsel van microservices. Een falende dienst kan een verstopping in de communicatie veroorzaken, met wachttijden, time-outs en uiteindelijk uitval tot gevolg.

Zonder circuit breaker blijven services fouten proberen te herstellen door aanhoudend te bellen, wat de situatie alleen maar verergert. Met een breaker voorkom je dat één falende dienst een domino-effect veroorzaakt. Zo garandeer je een stabiele en responsieve applicatie, zelfs onder zware belasting.

---

# Praktisch scenario: De trage calorie-tellerservice in een fitness-app
Stel je een fitness-app voor die diverse microservices combineert, waaronder een calorie-tellerservice die activiteiten analyseert.

Wanneer deze service traag wordt of niet reageert, zal de app lang wachten op antwoord of zelfs crashen als er geen foutafhandeling is.

**Zonder circuit breaker:**

- Calls blijven hangen met time-outs.
- Gebruikers ervaren frustraties en vertraging.
- Andere services kunnen overbelast raken door voortdurende pogingen.

**Met circuit breaker:**

- Na een aantal mislukkingen opent de breaker.
- Calls falen meteen zonder wachttijd.
- Gebruikers krijgen snelle fallback of cache-data.
- De breaker test regelmatig of de service weer stabiel is.

Door deze aanpak blijft de app soepel lopen, zelfs als één onderdeel hapert.

---

# Implementeren van circuit breaker in Java met Resilience4j

Hieronder volgt een concreet voorbeeld van het implementeren van een circuit breaker in Java met de Resilience4j-bibliotheek.

We configureren de breaker zodat deze opent wanneer meer dan 50% van de laatste 10 aanvragen zijn mislukt. Daarnaast blijft de breaker 10 seconden open (gevangen staat) voordat het weer half-open gaat om te testen of de service hersteld is.

```java
import io.github.resilience4j.circuitbreaker.CircuitBreaker;
import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import java.time.Duration;

// Configuratie van de circuit breaker
CircuitBreakerConfig config = CircuitBreakerConfig.custom()
    .failureRateThreshold(50) // circuit opent bij 50% fouten
    .waitDurationInOpenState(Duration.ofSeconds(10)) // open status duurt 10 seconden
    .slidingWindowSize(10) // evaluatie over de laatste 10 calls
    .build();

CircuitBreaker circuitBreaker = CircuitBreaker.of("calorieService", config);

// Wrap de API-call met de circuit breaker
Supplier<String> decoratedSupplier = CircuitBreaker
    .decorateSupplier(circuitBreaker, () -> callCalorieService());

try {
    String response = decoratedSupplier.get();
    System.out.println(response);
} catch (CallNotPermittedException ex) {
    System.out.println("Circuit is open, snelle fallback activeren");
}
```

Deze aanpak beschermt je applicatie tegen overbelasting en houdt de gebruikerservaring vloeiend.

---

# Best Practices voor het gebruik van circuit breakers

- **Monitor continu:** Houd open circuits, foutpercentages en latency in de gaten.
- **Fallback strategieën:** Implementeer alternatieven zoals cache of default antwoorden.
- **Stel thresholds slim in:** Pas parameters aan op basis van gedrag en gebruik.
- **Test met storingen:** Simuleer fouten om de werking te bevestigen.
- **Combineer met retries en time-outs:** Voor een complete foutafhandelingsstrategie.

*Een kleine tip:* probeer je circuit breaker te zien als die collega die zegt \"nee, ik neem even pauze\" voordat alles vastloopt. Zonder die pauzes zou je kantoor vol chaos zijn.

---

# Visualisatie van het circuit breaker-patroon

![Diagram van een circuit breaker patroon](/images/circuit-breaker-pattern-diagram.png "Circuit Breaker Patroon overzicht")

---

# Meer lezen
Voor een diepere duik in circuit breakers en microservices, raad ik aan de volgende bronnen te bekijken:

- [Resilience4j Documentation](https://resilience4j.readme.io/docs) – een uitgebreide library voor Java
- [Martin Fowler over Circuit Breakers](https://martinfowler.com/bliki/CircuitBreaker.html) – de bedenker van het patroon
- [Microservices.io Circuit Breaker Pattern](https://microservices.io/patterns/reliability/circuit-breaker.html) – praktische patronen en voorbeelden

---

# Conclusie
Het circuit breaker-patroon is essentieel voor robuuste microservice-architecturen. Door falende API-aanroepen slim af te handelen voorkom je cascades van fouten en houd je het systeem responsief.

Of je nu een beginnende ontwikkelaar bent of een doorgewinterde engineer: het implementeren van circuit breakers brengt je codebase een stuk stabieler en betrouwbaarder.

*Andy van Dongen*