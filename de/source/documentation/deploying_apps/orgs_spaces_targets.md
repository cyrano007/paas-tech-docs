## Organisationen, Räume &amp; Ziele

### Organisationen

Ihr Mandantenkonto gehört zu mindestens einer **Organisation** (kurz „org“) innerhalb des PaaS. Dies repräsentiert normalerweise die reale Organisation, Abteilung oder das Team, für das Sie arbeiten. Die Mandantenkonten Ihrer Kollegen gehören zur selben Organisation.

Normalerweise hat ein Team eine einzige Organisation, aber Sie können mehr als eine haben.

Um die Organisationen aufzulisten, auf die Ihr Konto zugreifen kann, führen Sie Folgendes aus:

`cf orgs`

Um Details zu einer Organisation anzuzeigen, führen Sie Folgendes aus:

`cf org ORGNAME`

wobei ORGNAME der Name der Organisation ist.

### Leerzeichen

Jede Organisation ist in einen oder mehrere **Bereiche** unterteilt, die zur Organisation der App-Entwicklung, -Bereitstellung und -Wartung verwendet werden. Beispielsweise könnten Sie Bereiche für Entwicklungs- und Produktionsversionen Ihrer App haben.

![Diagramm, das zeigt, dass eine Organisation mehrere Bereiche enthält](/documentation/figures/org-spaces.png)

Unterschiedliche Konten können unterschiedliche Berechtigungen für den Zugriff auf jeden Bereich haben, die durch Benutzerrollen gewährt werden. Beispielsweise können Sie entscheiden, einem Junior-Entwickler Zugriff auf Ihren `development` , aber nicht auf die `production` zu gewähren . Um zu ändern, welche Konten Zugriff haben, muss Ihr Konto die Rolle des Organisationsmanagers haben. Weitere Informationen zu Rollen finden Sie im Abschnitt zum [Verwalten von Benutzern](/#managing-users) .

Wenn wir Ihre Organisation einrichten, erstellen wir einen Standard- `sandbox` -Bereich, den Sie zum Experimentieren mit PaaS verwenden können.

Führen Sie Folgendes aus, um die Bereiche anzuzeigen, auf die Sie in Ihrer aktuellen Organisation zugreifen können:

`cf spaces`

### Ein Ziel setzen

Um eine App bereitzustellen, müssen Sie eine Kombination aus einer Organisation und einem Bereich angeben: Dies wird als **Ziel** bezeichnet.

Stellen Sie das Ziel ein mit:

`cf target -o ORGNAME -s SPACENAME`

Sobald Sie das Ziel festgelegt haben, merkt sich der Cloud Foundry-Client es, bis Sie es ändern.

Sie können den Bereich ändern, ohne die Organisation zu ändern, indem Sie Folgendes verwenden:

`cf target -s SPACENAME`

Sie sollten beim Testen Ihrer App auf den Sandbox-Bereich abzielen. Sie können dies tun, indem Sie Folgendes ausführen:

`cf target -s sandbox`

Wenn Sie ein neuer Benutzer sind, müssen Sie normalerweise nicht auf den Sandbox-Bereich abzielen, da dieser Bereich der Standard für neue Benutzer ist. Eine mögliche Ausnahme hiervon ist, wenn Ihre Organisation ausgereift ist und bereits Räume hat; Sie sollten sicherstellen, dass Sie den geeigneten Bereich zum Testen anvisieren.

### Räume verwalten

Sie können neue Bereiche innerhalb einer Organisation erstellen, wenn Ihr [Konto](/#org-manager) die Rolle „Organisationsmanager“ hat. Um herauszufinden, wer diese Rolle hat, führen Sie Folgendes aus:

`cf org-users ORGNAME`

wobei `ORGNAME` der Name der Organisation ist. Sie sehen eine Liste der Benutzer und ihrer Rollen.

Wenn Ihr Konto nicht die Rolle des Organisationsmanagers hat und Sie einen neuen Bereich erstellen müssen, sollten Sie einen Kollegen fragen, der ein Organisationsmanager für Ihre Organisation ist.

Als Organisationsmanager können Sie Folgendes verwenden:

`cf create-space SPACENAME -o ORGNAME`

einen neuen Raum zu schaffen. Sie müssen dann allen Mandantenkonten Zugriff gewähren, die diesen Bereich nutzen können sollen.

### Zugriff gewähren

Sie gewähren Zugriff auf einen Bereich, indem Sie einem Konto eine Rolle zuweisen. Um Rollen zuzuweisen, müssen Sie [Org Manager](/#org-manager) oder [Space Manager](/#space-manager) sein .

In den meisten Fällen ist die Rolle, die Sie zuweisen, [Space Developer](/#space-developer) , wodurch das Konto befähigt wird, Apps im angegebenen Space bereitzustellen und zu verwalten.

Führen Sie Folgendes aus, um einem anderen Konto Zugriff auf einen Bereich zu gewähren:

`cf set-space-role USERNAME ORGNAME SPACENAME ROLE`

Der `USERNAME` ist die E-Mail-Adresse, mit der sich der Benutzer anmeldet, wenn er den Befehlszeilen-Client verwendet.

Angenommen, Sie hätten gerade einen Bereich namens `test` in Ihrer Organisation mit dem Namen `acme` erstellt und Sie möchten, dass Ihre `ana@example.com` diesen Bereich als Entwickler verwenden kann, würden Sie Folgendes ausführen:

`cf set-space-role ana@example.com acme test SpaceDeveloper`

Weitere Informationen finden Sie in der [Cloud Foundry-Dokumentation zu Organisationen, Bereichen, Rollen und Berechtigungen](https://docs.cloudfoundry.org/concepts/roles.html) .
