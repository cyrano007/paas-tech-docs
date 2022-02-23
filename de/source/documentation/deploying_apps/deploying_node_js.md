## Stellen Sie eine Node.js-App bereit

In diesem Abschnitt wird beschrieben, wie Sie eine grundlegende Node.js-Anwendung für GOV.UK PaaS bereitstellen. Weitere Informationen finden Sie in den [Cloud Foundry-Tipps für Node.js-Anwendungen](https://docs.cloudfoundry.org/buildpacks/node/node-tips.html) .

> Wenn Ihre App einen [Sicherungsdienst](/deploying_services/#deploy-a-backing-or-routing-service) benötigt, muss sie mit einem der von PaaS unterstützten Dienste zusammenarbeiten können. In diesem Abschnitt finden Sie Anweisungen zum Bereitstellen sowohl von unterstützenden Dienst- als auch von nicht unterstützenden Dienst-Apps.

Diese Anweisungen gehen davon aus, dass Sie den [im](/get_started.html#get-started) Abschnitt „Erste Schritte“ erläuterten Einrichtungsprozess bereits durchgeführt haben.

Dies ist der Code für die Beispiel-App, die wir verwenden werden. Es ist ein einfacher Webserver, der mit einer „Hello World“-Nachricht antwortet.

```
const http = require('http');

const port = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, () => {
  console.log(`Server running on ${port}/`);
});
```

1. Speichern Sie den Code in einem neuen lokalen Verzeichnis als `example.js` .

2. Fügen Sie diese `manifest.yml` -Datei demselben Verzeichnis hinzu:

    ```
     ---
     applications:
     - name: my-node-app
       command: node example.js
       memory: 256M
       buildpacks:
       - nodejs_buildpack
    ```

    Ersetzen Sie `my-node-app` durch einen eindeutigen Namen für Ihre App. (Sie können `cf apps` verwenden, um bereits vorhandene Apps anzuzeigen).

    Die `memory` teilt dem PaaS mit, wie viel Speicher der App zugewiesen werden soll.

    Ein Buildpack bietet jegliche Framework- und Laufzeitunterstützung, die von einer App benötigt wird. Da die App in Node.js geschrieben ist, verwenden Sie in diesem Fall das `nodejs_buildpack` .

3. Schließen Sie eine npm-Datei „ `package.json` “ ein, um Abhängigkeiten anzugeben. Die Datei sollte auch einen `start` angeben, der zum Starten der App verwendet wird.

    Dies ist eine `package.json` -Datei für unsere Beispiel-App:

    ```
     {
       "name": "example",
       "version": "0.0.1",
       "author": "Demo",
       "engines": {
         "node": "6.11.x"
       }
     }
    ```

    Die `"engines"` -Werte geben die Versionen von Node.js und npm an, die PaaS zum Ausführen Ihrer App verwenden soll. Beachten Sie, dass ältere Versionen möglicherweise nicht verfügbar sind: Wenn Ihre Version nicht unterstützt wird, wird beim Versuch, die App hochzuladen und zu starten, eine Fehlermeldung angezeigt.

4. Sie können optional `npm install` ausführen, um Abhängigkeiten vorzuinstallieren, anstatt sie während des PaaS-Staging-Prozesses hinzuzufügen.

5. Führen Sie `cf push APPNAME` auf der obersten Ebene des Verzeichnisses aus, das alle Code- und Konfigurationsdateien enthält.

Wenn Sie die App hochladen möchten, ohne sie zu starten (z. B. wenn Sie einen PostgreSQL-Dienst erstellen müssen), führen Sie `cf push --no-start APPNAME` aus. Wenn Sie dann bereit sind, die App zu starten, führen Sie `cf start APPNAME` .

Weitere Informationen finden Sie in den [Cloud Foundry-Tipps für Node.js-Anwendungen](https://docs.cloudfoundry.org/buildpacks/node/node-tips.html) .

### PostgreSQL-Setup mit Node.js

Diese Anweisungen gelten für die Bereitstellung einer Node.js-App mit einer PostgreSQL-Datenbank und können auf andere unterstützende Dienste angewendet werden. Wenn Sie weitere Anleitungen zum Bereitstellen einer App mit [anderen unterstützten Unterstützungsdiensten](/deploying_services/#deploy-a-backing-or-routing-service) benötigen, kontaktieren Sie uns unter [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) .

Wenn Ihre App von einem Sicherungsdienst wie PostgreSQL abhängt, muss sie die Umgebungsvariable `VCAP_SERVICES` parsen, um erforderliche Details wie Dienst-URLs und Anmeldeinformationen abzurufen.

Sie müssen den Dienst erstellen und an Ihre Node.js-App binden, wie im [PostgreSQL](/deploying_services/postgresql/#postgresql) -Abschnitt beschrieben.

Sie können das [cfenv](https://www.npmjs.com/package/cfenv) -Modul verwenden, um beim Analysieren der Umgebungsvariablen zu helfen.

In Ihrer Datei `package.json` würden Sie `cfenv` als Abhängigkeit angeben:

```
    {
      // ...
      "dependencies": {
        "cfenv": "*",
        // ...
      }
    }
```

Dann können Sie in Ihrer App Konfigurationsinformationen für Sicherungsdienste abrufen. Dies ist ein Beispiel dafür, wie Sie eine Verbindung zu einem PostgreSQL-Dienst herstellen.

```
    var cfenv = require("cfenv");
    var pg = require('pg');
    var appEnv = cfenv.getAppEnv();
    var connectionString = appEnv.getServiceURL("SERVICE NAME");
    var client = new pg.Client({ connectionString, ssl: true });
    client.connect();
```

Ersetzen Sie „SERVICE NAME“ im obigen Code durch den genauen Namen des von Ihnen erstellten PostgreSQL-Dienstes. Die `getServiceURL` Funktion gibt eine Verbindungszeichenfolge zurück, die den Benutzernamen und das Kennwort enthält, die zum Herstellen einer Verbindung mit der Datenbank erforderlich sind.

Weitere Informationen finden Sie im:

- [Cloud Foundry-Community- `cf env` -Seite](https://github.com/cloudfoundry-community/node-cfenv/blob/master/README.md)
- die [Cloud Foundry-Dokumentation zu Umgebungsvariablen](https://docs.cloudfoundry.org/devguide/deploy-apps/environment-variable.html)

Sie sollten auch daran denken, Abhängigkeiten für alle Dienstbindungen in `package.json` .

```
{
  // ...
  "dependencies": {
    "pg": "*",
    // ...
  }
}
```

### Geben Sie eine Node.js-Version an

Sie sollten Cloud Foundry mitteilen, welche Version von Node.js Ihre App im Node.js-Buildpack verwendet.

Weitere Informationen finden Sie in der Dokumentation zu [Buildpack-Sprachversionsaktualisierungen](deploying_apps.html#buildpack-language-version-updates) .

Weitere Informationen finden Sie in der Cloud Foundry-Dokumentation zum [Angeben einer Node.js-Version](https://docs.cloudfoundry.org/buildpacks/node/index.html#runtime) .
