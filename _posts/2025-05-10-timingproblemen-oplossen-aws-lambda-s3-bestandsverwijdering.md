# Timingproblemen oplossen bij AWS Lambda-bestandsverwijdering via S3 triggers

Heb je ooit een AWS Lambda geschreven die automatisch bestanden uit een S3 bucket zou verwijderen nadat ze geüpload zijn, maar lukt het niet om dit 100% betrouwbaar te doen? Vaak krijg je de Lambda netjes getriggerd door een `ObjectCreated` event, maar blijft het bestand toch staan of zie je onverwachte verwijdermarkers. Hoe zorg je er nou voor dat je Lambda robuust met die timing omgaat? In dit artikel duiken we in de oorzaak, concrete voorbeelden en best practices.

---

## Waarom lukt het verwijderen niet altijd?
AWS S3 versiebeheer (versioning) in combinatie met eventgedreven Lambda functies zorgt soms voor ware raadsels. Bestanden worden als het ware versies in de bucket, met verwijdermarkers als speciale objectversies. De event trigger (bijv. `s3:ObjectCreated`) kan al afgevuurd worden voordat het object definitief volledig gecommit is in S3. Als je direct probeert te verwijderen, kan je Lambda daardoor failen of slechts een verwijdermarker plaatsen die blijft hangen.

> Debugging zo’n Lambda is een beetje als in een donkere grot zoeken naar een verloren sleutel. Zonder goede timing gaat je licht ook uit.

---

## Stap-voor-stap aanpak voor betrouwbare verwijdering

1. **Wacht eventjes:** laat je Lambda na trigger 15 seconden pauzeren. Zo krijgt AWS de tijd het object en de events volledig te synchroniseren.

2. **Verwijder alle versies en verwijdermarkers:** als je versiebeheer aanstaat, verwijder dan niet alleen de huidige versie maar ook eventuele oudere versies en verwijdermarkers (delete markers).

3. **Gebruik retries of secundaire triggers:** plan bijvoorbeeld een EventBridge trigger die na een paar minuten nog eens checkt of het object echt weg is en anders alsnog verwijdert.

4. **Logging en monitoring:** log expliciet de verwijdering en gebruik CloudWatch alarms om fouten snel te ontdekken.

---

## Praktische voorbeeldcode in Python (AWS Lambda)

```python
import time
import boto3

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    for record in event['Records']:
        bucket_name = record['s3']['bucket']['name']
        object_key = record['s3']['object']['key']
        version_id = record['s3']['object'].get('versionId')

        if not object_key.endswith('.allowed_extension'):
            print(f"Verwijderen bestand {object_key} wegens extensie")
            time.sleep(15)  # vertraging om timing issues te vermijden
            try:
                s3_client.delete_object(Bucket=bucket_name, Key=object_key, VersionId=version_id)
                delete_all_versions(s3_client, bucket_name, object_key)
                print(f"Verwijderd {object_key} inclusief alle versies")
            except Exception as e:
                print(f"Fout bij verwijderen: {str(e)}")


def delete_all_versions(s3_client, bucket_name, object_key):
    try:
        versions = s3_client.list_object_versions(Bucket=bucket_name, Prefix=object_key)
        for version in versions.get('Versions', []):
            s3_client.delete_object(Bucket=bucket_name, Key=object_key, VersionId=version['VersionId'])
        for marker in versions.get('DeleteMarkers', []):
            s3_client.delete_object(Bucket=bucket_name, Key=object_key, VersionId=marker['VersionId'])
    except Exception as e:
        print(f"Fout bij verwijderen alle versies: {str(e)}")
```

---

## Visueel overzicht

![AWS Lambda en S3 Verwijdering Flow](/images/aws-lambda-s3-deletion.png "Flow diagram van AWS Lambda werkend met S3 objectversies en verwijdering")

---

## Samenvatting

- AWS S3 versiebeheer kan ervoor zorgen dat events al eerder komen dan het object 100% beschikbaar is.
- Voeg altijd een korte vertraging toe in je Lambda vóór verwijderen.
- Verwijder alle objectversies en delete markers.
- Bouw eventueel retry-mechanismen in.
- Monitor en log de verwijderingen goed.

Met deze stappen voorkom je race condities en houd je je bucket schoon en overzichtelijk.

---

## Referenties

- [AWS Lambda met S3 events (officiële documentatie)](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)
- [S3 Versioning en verwijdering](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)

---

Succes met je serverless adventures! En vergeet niet: geduld is echt een schone zaak, zelfs bij AWS.

---