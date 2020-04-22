<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="96de6-101">In dieser Übung werden Sie das Microsoft Graph in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="96de6-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="96de6-102">Für diese Anwendung verwenden Sie die [Requests-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek, um Anrufe an Microsoft Graph zu tätigen.</span><span class="sxs-lookup"><span data-stu-id="96de6-102">For this application, you will use the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="96de6-103">Abrufen von Kalenderereignissen von Outlook</span><span class="sxs-lookup"><span data-stu-id="96de6-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="96de6-104">Fügen Sie zunächst eine Methode zu **./Tutorial/graph_helper. py** hinzu, um die Kalenderereignisse abzurufen.</span><span class="sxs-lookup"><span data-stu-id="96de6-104">Start by adding a method to **./tutorial/graph_helper.py** to fetch the calendar events.</span></span> <span data-ttu-id="96de6-105">Fügen Sie die folgende Methode hinzu.</span><span class="sxs-lookup"><span data-stu-id="96de6-105">Add the following method.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    <span data-ttu-id="96de6-106">Überlegen Sie sich, was dieser Code macht.</span><span class="sxs-lookup"><span data-stu-id="96de6-106">Consider what this code is doing.</span></span>

    - <span data-ttu-id="96de6-107">Die URL, die aufgerufen wird, lautet `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="96de6-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="96de6-108">Der `$select` Parameter schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf diejenigen ein, die von der Ansicht tatsächlich verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="96de6-108">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="96de6-109">Der `$orderby` Parameter sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu der Sie erstellt wurden, wobei das letzte Element zuerst angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="96de6-109">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="96de6-110">Ändern Sie in **./Tutorial/views.py**die `from tutorial.graph_helper import get_user` folgende Verbindung.</span><span class="sxs-lookup"><span data-stu-id="96de6-110">In **./tutorial/views.py**, change the `from tutorial.graph_helper import get_user` line to the following.</span></span>

    ```python
    from tutorial.graph_helper import get_user, get_calendar_events
    ```

1. <span data-ttu-id="96de6-111">Fügen Sie die folgende Ansicht zu **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="96de6-111">Add the following view to **./tutorial/views.py**.</span></span>

    ```python
    def calendar(request):
      context = initialize_context(request)

      token = get_token(request)

      events = get_calendar_events(token)

      context['errors'] = [
        { 'message': 'Events', 'debug': format(events)}
      ]

      return render(request, 'tutorial/home.html', context)
    ```

1. <span data-ttu-id="96de6-112">Öffnen Sie **./Tutorial/URLs.py** , und ersetzen `path` Sie die `calendar` vorhandenen Anweisungen für folgendermaßen.</span><span class="sxs-lookup"><span data-stu-id="96de6-112">Open **./tutorial/urls.py** and replace the existing `path` statements for `calendar` with the following.</span></span>

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. <span data-ttu-id="96de6-113">Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** .</span><span class="sxs-lookup"><span data-stu-id="96de6-113">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="96de6-114">Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="96de6-114">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="96de6-115">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="96de6-115">Display the results</span></span>

<span data-ttu-id="96de6-116">Jetzt können Sie eine Vorlage hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="96de6-116">Now you can add a template to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="96de6-117">Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis `calendar.html` mit dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="96de6-117">Create a new file in the **./tutorial/templates/tutorial** directory named `calendar.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    <span data-ttu-id="96de6-118">Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="96de6-118">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="96de6-119">Fügen Sie am `import` Anfang der Datei **./Tutorials/views.py** die folgende Anweisung hinzu.</span><span class="sxs-lookup"><span data-stu-id="96de6-119">Add the following `import` statement to the top of the **./tutorials/views.py** file.</span></span>

    ```python
    import dateutil.parser
    ```

1. <span data-ttu-id="96de6-120">Ersetzen Sie `calendar` die Ansicht in **./Tutorial/views.py** durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="96de6-120">Replace the `calendar` view in **./tutorial/views.py** with the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. <span data-ttu-id="96de6-121">Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="96de6-121">Refresh the page and the app should now render a table of events.</span></span>

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
