## Isolationssegmente

### Was Isolationssegmente sind

Zellen sind die zugrunde liegenden Server, die Ihre Apps erstellen und ausführen. GOV.UK PaaS plant Ihre Apps auf einer großen Anzahl identischer Zellen.

GOV.UK PaaS hat andere Zellen, die sich geringfügig von den regulären Zellen unterscheiden, auf denen Ihre Apps laufen. Diese Zellen werden Isolationssegmente genannt.

Sie können Isolationssegmente für einen Bereich oder für eine Organisation angeben.

#### Apps in einem Raum

Wenn Sie ein Isolationssegment für einen Bereich angeben, werden alle Apps in diesem Bereich auf den Zellen des Isolationssegments und nicht auf den normalen Zellen ausgeführt.

#### Apps innerhalb einer Organisation

Wenn Sie ein Isolationssegment für eine Organisation angeben, werden alle Apps in dieser Organisation auf den Zellen des Isolationssegments und nicht auf den normalen Zellen ausgeführt.

### Welche Isolationssegmente sind verfügbar

Wir bieten derzeit ein Isolationssegment an:

- `egress-restricted-1`

#### Ausgang eingeschränkt

Wenn Ihre Apps in einem Isolationssegment mit Ausgangsbeschränkung ausgeführt werden, können sie:

- Datenverkehr aus dem Internet bedienen (wenn Sie eine [öffentliche Route](/deploying_apps.html#names-routes-and-domains) abbilden)
- Verkehr intern bedienen (wenn Sie eine [private Route](/deploying_apps.html#deploying-private-apps) abbilden)
- mit anderen Apps innerhalb der Plattform sprechen (wenn Sie [Netzwerkrichtlinien](/deploying_apps.html#create-a-network-policy) einrichten)

Ihre Apps können jedoch nicht:

- mit dem Internet sprechen
- externe DNS-Anfragen stellen

### Wie Sie Isolationssegmente verwenden können

Wenn wir ein Isolationssegment für Ihre Organisation aktiviert haben und Sie ein Organisationsmanager sind, können Sie mithilfe des Isolationssegments ändern, wie Ihre Apps geplant werden.

Beispielsweise erstellen die folgenden Befehle einen neuen Bereich und stellen sicher, dass jede App, die Sie in diesen Bereich verschieben, auf einer Zelle mit Ausgangsbeschränkungen ausgeführt wird:

```
cf create-space my-locked-down-space
cf set-space-isolation-segment my-locked-down-space egress-restricted-1
```

Kontaktieren Sie uns unter [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) , wenn Sie möchten, dass wir ein Isolationssegment für Ihre Organisation aktivieren.
