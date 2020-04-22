<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6431b-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="6431b-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="6431b-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="6431b-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="6431b-103">In diesem Schritt integrieren Sie die [Requests-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="6431b-103">In this step you will integrate the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library into the application.</span></span>

1. <span data-ttu-id="6431b-104">Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts mit `oauth_settings.yml`dem Namen, und fügen Sie den folgenden Inhalt hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. <span data-ttu-id="6431b-105">Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR_APP_SECRET_HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.</span><span class="sxs-lookup"><span data-stu-id="6431b-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6431b-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die **oauth_settings. yml** -Datei aus der Quellcodeverwaltung auszuschließen, damit versehentlich Ihre APP-ID und Ihr Kennwort verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="6431b-106">If you're using source control such as git, now would be a good time to exclude the **oauth_settings.yml** file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="6431b-107">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="6431b-107">Implement sign-in</span></span>

1. <span data-ttu-id="6431b-108">Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis `auth_helper.py` mit dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-108">Create a new file in the **./tutorial** directory named `auth_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="6431b-109">In dieser Datei werden alle authentifizierungsbezogenen Methoden gespeichert.</span><span class="sxs-lookup"><span data-stu-id="6431b-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="6431b-110">Der `get_sign_in_url` generiert eine Autorisierungs-URL, `get_token_from_code` und die Methode tauscht die Autorisierungs Antwort für ein Zugriffstoken aus.</span><span class="sxs-lookup"><span data-stu-id="6431b-110">The `get_sign_in_url` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

1. <span data-ttu-id="6431b-111">Fügen Sie die `import` folgenden Anweisungen am Anfang von **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="6431b-111">Add the following `import` statements to the top of **./tutorial/views.py**.</span></span>

    ```python
    from django.urls import reverse
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code
    ```

1. <span data-ttu-id="6431b-112">Fügen Sie eine anmeldeansicht in der Datei **./Tutorial/views.py** hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-112">Add a sign-in view in the **./tutorial/views.py** file.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. <span data-ttu-id="6431b-113">Fügen Sie in der Datei **./Tutorial/views.py** eine Rückruf Ansicht hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-113">Add a callback view in the **./tutorial/views.py** file.</span></span>

    ```python
    def callback(request):
      # Get the state saved in session
      expected_state = request.session.pop('auth_state', '')
      # Make the token request
      token = get_token_from_code(request.get_full_path(), expected_state)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(token) }
      return HttpResponseRedirect(reverse('home'))
    ```

    <span data-ttu-id="6431b-114">Betrachten Sie diese Ansichten:</span><span class="sxs-lookup"><span data-stu-id="6431b-114">Consider what these views do:</span></span>

    - <span data-ttu-id="6431b-115">Durch `signin` die Aktion wird die Azure AD SignIn-URL generiert `state` , der vom OAuth-Client generierte Wert gespeichert und anschließend der Browser an die Azure AD SignIn-Seite umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="6431b-115">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    - <span data-ttu-id="6431b-116">In `callback` der Aktion wird Azure weitergeleitet, nachdem das SignIn abgeschlossen wurde.</span><span class="sxs-lookup"><span data-stu-id="6431b-116">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="6431b-117">Durch diese Aktion wird sicher `state` gestellt, dass der Wert mit dem gespeicherten Wert übereinstimmt, und dann wird der von Azure gesendete Autorisierungscode verwendet, um ein Zugriffstoken anzufordern.</span><span class="sxs-lookup"><span data-stu-id="6431b-117">That action makes sure the `state` value matches the saved value, then uses the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="6431b-118">Anschließend wird mit dem Zugriffstoken im temporären Fehlerwert zurück zur Startseite umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="6431b-118">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="6431b-119">Verwenden Sie diese, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="6431b-119">You'll use this to verify that our sign-in is working before moving on.</span></span>

1. <span data-ttu-id="6431b-120">Öffnen Sie **./Tutorial/URLs.py** , und ersetzen `path` Sie die `signin` vorhandenen Anweisungen für folgendermaßen.</span><span class="sxs-lookup"><span data-stu-id="6431b-120">Open **./tutorial/urls.py** and replace the existing `path` statements for `signin` with the following.</span></span>

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. <span data-ttu-id="6431b-121">Fügen Sie ein `path` neues für `callback` die Ansicht hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-121">Add a new `path` for the `callback` view.</span></span>

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. <span data-ttu-id="6431b-122">Starten Sie den Server, und `https://localhost:8000`navigieren Sie zu.</span><span class="sxs-lookup"><span data-stu-id="6431b-122">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="6431b-123">Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden.</span><span class="sxs-lookup"><span data-stu-id="6431b-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="6431b-124">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="6431b-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="6431b-125">Der Browser leitet zur App um, in der Sie das Token sehen.</span><span class="sxs-lookup"><span data-stu-id="6431b-125">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="6431b-126">Benutzerdetails abrufen</span><span class="sxs-lookup"><span data-stu-id="6431b-126">Get user details</span></span>

1. <span data-ttu-id="6431b-127">Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis `graph_helper.py` mit dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-127">Create a new file in the **./tutorial** directory named `graph_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="6431b-128">Die `get_user` -Methode führt eine GET-Anforderung an den `/me` Microsoft Graph-Endpunkt aus, um das Profil des Benutzers mithilfe des zuvor erworbenen Zugriffstokens abzurufen.</span><span class="sxs-lookup"><span data-stu-id="6431b-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

