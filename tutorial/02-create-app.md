<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung verwenden Sie [Django,](https://www.djangoproject.com/) um eine Web-App zu erstellen.

1. Wenn Sie Django noch nicht installiert haben, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.

    ```Shell
    pip install Django==3.2
    ```

1. Öffnen Sie Ihre CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über die Rechte zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Django-App zu erstellen.

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. Navigieren Sie zum **graph_tutorial,** und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.

    ```Shell
    python manage.py runserver
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8000`. Wenn alles funktioniert, wird eine Django-Willkommensseite angezeigt. Wenn Sie dies nicht sehen, überprüfen Sie die [Anleitung django erste Schritte](https://www.djangoproject.com/start/).

1. Fügen Sie dem Projekt eine App hinzu. Führen Sie den folgenden Befehl in Ihrer CLI aus.

    ```Shell
    python manage.py startapp tutorial
    ```

1. Öffnen **Sie ./graph_tutorial/settings.py,** und fügen Sie der Einstellung die `tutorial` neue App `INSTALLED_APPS` hinzu.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank für das Projekt zu initialisieren.

    ```Shell
    python manage.py migrate
    ```

1. Fügen Sie einen [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) für die `tutorial` App hinzu. Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial** `urls.py` namens, und fügen Sie den folgenden Code hinzu.

    ```python
    from django.urls import path

    from . import views

    urlpatterns = [
      # /
      path('', views.home, name='home'),
      # TEMPORARY
      path('signin', views.home, name='signin'),
      path('signout', views.home, name='signout'),
      path('calendar', views.home, name='calendar'),
    ]
    ```

1. Aktualisieren Sie das Projekt URLconf, um dieses zu importieren. Öffnen **Sie ./graph_tutorial/urls.py,** und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. Fügen Sie der App eine temporäre Ansicht `tutorials` hinzu, um zu überprüfen, ob das URL-Routing funktioniert. Öffnen **Sie ./tutorial/views.py,** und ersetzen Sie den gesamten Inhalt durch den folgenden Code.

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect
    from django.urls import reverse
    from datetime import datetime, timedelta

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Navigieren Sie zu `http://localhost:8000` . Sie sollten sehen `Welcome to the tutorial.`

## <a name="install-libraries"></a>Installieren von Bibliotheken

Installieren Sie vor dem Wechsel einige zusätzliche Bibliotheken, die Sie später verwenden werden:

- [Microsoft Authentication Library (MSAL) für Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) für die Verarbeitung von Anmelde- und OAuth-Tokenflüssen.
- [PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) zum Laden der Konfiguration aus einer YAML-Datei.
- [python-dateutil](https://pypi.org/project/python-dateutil/) für die Analyse von ISO 8601-Datumszeichenfolgen, die von Microsoft Graph zurückgegeben werden.

1. Führen Sie den folgenden Befehl in Ihrer CLI aus.

    ```Shell
    pip install msal==1.10.0
    pip install pyyaml==5.4.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a>Entwerfen der App

1. Erstellen Sie ein neues Verzeichnis im **Verzeichnis ./tutorial** mit dem Namen `templates` .

1. Erstellen Sie **im Verzeichnis ./tutorial/templates** ein neues Verzeichnis mit dem Namen `tutorial` .

1. Erstellen Sie **im Verzeichnis ./tutorial/templates/tutorial** eine neue Datei namens `layout.html` . Fügen Sie den folgenden Code in dieser Datei hinzu.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    Dieser Code fügt [Bootstrap für](http://getbootstrap.com/) einfache Formatierung und [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) für einige einfache Symbole hinzu. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

1. Erstellen Sie ein neues Verzeichnis im **Verzeichnis ./tutorial** mit dem Namen `static` .

1. Erstellen Sie **im Verzeichnis ./tutorial/static** ein neues Verzeichnis mit dem Namen `tutorial` .

1. Erstellen Sie **im Verzeichnis ./tutorial/static/tutorial** eine neue Datei mit dem Namen `app.css` . Fügen Sie den folgenden Code in dieser Datei hinzu.

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. Erstellen Sie eine Vorlage für die Homepage, die das Layout verwendet. Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial/templates/tutorial** `home.html` namens, und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. Öffnen Sie `./tutorial/views.py` die Datei, und fügen Sie die folgende neue Funktion hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. Ersetzen Sie die `home` vorhandene Ansicht durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. Fügen Sie eine PNG-Datei **namensno-profile-photo.png** im **Verzeichnis ./tutorial/static/tutorial** hinzu.

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Jetzt sollte die App ganz anders aussehen.

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
