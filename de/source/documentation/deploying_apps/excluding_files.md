## Ausgenommen Dateien

Cloud Foundry unterstützt die Versionskontrolle nicht, daher stellt `cf push` den Arbeitsstatus aller Dateien bereit, die Sie in diesem Verzeichnis haben. In den meisten Fällen möchten Sie [Dateien ausschließen,](https://docs.cloudfoundry.org/devguide/deploy-apps/prepare-to-deploy.html#exclude) die von Git ignoriert werden. Führen Sie in Ihrem Projektverzeichnis aus

`ln -s .gitignore .cfignore`

und übertragen Sie die `.cfignore` in Ihr Repository. Lesen Sie jedoch weiter, wenn Sie ein fortgeschritteneres CF-Setup haben.

Ein paar wichtige Punkte zu `.cfignore` :

1. Wenn Sie ein erweitertes App-Setup haben und Apps mit einem anderen `path` als dem Projektstamm haben (von wo aus Sie `cf push` ausführen), benötigen Sie eine zusätzliche `.cfignore` -Datei, die sich in jedem App- `path` befindet;

2. Beachten Sie auch, dass eine fortgeschrittenere `.gitignore` Syntax, wie z. B. der Platzhalter `**` für rekursive Unterverzeichnisse, von `.cfignore` *nicht* unterstützt werden.
