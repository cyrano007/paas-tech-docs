## Stellen Sie ein Docker-Image bereit

In diesem Abschnitt wird erläutert, wie Sie eine App aus einem Docker-Image bereitstellen. Die Rolle des Docker-Images besteht darin, als Verpackungsformat zu dienen, und die Anforderungen an die App und ihre Laufzeitumgebung sind dieselben wie für Apps, die mithilfe von Buildpacks bereitgestellt werden. Die Konfiguration mit manifest.yml bleibt ebenfalls gleich, obwohl Sie buildpack nicht angeben sollten. Wenn Sie dies tun, wird es ignoriert, was bei jedem, der das Manifest liest, zu Verwirrung führen kann. Unsere Plattform unterstützt derzeit die Bereitstellung von:

- Docker-Hub
- [Docker Trusted Registry](https://docs.docker.com/datacenter/dtr/2.1/guides/) [externe Seite]-Server
- Docker-Registrierungen, die eine Authentifizierung erfordern

Das Pushen lokaler Docker-Images wird nicht unterstützt.

So stellen Sie eine App mit einem in Docker Hub gespeicherten Docker-Image mit CF CLI bereit:

`cf push myapp --docker-image dockerhuborg/app_image:tag`

So stellen Sie eine App mit einem Docker-Image bereit, das in Docker Trusted Registry (DTR) mit CF CLI gespeichert ist:

`cf push myapp --docker-image MY-PRIVATE-REGISTRY.DOMAIN:5000/image/name:tag`

Um eine App mit einem Docker-Image bereitzustellen, das in einer Registrierung gespeichert ist, die eine Authentifizierung erfordert, befolgen Sie die [Anweisungen in der Cloud Foundry Docker-Image-Dokumentation](https://docs.cloudfoundry.org/devguide/deploy-apps/push-docker.html#private-repo) .

Möglicherweise stellen Sie fest, dass der Staging-Prozess beim Bereitstellen von einem Docker-Image vereinfacht wird, da die App nicht mithilfe eines Buildpacks aus dem gepushten Code erstellt werden muss. Außerdem wird das Buildpack für Apps, die von einem Docker-Image bereitgestellt werden, in den App-Informationen als `unknown` gemeldet ( `cf app <myapp>` ).

Es gibt ein paar weitere Einzelheiten zur Unterstützung von Docker-Images in Cloud Foundry:

- Der lokale Speicher in jeder Containerinstanz ist ephemer. Jeder Containerstart beginnt mit einem unveränderten Image.
- Der Datenverkehr wird an den niedrigsten exponierten Port geleitet (durch den Befehl `EXPOSE` in Dockerfile). Die Umgebungsvariable PORT wird ebenfalls auf diese Nummer gesetzt.
- Buildpack-basierte Apps verwenden bash, um den App-Startbefehl auszuführen ( `bash -c <command>` ), aber auf Docker-Images basierende Apps verwenden sh ( `sh -c <command>` ), da in Ihrem Docker-Image nicht davon ausgegangen werden kann, dass Bash vorhanden ist.
- Der privilegierte Containermodus wird nicht unterstützt. Funktionen, die davon abhängen, funktionieren nicht.
- Es werden nur Registrierungen unterstützt, die Docker Registry API V2 verwenden.
- Bilder werden basierend auf Tags zwischengespeichert. Die Verwendung eines Tags wie „ `latest` “ bei schnellen Iterationen kann zu Fehlern führen. Um dies zu verhindern, sollten Tags eindeutig sein. Zum Beispiel die Zeit, zu der das Bild erstellt wurde, oder ein Git-Commit-Hash.

Ihre Verantwortlichkeiten ändern sich, wenn Sie ein Docker-Image anstelle eines Buildpacks verwenden. Weitere Informationen finden Sie in den [Leitlinien zum Verantwortungsmodell](/guidance.html#responsibility-model) .

### Verfügbarkeitsanforderungen für die Docker-Registrierung

Das Bereitstellen, Skalieren, Neustarten oder erneute Staging einer App mit einem Docker-Image hängt davon ab, ob die Docker-Registrierung verfügbar ist, da diese Aktionen das Abrufen des Images aus der Registrierung erfordern.

Beachten Sie, dass Cloud Foundry Ihre Anwendung aufgrund von Umständen neu starten kann, die außerhalb Ihrer direkten Kontrolle liegen, wie z. B. Plattformskalierung oder -bereitstellungen oder Ausfall der zugrunde liegenden Hardwareplattform.

Wenn Cloud Foundry versucht, das Image aus der Registrierung abzurufen, obwohl die Registrierung nicht verfügbar ist, kann Ihre Anwendungsinstanz nicht gestartet werden, und Sie erhalten die Meldung `Failed to create container` werden. Sobald die Docker-Registrierung wieder verfügbar ist, können Sie Ihre App nach Bedarf erneut bereitstellen, neu starten oder erneut bereitstellen.

### Anwenden von Sicherheitsupdates

Wenn Ihre App mit einem Standard-Buildpack erstellt wird, sind Sie nur für den Code Ihrer App verantwortlich. Die Plattform stellt sowohl die Buildpacks als auch ein sicheres Root-Dateisystem ( `cflinuxfs3` ) bereit. Buildpack-basierte Apps haben niemals Root-Zugriff in ihrem Container, was eine weitere Sicherheitsebene hinzufügt. Wir wenden regelmäßig Sicherheits- und Funktionsupdates auf die Buildpacks, Container-Root-FS und die Plattform selbst an.

Wenn Ihre App aus dem Docker-Image bereitgestellt wird, verlagert sich die Verantwortung für das Root-Dateisystem und für den vollständigen Build Ihrer App auf Sie. Sie müssen sicherstellen, dass geeignete Maßnahmen ergriffen werden, um Ihre App und Ihr Root-Image sicher und mit den neuesten Sicherheitsupdates zu machen. Wir empfehlen Ihnen, eine der wichtigsten und gut unterstützten Linux-Distributionen als Basiscontainer zu verwenden. Auf diese Weise können Sie ein Image mit den neuesten Sicherheitspatches erstellen.
