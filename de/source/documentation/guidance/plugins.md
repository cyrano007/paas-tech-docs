## Nützliche Cloud Foundry-Plugins

Es gibt einige von der Community bereitgestellte Plugins, die in verschiedenen Situationen nützlich sind:

- Mit dem [App AutoScaler-Plug-](https://plugins.cloudfoundry.org/#app-autoscaler-plugin) in können Sie die Autoscaling- [Regeln](https://docs.cloud.service.gov.uk/managing_apps.html#autoscaling) für Ihre Anwendungen über Ihre Befehlszeilenschnittstelle (CLI) festlegen und anpassen.
- Mit dem [Buildpack-Nutzungs-Plugin](https://github.com/csterwa/cf_buildpacks_usage_cmd) können Sie überprüfen, ob Sie ein Buildpack verwenden, das betroffen ist, wenn das GOV.UK Platform as a Service (PaaS)-Team die Standard-Buildpacks aktualisiert
- Das [Log Cache CLI](https://github.com/cloudfoundry/log-cache-cli#installing-plugin) -Plug-in bietet Ihnen eine erweiterte Protokollfilterung aus dem Protokollpuffer, indem es Ihnen ermöglicht, nach Zeit, Typ oder Protokollklasse zu filtern
- Das [Open-Plugin](https://plugins.cloudfoundry.org/#open) gibt Ihnen die Links zu Ihren Anwendungen von Ihrer Befehlszeile aus und öffnet Ihren Browser direkt zu ihnen
- [Das CLI-Plug-in Route Lookup](https://github.com/18F/cf-route-lookup) bestimmt, welche Route mit welcher App verbunden ist
- Mit dem [Top-Plugin](https://plugins.cloudfoundry.org/#top) können Sie sehen, wie Ihre Anwendungen innerhalb von GOV.UK PaaS funktionieren, und warnen, wenn Ihnen der Arbeitsspeicher oder die CPU ausgehen

Die Open-Source-Community von Cloud Foundry pflegt diese Plugins, und wir sind nicht für deren Aktualisierung verantwortlich.

Andere [Cloud Foundry-Plug-ins](https://plugins.cloudfoundry.org/) sind ebenfalls verfügbar.

### Verwenden des Conduit-Plugins

Einige PaaS-Backing-Dienste von GOV.UK (PostgreSQL, MySQL und Redis) akzeptieren nur Verbindungen von PaaS-Apps von GOV.UK. Sie müssen sich über eine GOV.UK PaaS-App verbinden, um von Ihrem lokalen Computer aus auf einen dieser Sicherungsdienste zugreifen zu können. Wir haben das [Conduit-](https://plugins.cloudfoundry.org/#conduit) Plug-in erstellt und einige [Anleitungen zur Verwendung des Conduit-Plug-](/guidance.html#conduit) ins geschrieben, um diesen Prozess zu vereinfachen.

Mit Conduit können Sie von Ihrem lokalen System aus eine Verbindung zu Ihren Remote-Backing-Service-Instanzen herstellen. Auf diese Weise können Sie das Standardtool für Ihren Unterstützungsdienst verwenden, z. B. [`psql`](https://www.postgresql.org/docs/12/app-psql.html) für PostgreSQL oder [`mysql`](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) für MySQL CLI-Tools, [um Sicherungen zu erstellen](https://docs.cloud.service.gov.uk/deploying_services/postgresql/#paas-to-paas) und [Ihre Unterstützungsdienste abzufragen](https://docs.cloud.service.gov.uk/deploying_services/postgresql/#connect-to-a-postgresql-service-from-your-local-machine) .

### Stellen Sie von Ihrem lokalen Computer aus eine Verbindung zu einem Sicherungsdienst her

Um dieses Plugin zu installieren, führen Sie den folgenden Code von der Befehlszeile aus:

```
cf install-plugin conduit
```

Sie sehen die folgende Warnung:

```

Attention: Plugins are binaries written by potentially untrusted authors.
Install and use plugins at your own risk.
Do you want to uninstall the existing plugin and install conduit VERSION? [yN]:

```

Wählen Sie „y“, um mit der Installation fortzufahren.

Führen Sie nach Abschluss der Installation des Plugins den folgenden Code aus, um Anmeldeinformationen und eine lokale Adresse abzurufen, über die Sie auf den Sicherungsdienst zugreifen können:

```
cf conduit SERVICE_NAME
```

`SERVICE_NAME` sollte ein eindeutiger beschreibender Name für diese Dienstinstanz sein.

Um sich mit den von Ihnen gewählten Tools mit dem Sicherungsdienst zu verbinden, verwenden Sie den Benutzernamen, das Passwort, den Port und den URI, die in der Ausgabe von Conduit angegeben sind.

```
The following services are ready for you to connect to:

* service: SERVICE_NAME (BACKING_SERVICE_TYPE)
  host: 127.0.0.1
  jdbcuri: jdbc:postgresql://127.0.0.1:PORT/A_RESOURCE?password=PASSWORD&user=USERNAME
  name: A_RESOURCE
  password: PASSWORD
  port: PORT
  uri: postgres://USERNAME:PASSOWRD@127.0.0.1:PORT/A_RESOURCE
  username: USERNAME

```

Woher:

- BACKING_SERVICE_TYPE ist der Typ des Sicherungsdienstes (z. B. PostgreSQL)
- USERNAME ist der Benutzername für die Verbindung mit dem Dienst
- PASSWORD ist das Passwort für den Benutzer
- PORT ist der Port, an dem eine Verbindung hergestellt werden soll
- A_RESOURCE ist der Name der Ressource, wie er vom Dienstanbieter angegeben wird (z. B. der Datenbankname).

Führen Sie `cf conduit --help` aus, um weitere Optionen zu erhalten, und lesen Sie die [Conduit-README](https://github.com/alphagov/paas-cf-conduit/blob/master/README.md) -Datei für weitere Informationen zur Verwendung des Plugins.

Spezifische Informationen zur Verwendung von Conduit mit diesen unterstützenden Diensten finden Sie in der [PostgreSQL-Dokumentation](https://docs.cloud.service.gov.uk/deploying_services/postgresql/#connect-to-a-postgresql-service-from-your-local-machine) und der [MySQL-Dokumentation](https://docs.cloud.service.gov.uk/deploying_services/mysql/#connect-to-a-mysql-service-from-your-local-machine) .

## Löschen von Conduit-Anwendungen

Die Conduit-Anwendung erstellt und stellt GOV.UK PaaS-Apps mit Namen wie `__conduit_RANDOM_STRING__` (z. B. `__conduit_aeganmtr__` ) in Ihrem Bereich bereit. Conduit sollte die Apps am Ende der Sitzung von selbst aufräumen, aber wenn dies nicht der Fall ist, können Sie sie sicher selbst löschen. Stellen Sie vor dem Löschen sicher, dass niemand anderes in Ihrem Team Conduit verwendet, um eine Verbindung mit derselben Datenbank herzustellen.
