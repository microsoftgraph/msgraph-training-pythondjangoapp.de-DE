<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="cf571-101">In dieser Übung verwenden Sie [Django,](https://www.djangoproject.com/) um eine Web-App zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="cf571-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span>

1. <span data-ttu-id="cf571-102">Wenn Sie Django noch nicht installiert haben, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.</span><span class="sxs-lookup"><span data-stu-id="cf571-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    pip install Django==3.2
    ```

1. <span data-ttu-id="cf571-103">Öffnen Sie Ihre CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über die Rechte zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Django-App zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="cf571-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. <span data-ttu-id="cf571-104">Navigieren Sie zum **graph_tutorial,** und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="cf571-104">Navigate to the **graph_tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="cf571-105">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="cf571-105">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="cf571-106">Wenn alles funktioniert, wird eine Django-Willkommensseite angezeigt.</span><span class="sxs-lookup"><span data-stu-id="cf571-106">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="cf571-107">Wenn Sie dies nicht sehen, überprüfen Sie die [Anleitung django erste Schritte](https://www.djangoproject.com/start/).</span><span class="sxs-lookup"><span data-stu-id="cf571-107">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

1. <span data-ttu-id="cf571-108">Fügen Sie dem Projekt eine App hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-108">Add an app to the project.</span></span> <span data-ttu-id="cf571-109">Führen Sie den folgenden Befehl in Ihrer CLI aus.</span><span class="sxs-lookup"><span data-stu-id="cf571-109">Run the following command in your CLI.</span></span>

    ```Shell
    python manage.py startapp tutorial
    ```

1. <span data-ttu-id="cf571-110">Öffnen **Sie ./graph_tutorial/settings.py,** und fügen Sie der Einstellung die `tutorial` neue App `INSTALLED_APPS` hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-110">Open **./graph_tutorial/settings.py** and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. <span data-ttu-id="cf571-111">Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank für das Projekt zu initialisieren.</span><span class="sxs-lookup"><span data-stu-id="cf571-111">In your CLI, run the following command to initialize the database for the project.</span></span>

    ```Shell
    python manage.py migrate
    ```

1. <span data-ttu-id="cf571-112">Fügen Sie einen [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) für die `tutorial` App hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-112">Add a [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="cf571-113">Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial** `urls.py` namens, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-113">Create a new file in the **./tutorial** directory named `urls.py` and add the following code.</span></span>

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

1. <span data-ttu-id="cf571-114">Aktualisieren Sie das Projekt URLconf, um dieses zu importieren.</span><span class="sxs-lookup"><span data-stu-id="cf571-114">Update the project URLconf to import this one.</span></span> <span data-ttu-id="cf571-115">Öffnen **Sie ./graph_tutorial/urls.py,** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="cf571-115">Open **./graph_tutorial/urls.py** and replace the contents with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. <span data-ttu-id="cf571-116">Fügen Sie der App eine temporäre Ansicht `tutorials` hinzu, um zu überprüfen, ob das URL-Routing funktioniert.</span><span class="sxs-lookup"><span data-stu-id="cf571-116">Add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="cf571-117">Öffnen **Sie ./tutorial/views.py,** und ersetzen Sie den gesamten Inhalt durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="cf571-117">Open **./tutorial/views.py** and replace its entire contents with the following code.</span></span>

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect
    from django.urls import reverse
    from datetime import datetime, timedelta

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. <span data-ttu-id="cf571-118">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="cf571-118">Save all of your changes and restart the server.</span></span> <span data-ttu-id="cf571-119">Navigieren Sie zu `http://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="cf571-119">Browse to `http://localhost:8000`.</span></span> <span data-ttu-id="cf571-120">Sie sollten sehen `Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="cf571-120">You should see `Welcome to the tutorial.`</span></span>

## <a name="install-libraries"></a><span data-ttu-id="cf571-121">Installieren von Bibliotheken</span><span class="sxs-lookup"><span data-stu-id="cf571-121">Install libraries</span></span>

<span data-ttu-id="cf571-122">Installieren Sie vor dem Wechsel einige zusätzliche Bibliotheken, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="cf571-122">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="cf571-123">[Microsoft Authentication Library (MSAL) für Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) für die Verarbeitung von Anmelde- und OAuth-Tokenflüssen.</span><span class="sxs-lookup"><span data-stu-id="cf571-123">[Microsoft Authentication Library (MSAL) for Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="cf571-124">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) zum Laden der Konfiguration aus einer YAML-Datei.</span><span class="sxs-lookup"><span data-stu-id="cf571-124">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="cf571-125">[python-dateutil](https://pypi.org/project/python-dateutil/) für die Analyse von ISO 8601-Datumszeichenfolgen, die von Microsoft Graph zurückgegeben werden.</span><span class="sxs-lookup"><span data-stu-id="cf571-125">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

1. <span data-ttu-id="cf571-126">Führen Sie den folgenden Befehl in Ihrer CLI aus.</span><span class="sxs-lookup"><span data-stu-id="cf571-126">Run the following command in your CLI.</span></span>

    ```Shell
    pip install msal==1.10.0
    pip install pyyaml==5.4.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a><span data-ttu-id="cf571-127">Entwerfen der App</span><span class="sxs-lookup"><span data-stu-id="cf571-127">Design the app</span></span>

1. <span data-ttu-id="cf571-128">Erstellen Sie ein neues Verzeichnis im **Verzeichnis ./tutorial** mit dem Namen `templates` .</span><span class="sxs-lookup"><span data-stu-id="cf571-128">Create a new directory in the **./tutorial** directory named `templates`.</span></span>

1. <span data-ttu-id="cf571-129">Erstellen Sie **im Verzeichnis ./tutorial/templates** ein neues Verzeichnis mit dem Namen `tutorial` .</span><span class="sxs-lookup"><span data-stu-id="cf571-129">In the **./tutorial/templates** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="cf571-130">Erstellen Sie **im Verzeichnis ./tutorial/templates/tutorial** eine neue Datei namens `layout.html` .</span><span class="sxs-lookup"><span data-stu-id="cf571-130">In the **./tutorial/templates/tutorial** directory, create a new file named `layout.html`.</span></span> <span data-ttu-id="cf571-131">Fügen Sie den folgenden Code in dieser Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-131">Add the following code in that file.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    <span data-ttu-id="cf571-132">Dieser Code fügt [Bootstrap für](http://getbootstrap.com/) einfache Formatierung und [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) für einige einfache Symbole hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-132">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="cf571-133">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="cf571-133">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="cf571-134">Erstellen Sie ein neues Verzeichnis im **Verzeichnis ./tutorial** mit dem Namen `static` .</span><span class="sxs-lookup"><span data-stu-id="cf571-134">Create a new directory in the **./tutorial** directory named `static`.</span></span>

1. <span data-ttu-id="cf571-135">Erstellen Sie **im Verzeichnis ./tutorial/static** ein neues Verzeichnis mit dem Namen `tutorial` .</span><span class="sxs-lookup"><span data-stu-id="cf571-135">In the **./tutorial/static** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="cf571-136">Erstellen Sie **im Verzeichnis ./tutorial/static/tutorial** eine neue Datei mit dem Namen `app.css` .</span><span class="sxs-lookup"><span data-stu-id="cf571-136">In the **./tutorial/static/tutorial** directory, create a new file named `app.css`.</span></span> <span data-ttu-id="cf571-137">Fügen Sie den folgenden Code in dieser Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-137">Add the following code in that file.</span></span>

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. <span data-ttu-id="cf571-138">Erstellen Sie eine Vorlage für die Homepage, die das Layout verwendet.</span><span class="sxs-lookup"><span data-stu-id="cf571-138">Create a template for the home page that uses the layout.</span></span> <span data-ttu-id="cf571-139">Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial/templates/tutorial** `home.html` namens, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-139">Create a new file in the **./tutorial/templates/tutorial** directory named `home.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. <span data-ttu-id="cf571-140">Öffnen Sie `./tutorial/views.py` die Datei, und fügen Sie die folgende neue Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-140">Open the `./tutorial/views.py` file and add the following new function.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. <span data-ttu-id="cf571-141">Ersetzen Sie die `home` vorhandene Ansicht durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="cf571-141">Replace the existing `home` view with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. <span data-ttu-id="cf571-142">Fügen Sie eine PNG-Datei **namensno-profile-photo.png** im **Verzeichnis ./tutorial/static/tutorial** hinzu.</span><span class="sxs-lookup"><span data-stu-id="cf571-142">Add a PNG file named **no-profile-photo.png** in the **./tutorial/static/tutorial** directory.</span></span>

1. <span data-ttu-id="cf571-143">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="cf571-143">Save all of your changes and restart the server.</span></span> <span data-ttu-id="cf571-144">Jetzt sollte die App ganz anders aussehen.</span><span class="sxs-lookup"><span data-stu-id="cf571-144">Now, the app should look very different.</span></span>

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
