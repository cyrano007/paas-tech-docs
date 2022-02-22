## Stellen Sie eine Java-App bereit

In diesem Abschnitt wird beschrieben, wie Sie eine Java-Anwendung mithilfe des [Java-Buildpacks](https://github.com/cloudfoundry/java-buildpack) für GOV.UK PaaS bereitstellen.

### Bereitstellen einer JAR-Datei

Wenn Ihre Java-Anwendung als [selbstausführbare JAR-Datei](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/container-java_main.md) verpackt werden kann, können Sie sie mit `cf push -p [your-app].jar [your-app-name]` bereitstellen.

Dieses Beispiel führt Sie durch die Erstellung einer einfachen „Hello World“-Anwendung, die den beliebten [Jetty-Webserver](https://www.eclipse.org/jetty/) einbettet. Die Anweisungen gehen davon aus, dass Sie den [im](/get_started.html#get-started) Abschnitt „Erste Schritte“ erläuterten Einrichtungsprozess bereits durchgeführt haben.

1. Erstellen Sie ein Verzeichnis für Ihre Java-Anwendung:

    ```bash
    mkdir myapp
    cd myapp
    ```

2. Jetty herunterladen.

    ```bash
    curl -o ./jetty.jar http://central.maven.org/maven2/org/eclipse/jetty/aggregate/jetty-all/9.4.6.v20170531/jetty-all-9.4.6.v20170531-uber.jar
    ```

    Dieses Beispiel verwendet Version 9.4.6; Unter [http://www.eclipse.org/jetty/download.html](http://www.eclipse.org/jetty/download.html) finden Sie die neueste Version von Jetty.

3. Erstellen Sie eine `App.java` -Datei in Ihrem Anwendungsverzeichnis. Diese Klasse dient als Einstiegspunkt für Ihre Bewerbung:

    ```java
    package app;

    import java.io.IOException;
    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import org.eclipse.jetty.server.Request;
    import org.eclipse.jetty.server.Server;
    import org.eclipse.jetty.server.handler.AbstractHandler;


    public class App extends AbstractHandler
    {
        @Override
        public void handle( String target, Request baseRequest, HttpServletRequest request, HttpServletResponse response ) throws IOException, ServletException
        {
               response.setContentType("text/html; charset=utf-8");
               response.setStatus(HttpServletResponse.SC_OK);
               response.getWriter().println("<h1>Hello World</h1>");
               baseRequest.setHandled(true);
        }

        public static void main( String[] args ) throws Exception
        {
                int port = Integer.parseInt(System.getenv("PORT"));
                Server server = new Server(port);
                server.setHandler(new App());

                server.start();
                server.join();
        }
    }
    ```

4. Kompilieren Sie Ihre Klasse:

    ```bash
    javac -d . -cp ./jetty.jar App.java
    ```

5. Kombinieren Sie Ihre Klasse mit Jetty, um eine ausführbare JAR-Datei zu erstellen:

    ```bash
    jar uvfe ./jetty.jar app.App app/App.class
    ```

6. Pushen Sie Ihre JAR-Datei.

    ```bash
    cf push your-app-name -b java_buildpack -p ./jetty.jar
    ```

Sie haben jetzt Ihre Java-Anwendung bereitgestellt. Ihre Anwendung sollte jetzt über HTTPS von der in der Ausgabe angegebenen URL aus zugänglich sein.

Wenn Ihre Anwendung zum Starten Argumente benötigt, können Sie die Umgebungsvariable `JBP_CONFIG_JAVA_MAIN` . Um beispielsweise `server` und `config.yml` als Argumente für Ihre App hinzuzufügen, könnten Sie Folgendes zur `manifest.yml` Ihrer Anwendung hinzufügen:

```yaml
env:
  JBP_CONFIG_JAVA_MAIN: '{ arguments: "server config.yml" }
```

### Bereitstellen einer WAR-Datei

Wenn Ihre Anwendung als `.war` -Datei verpackt werden kann, können Sie Folgendes bereitstellen:

```bash
cf push your-app-name -b java_buildpack -p your-app.war
```

Wenn Sie eine bestimmte Version von Tomcat verwenden müssen, können Sie die Umgebungsvariable `JBP_CONFIG_TOMCAT` . Um beispielsweise Tomcat 8 zu verwenden, könnten Sie Folgendes zur `manifest.yml` Ihrer Anwendung hinzufügen:

```yaml
env:
  JBP_CONFIG_TOMCAT: '{ tomcat: { version: 8.0.+ } }'
```

Beachten Sie, dass Sie Tomcat nicht zusammen mit Ihrer Anwendung bereitstellen müssen. Das Java-Buildpack führt die Webanwendungen Servlet 2 und 3 aus.

Weitere Konfigurationsoptionen finden Sie in der [Tomcat-Container-Dokumentation](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/container-tomcat.md) .

### Geben Sie eine Java-Version an

Sie können die Version der Java Runtime Environment (JRE) angeben, indem Sie die Umgebungsvariable `JBP_CONFIG_OPEN_JDK_JRE` in der `manifest.yml` der Anwendung festlegen. So verwenden Sie beispielsweise JRE 11:

```yaml
  env:
    JBP_CONFIG_OPEN_JDK_JRE: '{ jre: { version: 11.+ } }'
```

Das `+` in diesem Beispiel bedeutet, dass das neueste verfügbare Point-Release verwendet wird.

### Bereitstellen anderer JVM-basierter Anwendungen

Das Java-Buildpack unterstützt die Ausführung beliebiger JVM-basierter Anwendungen (z. B. Scala oder Clojure) mit wenig oder gar keiner zusätzlichen Konfiguration. Weitere Informationen finden Sie in den folgenden Beispielen für die Bereitstellung von Anwendungen mit anderen Frameworks und JVM-basierten Sprachen:

- [Eingebetteter Webserver](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/example-embedded-web-server.md)
- [Grale](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/example-grails.md)
- [Groovig](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/example-groovy.md)
- [Java-Main](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/example-java_main.md)
- [Rahmen spielen](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/example-play_framework.md)
- [Servlet](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/example-servlet.md)
- [Spring Boot-CLI](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/example-spring_boot_cli.md)
- [Cloud Foundry Java Buildpack-Dokumentation](https://github.com/cloudfoundry/java-buildpack/blob/master/README.md)
- [Cloud Foundry-Tipps für Java-Entwickler](https://docs.cloudfoundry.org/buildpacks/java/java-tips.html) für weitere Informationen.
