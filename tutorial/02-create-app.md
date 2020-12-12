<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b6fd3-101">In dieser Übung werden Sie mit [Django](https://www.djangoproject.com/) eine Webanwendung erstellen.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span>

1. <span data-ttu-id="b6fd3-102">Wenn Django nicht bereits installiert ist, können Sie es über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    pip install --user Django==3.1.4
    ```

1. <span data-ttu-id="b6fd3-103">Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie Berechtigungen zum Erstellen von Dateien haben, und führen Sie den folgenden Befehl aus, um eine neue Django-APP zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. <span data-ttu-id="b6fd3-104">Navigieren Sie zum **graph_tutorial** Verzeichnis, und geben Sie den folgenden Befehl ein, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-104">Navigate to the **graph_tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="b6fd3-105">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-105">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="b6fd3-106">Wenn alles funktioniert, wird eine Django-Willkommensseite angezeigt.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-106">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="b6fd3-107">Wenn dies nicht der Fall ist, überprüfen Sie das [Django-Handbuch für erste Schritte](https://www.djangoproject.com/start/).</span><span class="sxs-lookup"><span data-stu-id="b6fd3-107">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

1. <span data-ttu-id="b6fd3-108">Fügen Sie dem Projekt eine APP hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-108">Add an app to the project.</span></span> <span data-ttu-id="b6fd3-109">Führen Sie den folgenden Befehl in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-109">Run the following command in your CLI.</span></span>

    ```Shell
    python manage.py startapp tutorial
    ```

1. <span data-ttu-id="b6fd3-110">Öffnen Sie **./graph_tutorial/Settings.py** , und fügen Sie die neue `tutorial` app zu der `INSTALLED_APPS` Einstellung hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-110">Open **./graph_tutorial/settings.py** and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. <span data-ttu-id="b6fd3-111">Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank für das Projekt zu initialisieren.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-111">In your CLI, run the following command to initialize the database for the project.</span></span>

    ```Shell
    python manage.py migrate
    ```

1. <span data-ttu-id="b6fd3-112">Fügen Sie ein [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) für die `tutorial` app hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-112">Add a [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="b6fd3-113">Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis mit dem Namen `urls.py` , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-113">Create a new file in the **./tutorial** directory named `urls.py` and add the following code.</span></span>

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

1. <span data-ttu-id="b6fd3-114">Aktualisieren Sie das Projekt URLconf, um dieses zu importieren.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-114">Update the project URLconf to import this one.</span></span> <span data-ttu-id="b6fd3-115">Öffnen Sie **graph_tutorial/URLs.py** , und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-115">Open **./graph_tutorial/urls.py** and replace the contents with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. <span data-ttu-id="b6fd3-116">Fügen Sie der App eine temporäre Ansicht hinzu, `tutorials` um zu überprüfen, ob das URL-Routing funktioniert.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-116">Add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="b6fd3-117">Öffnen Sie **./Tutorial/views.py** , und ersetzen Sie den gesamten Inhalt durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-117">Open **./tutorial/views.py** and replace its entire contents with the following code.</span></span>

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect
    from django.urls import reverse
    from datetime import datetime, timedelta
    from dateutil import tz, parser

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. <span data-ttu-id="b6fd3-118">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-118">Save all of your changes and restart the server.</span></span> <span data-ttu-id="b6fd3-119">Navigieren Sie zu `http://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="b6fd3-119">Browse to `http://localhost:8000`.</span></span> <span data-ttu-id="b6fd3-120">Sie sollten sehen `Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="b6fd3-120">You should see `Welcome to the tutorial.`</span></span>

## <a name="install-libraries"></a><span data-ttu-id="b6fd3-121">Installieren von Bibliotheken</span><span class="sxs-lookup"><span data-stu-id="b6fd3-121">Install libraries</span></span>

