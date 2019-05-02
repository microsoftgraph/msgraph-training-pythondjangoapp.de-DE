<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung zur Unterstützung der Authentifizierung mit Azure AD. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken für den Aufruf von Microsoft Graph abzurufen. In diesem Schritt integrieren Sie die Requests [-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek in die Anwendung.

Erstellen Sie eine neue Datei im Stamm des Projekts mit dem `oauth_settings.yml`Namen, und fügen Sie den folgenden Inhalt hinzu.

```text
app_id: YOUR_APP_ID_HERE
app_secret: YOUR_APP_PASSWORD_HERE
redirect: http://localhost:8000/tutorial/callback
scopes: openid profile offline_access user.read calendars.read
authority: https://login.microsoftonline.com/common
authorize_endpoint: /oauth2/v2.0/authorize
token_endpoint: /oauth2/v2.0/token
```

Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR_APP_SECRET_HERE` und ersetzen Sie es durch das von Ihnen generierte Kennwort.

> [!IMPORTANT]
> Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, um die `oauth_settings.yml` Datei aus der Quellcodeverwaltung auszuschließen, um versehentlich Ihre APP-ID und Ihr Kennwort zu verlieren.

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `auth_helper.py` dem Namen, und fügen Sie den folgenden Code hinzu.

```python
import yaml
from requests_oauthlib import OAuth2Session
import os
import time

# This is necessary for testing with non-HTTPS localhost
# Remove this if deploying to production
os.environ['OAUTHLIB_INSECURE_TRANSPORT'] = '1'

# This is necessary because Azure does not guarantee
# to return scopes in the same case and order as requested
os.environ['OAUTHLIB_RELAX_TOKEN_SCOPE'] = '1'
os.environ['OAUTHLIB_IGNORE_SCOPE_CHANGE'] = '1'

# Load the oauth_settings.yml file
stream = open('oauth_settings.yml', 'r')
settings = yaml.load(stream)
authorize_url = '{0}{1}'.format(settings['authority'], settings['authorize_endpoint'])
token_url = '{0}{1}'.format(settings['authority'], settings['token_endpoint'])

# Method to generate a sign-in url
def get_sign_in_url():
  # Initialize the OAuth client
  aad_auth = OAuth2Session(settings['app_id'],
    scope=settings['scopes'],
    redirect_uri=settings['redirect'])

  sign_in_url, state = aad_auth.authorization_url(authorize_url, prompt='login')

  return sign_in_url, state

# Method to exchange auth code for access token
def get_token_from_code(callback_url, expected_state):
  # Initialize the OAuth client
  aad_auth = OAuth2Session(settings['app_id'],
    state=expected_state,
    scope=settings['scopes'],
    redirect_uri=settings['redirect'])

  token = aad_auth.fetch_token(token_url,
    client_secret = settings['app_secret'],
    authorization_response=callback_url)

  return token
```

Diese Datei enthält alle authentifizierungsbezogenen Methoden. Der `get_sign_in_url` generiert eine Autorisierungs-URL, `get_token_from_code` und die-Methode tauscht die Autorisierungs Antwort für ein Zugriffstoken aus.

Fügen Sie die `import` folgenden Anweisungen am Anfang von `./tutorial/views.py`hinzu.

```python
from django.urls import reverse
from tutorial.auth_helper import get_sign_in_url, get_token_from_code
```

Fügen Sie nun eine Reihe neuer Ansichten in die `./tutorial/views.py` Datei ein.

```python
def sign_in(request):
  # Get the sign-in URL
  sign_in_url, state = get_sign_in_url()
  # Save the expected state so we can validate in the callback
  request.session['auth_state'] = state
  # Redirect to the Azure sign-in page
  return HttpResponseRedirect(sign_in_url)

def callback(request):
  # Get the state saved in session
  expected_state = request.session.pop('auth_state', '')
  # Make the token request
  token = get_token_from_code(request.get_full_path(), expected_state)
  # Temporary! Save the response in an error so it's displayed
  request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(token) }
  return HttpResponseRedirect(reverse('home'))
```

Dadurch werden zwei neue Ansichten definiert `signin` : `callback`und.

Die `signin` Aktion generiert die Azure AD SignIn-URL, speichert `state` den vom OAuth-Client generierten Wert und leitet den Browser dann an die Azure AD SignIn-Seite um.

Die `callback` Aktion ist die Stelle, an die Azure nach Abschluss der Anmeldung umleitet. Durch diese Aktion wird sicher `state` gestellt, dass der Wert mit dem gespeicherten Wert übereinstimmt, und anschließend wird der von Azure gesendete Autorisierungscode verwendet, um ein Zugriffstoken anzufordern. Anschließend wird zurück zur Homepage mit dem Zugriffstoken im temporären Fehlerwert umgeleitet. Verwenden Sie diese, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor Sie fortfahren. Bevor Sie testen, müssen Sie die Ansichten zu `./tutorial/urls.py`hinzufügen.

```python
path('signin', views.sign_in, name='signin'),
path('callback', views.callback, name='callback'),
```

Ersetzen Sie `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` die- `./tutorial/templates/tutorial/home.html` Reihe in durch Folgendes.

```html
<a href="{% url 'signin' %}" class="btn btn-primary btn-large">Click here to sign in</a>
```

Ersetzen Sie `<a href="#" class="nav-link">Sign In</a>` die- `./tutorial/templates/tutorial/layout.html` Reihe in durch Folgendes.

```html
<a href="{% url 'signin' %}" class="nav-link">Sign In</a>
```

Starten Sie den Server, und `https://localhost:8000/tutorial`navigieren Sie zu. Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den erforderlichen Berechtigungen zu. Der Browser leitet zur App um, wobei das Token angezeigt wird.

