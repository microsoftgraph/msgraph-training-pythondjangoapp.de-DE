<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten. In diesem Schritt integrieren Sie die [Requests-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek in die Anwendung.

1. Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts mit `oauth_settings.yml`dem Namen, und fügen Sie den folgenden Inhalt hinzu.

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR_APP_SECRET_HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.

> [!IMPORTANT]
> Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die **oauth_settings. yml** -Datei aus der Quellcodeverwaltung auszuschließen, damit versehentlich Ihre APP-ID und Ihr Kennwort verloren gehen.

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

1. Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis `auth_helper.py` mit dem Namen, und fügen Sie den folgenden Code hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    In dieser Datei werden alle authentifizierungsbezogenen Methoden gespeichert. Der `get_sign_in_url` generiert eine Autorisierungs-URL, `get_token_from_code` und die Methode tauscht die Autorisierungs Antwort für ein Zugriffstoken aus.

1. Fügen Sie die `import` folgenden Anweisungen am Anfang von **./Tutorial/views.py**.

    ```python
    from django.urls import reverse
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code
    ```

1. Fügen Sie eine anmeldeansicht in der Datei **./Tutorial/views.py** hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. Fügen Sie in der Datei **./Tutorial/views.py** eine Rückruf Ansicht hinzu.

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

    Betrachten Sie diese Ansichten:

    - Durch `signin` die Aktion wird die Azure AD SignIn-URL generiert `state` , der vom OAuth-Client generierte Wert gespeichert und anschließend der Browser an die Azure AD SignIn-Seite umgeleitet.

    - In `callback` der Aktion wird Azure weitergeleitet, nachdem das SignIn abgeschlossen wurde. Durch diese Aktion wird sicher `state` gestellt, dass der Wert mit dem gespeicherten Wert übereinstimmt, und dann wird der von Azure gesendete Autorisierungscode verwendet, um ein Zugriffstoken anzufordern. Anschließend wird mit dem Zugriffstoken im temporären Fehlerwert zurück zur Startseite umgeleitet. Verwenden Sie diese, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.

1. Öffnen Sie **./Tutorial/URLs.py** , und ersetzen `path` Sie die `signin` vorhandenen Anweisungen für folgendermaßen.

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. Fügen Sie ein `path` neues für `callback` die Ansicht hinzu.

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. Starten Sie den Server, und `https://localhost:8000`navigieren Sie zu. Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser leitet zur App um, in der Sie das Token sehen.

### <a name="get-user-details"></a>Benutzerdetails abrufen

1. Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis `graph_helper.py` mit dem Namen, und fügen Sie den folgenden Code hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    Die `get_user` -Methode führt eine GET-Anforderung an den `/me` Microsoft Graph-Endpunkt aus, um das Profil des Benutzers mithilfe des zuvor erworbenen Zugriffstokens abzurufen.

1. Aktualisieren Sie `callback` die-Methode in **./Tutorial/views.py** , um das Profil des Benutzers aus Microsoft Graph abzurufen. Fügen Sie die folgende `import`-Anweisung zum Anfang der Datei hinzu.

    ```python
    from tutorial.graph_helper import get_user
    ```

1. Ersetzen Sie die `callback`-Methode durch den folgenden Code.

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

Der neue Code Ruft die `get_user` Methode auf, um das Profil des Benutzers anzufordern. Das Benutzerobjekt wird der temporären Ausgabe zu Testzwecken hinzugefügt.

## <a name="storing-the-tokens"></a>Speichern des Tokens

Nun, da Sie Token abrufen können, ist es an der Zeit, eine Möglichkeit einzurichten, diese in der App zu speichern. Da es sich um eine Beispiel-App handelt, speichern Sie Sie aus Gründen der Einfachheit in der Sitzung. Eine echte App würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.

1. Fügen Sie die folgenden neuen Methoden zu **./Tutorial/auth_helper. py**hinzu.

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

1. Aktualisieren Sie `callback` die-Funktion in **./Tutorial/views.py** , um die Token in der Sitzung zu speichern und zur Hauptseite zurückzuleiten. Ersetzen Sie `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` die-Verbindung durch Folgendes.

    ```python
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
    ```

1. Ersetzen Sie `callback` die-Methode durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>Implementieren der Abmeldung

1. Hinzufügen einer `sign_out` neuen Ansicht in **./Tutorial/views.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. Öffnen Sie **./Tutorial/URLs.py** , und ersetzen `path` Sie die `signout` vorhandenen Anweisungen für folgendermaßen.

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort. Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach seiner Ausgabe ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss. Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.

1. Ersetzen Sie die `get_token` vorhandene Methode in **./Tutorial/auth_helper. py** durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="GetTokenSnippet":::

    Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen oder nahezu abläuft. Wenn dies der Fall ist, wird das Aktualisierungstoken verwendet, um neue Token abzurufen, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.
