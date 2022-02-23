## Stellen Sie eine .NET Core-App bereit

In diesem Abschnitt wird beschrieben, wie Sie eine .NET Core-Anwendung mithilfe des [dotnet-core-Buildpacks](https://github.com/cloudfoundry/dotnet-core-buildpack) für GOV.UK PaaS bereitstellen.

### Bereitstellen einer ASP.NET Core-Web-App

Dieses Beispiel führt Sie durch die Erstellung einer einfachen „Hello World“-Webanwendung mithilfe [der `dotnet` -Befehlszeilenschnittstelle](https://docs.microsoft.com/en-us/dotnet/core/tools/) . Die Anweisungen gehen davon aus, dass Sie den [im](/get_started.html#get-started) Abschnitt „Erste Schritte“ erläuterten Einrichtungsprozess bereits durchgeführt haben.

1. Installieren Sie die Befehlszeilenschnittstellen-Tools (CLI) von dotnet core

    [Anweisungen dazu finden Sie in der Microsoft-Dokumentation](https://docs.microsoft.com/en-us/dotnet/core/tools/#installation) .

2. Erstellen Sie mithilfe der dotnet-Befehlszeilenschnittstelle eine neue ASP.NET Core-Web-App

    ```bash
    dotnet new webapp --name your-app-name
    ```

3. Wechseln Sie in das Verzeichnis Ihrer neuen Anwendung

    ```bash
    cd your-app-name
    ```

4. Erstellen Sie ein Manifest, um zu beschreiben, wie Ihre Anwendung bereitgestellt werden soll

    ```yaml
    ---
    applications:
    - name: your-app-name
      buildpacks:
      - dotnet_core_buildpack
      memory: 256M
    ```

5. Wählen Sie eine Zielversion des .NET-Frameworks aus, die vom aktuellen Buildpack unterstützt wird

    1. Führen `cf buildpacks` aus und sehen Sie sich den Dateinamen für `dotnet_core_buildpack` an, um die Version zu ermitteln (wenn der Dateiname beispielsweise `dotnet-core-buildpack-cflinuxfs3-v2.3.2.zip` wäre, wäre die Version des Buildpacks `v2.3.2` ).

    2. Besuchen Sie die Versionsseite des Buildpacks, zum Beispiel [https://github.com/cloudfoundry/dotnet-core-buildpack/releases/tag/v2.3.2](https://github.com/cloudfoundry/dotnet-core-buildpack/releases/tag/v2.3.2)

    3. Überprüfen Sie die vom Buildpack unterstützten Versionen von `dotnet-sdk` sdk – Sie sollten normalerweise die neueste unterstützte Version verwenden, z. B. 3.0.100

    4. Bearbeiten Sie Ihre `.csproj` -Datei und ändern Sie den Inhalt des Elements `<TargetFramework>` in das Zielframework, das der unterstützten Frameworkversion entspricht (verwenden Sie beispielsweise `netcoreapp3.0` für eine 3.0.x-Version von `dotnet-sdk` ). Siehe [Microsoft-Dokumentation zu unterstützten Ziel-Framework-Versionen](https://docs.microsoft.com/en-us/dotnet/standard/frameworks#supported-target-framework-versions) .

    5. Erstellen Sie eine `buildpack.yml` -Datei, um anzugeben, welche Version des Frameworks das Buildpack verwenden soll:

        ```yaml
        ---
        dotnet-core:
          sdk: 3.x
        ```

6. Pushen Sie Ihre Anwendung zu GOV.UK PaaS

    ```bash
    cf push
    ```

Sie haben jetzt Ihre .NET Core-Anwendung bereitgestellt. Ihre Anwendung sollte jetzt über HTTPS von der in der Ausgabe angegebenen URL aus zugänglich sein.
