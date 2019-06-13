<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="29dbb-101">In diesem Lernprogramm erfahren Sie, wie Sie eine python django-Webanwendung erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="29dbb-101">This tutorial teaches you how to build a Python Django web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="29dbb-102">Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie es auf zwei Arten herunterladen.</span><span class="sxs-lookup"><span data-stu-id="29dbb-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="29dbb-103">Laden Sie den [python-Schnellstart](https://developer.microsoft.com/graph/quick-start?platform=option-Python) herunter, um in wenigen Minuten Arbeitscode zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="29dbb-103">Download the [Python quick start](https://developer.microsoft.com/graph/quick-start?platform=option-Python) to get working code in minutes.</span></span>
> - <span data-ttu-id="29dbb-104">Laden Sie das [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)herunter, oder Klonen Sie es.</span><span class="sxs-lookup"><span data-stu-id="29dbb-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="29dbb-105">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="29dbb-105">Prerequisites</span></span>

<span data-ttu-id="29dbb-106">Bevor Sie mit diesem Lernprogramm beginnen, sollten Sie [python](https://www.python.org/) (mit [PIP](https://pypi.org/project/pip/)) auf Ihrem Entwicklungscomputer installiert haben.</span><span class="sxs-lookup"><span data-stu-id="29dbb-106">Before you start this tutorial, you should have [Python](https://www.python.org/) (with [pip](https://pypi.org/project/pip/)) installed on your development machine.</span></span> <span data-ttu-id="29dbb-107">Wenn Sie nicht über python verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="29dbb-107">If you do not have Python, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="29dbb-108">Dieses Tutorial wurde mit Python Version 3.7.0 und Django Version 2.2.2 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="29dbb-108">This tutorial was written with Python version 3.7.0 and Django version 2.2.2.</span></span> <span data-ttu-id="29dbb-109">Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="29dbb-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="29dbb-110">Feedback</span><span class="sxs-lookup"><span data-stu-id="29dbb-110">Feedback</span></span>

<span data-ttu-id="29dbb-111">Geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)an.</span><span class="sxs-lookup"><span data-stu-id="29dbb-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span></span>
