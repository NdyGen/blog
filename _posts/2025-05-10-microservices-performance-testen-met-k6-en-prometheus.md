# Microservices performance testen: bottlenecks en load testing met k6 en Prometheus

Microservices-architecturen zijn geweldig flexibel, schaalbaar en modulair. Maar met grote voordelen komt ook de uitdaging om de performance goed te monitoren en te testen. Zeker als je afhankelijk bent van externe systemen met beperking in verwerkingssnelheid, is het cruciaal die bottlenecks vroegtijdig te spotten en je load tests slim op te zetten.

Je hoort het vaak: "Debuggen van die legacy monoliet is als grotten verkennen zonder kaart". Hetzelfde geldt ook zeker voor microservices: je hebt veel bewegende delen. Hoe weet je waar je performancevraagstukken ontstaan? En hoe simuleer je realistische workloads zodat je het echte gedrag kan voorspellen?

## Scenario: microservices met externe bottleneck

Stel je hebt een architectuur waarbij:
- Service A biedt endpoints om shipments aan te maken en te bewerken
- Service B ontvangt calls van Service A en heeft extra endpoints voor inventory en tracking
- Service C verwerkt berichten van Service B via Kafka Topic X
- Service C stuurt de verwerkte data door naar een extern systeem
- Dat externe systeem heeft limiet: maximaal 10 berichten per seconde (ongeacht onze load)

Hierbij is de bottleneck extern en onafhankelijk van ons. Dit scenario komt veel voor. Hoe test je dan de performance van je eigen services realistisch?

---

## Monitoring en load testing tools: k6 & Prometheus

**k6** is een open source tool om load tests te schrijven in JavaScript. Je kan met k6 HTTP requests nabootsen, verschillende gebruikers simuleren, acceptatiecriteria (thresholds) instellen en uitgebreide rapporten genereren.

**Prometheus** is een sterke monitoringtool met metrics verzameling, queryfunctionaliteit (PromQL) en alerting-mechanismen. Het integreert makkelijk met microservices. Door k6 en Prometheus samen te gebruiken krijg je live inzicht in de impact van je load tests op verschillende systeemonderdelen.

![Placeholder afbeelding k6 en Prometheus integratie](/images/1056.png "k6 & Prometheus integratie voor microservices")

---

## Performance testen - aandachtspunten

### 1. Meet endpoints en afhankelijkheden
Documenteer response tijden van elk endpoint en downstream services & databases.

### 2. Imiteer realistische load
Test het systeem met verschillende gebruikersaantallen, request types en wachttijden. 

### 3. Simuleer bottlenecks
Beperk de verwerkingssnelheid van je contacten, simuleer bijvoorbeeld het externe systeem met maximaal 10 berichten/sec.

### 4. Pieken versus stress
Piekbelasting test tijdelijk hoge loads, stress test gaat door tot belastbaarheid en uitval.

### 5. Controleren resources
CPU, geheugen, I/O en netwerkverbruik geven aanwijzingen over bottlenecks.

### 6. Wachtrijen en buffers
Monitor Kafka topic backlog en άλλα message queues om bottlenecks te signaleren.

### 7. Centraal metrics verzamelen
Gebruik uniforme metrics via OpenTelemetry of Prometheus exporters.


---

## Praktische tips voor je testplan

- Begin met baseline test (weinig load) en meet normale performance.
- Verhoog het aantal gebruikers stap voor stap en check latency en fouten.
- Test met bottleneck-simulators voor downstream systemen.
- Maak gebruik van dashboards (bv. Grafana) voor realtime inzicht.
- Zet alerts in op specifieke thresholds zoals wachtrijgroei.
- Documenteer je testcases en scripts goed voor reproduceerbaarheid.

### Voorbeeld k6 script
```js
import http from 'k6/http';
import { sleep } from 'k6';

export let options = {
  stages: [
    { duration: '1m', target: 20 },
    { duration: '3m', target: 50 },
    { duration: '5m', target: 10 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
  },
};

export default function() {
  http.get('http://serviceA/api/shipments');
  sleep(1);
}
```

---

## Samenvatting

Microservices performance testen vraagt om een combinatie van load testing (zoals met k6) en metrics monitoring (zoals Prometheus). 
Je moet externe en interne bottlenecks simuleren en realistische workloads aanzetten. 
Met een gedegen testplan voorkom je onverwachte problemen in productie.

Zoals een senior developer ooit zei: "Als je microservices niet onder echte chaos test, is het net alsof je skydivet zonder parachutecheck!"

Happy load testing!

---

**Bronnen:**
- [k6 docs](https://k6.io/docs/)
- [Prometheus intro](https://prometheus.io/docs/introduction/overview/)
- [Grafana](https://grafana.com/)

---