<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung integrieren Sie Microsoft Graph in die Anwendung. Für diese Anwendung verwenden Sie die Requests [-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek, um Aufrufe von Microsoft Graph zu tätigen.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen aus Outlook

Beginnen Sie mit dem Hinzufügen `./tutorial/graph_helper.py` einer Methode zum Abrufen der Kalenderereignisse. Fügen Sie die folgende Methode hinzu.

```python
def get_calendar_events(token):
  graph_client = OAuth2Session(token=token)

  # Configure query parameters to
  # modify the results
  query_params = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  # Send GET to /me/events
  events = graph_client.get('{0}/me/events'.format(graph_url), params=query_params)
  # Return the JSON result
  return events.json()
```

Überlegen Sie sich, was dieser Code tut.

- Die URL, die aufgerufen wird, `/v1.0/me/events`lautet.
- Der `$select` Parameter schränkt die für jedes Ereignis zurückgegebenen Felder auf diejenigen ein, die die Ansicht tatsächlich verwendet.
- Der `$orderby` Parameter sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu denen Sie erstellt wurden, wobei das neueste Element zuerst angezeigt wird.

Erstellen Sie nun eine Kalenderansicht. Ändern Sie zunächst `from tutorial.graph_helper import get_user` die folgende Codezeile.

```python
from tutorial.graph_helper import get_user, get_calendar_events
```

Fügen Sie dann die folgende Ansicht zu `./tutorial/views.py`hinzu.

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

Aktualisieren `./tutorial/urls.py` , um diese neue Ansicht hinzuzufügen.

```python
path('calendar', views.calendar, name='calendar'),
```

Aktualisieren Sie schließlich den Link **Kalender** in `./tutorial/templates/tutorial/layout.html` , um mit dieser Ansicht zu verknüpfen. Ersetzen Sie `<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>` die-Reihe durch Folgendes.

```html
<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="{% url 'calendar' %}">Calendar</a>
```

Jetzt können Sie dies testen. Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** . Wenn alles funktioniert, sollte ein JSON-Dump von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie eine Vorlage hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen. Erstellen Sie eine neue Datei im `./tutorial/templates/tutorial` Verzeichnis mit `calendar.html` dem Namen, und fügen Sie den folgenden Code hinzu.

```html
{% extends "tutorial/layout.html" %}
{% block content %}
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    {% if events %}
      {% for event in events %}
        <tr>
          <td>{{ event.organizer.emailAddress.name }}</td>
          <td>{{ event.subject }}</td>
          <td>{{ event.start.dateTime|date:'SHORT_DATETIME_FORMAT' }}</td>
          <td>{{ event.end.dateTime|date:'SHORT_DATETIME_FORMAT' }}</td>
        </tr>
      {% endfor %}
    {% endif %}
  </tbody>
</table>
{% endblock %}
```

, Der eine Auflistung von Ereignissen durchläuft und jeweils eine Tabellenzeile hinzufügt. Fügen Sie die `import` folgende Anweisung am Anfang der `./tutorials/views.py` Datei hinzu.

```python
import dateutil.parser
```

Ersetzen Sie `calendar` die Ansicht `./tutorial/views.py` in durch den folgenden Code.

```python
def calendar(request):
  context = initialize_context(request)

  token = get_token(request)

  events = get_calendar_events(token)

  if events:
    # Convert the ISO 8601 date times to a datetime object
    # This allows the Django template to format the value nicely
    for event in events['value']:
      event['start']['dateTime'] = dateutil.parser.parse(event['start']['dateTime'])
      event['end']['dateTime'] = dateutil.parser.parse(event['end']['dateTime'])

    context['events'] = events['value']

  return render(request, 'tutorial/calendar.html', context)
```

Aktualisieren Sie die Seite, und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.

![Screenshot der Ereignistabelle](./images/add-msgraph-01.png)