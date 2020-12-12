<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b83ed-101">In dieser Übung werden Sie das Microsoft Graph in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="b83ed-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="b83ed-102">Abrufen von Kalenderereignissen von Outlook</span><span class="sxs-lookup"><span data-stu-id="b83ed-102">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="b83ed-103">Beginnen Sie mit dem Hinzufügen einer Methode zu **./Tutorial/graph_helper. py** , um eine Ansicht des Kalenders zwischen zwei Datumsangaben abzurufen.</span><span class="sxs-lookup"><span data-stu-id="b83ed-103">Start by adding a method to **./tutorial/graph_helper.py** to fetch a view of the calendar between two dates.</span></span> <span data-ttu-id="b83ed-104">Fügen Sie die folgende Methode hinzu.</span><span class="sxs-lookup"><span data-stu-id="b83ed-104">Add the following method.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    <span data-ttu-id="b83ed-105">Überlegen Sie sich, was dieser Code macht.</span><span class="sxs-lookup"><span data-stu-id="b83ed-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="b83ed-106">Die URL, die aufgerufen wird, lautet `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="b83ed-106">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="b83ed-107">Die `Prefer: outlook.timezone` Kopfzeile bewirkt, dass die Anfangs-und Endzeiten in den Ergebnissen an die Zeitzone des Benutzers angepasst werden.</span><span class="sxs-lookup"><span data-stu-id="b83ed-107">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="b83ed-108">Die `startDateTime` `endDateTime` Parameter und legen den Anfang und das Ende der Ansicht fest.</span><span class="sxs-lookup"><span data-stu-id="b83ed-108">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="b83ed-109">Der `$select` Parameter schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf diejenigen ein, die von der Ansicht tatsächlich verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="b83ed-109">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="b83ed-110">Der `$orderby` Parameter sortiert die Ergebnisse nach dem Startzeitpunkt.</span><span class="sxs-lookup"><span data-stu-id="b83ed-110">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="b83ed-111">Der `$top` Parameter schränkt die Ergebnisse auf 50-Ereignisse ein.</span><span class="sxs-lookup"><span data-stu-id="b83ed-111">The `$top` parameter limits the results to 50 events.</span></span>

1. <span data-ttu-id="b83ed-112">Fügen Sie den folgenden Code zu **./Tutorial/graph_helper. py** hinzu, um einen [IANA-Zeitzonenbezeichner](https://www.iana.org/time-zones) basierend auf einem Windows-Zeitzonennamen zu suchen.</span><span class="sxs-lookup"><span data-stu-id="b83ed-112">Add the following code to **./tutorial/graph_helper.py** to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="b83ed-113">Dies ist erforderlich, da Microsoft Graph Zeitzonen als Windows-Zeitzonennamen zurückgeben kann, und die python- **DateTime** -Bibliothek erfordert IANA-Zeitzonenbezeichner.</span><span class="sxs-lookup"><span data-stu-id="b83ed-113">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Python **datetime** library requires IANA time zone identifiers.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="ZoneMappingsSnippet":::

1. <span data-ttu-id="b83ed-114">Fügen Sie die folgende Ansicht zu **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="b83ed-114">Add the following view to **./tutorial/views.py**.</span></span>

    ```python
    def calendar(request):
      context = initialize_context(request)
      user = context['user']

      # Load the user's time zone
      # Microsoft Graph can return the user's time zone as either
      # a Windows time zone name or an IANA time zone identifier
      # Python datetime requires IANA, so convert Windows to IANA
      time_zone = get_iana_from_windows(user['timeZone'])
      tz_info = tz.gettz(time_zone)

      # Get midnight today in user's time zone
      today = datetime.now(tz_info).replace(
        hour=0,
        minute=0,
        second=0,
        microsecond=0)

      # Based on today, get the start of the week (Sunday)
      if (today.weekday() != 6):
        start = today - timedelta(days=today.isoweekday())
      else:
        start = today

      end = start + timedelta(days=7)

      token = get_token(request)

      events = get_calendar_events(
        token,
        start.isoformat(timespec='seconds'),
        end.isoformat(timespec='seconds'),
        user['timeZone'])

      context['errors'] = [
        { 'message': 'Events', 'debug': format(events)}
      ]

      return render(request, 'tutorial/home.html', context)
    ```

1. <span data-ttu-id="b83ed-115">Öffnen Sie **./Tutorial/URLs.py** , und ersetzen Sie die vorhandenen `path` Anweisungen für `calendar` folgendermaßen.</span><span class="sxs-lookup"><span data-stu-id="b83ed-115">Open **./tutorial/urls.py** and replace the existing `path` statements for `calendar` with the following.</span></span>

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. <span data-ttu-id="b83ed-116">Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** .</span><span class="sxs-lookup"><span data-stu-id="b83ed-116">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="b83ed-117">Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="b83ed-117">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="b83ed-118">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="b83ed-118">Display the results</span></span>

<span data-ttu-id="b83ed-119">Jetzt können Sie eine Vorlage hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="b83ed-119">Now you can add a template to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="b83ed-120">Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis mit dem Namen `calendar.html` , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="b83ed-120">Create a new file in the **./tutorial/templates/tutorial** directory named `calendar.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    <span data-ttu-id="b83ed-121">Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="b83ed-121">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="b83ed-122">Ersetzen Sie die `calendar` Ansicht in **./Tutorial/views.py** durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="b83ed-122">Replace the `calendar` view in **./tutorial/views.py** with the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. <span data-ttu-id="b83ed-123">Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="b83ed-123">Refresh the page and the app should now render a table of events.</span></span>

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
