<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="8fa4e-101">In dieser Übung erweitern Sie die Anwendung von der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="8fa4e-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph abzurufen.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="8fa4e-103">In diesem Schritt integrieren Sie die [MSAL für Python-Bibliothek](https://github.com/AzureAD/microsoft-authentication-library-for-python) in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-103">In this step you will integrate the [MSAL for Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) library into the application.</span></span>

1. <span data-ttu-id="8fa4e-104">Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts namens `oauth_settings.yml` , und fügen Sie den folgenden Inhalt hinzu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. <span data-ttu-id="8fa4e-105">Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungsregistrierungsportal, und ersetzen Sie durch das von Ihnen `YOUR_APP_SECRET_HERE` generierte Kennwort.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="8fa4e-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, sollten Sie jetzt die **oauth_settings.yml-Datei** aus der Quellcodeverwaltung ausschließen, um versehentlich ein Leck ihrer App-ID und Ihres Kennworts zu vermeiden.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-106">If you're using source control such as git, now would be a good time to exclude the **oauth_settings.yml** file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="8fa4e-107">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="8fa4e-107">Implement sign-in</span></span>

1. <span data-ttu-id="8fa4e-108">Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial** `auth_helper.py` namens, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-108">Create a new file in the **./tutorial** directory named `auth_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="8fa4e-109">Diese Datei enthält alle Authentifizierungsmethoden.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="8fa4e-110">Der `get_sign_in_flow` generiert eine Autorisierungs-URL, und die `get_token_from_code` Methode tauscht die Autorisierungsantwort gegen ein Zugriffstoken aus.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-110">The `get_sign_in_flow` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

1. <span data-ttu-id="8fa4e-111">Fügen Sie die folgenden `import` Anweisungen oben in **./tutorial/views.py hinzu.**</span><span class="sxs-lookup"><span data-stu-id="8fa4e-111">Add the following `import` statements to the top of **./tutorial/views.py**.</span></span>

    ```python
    from dateutil import tz, parser
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code
    ```

1. <span data-ttu-id="8fa4e-112">Fügen Sie der Datei **./tutorial/views.py** eine Anmeldeansicht hinzu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-112">Add a sign-in view in the **./tutorial/views.py** file.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. <span data-ttu-id="8fa4e-113">Fügen Sie in der Datei **./tutorial/views.py** eine Rückrufansicht hinzu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-113">Add a callback view in the **./tutorial/views.py** file.</span></span>

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    <span data-ttu-id="8fa4e-114">Überlegen Sie, was diese Ansichten tun:</span><span class="sxs-lookup"><span data-stu-id="8fa4e-114">Consider what these views do:</span></span>

    - <span data-ttu-id="8fa4e-115">Die Aktion generiert die Azure AD-Anmelde-URL, speichert den vom OAuth-Client generierten Fluss und leitet den Browser dann zur `signin` Azure AD-Anmeldeseite um.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-115">The `signin` action generates the Azure AD signin URL, saves the flow generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    - <span data-ttu-id="8fa4e-116">Die `callback` Aktion leitet Azure nach Abschluss der Anmeldung um.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-116">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="8fa4e-117">Diese Aktion verwendet den gespeicherten Fluss und die von Azure gesendete Abfragezeichenfolge, um ein Zugriffstoken an verlangen.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-117">That action uses the saved flow and the query string sent by Azure to request an access token.</span></span> <span data-ttu-id="8fa4e-118">Anschließend wird die Antwort im temporären Fehlerwert zurück zur Startseite umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-118">It then redirects back to the home page with the response in the temporary error value.</span></span> <span data-ttu-id="8fa4e-119">Sie verwenden dies, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor Sie weiter.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-119">You'll use this to verify that our sign-in is working before moving on.</span></span>

1. <span data-ttu-id="8fa4e-120">Öffnen **Sie ./tutorial/urls.py,** und ersetzen Sie die vorhandenen `path` Anweisungen durch `signin` Folgendes.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-120">Open **./tutorial/urls.py** and replace the existing `path` statements for `signin` with the following.</span></span>

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. <span data-ttu-id="8fa4e-121">Fügen Sie eine `path` neue für die Ansicht `callback` hinzu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-121">Add a new `path` for the `callback` view.</span></span>

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. <span data-ttu-id="8fa4e-122">Starten Sie den Server, und navigieren Sie zu `https://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="8fa4e-122">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="8fa4e-123">Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="8fa4e-124">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="8fa4e-125">Der Browser wird an die App umgeleitet und zeigt die Antwort an, einschließlich des Zugriffstokens.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-125">The browser redirects to the app, showing the response, including the access token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="8fa4e-126">Benutzerdetails abrufen</span><span class="sxs-lookup"><span data-stu-id="8fa4e-126">Get user details</span></span>

1. <span data-ttu-id="8fa4e-127">Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial** `graph_helper.py` namens, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-127">Create a new file in the **./tutorial** directory named `graph_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="8fa4e-128">Die Methode stellt eine GET-Anforderung an den Microsoft Graph-Endpunkt, um das Benutzerprofil mithilfe des zuvor erworbenen `get_user` `/me` Zugriffstokens abzurufen.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

