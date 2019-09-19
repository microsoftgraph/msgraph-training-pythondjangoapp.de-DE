<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie mit [Django](https://www.djangoproject.com/) eine Webanwendung erstellen. Wenn Django nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.

```Shell
pip install Django=2.2.5
```

Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Django-APP zu erstellen.

```Shell
django-admin startproject graph_tutorial
```

Django erstellt ein neues Verzeichnis namens `graph_tutorial` und ein Gerüst für eine Django-Webanwendung. Navigieren Sie zu diesem neuen Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.

```Shell
python manage.py runserver
```

Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8000`. Wenn alles funktioniert, wird eine Django-Willkommensseite angezeigt. Wenn dies nicht der Fall ist, überprüfen Sie das [Django-Handbuch für erste Schritte](https://www.djangoproject.com/start/).

Nachdem Sie das Projekt überprüft haben, fügen Sie dem Projekt eine APP hinzu. Führen Sie den folgenden Befehl in der CLI aus.

```Shell
python manage.py startapp tutorial
```

Dadurch wird eine neue APP im `./tutorial` Verzeichnis erstellt. Öffnen Sie `./graph_tutorial/settings.py` die, und fügen `tutorial` Sie die neue `INSTALLED_APPS` APP zu der Einstellung hinzu.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tutorial',
]
```

Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank für das Projekt zu initialisieren.

```Shell
python manage.py migrate
```

Fügen Sie ein [URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) für `tutorial` die APP hinzu. Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `urls.py` dem Namen, und fügen Sie den folgenden Code hinzu.

```python
from django.urls import path

from . import views

urlpatterns = [
  # /tutorial
  path('', views.home, name='home'),
]
```

Aktualisieren Sie nun das Projekt URLconf, um dieses zu importieren. Öffnen Sie `./graph_tutorial/urls.py` die Datei, und ersetzen Sie den Inhalt durch Folgendes.

```python
from django.contrib import admin
from django.urls import path, include
from tutorial import views

urlpatterns = [
    path('tutorial/', include('tutorial.urls')),
    path('admin/', admin.site.urls),
]
```

Fügen Sie der `tutorials` APP schließlich eine temporäre Ansicht hinzu, um zu überprüfen, ob das URL-Routing funktioniert. Öffnen Sie die Datei `./tutorial/views.py`, und fügen Sie den folgenden Code hinzu.

```python
from django.http import HttpResponse, HttpResponseRedirect

def home(request):
  # Temporary!
  return HttpResponse("Welcome to the tutorial.")
```

Speichern Sie alle Änderungen, und starten Sie den Server neu. Navigieren Sie `http://localhost:8000/tutorial`zu. Sie sollten sehen`Welcome to the tutorial.`

Installieren Sie vor dem Verschieben einige zusätzliche Bibliotheken, die Sie später verwenden werden:

- [Anforderungen-OAuthlib: OAuth für Menschen](https://requests-oauthlib.readthedocs.io/en/latest/) zum Verarbeiten von Anmelde-und OAuth-Token-Flows sowie zum tätigen von Anrufen an Microsoft Graph.
- [Autoyaml](https://pyyaml.org/wiki/PyYAMLDocumentation) zum Laden der Konfiguration aus einer YAML-Datei.
- [python-dateutil](https://pypi.org/project/python-dateutil/) zum Analysieren von ISO 8601-Datumszeichenfolgen, die von Microsoft Graph zurückgegeben werden.

Führen Sie den folgenden Befehl in der CLI aus.

```Shell
pip install requests_oauthlib==1.2.0
pip install pyyaml==5.1
pip install python-dateutil==2.8.0
```

## <a name="design-the-app"></a>Entwerfen der APP

Erstellen Sie zunächst ein Vorlagenverzeichnis, und definieren Sie ein globales Layout für die app. Erstellen Sie ein neues Verzeichnis im `./tutorial` Verzeichnis mit `templates`dem Namen. Erstellen Sie `templates` im Verzeichnis ein neues Verzeichnis mit dem `tutorial`Namen. Erstellen Sie schließlich eine neue Datei in diesem Verzeichnis mit `layout.html`dem Namen. Der relative Pfad aus dem Stamm des Projekts sollte sein `./tutorial/templates/tutorial/layout.html`. Fügen Sie den folgenden Code in die Datei ein.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Python Graph Tutorial</title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    {% load static %}
    <link rel="stylesheet" href="{% static "tutorial/app.css" %}">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <a href="{% url 'home' %}" class="navbar-brand">Python Graph Tutorial</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <a href="{% url 'home' %}" class="nav-link{% if request.resolver_match.view_name == 'home' %} active{% endif %}">Home</a>
            </li>
            {% if user.is_authenticated %}
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>
              </li>
            {% endif %}
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            {% if user.is_authenticated %}
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  {% if user.avatar %}
                    <img src="{{ user.avatar }}" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  {% else %}
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  {% endif %}
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0">{{ user.name }}</h5>
                  <p class="dropdown-item-text text-muted mb-0">{{ user.email }}</p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            {% else %}
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            {% endif %}
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      {% if errors %}
        {% for error in errors %}
          <div class="alert alert-danger" role="alert">
            <p class="mb-3">{{ error.message }}</p>
            {% if error.debug %}
              <pre class="alert-pre border bg-light p-2"><code>{{ error.debug }}</code></pre>
            {% endif %}
          </div>
        {% endfor %}
      {% endif %}
      {% block content %}{% endblock %}
    </main>
  </body>
</html>
```

Dieser Code fügt [Bootstrap](http://getbootstrap.com/) für einfaches Styling und [Font awesome](https://fontawesome.com/) für einige einfache Symbole hinzu. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

Erstellen Sie nun ein neues Verzeichnis im `./tutorial` Verzeichnis mit `static`dem Namen. Erstellen Sie `static` im Verzeichnis ein neues Verzeichnis mit dem `tutorial`Namen. Erstellen Sie schließlich eine neue Datei in diesem Verzeichnis mit `app.css`dem Namen. Der relative Pfad aus dem Stamm des Projekts sollte sein `./tutorial/static/tutorial/app.css`. Fügen Sie den folgenden Code in die Datei ein.

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

Erstellen Sie als nächstes eine Vorlage für die Startseite, die das Layout verwendet. Erstellen Sie eine neue Datei im `./tutorial/templates/tutorial` Verzeichnis mit `home.html` dem Namen, und fügen Sie den folgenden Code hinzu.

```html
{% extends "tutorial/layout.html" %}
{% block content %}
<div class="jumbotron">
  <h1>Python Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Python</p>
  {% if user.is_authenticated %}
    <h4>Welcome {{ user.name }}!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  {% else %}
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  {% endif %}
</div>
{% endblock %}
```

Aktualisieren Sie `home` die Ansicht, um diese Vorlage zu verwenden. Öffnen Sie `./tutorial/views.py` die Datei, und fügen Sie die folgende neue Funktion hinzu.

```python
def initialize_context(request):
  context = {}

  # Check for any errors in the session
  error = request.session.pop('flash_error', None)

  if error != None:
    context['errors'] = []
    context['errors'].append(error)

  # Check for user in the session
  context['user'] = request.session.get('user', {'is_authenticated': False})
  return context
```

Ersetzen Sie dann die `home` vorhandene Ansicht durch Folgendes.

```python
def home(request):
  context = initialize_context(request)

  return render(request, 'tutorial/home.html', context)
```

Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

![Screenshot der neu gestalteten Startseite](./images/create-app-01.png)
