<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie das Microsoft Graph in die Anwendung integrieren.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen von Outlook

1. Beginnen Sie mit dem Hinzufügen einer Methode zu **./Tutorial/graph_helper. py** , um eine Ansicht des Kalenders zwischen zwei Datumsangaben abzurufen. Fügen Sie die folgende Methode hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    Überlegen Sie sich, was dieser Code macht.

    - Die URL, die aufgerufen wird, lautet `/v1.0/me/calendarview`.
        - Die `Prefer: outlook.timezone` Kopfzeile bewirkt, dass die Anfangs-und Endzeiten in den Ergebnissen an die Zeitzone des Benutzers angepasst werden.
        - Die `startDateTime` `endDateTime` Parameter und legen den Anfang und das Ende der Ansicht fest.
        - Der `$select` Parameter schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf diejenigen ein, die von der Ansicht tatsächlich verwendet werden.
        - Der `$orderby` Parameter sortiert die Ergebnisse nach dem Startzeitpunkt.
        - Der `$top` Parameter schränkt die Ergebnisse auf 50-Ereignisse ein.

1. Fügen Sie den folgenden Code zu **./Tutorial/graph_helper. py** hinzu, um einen [IANA-Zeitzonenbezeichner](https://www.iana.org/time-zones) basierend auf einem Windows-Zeitzonennamen zu suchen. Dies ist erforderlich, da Microsoft Graph Zeitzonen als Windows-Zeitzonennamen zurückgeben kann, und die python- **DateTime** -Bibliothek erfordert IANA-Zeitzonenbezeichner.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="ZoneMappingsSnippet":::

1. Fügen Sie die folgende Ansicht zu **./Tutorial/views.py**.

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

1. Öffnen Sie **./Tutorial/URLs.py** , und ersetzen Sie die vorhandenen `path` Anweisungen für `calendar` folgendermaßen.

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** . Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie eine Vorlage hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.

1. Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis mit dem Namen `calendar.html` , und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.

1. Ersetzen Sie die `calendar` Ansicht in **./Tutorial/views.py** durch den folgenden Code.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