1. <span data-ttu-id="6431b-129">Aktualisieren Sie `callback` die-Methode in **./Tutorial/views.py** , um das Profil des Benutzers aus Microsoft Graph abzurufen.</span><span class="sxs-lookup"><span data-stu-id="6431b-129">Update the `callback` method in **./tutorial/views.py** to get the user's profile from Microsoft Graph.</span></span> <span data-ttu-id="6431b-130">Fügen Sie die folgende `import`-Anweisung zum Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-130">Add the following `import` statement to the top of the file.</span></span>

    ```python
    from tutorial.graph_helper import get_user
    ```

1. <span data-ttu-id="6431b-131">Ersetzen Sie die `callback`-Methode durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="6431b-131">Replace the `callback` method with the following code.</span></span>

    ```python
    def callback(request):
      # Get the state saved in session
      expected_state = request.session.pop('auth_state', '')
      # Make the token request
      token = get_token_from_code(request.get_full_path(), expected_state)

      # Get the user's profile
      user = get_user(token)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved',
        'debug': 'User: {0}\nToken: {1}'.format(user, token) }
      return HttpResponseRedirect(reverse('home'))
    ```

<span data-ttu-id="6431b-132">Der neue Code Ruft die `get_user` Methode auf, um das Profil des Benutzers anzufordern.</span><span class="sxs-lookup"><span data-stu-id="6431b-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="6431b-133">Das Benutzerobjekt wird der temporären Ausgabe zu Testzwecken hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="6431b-133">It adds the user object to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="6431b-134">Speichern des Tokens</span><span class="sxs-lookup"><span data-stu-id="6431b-134">Storing the tokens</span></span>

<span data-ttu-id="6431b-135">Nun, da Sie Token abrufen können, ist es an der Zeit, eine Möglichkeit einzurichten, diese in der App zu speichern.</span><span class="sxs-lookup"><span data-stu-id="6431b-135">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="6431b-136">Da es sich um eine Beispiel-App handelt, speichern Sie Sie aus Gründen der Einfachheit in der Sitzung.</span><span class="sxs-lookup"><span data-stu-id="6431b-136">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="6431b-137">Eine echte App würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.</span><span class="sxs-lookup"><span data-stu-id="6431b-137">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="6431b-138">Fügen Sie die folgenden neuen Methoden zu **./Tutorial/auth_helper. py**hinzu.</span><span class="sxs-lookup"><span data-stu-id="6431b-138">Add the following new methods to **./tutorial/auth_helper.py**.</span></span>

    ```python
    def store_token(request, token):
      request.session['oauth_token'] = token

    def store_user(request, user):
      request.session['user'] = {
        'is_authenticated': True,
        'name': user['displayName'],
        'email': user['mail'] if (user['mail'] != None) else user['userPrincipalName']
      }

    def get_token(request):
      token = request.session['oauth_token']
      return token

    def remove_user_and_token(request):
      if 'oauth_token' in request.session:
        del request.session['oauth_token']

      if 'user' in request.session:
        del request.session['user']
    ```

1. <span data-ttu-id="6431b-139">Aktualisieren Sie `callback` die-Funktion in **./Tutorial/views.py** , um die Token in der Sitzung zu speichern und zur Hauptseite zurückzuleiten.</span><span class="sxs-lookup"><span data-stu-id="6431b-139">Update the `callback` function in **./tutorial/views.py** to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="6431b-140">Ersetzen Sie `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` die-Verbindung durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="6431b-140">Replace the `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` line with the following.</span></span>

    ```python
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
    ```

1. <span data-ttu-id="6431b-141">Ersetzen Sie `callback` die-Methode durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="6431b-141">Replace the `callback` method with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="6431b-142">Implementieren der Abmeldung</span><span class="sxs-lookup"><span data-stu-id="6431b-142">Implement sign-out</span></span>

1. <span data-ttu-id="6431b-143">Hinzufügen einer `sign_out` neuen Ansicht in **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="6431b-143">Add a new `sign_out` view in **./tutorial/views.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. <span data-ttu-id="6431b-144">Öffnen Sie **./Tutorial/URLs.py** , und ersetzen `path` Sie die `signout` vorhandenen Anweisungen für folgendermaßen.</span><span class="sxs-lookup"><span data-stu-id="6431b-144">Open **./tutorial/urls.py** and replace the existing `path` statements for `signout` with the following.</span></span>

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. <span data-ttu-id="6431b-145">Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort.</span><span class="sxs-lookup"><span data-stu-id="6431b-145">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="6431b-146">Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="6431b-146">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. <span data-ttu-id="6431b-148">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="6431b-148">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="6431b-149">Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="6431b-149">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="6431b-151">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="6431b-151">Refreshing tokens</span></span>

<span data-ttu-id="6431b-152">Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="6431b-152">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="6431b-153">Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="6431b-153">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="6431b-154">Dieses Token ist jedoch nur kurzzeitig verfügbar.</span><span class="sxs-lookup"><span data-stu-id="6431b-154">However, this token is short-lived.</span></span> <span data-ttu-id="6431b-155">Das Token läuft eine Stunde nach seiner Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="6431b-155">The token expires an hour after it is issued.</span></span> <span data-ttu-id="6431b-156">An dieser Stelle kommt das Aktualisierungstoken ins Spiel.</span><span class="sxs-lookup"><span data-stu-id="6431b-156">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="6431b-157">Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="6431b-157">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="6431b-158">Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="6431b-158">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="6431b-159">Ersetzen Sie die `get_token` vorhandene Methode in **./Tutorial/auth_helper. py** durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="6431b-159">Replace the existing `get_token` method in **./tutorial/auth_helper.py** with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="GetTokenSnippet":::

    <span data-ttu-id="6431b-160">Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen oder nahezu abläuft.</span><span class="sxs-lookup"><span data-stu-id="6431b-160">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="6431b-161">Wenn dies der Fall ist, wird das Aktualisierungstoken verwendet, um neue Token abzurufen, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="6431b-161">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