<span data-ttu-id="b6fd3-122">Installieren Sie vor dem Verschieben einige zusätzliche Bibliotheken, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="b6fd3-122">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="b6fd3-123">[Microsoft Authentication Library (MSAL) für python](https://github.com/AzureAD/microsoft-authentication-library-for-python) zur Verarbeitung von Anmelde-und OAuth-Token-Flows.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-123">[Microsoft Authentication Library (MSAL) for Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="b6fd3-124">[Anforderungen: http für Menschen](https://requests.readthedocs.io/en/master/) zum tätigen von Anrufen an Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-124">[Requests: HTTP for Humans](https://requests.readthedocs.io/en/master/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="b6fd3-125">[Autoyaml](https://pyyaml.org/wiki/PyYAMLDocumentation) zum Laden der Konfiguration aus einer YAML-Datei.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-125">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="b6fd3-126">[python-dateutil](https://pypi.org/project/python-dateutil/) zum Analysieren von ISO 8601-Datumszeichenfolgen, die von Microsoft Graph zurückgegeben werden.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-126">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

1. <span data-ttu-id="b6fd3-127">Führen Sie den folgenden Befehl in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-127">Run the following command in your CLI.</span></span>

    ```Shell
    pip install msal==1.7.0
    pip install requests==2.25.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a><span data-ttu-id="b6fd3-128">Entwerfen der App</span><span class="sxs-lookup"><span data-stu-id="b6fd3-128">Design the app</span></span>

1. <span data-ttu-id="b6fd3-129">Erstellen Sie ein neues Verzeichnis im **./Tutorial** -Verzeichnis mit dem Namen `templates` .</span><span class="sxs-lookup"><span data-stu-id="b6fd3-129">Create a new directory in the **./tutorial** directory named `templates`.</span></span>

1. <span data-ttu-id="b6fd3-130">Erstellen Sie im Verzeichnis **./Tutorial/Templates** ein neues Verzeichnis mit dem Namen `tutorial` .</span><span class="sxs-lookup"><span data-stu-id="b6fd3-130">In the **./tutorial/templates** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="b6fd3-131">Erstellen Sie im Verzeichnis **./Tutorial/Templates/Tutorial** eine neue Datei mit dem Namen `layout.html` .</span><span class="sxs-lookup"><span data-stu-id="b6fd3-131">In the **./tutorial/templates/tutorial** directory, create a new file named `layout.html`.</span></span> <span data-ttu-id="b6fd3-132">Fügen Sie den folgenden Code in die Datei ein.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-132">Add the following code in that file.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    <span data-ttu-id="b6fd3-133">Dieser Code fügt [Bootstrap](http://getbootstrap.com/) für einfaches Styling und [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) für einige einfache Symbole hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-133">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="b6fd3-134">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-134">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="b6fd3-135">Erstellen Sie ein neues Verzeichnis im **./Tutorial** -Verzeichnis mit dem Namen `static` .</span><span class="sxs-lookup"><span data-stu-id="b6fd3-135">Create a new directory in the **./tutorial** directory named `static`.</span></span>

1. <span data-ttu-id="b6fd3-136">Erstellen Sie im Verzeichnis **./Tutorial/static** ein neues Verzeichnis mit dem Namen `tutorial` .</span><span class="sxs-lookup"><span data-stu-id="b6fd3-136">In the **./tutorial/static** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="b6fd3-137">Erstellen Sie im Verzeichnis **./Tutorial/static/Tutorial** eine neue Datei mit dem Namen `app.css` .</span><span class="sxs-lookup"><span data-stu-id="b6fd3-137">In the **./tutorial/static/tutorial** directory, create a new file named `app.css`.</span></span> <span data-ttu-id="b6fd3-138">Fügen Sie den folgenden Code in die Datei ein.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-138">Add the following code in that file.</span></span>

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. <span data-ttu-id="b6fd3-139">Erstellen Sie eine Vorlage für die Startseite, die das Layout verwendet.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-139">Create a template for the home page that uses the layout.</span></span> <span data-ttu-id="b6fd3-140">Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis mit dem Namen `home.html` , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-140">Create a new file in the **./tutorial/templates/tutorial** directory named `home.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. <span data-ttu-id="b6fd3-141">Öffnen `./tutorial/views.py` Sie die Datei, und fügen Sie die folgende neue Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-141">Open the `./tutorial/views.py` file and add the following new function.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. <span data-ttu-id="b6fd3-142">Ersetzen Sie die vorhandene `home` Ansicht durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-142">Replace the existing `home` view with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. <span data-ttu-id="b6fd3-143">Fügen Sie im Verzeichnis **./Tutorial/static/Tutorial** eine PNG-Datei mit dem Namen **no-profile-photo.png** hinzu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-143">Add a PNG file named **no-profile-photo.png** in the **./tutorial/static/tutorial** directory.</span></span>

1. <span data-ttu-id="b6fd3-144">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-144">Save all of your changes and restart the server.</span></span> <span data-ttu-id="b6fd3-145">Nun sollte die APP sehr unterschiedlich aussehen.</span><span class="sxs-lookup"><span data-stu-id="b6fd3-145">Now, the app should look very different.</span></span>

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
