# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="0990c-101">Ausführen des abgeschlossenen Projekts</span><span class="sxs-lookup"><span data-stu-id="0990c-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="0990c-102">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="0990c-102">Prerequisites</span></span>

<span data-ttu-id="0990c-103">Zum Ausführen des abgeschlossenen Projekts in diesem Ordner benötigen Sie Folgendes:</span><span class="sxs-lookup"><span data-stu-id="0990c-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="0990c-104">[Python](https://www.python.org/) (mit [PIP](https://pypi.org/project/pip/)) auf dem Entwicklungscomputer installiert.</span><span class="sxs-lookup"><span data-stu-id="0990c-104">[Python](https://www.python.org/) (with [pip](https://pypi.org/project/pip/)) installed on your development machine.</span></span> <span data-ttu-id="0990c-105">Wenn Sie nicht über python verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="0990c-105">If you do not have Python, visit the previous link for download options.</span></span> <span data-ttu-id="0990c-106">(**Hinweis:** dieses Tutorial wurde mit Python-Version 3.7.0 und Django Version 2,1 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="0990c-106">(**Note:** This tutorial was written with Python version 3.7.0 and Django version 2.1.</span></span> <span data-ttu-id="0990c-107">Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, die jedoch nicht getestet wurden.)</span><span class="sxs-lookup"><span data-stu-id="0990c-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="0990c-108">Entweder ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder ein Microsoft-Geschäfts-oder Schulkonto.</span><span class="sxs-lookup"><span data-stu-id="0990c-108">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="0990c-109">Wenn Sie nicht über ein Microsoft-Konto verfügen, gibt es eine Reihe von Optionen, um ein kostenloses Konto zu erhalten:</span><span class="sxs-lookup"><span data-stu-id="0990c-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="0990c-110">Sie können [sich für ein neues persönliches Microsoft-Konto registrieren](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="0990c-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="0990c-111">Sie können [sich für das office 365-Entwicklerprogramm anmelden](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="0990c-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-application-registration-portal"></a><span data-ttu-id="0990c-112">Registrieren einer Webanwendung mit dem Anwendungs Registrierungs Portal</span><span class="sxs-lookup"><span data-stu-id="0990c-112">Register a web application with the Application Registration Portal</span></span>

1. <span data-ttu-id="0990c-113">Öffnen Sie einen Browser, und navigieren Sie zum [Anwendungs Registrierungs Portal](https://apps.dev.microsoft.com).</span><span class="sxs-lookup"><span data-stu-id="0990c-113">Open a browser and navigate to the [Application Registration Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="0990c-114">Melden Sie sich über ein **persönliches Konto** (aka: Microsoft-Konto) oder ein Geschäfts- **oder Schulkonto**an.</span><span class="sxs-lookup"><span data-stu-id="0990c-114">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="0990c-115">Wählen Sie oben auf der Seite **eine APP hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="0990c-115">Select **Add an app** at the top of the page.</span></span>

    > <span data-ttu-id="0990c-116">**Hinweis:** Wenn auf der Seite mehr als eine Schaltfläche **app hinzufügen** angezeigt wird, wählen Sie diejenige aus, die der Liste **konvergierter apps** entspricht.</span><span class="sxs-lookup"><span data-stu-id="0990c-116">**Note:** If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="0990c-117">Legen Sie auf der Seite **Anwendung registrieren** den **Anwendungsnamen** in **python Graph Tutorial** fest, und wählen Sie **Erstellen**aus.</span><span class="sxs-lookup"><span data-stu-id="0990c-117">On the **Register your application** page, set the **Application Name** to **Python Graph Tutorial** and select **Create**.</span></span>

    ![Screenshot des Erstellens einer neuen app in der APP-Registrierungs Portal-Website](/Images/arp-create-app-01.png)

1. <span data-ttu-id="0990c-119">Kopieren Sie auf der Seite mit der **python-Diagramm-Lernprogramm Registrierung** im Abschnitt **Eigenschaften** die **Anwendungs-ID** , so wie Sie Sie später benötigen.</span><span class="sxs-lookup"><span data-stu-id="0990c-119">On the **Python Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![Screenshot der neu erstellten Anwendungs-ID](/Images/arp-create-app-02.png)

1. <span data-ttu-id="0990c-121">Scrollen Sie nach unten zum Abschnitt **Anwendungs Geheimnisse** .</span><span class="sxs-lookup"><span data-stu-id="0990c-121">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="0990c-122">Wählen Sie **Neues Kennwort generieren**aus.</span><span class="sxs-lookup"><span data-stu-id="0990c-122">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="0990c-123">Kopieren Sie im Dialogfeld **Neues Kennwort generiert** den Inhalt des Felds, so wie Sie es später benötigen.</span><span class="sxs-lookup"><span data-stu-id="0990c-123">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > <span data-ttu-id="0990c-124">**Wichtig:** Dieses Kennwort wird nie wieder angezeigt, stellen Sie daher sicher, dass Sie es jetzt kopieren.</span><span class="sxs-lookup"><span data-stu-id="0990c-124">**Important:** This password is never shown again, so make sure you copy it now.</span></span>

    ![Screenshot des Kennworts der neu erstellten Anwendung](/Images/arp-create-app-03.png)

1. <span data-ttu-id="0990c-126">Scrollen Sie nach unten zum Abschnitt **Plattformen** .</span><span class="sxs-lookup"><span data-stu-id="0990c-126">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="0990c-127">Wählen Sie **Plattform hinzufügen**aus.</span><span class="sxs-lookup"><span data-stu-id="0990c-127">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="0990c-128">Wählen Sie im Dialogfeld **Plattform hinzufügen** die Option **Web**aus.</span><span class="sxs-lookup"><span data-stu-id="0990c-128">In the **Add Platform** dialog, select **Web**.</span></span>

        ![Screenshot Erstellen einer Plattform für die APP](/Images/arp-create-app-04.png)

    1. <span data-ttu-id="0990c-130">Geben Sie \*\*\*\* im Feld Webplattform die URL `http://localhost:8000/tutorial/callback` für die Umleitungs- **URLs**ein.</span><span class="sxs-lookup"><span data-stu-id="0990c-130">In the **Web** platform box, enter the URL `http://localhost:8000/tutorial/callback` for the **Redirect URLs**.</span></span>

        ![Screenshot der neu hinzugefügten Webplattform für die Anwendung](/Images/arp-create-app-05.png)

1. <span data-ttu-id="0990c-132">Scrollen Sie zum unteren Rand der Seite, und wählen Sie **Speichern**aus.</span><span class="sxs-lookup"><span data-stu-id="0990c-132">Scroll to the bottom of the page and select **Save**.</span></span>

## <a name="configure-the-sample"></a><span data-ttu-id="0990c-133">Konfigurieren des Beispiels</span><span class="sxs-lookup"><span data-stu-id="0990c-133">Configure the sample</span></span>

1. <span data-ttu-id="0990c-134">Benennen Sie `oauth_settings.yml.example` die Datei `oauth_settings.yml`in um.</span><span class="sxs-lookup"><span data-stu-id="0990c-134">Rename the `oauth_settings.yml.example` file to `oauth_settings.yml`.</span></span>
1. <span data-ttu-id="0990c-135">Bearbeiten Sie `oauth_settings.yml` die Datei, und nehmen Sie die folgenden Änderungen vor.</span><span class="sxs-lookup"><span data-stu-id="0990c-135">Edit the `oauth_settings.yml` file and make the following changes.</span></span>
    1. <span data-ttu-id="0990c-136">Ersetzen `YOUR_APP_ID_HERE` Sie durch die **Anwendungs-ID** , die Sie im App-Registrierungs Portal erhalten haben.</span><span class="sxs-lookup"><span data-stu-id="0990c-136">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="0990c-137">Ersetzen `YOUR_APP_PASSWORD_HERE` Sie durch das Kennwort, das Sie im App-Registrierungs Portal erhalten haben.</span><span class="sxs-lookup"><span data-stu-id="0990c-137">Replace `YOUR_APP_PASSWORD_HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="0990c-138">Navigieren Sie in der Befehlszeilenschnittstelle zu diesem Verzeichnis, und führen Sie den folgenden Befehl aus, um die Anforderungen zu installieren.</span><span class="sxs-lookup"><span data-stu-id="0990c-138">In your command-line interface (CLI), navigate to this directory and run the following command to install requirements.</span></span>

    ```Shell
    pip install -r requirements.txt
    ```

1. <span data-ttu-id="0990c-139">Führen Sie in der CLI den folgenden Befehl aus, um die Datenbank der APP zu initialisieren.</span><span class="sxs-lookup"><span data-stu-id="0990c-139">In your CLI, run the following command to initialize the app's database.</span></span>

    ```Shell
    python manage.py migrate
    ```

## <a name="run-the-sample"></a><span data-ttu-id="0990c-140">Ausführen des Beispiels</span><span class="sxs-lookup"><span data-stu-id="0990c-140">Run the sample</span></span>

1. <span data-ttu-id="0990c-141">Führen Sie den folgenden Befehl in der CLI aus, um die Anwendung zu starten.</span><span class="sxs-lookup"><span data-stu-id="0990c-141">Run the following command in your CLI to start the application.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="0990c-142">Öffnen Sie einen Browser, und navigieren Sie zu `http://localhost:8000/tutorial`.</span><span class="sxs-lookup"><span data-stu-id="0990c-142">Open a browser and browse to `http://localhost:8000/tutorial`.</span></span>