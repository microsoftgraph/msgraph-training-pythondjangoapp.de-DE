<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="90499-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="90499-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="90499-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="90499-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="90499-103">In diesem Schritt integrieren Sie die Requests [-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="90499-103">In this step you will integrate the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library into the application.</span></span>

<span data-ttu-id="90499-104">Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts mit `oauth_settings.yml`dem Namen, und fügen Sie den folgenden Inhalt hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

```text
app_id: YOUR_APP_ID_HERE
app_secret: YOUR_APP_PASSWORD_HERE
redirect: http://localhost:8000/tutorial/callback
scopes: openid profile offline_access user.read calendars.read
authority: https://login.microsoftonline.com/common
authorize_endpoint: /oauth2/v2.0/authorize
token_endpoint: /oauth2/v2.0/token
```

<span data-ttu-id="90499-105">Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR_APP_SECRET_HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.</span><span class="sxs-lookup"><span data-stu-id="90499-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="90499-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die Datei `oauth_settings.yml` aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="90499-106">If you're using source control such as git, now would be a good time to exclude the `oauth_settings.yml` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="90499-107">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="90499-107">Implement sign-in</span></span>

<span data-ttu-id="90499-108">Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `auth_helper.py` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-108">Create a new file in the `./tutorial` directory named `auth_helper.py` and add the following code.</span></span>

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
settings = yaml.load(stream, yaml.SafeLoader)
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

<span data-ttu-id="90499-109">In dieser Datei werden alle authentifizierungsbezogenen Methoden gespeichert.</span><span class="sxs-lookup"><span data-stu-id="90499-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="90499-110">Der `get_sign_in_url` generiert eine Autorisierungs-URL, `get_token_from_code` und die Methode tauscht die Autorisierungs Antwort für ein Zugriffstoken aus.</span><span class="sxs-lookup"><span data-stu-id="90499-110">The `get_sign_in_url` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

<span data-ttu-id="90499-111">Fügen Sie die `import` folgenden Anweisungen am Anfang von `./tutorial/views.py`hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-111">Add the following `import` statements to the top of `./tutorial/views.py`.</span></span>

```python
from django.urls import reverse
from tutorial.auth_helper import get_sign_in_url, get_token_from_code
```

<span data-ttu-id="90499-112">Fügen Sie nun ein paar neue Ansichten in die `./tutorial/views.py` Datei ein.</span><span class="sxs-lookup"><span data-stu-id="90499-112">Now add a couple of new views in the `./tutorial/views.py` file.</span></span>

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

<span data-ttu-id="90499-113">Dadurch werden zwei neue Ansichten definiert `signin` : `callback`und.</span><span class="sxs-lookup"><span data-stu-id="90499-113">This defines two new views: `signin` and `callback`.</span></span>

<span data-ttu-id="90499-114">Durch `signin` die Aktion wird die Azure AD SignIn-URL generiert `state` , der vom OAuth-Client generierte Wert gespeichert und anschließend der Browser an die Azure AD SignIn-Seite umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="90499-114">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

<span data-ttu-id="90499-115">In `callback` der Aktion wird Azure weitergeleitet, nachdem das SignIn abgeschlossen wurde.</span><span class="sxs-lookup"><span data-stu-id="90499-115">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="90499-116">Durch diese Aktion wird sicher `state` gestellt, dass der Wert mit dem gespeicherten Wert übereinstimmt, und dann wird der von Azure gesendete Autorisierungscode verwendet, um ein Zugriffstoken anzufordern.</span><span class="sxs-lookup"><span data-stu-id="90499-116">That action makes sure the `state` value matches the saved value, then uses the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="90499-117">Anschließend wird mit dem Zugriffstoken im temporären Fehlerwert zurück zur Startseite umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="90499-117">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="90499-118">Verwenden Sie diese, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="90499-118">You'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="90499-119">Bevor Sie testen, müssen Sie die Ansichten zu `./tutorial/urls.py`hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="90499-119">Before you test, you need to add the views to `./tutorial/urls.py`.</span></span>

```python
path('signin', views.sign_in, name='signin'),
path('callback', views.callback, name='callback'),
```

<span data-ttu-id="90499-120">Ersetzen Sie `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` die- `./tutorial/templates/tutorial/home.html` Verbindung durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90499-120">Replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line in `./tutorial/templates/tutorial/home.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="btn btn-primary btn-large">Click here to sign in</a>
```

<span data-ttu-id="90499-121">Ersetzen Sie `<a href="#" class="nav-link">Sign In</a>` die- `./tutorial/templates/tutorial/layout.html` Verbindung durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90499-121">Replace the `<a href="#" class="nav-link">Sign In</a>` line in `./tutorial/templates/tutorial/layout.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="nav-link">Sign In</a>
```

<span data-ttu-id="90499-122">Starten Sie den Server, und `https://localhost:8000/tutorial`navigieren Sie zu.</span><span class="sxs-lookup"><span data-stu-id="90499-122">Start the server and browse to `https://localhost:8000/tutorial`.</span></span> <span data-ttu-id="90499-123">Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="90499-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="90499-124">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="90499-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="90499-125">Der Browser wird an die APP umgeleitet, wobei das Token angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="90499-125">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="90499-126">Abrufen von Benutzer Details</span><span class="sxs-lookup"><span data-stu-id="90499-126">Get user details</span></span>

<span data-ttu-id="90499-127">Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `graph_helper.py` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-127">Create a new file in the `./tutorial` directory named `graph_helper.py` and add the following code.</span></span>

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

<span data-ttu-id="90499-128">Die `get_user` -Methode führt eine GET-Anforderung an den `/me` Microsoft Graph-Endpunkt aus, um das Profil des Benutzers mithilfe des zuvor erworbenen Zugriffstokens abzurufen.</span><span class="sxs-lookup"><span data-stu-id="90499-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

<span data-ttu-id="90499-129">Aktualisieren Sie `callback` die- `./tutorial/views.py` Methode in, um das Profil des Benutzers aus Microsoft Graph abzurufen.</span><span class="sxs-lookup"><span data-stu-id="90499-129">Update the `callback` method in `./tutorial/views.py` to get the user's profile from Microsoft Graph.</span></span>

<span data-ttu-id="90499-130">Fügen Sie zunächst die folgende `import` Anweisung am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-130">First, add the following `import` statement to the top of the file.</span></span>

```python
from tutorial.graph_helper import get_user
```

<span data-ttu-id="90499-131">Ersetzen Sie `callback` die-Methode durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="90499-131">Replace the `callback` method with the following code.</span></span>

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

<span data-ttu-id="90499-132">Der neue Code Ruft die `get_user` Methode auf, um das Profil des Benutzers anzufordern.</span><span class="sxs-lookup"><span data-stu-id="90499-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="90499-133">Das Benutzerobjekt wird der temporären Ausgabe zu Testzwecken hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="90499-133">It adds the user object to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="90499-134">Speichern der Token</span><span class="sxs-lookup"><span data-stu-id="90499-134">Storing the tokens</span></span>

<span data-ttu-id="90499-135">Nachdem Sie nun Token erhalten können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="90499-135">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="90499-136">Da es sich um eine Beispiel-App handelt, speichern Sie Sie aus Gründen der Einfachheit in der Sitzung.</span><span class="sxs-lookup"><span data-stu-id="90499-136">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="90499-137">Eine reale APP würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.</span><span class="sxs-lookup"><span data-stu-id="90499-137">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="90499-138">Fügen Sie die folgenden neuen Methoden `./tutorial/auth_helper.py`zu hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-138">Add the following new methods to `./tutorial/auth_helper.py`.</span></span>

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

<span data-ttu-id="90499-139">Aktualisieren Sie dann die `callback` -Funktion `./tutorial/views.py` in, um die Token in der Sitzung zu speichern und zur Hauptseite zurückzuleiten.</span><span class="sxs-lookup"><span data-stu-id="90499-139">Then, update the `callback` function in `./tutorial/views.py` to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="90499-140">Ersetzen Sie `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` die-Verbindung durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90499-140">Replace the `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` line with the following.</span></span>

```python
from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
```

<span data-ttu-id="90499-141">Ersetzen Sie `callback` die-Methode durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90499-141">Replace the `callback` method with the following.</span></span>

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

## <a name="implement-sign-out"></a><span data-ttu-id="90499-142">Implementieren der Abmeldung</span><span class="sxs-lookup"><span data-stu-id="90499-142">Implement sign-out</span></span>

<span data-ttu-id="90499-143">Bevor Sie dieses neue Feature testen, fügen Sie eine Möglichkeit zum Abmelden hinzu. Fügen Sie eine `sign_out` neue Ansicht `./tutorial/views.py`in hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-143">Before you test this new feature, add a way to sign out. Add a new `sign_out` view in `./tutorial/views.py`.</span></span>

```python
def sign_out(request):
  # Clear out the user and token
  remove_user_and_token(request)

  return HttpResponseRedirect(reverse('home'))
```

<span data-ttu-id="90499-144">Fügen Sie nun diese Ansicht `./tutorial/urls.py`zu hinzu.</span><span class="sxs-lookup"><span data-stu-id="90499-144">Now add this view to `./tutorial/urls.py`.</span></span>

```python
path('signout', views.sign_out, name='signout'),
```

<span data-ttu-id="90499-145">Aktualisieren Sie \*\*\*\* den Link abmelden `./tutorial/templates/tutorial/layout.html` in, um diese neue Ansicht zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="90499-145">Update the **Sign Out** link in `./tutorial/templates/tutorial/layout.html` to use this new view.</span></span> <span data-ttu-id="90499-146">Ersetzen Sie `<a href="#" class="dropdown-item">Sign Out</a>` die-Verbindung durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90499-146">Replace the `<a href="#" class="dropdown-item">Sign Out</a>` line with the following.</span></span>

```html
<a href="{% url 'signout' %}" class="dropdown-item">Sign Out</a>
```

<span data-ttu-id="90499-147">Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort.</span><span class="sxs-lookup"><span data-stu-id="90499-147">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="90499-148">Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="90499-148">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Ein Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

<span data-ttu-id="90499-150">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers \*\*\*\* , um auf den Abmeldelink zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="90499-150">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="90499-151">Durch \*\*\*\* klicken auf Abmelden wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="90499-151">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Screenshot des Dropdownmenüs mit dem Link zum Abmelden](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="90499-153">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="90499-153">Refreshing tokens</span></span>

<span data-ttu-id="90499-154">Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="90499-154">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="90499-155">Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="90499-155">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="90499-156">Dieses Token ist jedoch nur kurzlebig.</span><span class="sxs-lookup"><span data-stu-id="90499-156">However, this token is short-lived.</span></span> <span data-ttu-id="90499-157">Das Token läuft eine Stunde nach seiner Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="90499-157">The token expires an hour after it is issued.</span></span> <span data-ttu-id="90499-158">Hier wird das Aktualisierungstoken nützlich.</span><span class="sxs-lookup"><span data-stu-id="90499-158">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="90499-159">Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass sich der Benutzer erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="90499-159">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="90499-160">Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="90499-160">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="90499-161">Ersetzen Sie die `get_token` vorhandene Methode `./tutorial/auth_helper.py` in durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90499-161">Replace the existing `get_token` method in `./tutorial/auth_helper.py` with the following.</span></span>

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

<span data-ttu-id="90499-162">Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen oder nahezu abläuft.</span><span class="sxs-lookup"><span data-stu-id="90499-162">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="90499-163">Wenn dies der Fall ist, wird das Aktualisierungstoken verwendet, um neue Token abzurufen, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="90499-163">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
