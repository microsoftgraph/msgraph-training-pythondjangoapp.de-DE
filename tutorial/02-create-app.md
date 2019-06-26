<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="007b6-101">In dieser Übung werden Sie mit [Django](https://www.djangoproject.com/) eine Webanwendung erstellen.</span><span class="sxs-lookup"><span data-stu-id="007b6-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span> <span data-ttu-id="007b6-102">Wenn Django nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.</span><span class="sxs-lookup"><span data-stu-id="007b6-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
pip install Django=2.2.2
```

<span data-ttu-id="007b6-103">Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Django-APP zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="007b6-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

```Shell
django-admin startproject graph_tutorial
```

<span data-ttu-id="007b6-104">Django erstellt ein neues Verzeichnis namens `graph_tutorial` und ein Gerüst für eine Django-Webanwendung.</span><span class="sxs-lookup"><span data-stu-id="007b6-104">Django creates a new directory called `graph_tutorial` and scaffolds a Django web app.</span></span> <span data-ttu-id="007b6-105">Navigieren Sie zu diesem neuen Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="007b6-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
python manage.py runserver
```

<span data-ttu-id="007b6-106">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="007b6-106">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="007b6-107">Wenn alles funktioniert, wird eine Django-Willkommensseite angezeigt.</span><span class="sxs-lookup"><span data-stu-id="007b6-107">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="007b6-108">Wenn dies nicht der Fall ist, überprüfen Sie das [Django-Handbuch für erste Schritte](https://www.djangoproject.com/start/).</span><span class="sxs-lookup"><span data-stu-id="007b6-108">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

<span data-ttu-id="007b6-109">Nachdem Sie das Projekt überprüft haben, fügen Sie dem Projekt eine APP hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-109">Now that you've verified the project, add an app to the project.</span></span> <span data-ttu-id="007b6-110">Führen Sie den folgenden Befehl in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="007b6-110">Run the following command in your CLI.</span></span>

```Shell
python manage.py startapp tutorial
```

<span data-ttu-id="007b6-111">Dadurch wird eine neue APP im `./tutorial` Verzeichnis erstellt.</span><span class="sxs-lookup"><span data-stu-id="007b6-111">This creates a new app in the `./tutorial` directory.</span></span> <span data-ttu-id="007b6-112">Öffnen Sie `./graph_tutorial/settings.py` die, und fügen `tutorial` Sie die neue `INSTALLED_APPS` APP zu der Einstellung hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-112">Open the `./graph_tutorial/settings.py` and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

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

<span data-ttu-id="007b6-113">Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank für das Projekt zu initialisieren.</span><span class="sxs-lookup"><span data-stu-id="007b6-113">In your CLI, run the following command to initialize the database for the project.</span></span>

```Shell
python manage.py migrate
```

<span data-ttu-id="007b6-114">Fügen Sie ein [URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) für `tutorial` die APP hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-114">Add a [URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="007b6-115">Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `urls.py` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-115">Create a new file in the `./tutorial` directory named `urls.py` and add the following code.</span></span>

```python
from django.urls import path

from . import views

urlpatterns = [
  # /tutorial
  path('', views.home, name='home'),
]
```

<span data-ttu-id="007b6-116">Aktualisieren Sie nun das Projekt URLconf, um dieses zu importieren.</span><span class="sxs-lookup"><span data-stu-id="007b6-116">Now update the project URLconf to import this one.</span></span> <span data-ttu-id="007b6-117">Öffnen Sie `./graph_tutorial/urls.py` die Datei, und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="007b6-117">Open the `./graph_tutorial/urls.py` file and replace the contents with the following.</span></span>

```python
from django.contrib import admin
from django.urls import path, include
from tutorial import views

urlpatterns = [
    path('tutorial/', include('tutorial.urls')),
    path('admin/', admin.site.urls),
]
```

<span data-ttu-id="007b6-118">Fügen Sie der `tutorials` APP schließlich eine temporäre Ansicht hinzu, um zu überprüfen, ob das URL-Routing funktioniert.</span><span class="sxs-lookup"><span data-stu-id="007b6-118">Finally add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="007b6-119">Öffnen Sie die Datei `./tutorial/views.py`, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-119">Open the `./tutorial/views.py` file and add the following code.</span></span>

```python
from django.http import HttpResponse, HttpResponseRedirect

def home(request):
  # Temporary!
  return HttpResponse("Welcome to the tutorial.")
```

<span data-ttu-id="007b6-120">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="007b6-120">Save all of your changes and restart the server.</span></span> <span data-ttu-id="007b6-121">Navigieren Sie `http://localhost:8000/tutorial`zu.</span><span class="sxs-lookup"><span data-stu-id="007b6-121">Browse to `http://localhost:8000/tutorial`.</span></span> <span data-ttu-id="007b6-122">Sie sollten sehen`Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="007b6-122">You should see `Welcome to the tutorial.`</span></span>

<span data-ttu-id="007b6-123">Installieren Sie vor dem Verschieben einige zusätzliche Bibliotheken, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="007b6-123">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="007b6-124">[Anforderungen-OAuthlib: OAuth für Menschen](https://requests-oauthlib.readthedocs.io/en/latest/) zum Verarbeiten von Anmelde-und OAuth-Token-Flows sowie zum tätigen von Anrufen an Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="007b6-124">[Requests-OAuthlib: OAuth for Humans](https://requests-oauthlib.readthedocs.io/en/latest/) for handling sign-in and OAuth token flows, and for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="007b6-125">[Autoyaml](https://pyyaml.org/wiki/PyYAMLDocumentation) zum Laden der Konfiguration aus einer YAML-Datei.</span><span class="sxs-lookup"><span data-stu-id="007b6-125">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="007b6-126">[python-dateutil](https://pypi.org/project/python-dateutil/) zum Analysieren von ISO 8601-Datumszeichenfolgen, die von Microsoft Graph zurückgegeben werden.</span><span class="sxs-lookup"><span data-stu-id="007b6-126">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

<span data-ttu-id="007b6-127">Führen Sie den folgenden Befehl in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="007b6-127">Run the following command in your CLI.</span></span>

```Shell
pip install requests_oauthlib==1.2.0
pip install pyyaml==5.1
pip install python-dateutil==2.8.0
```

## <a name="design-the-app"></a><span data-ttu-id="007b6-128">Entwerfen der APP</span><span class="sxs-lookup"><span data-stu-id="007b6-128">Design the app</span></span>

<span data-ttu-id="007b6-129">Erstellen Sie zunächst ein Vorlagenverzeichnis, und definieren Sie ein globales Layout für die app.</span><span class="sxs-lookup"><span data-stu-id="007b6-129">Start by creating a templates directory and defining a global layout for the app.</span></span> <span data-ttu-id="007b6-130">Erstellen Sie ein neues Verzeichnis im `./tutorial` Verzeichnis mit `templates`dem Namen.</span><span class="sxs-lookup"><span data-stu-id="007b6-130">Create a new directory in the `./tutorial` directory named `templates`.</span></span> <span data-ttu-id="007b6-131">Erstellen Sie `templates` im Verzeichnis ein neues Verzeichnis mit dem `tutorial`Namen.</span><span class="sxs-lookup"><span data-stu-id="007b6-131">In the `templates` directory, create a new directory named `tutorial`.</span></span> <span data-ttu-id="007b6-132">Erstellen Sie schließlich eine neue Datei in diesem Verzeichnis mit `layout.html`dem Namen.</span><span class="sxs-lookup"><span data-stu-id="007b6-132">Finally, create a new file in this directory named `layout.html`.</span></span> <span data-ttu-id="007b6-133">Der relative Pfad aus dem Stamm des Projekts sollte sein `./tutorial/templates/tutorial/layout.html`.</span><span class="sxs-lookup"><span data-stu-id="007b6-133">The relative path from the root of your project should be `./tutorial/templates/tutorial/layout.html`.</span></span> <span data-ttu-id="007b6-134">Fügen Sie den folgenden Code in die Datei ein.</span><span class="sxs-lookup"><span data-stu-id="007b6-134">Add the following code in that file.</span></span>

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

<span data-ttu-id="007b6-135">Dieser Code fügt [Bootstrap](http://getbootstrap.com/) für einfaches Styling und [Font awesome](https://fontawesome.com/) für einige einfache Symbole hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-135">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="007b6-136">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="007b6-136">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="007b6-137">Erstellen Sie nun ein neues Verzeichnis im `./tutorial` Verzeichnis mit `static`dem Namen.</span><span class="sxs-lookup"><span data-stu-id="007b6-137">Now create a new directory in the `./tutorial` directory named `static`.</span></span> <span data-ttu-id="007b6-138">Erstellen Sie `static` im Verzeichnis ein neues Verzeichnis mit dem `tutorial`Namen.</span><span class="sxs-lookup"><span data-stu-id="007b6-138">In the `static` directory, create a new directory named `tutorial`.</span></span> <span data-ttu-id="007b6-139">Erstellen Sie schließlich eine neue Datei in diesem Verzeichnis mit `app.css`dem Namen.</span><span class="sxs-lookup"><span data-stu-id="007b6-139">Finally, create a new file in this directory named `app.css`.</span></span> <span data-ttu-id="007b6-140">Der relative Pfad aus dem Stamm des Projekts sollte sein `./tutorial/static/tutorial/app.css`.</span><span class="sxs-lookup"><span data-stu-id="007b6-140">The relative path from the root of your project should be `./tutorial/static/tutorial/app.css`.</span></span> <span data-ttu-id="007b6-141">Fügen Sie den folgenden Code in die Datei ein.</span><span class="sxs-lookup"><span data-stu-id="007b6-141">Add the following code in that file.</span></span>

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

<span data-ttu-id="007b6-142">Erstellen Sie als nächstes eine Vorlage für die Startseite, die das Layout verwendet.</span><span class="sxs-lookup"><span data-stu-id="007b6-142">Next, create a template for the home page that uses the layout.</span></span> <span data-ttu-id="007b6-143">Erstellen Sie eine neue Datei im `./tutorial/templates/tutorial` Verzeichnis mit `home.html` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-143">Create a new file in the `./tutorial/templates/tutorial` directory named `home.html` and add the following code.</span></span>

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

<span data-ttu-id="007b6-144">Aktualisieren Sie `home` die Ansicht, um diese Vorlage zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="007b6-144">Update the `home` view to use this template.</span></span> <span data-ttu-id="007b6-145">Öffnen Sie `./tutorial/views.py` die Datei, und fügen Sie die folgende neue Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="007b6-145">Open the `./tutorial/views.py` file and add the following new function.</span></span>

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

<span data-ttu-id="007b6-146">Ersetzen Sie dann die `home` vorhandene Ansicht durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="007b6-146">Then replace the existing `home` view with the following.</span></span>

```python
def home(request):
  context = initialize_context(request)

  return render(request, 'tutorial/home.html', context)
```

<span data-ttu-id="007b6-147">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="007b6-147">Save all of your changes and restart the server.</span></span> <span data-ttu-id="007b6-148">Nun sollte die APP sehr unterschiedlich aussehen.</span><span class="sxs-lookup"><span data-stu-id="007b6-148">Now, the app should look very different.</span></span>

![Screenshot der neu gestalteten Startseite](./images/create-app-01.png)
