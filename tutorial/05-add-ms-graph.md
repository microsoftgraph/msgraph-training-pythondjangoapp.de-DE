<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie das Microsoft Graph in die Anwendung integrieren. Für diese Anwendung verwenden Sie die [Requests-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek, um Anrufe an Microsoft Graph zu tätigen.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen von Outlook

1. Fügen Sie zunächst eine Methode zu **./Tutorial/graph_helper. py** hinzu, um die Kalenderereignisse abzurufen. Fügen Sie die folgende Methode hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    Überlegen Sie sich, was dieser Code macht.

    - Die URL, die aufgerufen wird, lautet `/v1.0/me/events`.
    - Der `$select` Parameter schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf diejenigen ein, die von der Ansicht tatsächlich verwendet werden.
    - Der `$orderby` Parameter sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu der Sie erstellt wurden, wobei das letzte Element zuerst angezeigt wird.

1. Ändern Sie in **./Tutorial/views.py**die `from tutorial.graph_helper import get_user` folgende Verbindung.

    ```python
    from tutorial.graph_helper import get_user, get_calendar_events
    ```

1. Fügen Sie die folgende Ansicht zu **./Tutorial/views.py**.

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

1. Öffnen Sie **./Tutorial/URLs.py** , und ersetzen `path` Sie die `calendar` vorhandenen Anweisungen für folgendermaßen.

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** . Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie eine Vorlage hinzufügen, um die Ergebnisse auf eine benutzerfreundlichere Weise anzuzeigen.

1. Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis `calendar.html` mit dem Namen, und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.

1. Fügen Sie am `import` Anfang der Datei **./Tutorials/views.py** die folgende Anweisung hinzu.

    ```python
    import dateutil.parser
    ```

1. Ersetzen Sie `calendar` die Ansicht in **./Tutorial/views.py** durch den folgenden Code.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
