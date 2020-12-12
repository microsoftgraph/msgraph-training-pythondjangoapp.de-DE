<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="48a37-101">In diesem Abschnitt können Sie die Möglichkeit zum Erstellen von Ereignissen im Kalender des Benutzers hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="48a37-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="48a37-102">Fügen Sie die folgende Methode zu **./Tutorial/graph_helper. py** hinzu, um ein neues Ereignis zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="48a37-102">Add the following method to **./tutorial/graph_helper.py** to create a new event.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="CreateEventSnippet":::

## <a name="create-a-new-event-form"></a><span data-ttu-id="48a37-103">Erstellen eines neuen Ereignis Formulars</span><span class="sxs-lookup"><span data-stu-id="48a37-103">Create a new event form</span></span>

1. <span data-ttu-id="48a37-104">Erstellen Sie eine neue Datei im **./Tutorial/Templates/Tutorial** -Verzeichnis mit dem Namen `newevent.html` , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="48a37-104">Create a new file in the **./tutorial/templates/tutorial** directory named `newevent.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/newevent.html" id="NewEventSnippet":::

1. <span data-ttu-id="48a37-105">Fügen Sie die folgende Ansicht zu **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="48a37-105">Add the following view to **./tutorial/views.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="NewEventViewSnippet":::

1. <span data-ttu-id="48a37-106">Öffnen Sie **/Tutorial/URLs.py** , und fügen Sie eine `path` Anweisung für die `newevent` Ansicht hinzu.</span><span class="sxs-lookup"><span data-stu-id="48a37-106">Open **./tutorial/urls.py** and add a `path` statements for the `newevent` view.</span></span>

    ```python
    path('calendar/new', views.newevent, name='newevent'),
    ```

1. <span data-ttu-id="48a37-107">Speichern Sie Ihre Änderungen, und aktualisieren Sie die app.</span><span class="sxs-lookup"><span data-stu-id="48a37-107">Save your changes and refresh the app.</span></span> <span data-ttu-id="48a37-108">Klicken Sie auf der Seite **Kalender** auf die Schaltfläche **Neues Ereignis** .</span><span class="sxs-lookup"><span data-stu-id="48a37-108">On the **Calendar** page, select the **New event** button.</span></span> <span data-ttu-id="48a37-109">Füllen Sie das Formular aus, und wählen Sie **Erstellen** aus, um das Ereignis zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="48a37-109">Fill in the form and select **Create** to create the event.</span></span>

    ![Screenshot des neuen Ereignis Formulars](images/create-event-01.png)
