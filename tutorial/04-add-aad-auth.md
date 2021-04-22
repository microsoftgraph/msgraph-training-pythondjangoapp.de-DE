<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung von der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph abzurufen. In diesem Schritt integrieren Sie die [MSAL für Python-Bibliothek](https://github.com/AzureAD/microsoft-authentication-library-for-python) in die Anwendung.

1. Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts namens `oauth_settings.yml` , und fügen Sie den folgenden Inhalt hinzu.

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungsregistrierungsportal, und ersetzen Sie durch das von Ihnen `YOUR_APP_SECRET_HERE` generierte Kennwort.

> [!IMPORTANT]
> Wenn Sie die Quellcodeverwaltung wie git verwenden, sollten Sie jetzt die **oauth_settings.yml-Datei** aus der Quellcodeverwaltung ausschließen, um versehentlich ein Leck ihrer App-ID und Ihres Kennworts zu vermeiden.

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

1. Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial** `auth_helper.py` namens, und fügen Sie den folgenden Code hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    Diese Datei enthält alle Authentifizierungsmethoden. Der `get_sign_in_flow` generiert eine Autorisierungs-URL, und die `get_token_from_code` Methode tauscht die Autorisierungsantwort gegen ein Zugriffstoken aus.

1. Fügen Sie die folgenden `import` Anweisungen oben in **./tutorial/views.py hinzu.**

    ```python
    from dateutil import tz, parser
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code
    ```

1. Fügen Sie der Datei **./tutorial/views.py** eine Anmeldeansicht hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. Fügen Sie in der Datei **./tutorial/views.py** eine Rückrufansicht hinzu.

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    Überlegen Sie, was diese Ansichten tun:

    - Die Aktion generiert die Azure AD-Anmelde-URL, speichert den vom OAuth-Client generierten Fluss und leitet den Browser dann zur `signin` Azure AD-Anmeldeseite um.

    - Die `callback` Aktion leitet Azure nach Abschluss der Anmeldung um. Diese Aktion verwendet den gespeicherten Fluss und die von Azure gesendete Abfragezeichenfolge, um ein Zugriffstoken an verlangen. Anschließend wird die Antwort im temporären Fehlerwert zurück zur Startseite umgeleitet. Sie verwenden dies, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor Sie weiter.

1. Öffnen **Sie ./tutorial/urls.py,** und ersetzen Sie die vorhandenen `path` Anweisungen durch `signin` Folgendes.

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. Fügen Sie eine `path` neue für die Ansicht `callback` hinzu.

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. Starten Sie den Server, und navigieren Sie zu `https://localhost:8000` . Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser wird an die App umgeleitet und zeigt die Antwort an, einschließlich des Zugriffstokens.

### <a name="get-user-details"></a>Benutzerdetails abrufen

1. Erstellen Sie eine neue Datei im **Verzeichnis ./tutorial** `graph_helper.py` namens, und fügen Sie den folgenden Code hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    Die Methode stellt eine GET-Anforderung an den Microsoft Graph-Endpunkt, um das Benutzerprofil mithilfe des zuvor erworbenen `get_user` `/me` Zugriffstokens abzurufen.

1. Aktualisieren Sie `callback` die Methode in **./tutorial/views.py,** um das Benutzerprofil von Microsoft Graph zu erhalten. Fügen Sie die folgende `import`-Anweisung zum Anfang der Datei hinzu.

    ```python
    from tutorial.graph_helper import *
    ```

1. Ersetzen Sie die `callback`-Methode durch den folgenden Code.

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

    Der neue Code ruft die `get_user` -Methode auf, um das Profil des Benutzers an fordern. Das Benutzerobjekt wird der temporären Ausgabe zu Testzwecken hinzufügt.

1. Fügen Sie die folgenden neuen Methoden **zu ./tutorial/auth_helper.py hinzu.**

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="SecondCodeSnippet":::

1. Aktualisieren Sie `callback` die Funktion in **./tutorial/views.py,** um den Benutzer in der Sitzung zu speichern und zur Hauptseite zurückzuleiten. Ersetzen Sie `from tutorial.auth_helper import get_sign_in_flow, get_token_from_code` die Zeile durch Folgendes.

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code, store_user, remove_user_and_token, get_token
    ```

1. Ersetzen Sie `callback` die Methode durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>Implementieren des Abmeldens

1. Fügen Sie eine neue `sign_out` Ansicht in **./tutorial/views.py hinzu.**

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. Öffnen **Sie ./tutorial/urls.py,** und ersetzen Sie die vorhandenen `path` Anweisungen durch `signout` Folgendes.

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. Starten Sie den Server neu, und gehen Sie durch den Anmeldevorgang. Sie sollten wieder auf der Startseite landen, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Benutzer-Avatar, um auf den **Link Abmelden zu** zugreifen. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der Kopfzeile von `Authorization` API-Aufrufen gesendet wird. Dies ist das Token, mit dem die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach der Enerbung ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.

Da in diesem Beispiel MSAL verwendet wird, müssen Sie keinen bestimmten Code schreiben, um das Token zu aktualisieren. Die MSAL-Methode `acquire_token_silent` behandelt bei Bedarf das Aktualisieren des Tokens.
