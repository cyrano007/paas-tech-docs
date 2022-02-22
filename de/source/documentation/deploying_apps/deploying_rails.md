## Stellen Sie eine Ruby on Rails-App bereit

In diesem Abschnitt werden die minimalen Schritte zum Bereitstellen einer einfachen Rails-App erläutert. Weitere Informationen finden Sie im [Cloud Foundry-Leitfaden zur Bereitstellung von Ruby on Rails-Apps](https://docs.cloudfoundry.org/buildpacks/ruby/gsg-ror.html) .

> Wenn Ihre App einen [Sicherungsdienst](/deploying_services/#deploy-a-backing-or-routing-service) benötigt, muss sie mit einem der von PaaS unterstützten Dienste zusammenarbeiten können. In diesem Abschnitt finden Sie Anweisungen zum Bereitstellen sowohl von unterstützenden Dienst- als auch von nicht unterstützenden Dienst-Apps.

Bei diesen Schritten wird davon ausgegangen, dass Sie den [im](/get_started.html#get-started) Abschnitt „Erste Schritte“ erläuterten Einrichtungsprozess bereits durchgeführt haben.

Wenn Sie eine App bereitstellen, müssen Sie eine Kombination aus einer Organisation und einem Bereich auswählen. Dies wird als [**Ziel**](/deploying_apps.html#set-a-target) bezeichnet.

Wir haben einen `sandbox` -Bereich bereitgestellt, den Sie zum Lernen über PaaS nutzen können. Möglicherweise möchten Sie die Sandbox während des Testens ansprechen, indem Sie Folgendes ausführen:

`cf target -s sandbox`

Es ist auch wichtig zu wissen, dass das Original ersetzt wird, wenn Sie eine App bereitstellen, die denselben Namen und dasselbe Ziel wie eine vorhandene App verwendet. Wenn Sie sich nicht sicher sind, wo Sie Ihre App bereitstellen sollen, wenden Sie sich an den Rest Ihres Teams.

### Bereitstellen einer App

So stellen Sie eine Rails-App bereit, die keinen Sicherungsdienst benötigt:

1. Legen Sie den Code für Ihre Rails-App in einem lokalen Verzeichnis ab (z. B. indem Sie ihn aus der Versionskontrolle auschecken).

2. [Von Git ignorierte Dateien ausschließen](/deploying_apps.html#excluding-files) .

3. Wenn Sie Rails 4 verwenden, [fügen Sie das `rails_12factor`](https://github.com/heroku/rails_12factor#install) für eine bessere Protokollierung hinzu. Rails 5 hat diese Funktionalität standardmäßig eingebaut.

4. Erstellen Sie eine manifest.yml-Datei in dem Ordner, in dem Sie Ihre App ausgecheckt haben.

    ```
     ---
     applications:
     - name: my-rails-app
       memory: 256M
       buildpacks:
       - ruby_buildpack
    ```

    Ersetzen Sie `my-rails-app` durch einen eindeutigen Namen für Ihre App. (Sie können `cf apps` verwenden, um bereits vorhandene Apps anzuzeigen).

    Die `memory` teilt dem PaaS mit, wie viel Speicher der App zugewiesen werden soll.

    Ein Buildpack bietet jegliche Framework- und Laufzeitunterstützung, die von einer App benötigt wird. Da die App in diesem Fall in Ruby geschrieben ist, verwenden Sie das `ruby_buildpack` .

5. Laden Sie die Anwendung hoch und starten Sie sie, indem Sie Folgendes ausführen:

    ```
    cf push APPNAME
    ```

    aus dem Verzeichnis, das alle Code- und Konfigurationsdateien für Ihre App enthält.

    Wenn Sie nach dem Befehl `cf push` keinen Namen für die App angeben, wird der Name aus der Manifestdatei verwendet.

6. Legen Sie alle zusätzlichen [Umgebungsvariablen fest,](/deploying_apps.html#environment-variables) die von Ihrer App benötigt werden. Zum Beispiel:

    ```
    cf set-env APPNAME VARIABLE `value`
    ```

    wobei VARIABLE ein eindeutiger Name für die Variable und `value` der festzulegende Wert ist.

Ihre App sollte jetzt unter Ihrer [App-Domain verfügbar](/orgs_spaces_users.html#regions) sein.

Informationen zu einer Produktions-App finden Sie in der [Dokumentation zum Bereitstellen einer App in der Produktion](/deploying_apps.html#deploy-an-app-to-production) .

### Bereitstellung mit einer PostgreSQL-Datenbank

Diese Anweisungen gelten für die Bereitstellung einer Rails-App mit einer PostgreSQL-Datenbank und können auf andere unterstützende Dienste angewendet werden. Wenn Sie weitere Anleitungen zum Bereitstellen einer App mit [anderen unterstützten Unterstützungsdiensten](/deploying_services/#deploy-a-backing-or-routing-service) benötigen, kontaktieren Sie uns unter [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) .

Das Cloud Foundry-Buildpack für Ruby ruft automatisch die Details des ersten verfügbaren PostgreSQL-Dienstes aus der `VCAP_SERVICES` Umgebungsvariable ab und legt die Ruby-Umgebungsvariable `DATABASE_URL` entsprechend fest. Stellen Sie sicher, dass Ihre App so konfiguriert ist, dass sie `DATABASE_URL` verwendet, um ihre Datenbankkonfiguration festzulegen, wenn sie in PaaS bereitgestellt wird.

1. Legen Sie den Code für Ihre Rails-App in einem lokalen Verzeichnis ab (z. B. indem Sie ihn aus der Versionskontrolle auschecken).

2. Wenn Sie Git verwenden, möchten Sie möglicherweise [Dateien ausschließen, die von Git ignoriert werden](/deploying_apps.html#excluding-files) .

3. Wenn Sie Rails 4 verwenden, [fügen Sie das `rails_12factor`](https://github.com/heroku/rails_12factor#install) für eine bessere Protokollierung hinzu. Rails 5 hat diese Funktionalität standardmäßig eingebaut.

4. Erstellen Sie eine manifest.yml-Datei in dem Verzeichnis, in dem Sie Ihre App ausgecheckt haben.

    ```
     ---
     applications:
     - name: my-rails-app
       memory: 256M
       buildpacks:
       - ruby_buildpack
    ```

    Ersetzen Sie `my-rails-app` durch einen eindeutigen Namen für Ihre App. (Sie können `cf apps` verwenden, um bereits vorhandene Apps anzuzeigen).

    Die `memory` teilt dem PaaS mit, wie viel Speicher der App zugewiesen werden soll.

    Ein Buildpack bietet jegliche Framework- und Laufzeitunterstützung, die von einer App benötigt wird. Da die App in diesem Fall in Ruby geschrieben ist, verwenden Sie das `ruby_buildpack` .

5. Laden Sie die App hoch, ohne sie mit diesem Befehl zu starten:

    ```
    cf push --no-start APPNAME
    ```

    aus dem Verzeichnis, das alle Code- und Konfigurationsdateien für Ihre App enthält.

    Wenn Sie nach dem Befehl `cf push` keinen Namen für die App angeben, wird der Name aus der Manifestdatei verwendet.

6. Legen Sie alle zusätzlichen [Umgebungsvariablen fest,](/deploying_apps.html#environment-variables) die von Ihrer App benötigt werden. Zum Beispiel:

    ```
    cf set-env APPNAME VARIABLE `value`
    ```

    wobei VARIABLE ein eindeutiger Name für die Variable und `value` der festzulegende Wert ist.

7. [Erstellen Sie einen PostgreSQL-Unterstützungsdienst (falls erforderlich)](/deploying_services/postgresql/#set-up-a-postgresql-service) und[binden Sie ihn an Ihre App](/deploying_services/postgresql/#bind-a-postgresql-service-to-your-app) .

    Um die Rails-Unterstützung für Datenbankmigrationen zu aktivieren, möchten Sie möglicherweise eine `Procfile` im selben Verzeichnis wie Ihre `manifest.yml` und `Gemfile` . Die `Procfile` ist eine Möglichkeit, Befehle anzugeben, die beim Bereitstellen Ihrer App ausgeführt werden sollen.

    Dies ist ein minimales Beispiel des `Procfile` Inhalts für *Rails 5.0* :

    ```
    web: rake db:migrate && bin/rails server
    ```

8. Starten Sie Ihre App, indem Sie Folgendes ausführen:

    ```
    cf start APPNAME
    ```

Ihre App sollte jetzt unter Ihrer [App-Domain verfügbar](/orgs_spaces_users.html#regions) sein.

Informationen zu einer Produktions-App finden Sie in der [Dokumentation zum Bereitstellen einer App in der Produktion](/deploying_apps.html#deploy-an-app-to-production) .

### Geben Sie eine Ruby-Version an

Sie sollten Cloud Foundry mitteilen, welche Ruby-Version Ihre App im Ruby-Buildpack verwendet.

Weitere Informationen finden Sie in der Dokumentation zu [Buildpack-Sprachversionsaktualisierungen](deploying_apps.html#buildpack-language-version-updates) .

Weitere Informationen finden Sie in der [Cloud Foundry-Dokumentation zum Angeben einer Ruby-Version](https://docs.cloudfoundry.org/buildpacks/ruby/index.html#runtime) .

### Webserver

Standardmäßig führt das Cloud Foundry Ruby-Buildpack den [`bin/rails server`](https://github.com/cloudfoundry/ruby-buildpack/blob/1f0ac3ce10866390d161c3f27e71d64890859454/lib/language_pack/rails4.rb#L27) aus, um die Anwendung hochzufahren. In Rails 4 und darunter wird WEBrick als Webserver verwendet. In Rails 5 und höher ist der Standardwert [puma](http://guides.rubyonrails.org/getting_started.html#starting-up-the-web-server) .

Möglicherweise möchten Sie in der Produktion einen anderen Webserver verwenden. Weitere Informationen finden Sie in der [Cloud Foundry-Dokumentation zum Konfigurieren eines Produktionsservers](https://docs.cloudfoundry.org/buildpacks/prod-server.html) .

### Fehlerbehebung bei der Asset-Vorkompilierung

Standardmäßig führt das Rails-Buildpack während der Staging-Phase eine Asset-Vorkompilierung durch. Dies ist für die meisten Rails-Apps in Ordnung, funktioniert jedoch nicht für diejenigen, die während der Asset-Kompilierung eine Verbindung zu Diensten (z. B. der Datenbank) herstellen müssen. (Beispiel siehe [MyUSA Ausgabe Nr. 636](https://github.com/18F/myusa/issues/636) )

Dafür gibt es mehrere mögliche Lösungen. Weitere Informationen finden Sie in der [Cloud Foundry-Dokumentation zur Asset-Vorkompilierung](https://docs.cloudfoundry.org/buildpacks/ruby/ruby-tips.html#precompile) .
