<!-- markdownlint-disable MD002 MD041 -->

In diesem Abschnitt können Sie die Möglichkeit zum Erstellen von Ereignissen im Kalender des Benutzers hinzufügen.

1. Fügen Sie die folgende Methode zu **./Tutorial/graph_helper. py** hinzu, um ein neues Ereignis zu erstellen.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="CreateEventSnippet":::

## <a name="create-a-new-event-form"></a>Erstellen eines neuen Ereignis Formulars

1. Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis mit dem Namen `newevent.html` , und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/newevent.html" id="NewEventSnippet":::

1. Fügen Sie die folgende Ansicht zu **./Tutorial/views.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="NewEventViewSnippet":::

1. Öffnen Sie **/Tutorial/URLs.py** , und fügen Sie eine `path` Anweisung für die `newevent` Ansicht hinzu.

    ```python
    path('calendar/new', views.newevent, name='newevent'),
    ```

1. Speichern Sie Ihre Änderungen, und aktualisieren Sie die app. Klicken Sie auf der Seite **Kalender** auf die Schaltfläche **Neues Ereignis** . Füllen Sie das Formular aus, und wählen Sie **Erstellen** aus, um das Ereignis zu erstellen.

    ![Screenshot des neuen Ereignis Formulars](images/create-event-01.png)