1. <span data-ttu-id="8fa4e-129">Aktualisieren Sie `callback` die Methode in **./tutorial/views.py,** um das Benutzerprofil von Microsoft Graph zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-129">Update the `callback` method in **./tutorial/views.py** to get the user's profile from Microsoft Graph.</span></span> <span data-ttu-id="8fa4e-130">Fügen Sie die folgende `import`-Anweisung zum Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-130">Add the following `import` statement to the top of the file.</span></span>

    ```python
    from tutorial.graph_helper import *
    ```

1. <span data-ttu-id="8fa4e-131">Ersetzen Sie die `callback`-Methode durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-131">Replace the `callback` method with the following code.</span></span>

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)

      #Get the user's profile
      user = get_user(result['access_token'])
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': 'User: {0}\nToken: {1}'.format(user, result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    <span data-ttu-id="8fa4e-132">Der neue Code ruft die `get_user` -Methode auf, um das Profil des Benutzers an fordern.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="8fa4e-133">Das Benutzerobjekt wird der temporären Ausgabe zu Testzwecken hinzufügt.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-133">It adds the user object to the temporary output for testing.</span></span>

1. <span data-ttu-id="8fa4e-134">Fügen Sie die folgenden neuen Methoden **zu ./tutorial/auth_helper.py hinzu.**</span><span class="sxs-lookup"><span data-stu-id="8fa4e-134">Add the following new methods to **./tutorial/auth_helper.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="SecondCodeSnippet":::

1. <span data-ttu-id="8fa4e-135">Aktualisieren Sie `callback` die Funktion in **./tutorial/views.py,** um den Benutzer in der Sitzung zu speichern und zur Hauptseite zurückzuleiten.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-135">Update the `callback` function in **./tutorial/views.py** to store the user in the session and redirect back to the main page.</span></span> <span data-ttu-id="8fa4e-136">Ersetzen Sie `from tutorial.auth_helper import get_sign_in_flow, get_token_from_code` die Zeile durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-136">Replace the `from tutorial.auth_helper import get_sign_in_flow, get_token_from_code` line with the following.</span></span>

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code, store_user, remove_user_and_token, get_token
    ```

1. <span data-ttu-id="8fa4e-137">Ersetzen Sie `callback` die Methode durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-137">Replace the `callback` method with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="8fa4e-138">Implementieren des Abmeldens</span><span class="sxs-lookup"><span data-stu-id="8fa4e-138">Implement sign-out</span></span>

1. <span data-ttu-id="8fa4e-139">Fügen Sie eine neue `sign_out` Ansicht in **./tutorial/views.py hinzu.**</span><span class="sxs-lookup"><span data-stu-id="8fa4e-139">Add a new `sign_out` view in **./tutorial/views.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. <span data-ttu-id="8fa4e-140">Öffnen **Sie ./tutorial/urls.py,** und ersetzen Sie die vorhandenen `path` Anweisungen durch `signout` Folgendes.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-140">Open **./tutorial/urls.py** and replace the existing `path` statements for `signout` with the following.</span></span>

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. <span data-ttu-id="8fa4e-141">Starten Sie den Server neu, und gehen Sie durch den Anmeldevorgang.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-141">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="8fa4e-142">Sie sollten wieder auf der Startseite landen, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-142">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. <span data-ttu-id="8fa4e-144">Klicken Sie in der oberen rechten Ecke auf den Benutzer-Avatar, um auf den **Link Abmelden zu** zugreifen.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-144">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="8fa4e-145">Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-145">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="8fa4e-147">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="8fa4e-147">Refreshing tokens</span></span>

<span data-ttu-id="8fa4e-148">An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der Kopfzeile von `Authorization` API-Aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="8fa4e-149">Dies ist das Token, mit dem die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-149">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="8fa4e-150">Dieses Token ist jedoch nur kurzzeitig verfügbar.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-150">However, this token is short-lived.</span></span> <span data-ttu-id="8fa4e-151">Das Token läuft eine Stunde nach der Enerbung ab.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="8fa4e-152">An dieser Stelle kommt das Aktualisierungstoken ins Spiel.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="8fa4e-153">Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="8fa4e-154">Da in diesem Beispiel MSAL verwendet wird, müssen Sie keinen bestimmten Code schreiben, um das Token zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-154">Because this sample uses MSAL, you do not have to write any specific code to refresh the token.</span></span> <span data-ttu-id="8fa4e-155">Die MSAL-Methode `acquire_token_silent` behandelt bei Bedarf das Aktualisieren des Tokens.</span><span class="sxs-lookup"><span data-stu-id="8fa4e-155">MSAL's `acquire_token_silent` method handles refreshing the token if needed.</span></span>
