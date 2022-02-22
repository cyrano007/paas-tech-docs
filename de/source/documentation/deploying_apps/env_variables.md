## Umgebungsvariablen

Sie müssen alle Konfigurationsinformationen für Ihre App als Umgebungsvariablen speichern.

Dies könnte beinhalten:

- Anmeldedaten für externe Dienste, die Ihre App verwendet, z. B. ein Twitter-Konto
- Werte, die bei jeder Bereitstellung der App variieren, z. B. die kanonische URL

Um die aktuellen Umgebungsvariablen einer App anzuzeigen, führen `cf env APP_NAME` .

Um eine Variable zu erstellen oder zu aktualisieren, führen Sie Folgendes aus:

```
cf set-env APP_NAME ENV_VAR_NAME ENV_VAR_VALUE
```

Wenn Sie eine bereits vorhandene App bereitstellen, sollten Sie in der Dokumentation der App nach Umgebungsvariablen suchen, die Sie festlegen müssen.

Wenn die App beispielsweise diese Anweisungen zum Bereitstellen in Heroku enthält:

```
heroku config:set VARIABLE=value
```

dann sollten Sie den entsprechenden Befehl mit `cf set-env` ausführen:

```
cf set-env APP_NAME VARIABLE value
```

### Vom System bereitgestellte Umgebungsvariablen

Vom System bereitgestellte Umgebungsvariablen informieren Sie über Konfigurationsdetails, die von PaaS verarbeitet werden, zum Beispiel:

- der Port, auf dem die App lauscht
- der maximale Arbeitsspeicher, den jede App-Instanz verwenden kann
- die externe IP-Adresse der App-Instanz

Versuchen Sie nicht, die Werte dieser vom System bereitgestellten Variablen über die Befehlszeile oder den Code Ihrer App zu ändern.

`VCAP_SERVICES` und `VCAP_APPLICATION` sind zwei wichtige Variablen für die Ersteinrichtung:

- `VCAP_SERVICES` enthält Details, einschließlich Anmeldeinformationen, aller Sicherungsdienste, die an die App gebunden sind

- `VCAP_APPLICATION` stellt Details der aktuell ausgeführten Anwendung bereit, beispielsweise die Sprachlaufzeitversion

Um die Werte der vom System bereitgestellten Variablen anzuzeigen, führen `cf env APP_NAME` .

Wenn Ihre App eine Verbindung zu einem Sicherungsdienst herstellt, muss Ihre App möglicherweise `VCAP_SERVICES` parsen, um die Anmeldeinformationen und andere Einstellungen für diesen Dienst abzurufen und die entsprechenden Umgebungsvariablen festzulegen.

Einige Buildpacks erledigen dies automatisch für Sie. Weitere Informationen finden Sie in den Bereitstellungsanweisungen für die Sprache oder das Framework, das Sie verwenden.

Hier ist ein Beispiel für die Struktur der in `VCAP_SERVICES` enthaltenen Informationen:

```
{
 "VCAP_SERVICES": {
  "postgres": [
   {
    "binding_name": null,
    "credentials": {
     "host": "rdsbroker-66ecd739-2e98-401a-9e45-15436165be06.c7uewwm9qebj.eu-west-1.rds.amazonaws.com",
     "jdbcuri": "jdbc:postgresql://rdsbroker-66ecd739-2e98-401a-9e45-17938165be06.c7uewwm9qebj.eu-west-1.rds.amazonaws.com:5432/DATABASE_NAME?password=PASSWORD\u0026ssl=true\u0026user=USERNAME",
     "name": "DATABASE_NAME",
     "password": "PASSWORD",
     "port": 5432,
     "uri": "postgres://USERNAME:PASSWORD@rdsbroker-66ecd739-2e98-401a-9e45-17938165be06.c7uewwm9qebj.eu-west-1.rds.amazonaws.com:5432/DATABASE_NAME",
     "username": "USERNAME"
    },
    "instance_name": "SERVICE_NAME",
    "label": "postgres",
    "name": "SERVICE_NAME",
    "plan": "PLAN",
    "provider": null,
    "syslog_drain_url": null,
    "tags": [
     "postgres",
     "relational"
    ],
    "volume_mounts": []
   }
  ]
 }
}
```

Weitere Informationen finden Sie in der [Cloud Foundry-Dokumentation zu Umgebungsvariablen](https://docs.cloudfoundry.org/devguide/deploy-apps/environment-variable.html) .
