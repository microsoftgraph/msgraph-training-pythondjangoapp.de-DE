<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie mit [Django](https://www.djangoproject.com/) eine Webanwendung erstellen.

1. Wenn Django nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.

    ```Shell
    pip install Django==3.0.4
    ```

1. Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Django-APP zu erstellen.

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. Navigieren Sie zum **graph_tutorial** Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.

    ```Shell
    python manage.py runserver
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8000`. Wenn alles funktioniert, wird eine Django-Willkommensseite angezeigt. Wenn dies nicht der Fall ist, überprüfen Sie das [Django-Handbuch für erste Schritte](https://www.djangoproject.com/start/).

1. Fügen Sie dem Projekt eine APP hinzu. Führen Sie den folgenden Befehl in der CLI aus.

    ```Shell
    python manage.py startapp tutorial
    ```

1. Öffnen Sie **./graph_tutorial/Settings.py** , und fügen `tutorial` Sie die neue `INSTALLED_APPS` APP zu der Einstellung hinzu.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank für das Projekt zu initialisieren.

    ```Shell
    python manage.py migrate
    ```

1. Fügen Sie ein [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) für `tutorial` die APP hinzu. Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis `urls.py` mit dem Namen, und fügen Sie den folgenden Code hinzu.

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

1. Aktualisieren Sie das Projekt URLconf, um dieses zu importieren. Öffnen Sie **graph_tutorial/URLs.py** , und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. Fügen Sie der `tutorials` App eine temporäre Ansicht hinzu, um zu überprüfen, ob das URL-Routing funktioniert. Öffnen Sie **/Tutorial/views.py** , und fügen Sie den folgenden Code hinzu.

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Navigieren Sie `http://localhost:8000`zu. Sie sollten sehen`Welcome to the tutorial.`

## <a name="install-libraries"></a>Installieren von Bibliotheken

Installieren Sie vor dem Verschieben einige zusätzliche Bibliotheken, die Sie später verwenden werden:

- [Anforderungen-OAuthlib: OAuth für Menschen](https://requests-oauthlib.readthedocs.io/en/latest/) zum Verarbeiten von Anmelde-und OAuth-Token-Flows sowie zum tätigen von Anrufen an Microsoft Graph.
- [Autoyaml](https://pyyaml.org/wiki/PyYAMLDocumentation) zum Laden der Konfiguration aus einer YAML-Datei.
- [python-dateutil](https://pypi.org/project/python-dateutil/) zum Analysieren von ISO 8601-Datumszeichenfolgen, die von Microsoft Graph zurückgegeben werden.

1. Führen Sie den folgenden Befehl in der CLI aus.

    ```Shell
    pip install requests_oauthlib==1.3.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a>Entwerfen der App

1. Erstellen Sie ein neues Verzeichnis im **./Tutorial** -Verzeichnis `templates`mit dem Namen.

1. Erstellen Sie im Verzeichnis **./Tutorial/Templates** ein neues Verzeichnis mit dem `tutorial`Namen.

1. Erstellen Sie im Verzeichnis **./Tutorial/Templates/Tutorial** eine neue Datei mit dem `layout.html`Namen. Fügen Sie den folgenden Code in die Datei ein.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    Dieser Code fügt [Bootstrap](http://getbootstrap.com/) zur einfachen Formatierung hinzu, und [Font Awesome](https://fontawesome.com/) für einige einfache Symbole. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

1. Erstellen Sie ein neues Verzeichnis im **./Tutorial** -Verzeichnis `static`mit dem Namen.

1. Erstellen Sie im Verzeichnis **./Tutorial/static** ein neues Verzeichnis mit dem `tutorial`Namen.

1. Erstellen Sie im Verzeichnis **./Tutorial/static/Tutorial** eine neue Datei mit dem `app.css`Namen. Fügen Sie den folgenden Code in die Datei ein.

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. Erstellen Sie eine Vorlage für die Startseite, die das Layout verwendet. Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis `home.html` mit dem Namen, und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. Öffnen Sie `./tutorial/views.py` die Datei, und fügen Sie die folgende neue Funktion hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. Ersetzen Sie die `home` vorhandene Ansicht durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
