## Namen, Routen und Domänen

### App-Namen und Domain-Hostname kollidieren

Wenn Sie eine App pushen, müssen Sie einen App-Namen zuweisen, entweder im Manifest oder über die Befehlszeile. Der App-Name wird auch als Standard-Hostname der [App-Domäne](/orgs_spaces_users.html#regions) verwendet, in der die App gehostet wird.

Wenn Sie beispielsweise eine App namens `myapp` in der Region London pushen, versucht PaaS, sie unter `myapp.london.cloudapps.digital` zu hosten. Dies erfolgt mithilfe der **Routenfunktionen** von Cloud Foundry.

Die [App-Domäne](/orgs_spaces_users.html#regions) wird von allen PaaS-Mandanten innerhalb einer Region gemeinsam genutzt. Wenn in diesem Beispiel ein anderer `myapp.cloudapps.digital` in der Region Irland verwendet und Sie versuchen, eine App namens `myapp` in dieselbe Region zu pushen, erhalten Sie eine Fehlermeldung wie die folgende:

```
Using route myapp.london.cloudapps.digital
Binding myapp.london.cloudapps.digital to myapp...
FAILED
The route myapp.london.cloudapps.digital is already in use.
```

Dies geschieht auch, wenn es in Ihrer eigenen Organisation eine App mit demselben Namen, aber in einem anderen Bereich gibt.

Es gibt einige mögliche Lösungen für dieses Problem:

1. Minimieren Sie die Wahrscheinlichkeit von Namenskonflikten durch die Art und Weise, wie Sie Ihre Apps benennen. Integrieren Sie den Namen Ihres Teams, Ihrer Abteilung oder Ihres Dienstes, um Namen zu erstellen, die wahrscheinlich nicht mit anderen PaaS-Mandanten kollidieren. Wenn Sie beispielsweise für das Friedensministerium arbeiten, können Sie anstelle von `myapp` `minipeace-myapp` .

    Wenn Sie verschiedene Versionen einer App in verschiedenen Bereichen ausführen, müssen Sie sie auch unterschiedlich benennen: Sie könnten beispielsweise `minipeace-myapp-dev` , `minipeace-myapp-test` , `minipeace-myapp-production` und so weiter haben.

2. Geben Sie manuell einen Hostnamen an, der sich vom App-Namen unterscheidet. Sie können dies als Befehlszeilenoption tun, wenn Sie Folgendes drücken:

    `cf push myapp -n HOSTNAME`

    oder mit einer Zeile in der `manifest.yml` -Datei der App:

    ```
     ---
         ...
         host: HOSTNAME
    ```

    Sie müssen immer noch einen Hostnamen auswählen, der nicht verwendet wird.

    Diese Lösung bedeutet, dass Sie den Hostnamen ändern können, während der App-Name gleich bleibt. Dies ist flexibler, bedeutet aber, dass Sie sowohl einen Hostnamen als auch einen App-Namen für jede App im Auge behalten müssen. Es könnte möglicherweise die Bereitstellung für das falsche Ziel erleichtern.

3. Verwenden Sie die `random-route` Option. Dies hängt ein paar zufällige Wörter an den Hostnamen an, um Konflikte zu vermeiden. Wenn Ihre App beispielsweise in der Region London gehostet wird und Sie Folgendes ausführen:

    `cf push myapp --random-route`

    Die App wird etwa unter `https://myapp-mummifying-giraffe.london.cloudapps.digital` gehostet.

    Sie können dies auch in der `manifest.yml` -Datei Ihrer App angeben:

    `random-route: true`

    Dies ist eine bequeme Möglichkeit, Kollisionen automatisch zu vermeiden. Der Nachteil ist, dass Ihre Endbenutzer, wenn sie die resultierenden URLs sehen können, die zufälligen Wörter möglicherweise seltsam finden.

### Benutzerdefinierte Domänen

In der Produktion möchten Sie wahrscheinlich, dass Ihre App über Ihre eigene URL verfügbar ist (z. B. `yourapp.service.gov.uk` ). Siehe [Verwalten von benutzerdefinierten Domänen mit dem cdn-route-Dienst](deploying_services/use_a_custom_domain/#managing-custom-domains-using-the-cdn-route-service) , um zu sehen, wie Sie dies einrichten.
