## Stellen Sie eine Django-App bereit

In diesem Abschnitt wird erläutert, wie Sie eine App mithilfe des Django-Frameworks bereitstellen. Möglicherweise müssen Sie auch in der [Cloud Foundry-Dokumentation zum Python-Buildpack nachschlagen](https://docs.cloudfoundry.org/buildpacks/python/index.html) .

> Wenn Ihre App einen [Sicherungsdienst](/deploying_services/#deploy-a-backing-or-routing-service) benötigt, muss sie mit einem der von PaaS unterstützten Dienste zusammenarbeiten können. In diesem Abschnitt finden Sie Anweisungen zum Bereitstellen sowohl von unterstützenden Dienst- als auch von nicht unterstützenden Dienst-Apps.

Vor dem Bereitstellen einer Django-App müssen Sie:

- Melden Sie sich bei Ihrem [PaaS-Konto](get_started.html#get-an-account) an
- Richten Sie die [Cloud Foundry-Befehlszeile ein](get_started.html#set-up-command-line)
- [Zielen Sie auf](deploying_apps.html#set-a-target) den entsprechenden Raum

1. Legen Sie den Code für Ihre Django-App in einem lokalen Verzeichnis ab (z. B. indem Sie ihn aus der Versionskontrolle auschecken).

2. Wenn Sie Git verwenden, fügen Sie Ihrer `.gitignore` -Datei `*.pyc` und `local_settings.py` hinzu und schließen Sie dann [Dateien aus, die von Git ignoriert werden](/deploying_apps.html#excluding-files) , damit Cloud Foundry sie ebenfalls ignoriert.

3. Führen `cf buildpacks` in der Befehlszeile aus, um die Version des Python-Buildpacks zu überprüfen.

4. Rufen Sie die [Python-Buildpack-Seite auf GitHub](https://github.com/cloudfoundry/python-buildpack/releases) auf, um zu prüfen, welche Versionen von Python derzeit im Python-Buildpack enthalten sind.

5. Teilen Sie Cloud Foundry mit, welche Version von Python verwendet werden soll, indem Sie eine `runtime.txt` -Datei im Stammverzeichnis des lokalen Verzeichnisses erstellen. Dieser Schritt ist optional.

    Weitere Informationen finden Sie in der Dokumentation zu [Buildpack-Sprachversionsaktualisierungen](deploying_apps.html#buildpack-language-version-updates) .

    Weitere Informationen finden Sie in der [Cloud Foundry-Dokumentation zum Angeben einer Python-Version](https://docs.cloudfoundry.org/buildpacks/python/index.html#runtime) .

6. Stellen Sie sicher, dass Sie alle erforderlichen Module für Ihr Projekt in Ihrer virtuellen Umgebung installiert haben (einschließlich Django).

7. Generieren Sie eine „ `requirements.txt` “-Datei, falls Ihr Projekt noch keine hat, indem Sie „ `pip freeze > requirements.txt` im Stammverzeichnis des lokalen Ordners ausführen.

    Fügen Sie der Datei `requirements.txt` die folgenden Zeilen hinzu:

    ```
     whitenoise==5.2.0  # manages static assets
     waitress==2.0.0    # a pure python WSGI server that is a replacement for gunicorn
    ```

8. Weisen Sie Django an, `whitenoise` zu verwenden, um statische Dateien bereitzustellen. Fügen Sie in `settings.py` im Projektordner diese Zeile zu `MIDDLEWARE` , unmittelbar nach `middleware.security.SecurityMiddleware` :

    ```
     'whitenoise.middleware.WhiteNoiseMiddleware',
    ```

    Um die Dateien so bereitzustellen, dass sie komprimiert und zwischengespeichert werden können, fügen Sie die folgende Zeile zu `settings.py` hinzu:

    ```
     STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
    ```

    [Einzelheiten](http://whitenoise.evans.io/en/stable/django.html) finden Sie in der `whitenoise` Dokumentation.

9. Sie sollten Django nun mitteilen, wo nach statischen Dateien gesucht werden soll. Fügen Sie in `settings.py` im Projektordner diese Zeilen unter der `import os` Anweisung hinzu.

    ```
     PROJECT_ROOT = os.path.dirname(os.path.abspath(__file__))

     STATIC_ROOT = os.path.join(PROJECT_ROOT, 'static')
     STATIC_URL = '/static/'
    ```

    In diesem Fall weist die Variable STATIC_ROOT Django an, nach statischen Dateien in einem Verzeichnis namens `static` innerhalb des Projektordners zu suchen, und die Variable STATIC_URL legt den Pfad fest, in dem diese Dateien bereitgestellt werden.

    Je nachdem, wie statische Dateien in Ihrer App behandelt werden, müssen Sie diese Werte möglicherweise ändern.

    Wenn sich Ihre statischen Dateien in mehreren Ordnern befinden, müssen Sie möglicherweise die Variable STATICFILES_DIRS verwenden. Siehe die Django-Dokumentation für [vollständige Details zur Verwaltung statischer Dateien](https://docs.djangoproject.com/en/1.9/howto/static-files/) .

10. Erstellen Sie eine Datei namens `Procfile` im Stammverzeichnis Ihres lokalen Ordners und fügen Sie sie ein:

    ```
         web: python manage.py migrate && waitress-serve --port=$PORT PROJECTNAME.wsgi:application
    ```

    Die Procfile ist eine Möglichkeit, Befehle anzugeben, die beim Bereitstellen Ihrer App ausgeführt werden sollen. in diesem Fall für die Datenbankmigration.

    `PROJECTNAME` sollte durch den Namen Ihres WSGI-Moduls ersetzt werden. Standardmäßig ist dies derselbe wie der Name Ihres Projektmoduls, aber er kann mit der Umgebungsvariable DJANGO_SETTINGS_MODULE geändert werden. Bei Verwendung dieser Konfiguration werden alle Datenbankmigrationen automatisch angewendet.

11. Erstellen Sie eine `manifest.yml` -Datei im Stammverzeichnis Ihres lokalen Ordners.

    ```
     ---
     applications:
     - name: my-app
       memory: 512M
       buildpacks:
       - python_buildpack
    ```

    wobei `my-django-app` der Name ist, der für die App innerhalb von GOV.UK PaaS verwendet wird.

    Ersetzen Sie `my-django-app` durch einen eindeutigen Namen für Ihre App. (Sie können `cf apps` verwenden, um bereits vorhandene Apps anzuzeigen).

    Die `memory` teilt dem PaaS mit, wie viel Speicher der App zugewiesen werden soll.

12. Wenn Ihre App eine Datenbank benötigt, [erstellen Sie einen PostgreSQL-Backing-Service](/deploying_services/postgresql/#set-up-a-postgresql-service) und[binden Sie ihn an Ihre App](/deploying_services/postgresql/#bind-a-postgresql-service-to-your-app) . Sehen Sie sich dann den Abschnitt zur [Einrichtung von PostgreSQL](/deploying_apps.html#postgresql-setup-with-django) weiter unten an.

13. Gehen Sie wie folgt vor, um Ihre App zu pushen:

    `cf push APPNAME`

    aus dem Verzeichnis, das alle Code- und Konfigurationsdateien enthält.

    Wenn Sie die App hochladen möchten, ohne sie zu starten (z. B. wenn Sie einen PostgreSQL-Dienst erstellen müssen), führen Sie `cf push --no-start APPNAME` aus. Wenn Sie dann bereit sind, die App zu starten, führen Sie `cf start APPNAME` .

Sie können Ihre App jetzt in Ihrer [App-Domain](/orgs_spaces_users.html#regions) anzeigen.

### PostgreSQL-Setup mit Django

Diese Anweisungen gelten für die Bereitstellung einer Django-App mit einer PostgreSQL-Datenbank und können auf andere unterstützende Dienste angewendet werden. Wenn Sie weitere Anleitungen zum Bereitstellen einer App mit [anderen unterstützten Unterstützungsdiensten](/deploying_services/#deploy-a-backing-or-routing-service) benötigen, kontaktieren Sie uns unter [gov-uk-paas-support@digital.cabinet-office.gov.uk](mailto:gov-uk-paas-support@digital.cabinet-office.gov.uk) .

Fügen Sie diese Zeilen zu Ihrer `requirements.txt` :

```
psycopg2==2.6.2 #installs the postgres driver
dj-database-url==0.3.0 #grabs environment variables and dumps them into a Django settings file
```

Stellen Sie in Ihrer Datei `settings.py` sicher, dass Sie das Paket `dj_database_url` importieren, das wir der Datei `requirements.txt` oben hinzugefügt haben:

```
    import dj_database_url
```

Dieses Paket analysiert automatisch die Umgebungsvariable `VCAP_SERVICES` und setzt DATABASE_URL auf die erste gefundene Datenbank.

Dann müssen Sie eine `DATABASES` Einstellung hinzufügen. Am besten fügen Sie dies der Datei `settings.py` hinzu.

```
    DATABASES = {}
    DATABASES['default'] =  dj_database_url.config()
```

Ihre `local_settings.py` -Datei überschreibt dies, wenn Sie lokal arbeiten.

Die im obigen Abschnitt bereitgestellte `Procfile` Konfiguration wendet Datenbankmigrationen automatisch an.
