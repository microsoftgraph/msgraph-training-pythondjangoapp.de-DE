<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="3b82e-101">In dieser Übung werden Sie mit [Django](https://www.djangoproject.com/) eine Webanwendung erstellen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span>

1. <span data-ttu-id="3b82e-102">Wenn Django nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.</span><span class="sxs-lookup"><span data-stu-id="3b82e-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    pip install Django==3.0.4
    ```

1. <span data-ttu-id="3b82e-103">Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Django-APP zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. <span data-ttu-id="3b82e-104">Navigieren Sie zum **graph_tutorial** Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="3b82e-104">Navigate to the **graph_tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="3b82e-105">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="3b82e-105">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="3b82e-106">Wenn alles funktioniert, wird eine Django-Willkommensseite angezeigt.</span><span class="sxs-lookup"><span data-stu-id="3b82e-106">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="3b82e-107">Wenn dies nicht der Fall ist, überprüfen Sie das [Django-Handbuch für erste Schritte](https://www.djangoproject.com/start/).</span><span class="sxs-lookup"><span data-stu-id="3b82e-107">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

1. <span data-ttu-id="3b82e-108">Fügen Sie dem Projekt eine APP hinzu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-108">Add an app to the project.</span></span> <span data-ttu-id="3b82e-109">Führen Sie den folgenden Befehl in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="3b82e-109">Run the following command in your CLI.</span></span>

    ```Shell
    python manage.py startapp tutorial
    ```

1. <span data-ttu-id="3b82e-110">Öffnen Sie **./graph_tutorial/Settings.py** , und fügen `tutorial` Sie die neue `INSTALLED_APPS` APP zu der Einstellung hinzu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-110">Open **./graph_tutorial/settings.py** and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. <span data-ttu-id="3b82e-111">Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank für das Projekt zu initialisieren.</span><span class="sxs-lookup"><span data-stu-id="3b82e-111">In your CLI, run the following command to initialize the database for the project.</span></span>

    ```Shell
    python manage.py migrate
    ```

1. <span data-ttu-id="3b82e-112">Fügen Sie ein [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) für `tutorial` die APP hinzu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-112">Add a [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="3b82e-113">Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis `urls.py` mit dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-113">Create a new file in the **./tutorial** directory named `urls.py` and add the following code.</span></span>

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

1. <span data-ttu-id="3b82e-114">Aktualisieren Sie das Projekt URLconf, um dieses zu importieren.</span><span class="sxs-lookup"><span data-stu-id="3b82e-114">Update the project URLconf to import this one.</span></span> <span data-ttu-id="3b82e-115">Öffnen Sie **graph_tutorial/URLs.py** , und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="3b82e-115">Open **./graph_tutorial/urls.py** and replace the contents with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. <span data-ttu-id="3b82e-116">Fügen Sie der `tutorials` App eine temporäre Ansicht hinzu, um zu überprüfen, ob das URL-Routing funktioniert.</span><span class="sxs-lookup"><span data-stu-id="3b82e-116">Add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="3b82e-117">Öffnen Sie **/Tutorial/views.py** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-117">Open **./tutorial/views.py** and add the following code.</span></span>

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. <span data-ttu-id="3b82e-118">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-118">Save all of your changes and restart the server.</span></span> <span data-ttu-id="3b82e-119">Navigieren Sie `http://localhost:8000`zu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-119">Browse to `http://localhost:8000`.</span></span> <span data-ttu-id="3b82e-120">Sie sollten sehen`Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="3b82e-120">You should see `Welcome to the tutorial.`</span></span>

## <a name="install-libraries"></a><span data-ttu-id="3b82e-121">Installieren von Bibliotheken</span><span class="sxs-lookup"><span data-stu-id="3b82e-121">Install libraries</span></span>

<span data-ttu-id="3b82e-122">Installieren Sie vor dem Verschieben einige zusätzliche Bibliotheken, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="3b82e-122">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="3b82e-123">[Anforderungen-OAuthlib: OAuth für Menschen](https://requests-oauthlib.readthedocs.io/en/latest/) zum Verarbeiten von Anmelde-und OAuth-Token-Flows sowie zum tätigen von Anrufen an Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="3b82e-123">[Requests-OAuthlib: OAuth for Humans](https://requests-oauthlib.readthedocs.io/en/latest/) for handling sign-in and OAuth token flows, and for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="3b82e-124">[Autoyaml](https://pyyaml.org/wiki/PyYAMLDocumentation) zum Laden der Konfiguration aus einer YAML-Datei.</span><span class="sxs-lookup"><span data-stu-id="3b82e-124">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="3b82e-125">[python-dateutil](https://pypi.org/project/python-dateutil/) zum Analysieren von ISO 8601-Datumszeichenfolgen, die von Microsoft Graph zurückgegeben werden.</span><span class="sxs-lookup"><span data-stu-id="3b82e-125">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

1. <span data-ttu-id="3b82e-126">Führen Sie den folgenden Befehl in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="3b82e-126">Run the following command in your CLI.</span></span>

    ```Shell
    pip install requests_oauthlib==1.3.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a><span data-ttu-id="3b82e-127">Entwerfen der App</span><span class="sxs-lookup"><span data-stu-id="3b82e-127">Design the app</span></span>

1. <span data-ttu-id="3b82e-128">Erstellen Sie ein neues Verzeichnis im **./Tutorial** -Verzeichnis `templates`mit dem Namen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-128">Create a new directory in the **./tutorial** directory named `templates`.</span></span>

1. <span data-ttu-id="3b82e-129">Erstellen Sie im Verzeichnis **./Tutorial/Templates** ein neues Verzeichnis mit dem `tutorial`Namen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-129">In the **./tutorial/templates** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="3b82e-130">Erstellen Sie im Verzeichnis **./Tutorial/Templates/Tutorial** eine neue Datei mit dem `layout.html`Namen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-130">In the **./tutorial/templates/tutorial** directory, create a new file named `layout.html`.</span></span> <span data-ttu-id="3b82e-131">Fügen Sie den folgenden Code in die Datei ein.</span><span class="sxs-lookup"><span data-stu-id="3b82e-131">Add the following code in that file.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    <span data-ttu-id="3b82e-132">Dieser Code fügt [Bootstrap](http://getbootstrap.com/) zur einfachen Formatierung hinzu, und [Font Awesome](https://fontawesome.com/) für einige einfache Symbole.</span><span class="sxs-lookup"><span data-stu-id="3b82e-132">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="3b82e-133">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="3b82e-133">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="3b82e-134">Erstellen Sie ein neues Verzeichnis im **./Tutorial** -Verzeichnis `static`mit dem Namen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-134">Create a new directory in the **./tutorial** directory named `static`.</span></span>

1. <span data-ttu-id="3b82e-135">Erstellen Sie im Verzeichnis **./Tutorial/static** ein neues Verzeichnis mit dem `tutorial`Namen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-135">In the **./tutorial/static** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="3b82e-136">Erstellen Sie im Verzeichnis **./Tutorial/static/Tutorial** eine neue Datei mit dem `app.css`Namen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-136">In the **./tutorial/static/tutorial** directory, create a new file named `app.css`.</span></span> <span data-ttu-id="3b82e-137">Fügen Sie den folgenden Code in die Datei ein.</span><span class="sxs-lookup"><span data-stu-id="3b82e-137">Add the following code in that file.</span></span>

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. <span data-ttu-id="3b82e-138">Erstellen Sie eine Vorlage für die Startseite, die das Layout verwendet.</span><span class="sxs-lookup"><span data-stu-id="3b82e-138">Create a template for the home page that uses the layout.</span></span> <span data-ttu-id="3b82e-139">Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis `home.html` mit dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-139">Create a new file in the **./tutorial/templates/tutorial** directory named `home.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. <span data-ttu-id="3b82e-140">Öffnen Sie `./tutorial/views.py` die Datei, und fügen Sie die folgende neue Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-140">Open the `./tutorial/views.py` file and add the following new function.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. <span data-ttu-id="3b82e-141">Ersetzen Sie die `home` vorhandene Ansicht durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="3b82e-141">Replace the existing `home` view with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. <span data-ttu-id="3b82e-142">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="3b82e-142">Save all of your changes and restart the server.</span></span> <span data-ttu-id="3b82e-143">Nun sollte die APP sehr unterschiedlich aussehen.</span><span class="sxs-lookup"><span data-stu-id="3b82e-143">Now, the app should look very different.</span></span>

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
