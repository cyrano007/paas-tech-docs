## Stellen Sie eine App in der Produktion bereit

Um eine App in einer Produktionsumgebung bereitzustellen, muss Ihre Abteilung, Agentur oder Ihr Team über ein [kostenpflichtiges Konto](/get_started.html#trial-and-paid-accounts) bei GOV.UK PaaS verfügen.

Bevor Sie eine App in einer Produktionsumgebung bereitstellen, müssen Sie eine Domäne einrichten.

Wenn Sie ein zentraler Regierungsdienst sind, der die Domäne GOV.UK verwendet, müssen [Sie eine Domäne von GOV.UK](https://www.gov.uk/service-manual/technology/get-a-domain-name) für Ihren Dienst erhalten und die für Ihren cdn [-route-Dienst erforderlichen DNS-Einträge](/deploying_services/use_a_custom_domain/#set-up-a-cdn-route-service-with-one-or-more-custom-domains) festlegen.

Wenn Sie Ihr eigenes Content Distribution Network (CDN) verwenden, müssen Sie [eine Domäne einrichten, indem Sie Ihr CDN konfigurieren](/deploying_services/configure_cdn/#set-up-a-custom-domain-by-configuring-your-own-cdn) .

Sie müssen außerdem:

- Erstellen Sie Ihre App nach den [12-Faktor-App-Prinzipien](/architecture.html#12-factor-application-principles)
- Verwenden Sie eine separate Staging-Umgebung, um Ihre App zu testen (weitere Informationen finden Sie in den [Fallstudien zum Verwalten von Organisationen, Benutzern und Bereichen](/orgs_spaces_users.html#case-studies) ).
- [Konfigurieren Sie eine `http` -Zustandsprüfung](https://docs.cloudfoundry.org/devguide/deploy-apps/healthchecks.html) , damit Cloud Foundry fehlerhafte App-Instanzen erkennen und versuchen kann, sie zu ersetzen

Wenn Sie eine App in einer Produktionsumgebung bereitstellen, sollten Sie:

- [Verwenden Sie null Ausfallzeiten, fortlaufende Bereitstellungen](/get_started.html#deploying-with-zero-downtime)
- [Führen Sie mehr als eine Instanz Ihrer App](/managing_apps.html#scaling) aus, um sicherzustellen, dass Ihre App immer verfügbar ist
- Wählen Sie einen Hochverfügbarkeitsplan für den [Sicherungsdienst](/deploying_services/#deploy-a-backing-or-routing-service) Ihrer App aus, wenn Ihre App einen Sicherungsdienst verwendet
