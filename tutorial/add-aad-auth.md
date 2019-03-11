<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="40ffd-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung zur Unterstützung der Authentifizierung mit Azure AD.</span><span class="sxs-lookup"><span data-stu-id="40ffd-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="40ffd-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken für den Aufruf von Microsoft Graph abzurufen.</span><span class="sxs-lookup"><span data-stu-id="40ffd-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="40ffd-103">In diesem Schritt integrieren Sie die reQuests [-OAuthlib-](https://requests-oauthlib.readthedocs.io/en/latest/) Bibliothek in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="40ffd-103">In this step you will integrate the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library into the application.</span></span>

<span data-ttu-id="40ffd-104">Erstellen Sie eine neue Datei im Stamm des Projekts mit dem `oauth_settings.yml`Namen, und fügen Sie den folgenden Inhalt hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

```text
app_id: YOUR_APP_ID_HERE
app_secret: YOUR_APP_PASSWORD_HERE
redirect: http://localhost:8000/tutorial/callback
scopes: openid profile offline_access user.read calendars.read
authority: https://login.microsoftonline.com/common
authorize_endpoint: /oauth2/v2.0/authorize
token_endpoint: /oauth2/v2.0/token
```

<span data-ttu-id="40ffd-105">Ersetzen `YOUR_APP_ID_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR_APP_SECRET_HERE` und ersetzen Sie es durch das von Ihnen generierte Kennwort.</span><span class="sxs-lookup"><span data-stu-id="40ffd-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="40ffd-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, um die `oauth_settings.yml` Datei aus der Quellcodeverwaltung auszuschließen, um versehentlich Ihre APP-ID und Ihr Kennwort zu verlieren.</span><span class="sxs-lookup"><span data-stu-id="40ffd-106">If you're using source control such as git, now would be a good time to exclude the `oauth_settings.yml` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="40ffd-107">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="40ffd-107">Implement sign-in</span></span>

<span data-ttu-id="40ffd-108">Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `auth_helper.py` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-108">Create a new file in the `./tutorial` directory named `auth_helper.py` and add the following code.</span></span>

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

<span data-ttu-id="40ffd-109">Diese Datei enthält alle authentifizierungsbezogenen Methoden.</span><span class="sxs-lookup"><span data-stu-id="40ffd-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="40ffd-110">Der `get_sign_in_url` generiert eine Autorisierungs-URL, `get_token_from_code` und die-Methode tauscht die Autorisierungs Antwort für ein Zugriffstoken aus.</span><span class="sxs-lookup"><span data-stu-id="40ffd-110">The `get_sign_in_url` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

<span data-ttu-id="40ffd-111">Fügen Sie die `import` folgenden Anweisungen am Anfang von `./tutorial/views.py`hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-111">Add the following `import` statements to the top of `./tutorial/views.py`.</span></span>

```python
from django.urls import reverse
from tutorial.auth_helper import get_sign_in_url, get_token_from_code
```

<span data-ttu-id="40ffd-112">Fügen Sie nun eine Reihe neuer Ansichten in die `./tutorial/views.py` Datei ein.</span><span class="sxs-lookup"><span data-stu-id="40ffd-112">Now add a couple of new views in the `./tutorial/views.py` file.</span></span>

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

<span data-ttu-id="40ffd-113">Dadurch werden zwei neue Ansichten definiert `signin` : `callback`und.</span><span class="sxs-lookup"><span data-stu-id="40ffd-113">This defines two new views: `signin` and `callback`.</span></span>

<span data-ttu-id="40ffd-114">Die `signin` Aktion generiert die Azure AD SignIn-URL, speichert `state` den vom OAuth-Client generierten Wert und leitet den Browser dann an die Azure AD SignIn-Seite um.</span><span class="sxs-lookup"><span data-stu-id="40ffd-114">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

<span data-ttu-id="40ffd-115">Die `callback` Aktion ist die Stelle, an die Azure nach Abschluss der Anmeldung umleitet.</span><span class="sxs-lookup"><span data-stu-id="40ffd-115">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="40ffd-116">Durch diese Aktion wird sicher `state` gestellt, dass der Wert mit dem gespeicherten Wert übereinstimmt, und anschließend wird der von Azure gesendete Autorisierungscode verwendet, um ein Zugriffstoken anzufordern.</span><span class="sxs-lookup"><span data-stu-id="40ffd-116">That action makes sure the `state` value matches the saved value, then uses the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="40ffd-117">Anschließend wird zurück zur Homepage mit dem Zugriffstoken im temporären Fehlerwert umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="40ffd-117">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="40ffd-118">Verwenden Sie diese, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="40ffd-118">You'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="40ffd-119">Bevor Sie testen, müssen Sie die Ansichten zu `./tutorial/urls.py`hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="40ffd-119">Before you test, you need to add the views to `./tutorial/urls.py`.</span></span>

```python
path('signin', views.sign_in, name='signin'),
path('callback', views.callback, name='callback'),
```

<span data-ttu-id="40ffd-120">Ersetzen Sie `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` die- `./tutorial/templates/tutorial/home.html` Reihe in durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="40ffd-120">Replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line in `./tutorial/templates/tutorial/home.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="btn btn-primary btn-large">Click here to sign in</a>
```

<span data-ttu-id="40ffd-121">Ersetzen Sie `<a href="#" class="nav-link">Sign In</a>` die- `./tutorial/templates/tutorial/layout.html` Reihe in durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="40ffd-121">Replace the `<a href="#" class="nav-link">Sign In</a>` line in `./tutorial/templates/tutorial/layout.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="nav-link">Sign In</a>
```

<span data-ttu-id="40ffd-122">Starten Sie den Server, und `https://localhost:8000/tutorial`navigieren Sie zu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-122">Start the server and browse to `https://localhost:8000/tutorial`.</span></span> <span data-ttu-id="40ffd-123">Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="40ffd-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="40ffd-124">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den erforderlichen Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="40ffd-125">Der Browser leitet zur App um, wobei das Token angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="40ffd-125">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="40ffd-126">Benutzer Details abrufen</span><span class="sxs-lookup"><span data-stu-id="40ffd-126">Get user details</span></span>

<span data-ttu-id="40ffd-127">Erstellen Sie eine neue Datei im `./tutorial` Verzeichnis mit `graph_helper.py` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-127">Create a new file in the `./tutorial` directory named `graph_helper.py` and add the following code.</span></span>

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

<span data-ttu-id="40ffd-128">Die `get_user` -Methode führt eine GET-Anforderung an den `/me` Microsoft Graph-Endpunkt aus, um das Benutzerprofil mithilfe des zuvor erworbenen Zugriffstokens abzurufen.</span><span class="sxs-lookup"><span data-stu-id="40ffd-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

<span data-ttu-id="40ffd-129">Aktualisieren Sie `callback` die- `./tutorial/views.py` Methode in, um das Benutzerprofil von Microsoft Graph abzurufen.</span><span class="sxs-lookup"><span data-stu-id="40ffd-129">Update the `callback` method in `./tutorial/views.py` to get the user's profile from Microsoft Graph.</span></span>

<span data-ttu-id="40ffd-130">Fügen Sie zunächst die folgende `import` Anweisung am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-130">First, add the following `import` statement to the top of the file.</span></span>

```python
from tutorial.graph_helper import get_user
```

<span data-ttu-id="40ffd-131">Ersetzen Sie `callback` die Methode durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="40ffd-131">Replace the `callback` method with the following code.</span></span>

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

<span data-ttu-id="40ffd-132">Der neue Code Ruft die `get_user` Methode auf, um das Profil des Benutzers anzufordern.</span><span class="sxs-lookup"><span data-stu-id="40ffd-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="40ffd-133">Das Benutzerobjekt wird der temporären Ausgabe zu Testzwecken hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="40ffd-133">It adds the user object to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="40ffd-134">Speichern der Token</span><span class="sxs-lookup"><span data-stu-id="40ffd-134">Storing the tokens</span></span>

<span data-ttu-id="40ffd-135">Da Sie jetzt Tokens abrufen können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="40ffd-135">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="40ffd-136">Da es sich um eine Beispiel-App handelt, speichern Sie Sie in der Sitzung.</span><span class="sxs-lookup"><span data-stu-id="40ffd-136">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="40ffd-137">Eine echte APP würde eine zuverlässigere Secure Storage-Lösung wie eine Datenbank verwenden.</span><span class="sxs-lookup"><span data-stu-id="40ffd-137">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="40ffd-138">Fügen Sie die folgenden neuen Methoden `./tutorial/auth_helper.py`zu hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-138">Add the following new methods to `./tutorial/auth_helper.py`.</span></span>

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

<span data-ttu-id="40ffd-139">Aktualisieren Sie dann die `callback` -Funktion `./tutorial/views.py` in, um die Token in der Sitzung zu speichern und zurück zur Hauptseite umzuleiten.</span><span class="sxs-lookup"><span data-stu-id="40ffd-139">Then, update the `callback` function in `./tutorial/views.py` to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="40ffd-140">Ersetzen Sie `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` die-Reihe durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="40ffd-140">Replace the `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` line with the following.</span></span>

```python
from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
```

<span data-ttu-id="40ffd-141">Ersetzen Sie `callback` die Methode durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="40ffd-141">Replace the `callback` method with the following.</span></span>

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

## <a name="implement-sign-out"></a><span data-ttu-id="40ffd-142">Implementierung der Abmeldung</span><span class="sxs-lookup"><span data-stu-id="40ffd-142">Implement sign-out</span></span>

<span data-ttu-id="40ffd-143">Bevor Sie dieses neue Feature testen, fügen Sie eine Möglichkeit zum Abmelden hinzu. Fügen Sie eine `sign_out` neue Ansicht `./tutorial/views.py`in hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-143">Before you test this new feature, add a way to sign out. Add a new `sign_out` view in `./tutorial/views.py`.</span></span>

```python
def sign_out(request):
  # Clear out the user and token
  remove_user_and_token(request)

  return HttpResponseRedirect(reverse('home'))
```

<span data-ttu-id="40ffd-144">Fügen Sie nun diese Ansicht `./tutorial/urls.py`zu hinzu.</span><span class="sxs-lookup"><span data-stu-id="40ffd-144">Now add this view to `./tutorial/urls.py`.</span></span>

```python
path('signout', views.sign_out, name='signout'),
```

<span data-ttu-id="40ffd-145">Aktualisieren Sie \*\*\*\* den Link abmelden `./tutorial/templates/tutorial/layout.html` in, um diese neue Ansicht zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="40ffd-145">Update the **Sign Out** link in `./tutorial/templates/tutorial/layout.html` to use this new view.</span></span> <span data-ttu-id="40ffd-146">Ersetzen Sie `<a href="#" class="dropdown-item">Sign Out</a>` die-Reihe durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="40ffd-146">Replace the `<a href="#" class="dropdown-item">Sign Out</a>` line with the following.</span></span>

```html
<a href="{% url 'signout' %}" class="dropdown-item">Sign Out</a>
```

<span data-ttu-id="40ffd-147">Starten Sie den Server neu, und führen Sie den Anmeldevorgang durch.</span><span class="sxs-lookup"><span data-stu-id="40ffd-147">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="40ffd-148">Sie sollten auf der Startseite zurückkehren, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="40ffd-148">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

<span data-ttu-id="40ffd-150">Klicken Sie auf den Benutzer Avatar in der oberen rechten Ecke, \*\*\*\* um auf den Link abmelden zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="40ffd-150">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="40ffd-151">Wenn \*\*\*\* Sie auf Abmelden klicken, wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite.</span><span class="sxs-lookup"><span data-stu-id="40ffd-151">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Screenshot des Dropdownmenüs mit dem Link "abMelden"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="40ffd-153">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="40ffd-153">Refreshing tokens</span></span>

<span data-ttu-id="40ffd-154">Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="40ffd-154">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="40ffd-155">Dies ist das Token, mit dem die APP auf Microsoft Graph im Namen des Benutzers zugreifen kann.</span><span class="sxs-lookup"><span data-stu-id="40ffd-155">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="40ffd-156">Dieses Token ist jedoch kurzlebig.</span><span class="sxs-lookup"><span data-stu-id="40ffd-156">However, this token is short-lived.</span></span> <span data-ttu-id="40ffd-157">Das Token läuft eine Stunde nach der Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="40ffd-157">The token expires an hour after it is issued.</span></span> <span data-ttu-id="40ffd-158">An dieser Stelle wird das Aktualisierungstoken nützlich.</span><span class="sxs-lookup"><span data-stu-id="40ffd-158">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="40ffd-159">Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="40ffd-159">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="40ffd-160">Aktualisieren Sie den Token-Verwaltungscode, um die Token-Aktualisierung zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="40ffd-160">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="40ffd-161">Ersetzen Sie die `get_token` vorhandene- `./tutorial/auth_helper.py` Methode mit dem folgenden.</span><span class="sxs-lookup"><span data-stu-id="40ffd-161">Replace the existing `get_token` method in `./tutorial/auth_helper.py` with the following.</span></span>

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

<span data-ttu-id="40ffd-162">Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen ist oder in Kürze abläuft.</span><span class="sxs-lookup"><span data-stu-id="40ffd-162">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="40ffd-163">Wenn dies der Fall ist, wird das Aktualisierungstoken zum Abrufen neuer Token verwendet, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="40ffd-163">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>