### <a name="get-user-details"></a>Benutzer Details abrufen

Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `graph_helper.py` dem Namen, und fügen Sie den folgenden Code hinzu.

```python
from requests_oauthlib import OAuth2Session

graph_url = 'https://graph.microsoft.com/v1.0'

def get_user(token):
  graph_client = OAuth2Session(token=token)
  # Send GET to /me
  user = graph_client.get('{0}/me'.format(graph_url))
  # Return the JSON result
  return user.json()
```

Die `get_user` -Methode führt eine GET-Anforderung an den `/me` Microsoft Graph-Endpunkt aus, um das Benutzerprofil mithilfe des zuvor erworbenen Zugriffstokens abzurufen.

Aktualisieren Sie `callback` die- `./tutorial/views.py` Methode in, um das Benutzerprofil von Microsoft Graph abzurufen.

Fügen Sie zunächst die folgende `import` Anweisung am Anfang der Datei hinzu.

```python
from tutorial.graph_helper import get_user
```

Ersetzen Sie `callback` die Methode durch den folgenden Code.

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

## <a name="storing-the-tokens"></a>Speichern der Token

Da Sie jetzt Tokens abrufen können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren. Da es sich um eine Beispiel-App handelt, speichern Sie Sie in der Sitzung. Eine echte APP würde eine zuverlässigere Secure Storage-Lösung wie eine Datenbank verwenden.

Fügen Sie die folgenden neuen Methoden `./tutorial/auth_helper.py`zu hinzu.

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

Aktualisieren Sie dann die `callback` -Funktion `./tutorial/views.py` in, um die Token in der Sitzung zu speichern und zurück zur Hauptseite umzuleiten. Ersetzen Sie `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` die-Reihe durch Folgendes.

```python
from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
```

Ersetzen Sie `callback` die Methode durch Folgendes.

```python
def callback(request):
  # Get the state saved in session
  expected_state = request.session.pop('auth_state', '')
  # Make the token request
  token = get_token_from_code(request.get_full_path(), expected_state)

  # Get the user's profile
  user = get_user(token)

  # Save token and user
  store_token(request, token)
  store_user(request, user)

  return HttpResponseRedirect(reverse('home'))
```

## <a name="implement-sign-out"></a>Implementierung der Abmeldung

Bevor Sie dieses neue Feature testen, fügen Sie eine Möglichkeit zum Abmelden hinzu. Fügen Sie eine `sign_out` neue Ansicht `./tutorial/views.py`in hinzu.

```python
def sign_out(request):
  # Clear out the user and token
  remove_user_and_token(request)

  return HttpResponseRedirect(reverse('home'))
```

Fügen Sie nun diese Ansicht `./tutorial/urls.py`zu hinzu.

```python
path('signout', views.sign_out, name='signout'),
```

Aktualisieren Sie **** den Link abmelden `./tutorial/templates/tutorial/layout.html` in, um diese neue Ansicht zu verwenden. Ersetzen Sie `<a href="#" class="dropdown-item">Sign Out</a>` die-Reihe durch Folgendes.

```html
<a href="{% url 'signout' %}" class="dropdown-item">Sign Out</a>
```

Starten Sie den Server neu, und führen Sie den Anmeldevorgang durch. Sie sollten auf der Startseite zurückkehren, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

![Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

Klicken Sie auf den Benutzer Avatar in der oberen rechten Ecke, **** um auf den Link abmelden zuzugreifen. Wenn **** Sie auf Abmelden klicken, wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite.

![Screenshot des Dropdownmenüs mit dem Link "Abmelden"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, mit dem die APP auf Microsoft Graph im Namen des Benutzers zugreifen kann.

Dieses Token ist jedoch kurzlebig. Das Token läuft eine Stunde nach der Ausgabe ab. An dieser Stelle wird das Aktualisierungstoken nützlich. Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss. Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.

Ersetzen Sie die `get_token` vorhandene- `./tutorial/auth_helper.py` Methode mit dem folgenden.

```python
def get_token(request):
  token = request.session['oauth_token']
  if token != None:
    # Check expiration
    now = time.time()
    # Subtract 5 minutes from expiration to account for clock skew
    expire_time = token['expires_at'] - 300
    if now >= expire_time:
      # Refresh the token
      aad_auth = OAuth2Session(settings['app_id'],
        token = token,
        scope=settings['scopes'],
        redirect_uri=settings['redirect'])

      refresh_params = {
        'client_id': settings['app_id'],
        'client_secret': settings['app_secret'],
      }
      new_token = aad_auth.refresh_token(token_url, **refresh_params)

      # Save new token
      store_token(request, new_token)

      # Return new access token
      return new_token

    else:
      # Token still valid, just return it
      return token
```

Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen ist oder in Kürze abläuft. Wenn dies der Fall ist, wird das Aktualisierungstoken zum Abrufen neuer Token verwendet, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.