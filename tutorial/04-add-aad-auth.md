<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten. In diesem Schritt integrieren Sie die Bibliothek [MSAL für python](https://github.com/AzureAD/microsoft-authentication-library-for-python) in die Anwendung.

1. Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts mit dem Namen `oauth_settings.yml` , und fügen Sie den folgenden Inhalt hinzu.

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, und ersetzen `YOUR_APP_SECRET_HERE` Sie durch das Kennwort, das Sie generiert haben.

> [!IMPORTANT]
> Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die **oauth_settings. yml** -Datei aus der Quellcodeverwaltung auszuschließen, damit versehentlich Ihre APP-ID und Ihr Kennwort verloren gehen.

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

1. Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis mit dem Namen `auth_helper.py` , und fügen Sie den folgenden Code hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    In dieser Datei werden alle authentifizierungsbezogenen Methoden gespeichert. Der `get_sign_in_flow` generiert eine Autorisierungs-URL, und die `get_token_from_code` Methode tauscht die Autorisierungs Antwort für ein Zugriffstoken aus.

1. Fügen Sie die folgende `import` Anweisung am Anfang von **./Tutorial/views.py**.

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code
    ```

1. Fügen Sie eine anmeldeansicht in der Datei **./Tutorial/views.py** hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. Fügen Sie in der Datei **./Tutorial/views.py** eine Rückruf Ansicht hinzu.

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    Betrachten Sie diese Ansichten:

    - `signin`Durch die Aktion wird die Azure AD SignIn-URL generiert, der vom OAuth-Client generierte Ablauf gespeichert und anschließend der Browser an die Azure AD SignIn-Seite umgeleitet.

    - `callback`In der Aktion wird Azure weitergeleitet, nachdem das SignIn abgeschlossen wurde. Bei dieser Aktion wird der gespeicherte Fluss und die von Azure gesendete Abfragezeichenfolge verwendet, um ein Zugriffstoken anzufordern. Anschließend wird die Antwort im temporären Fehlerwert zurück zur Startseite umgeleitet. Verwenden Sie diese, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.

1. Öffnen Sie **./Tutorial/URLs.py** , und ersetzen Sie die vorhandenen `path` Anweisungen für `signin` folgendermaßen.

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. Fügen Sie ein neues `path` für die `callback` Ansicht hinzu.

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. Starten Sie den Server, und navigieren Sie zu `https://localhost:8000` . Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser wird an die APP umgeleitet und zeigt die Antwort an, einschließlich des Zugriffstokens.

### <a name="get-user-details"></a>Benutzerdetails abrufen

1. Erstellen Sie eine neue Datei im **./Tutorial** -Verzeichnis mit dem Namen `graph_helper.py` , und fügen Sie den folgenden Code hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    Die `get_user` -Methode führt eine GET-Anforderung an den Microsoft Graph- `/me` Endpunkt aus, um das Profil des Benutzers mithilfe des zuvor erworbenen Zugriffstokens abzurufen.

1. Aktualisieren Sie die `callback` -Methode in **./Tutorial/views.py** , um das Profil des Benutzers aus Microsoft Graph abzurufen. Fügen Sie die folgende `import`-Anweisung zum Anfang der Datei hinzu.

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

    Der neue Code Ruft die `get_user` Methode auf, um das Profil des Benutzers anzufordern. Das Benutzerobjekt wird der temporären Ausgabe zu Testzwecken hinzugefügt.

1. Fügen Sie die folgenden neuen Methoden zu **./Tutorial/auth_helper. py** hinzu.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="SecondCodeSnippet":::

1. Aktualisieren Sie die- `callback` Funktion in **./Tutorial/views.py** , um den Benutzer in der Sitzung zu speichern und zur Hauptseite zurückzuleiten. Ersetzen Sie die `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` -Verbindung durch Folgendes.

    ```python
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_user, remove_user_and_token, get_token
    ```

1. Ersetzen Sie die `callback` -Methode durch Folgendes.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>Implementieren der Abmeldung

1. Hinzufügen einer neuen `sign_out` Ansicht in **./Tutorial/views.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. Öffnen Sie **./Tutorial/URLs.py** , und ersetzen Sie die vorhandenen `path` Anweisungen für `signout` folgendermaßen.

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort. Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach seiner Ausgabe ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.

Da in diesem Beispiel MSAL verwendet wird, müssen Sie keinen spezifischen Code zum Aktualisieren des Tokens schreiben. Die MSAL `acquire_token_silent` -Methode verarbeitet das Aktualisieren des Tokens, falls erforderlich.
