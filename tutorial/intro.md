<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="eddd7-101">In diesem Tutorial erfahren Sie, wie Sie eine python-django-Web-App erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="eddd7-101">This tutorial teaches you how to build a Python Django web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="eddd7-102">Wenn Sie das fertige Tutorial lieber herunterladen möchten, können Sie es auf zweierlei Weise herunterladen.</span><span class="sxs-lookup"><span data-stu-id="eddd7-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="eddd7-103">Laden Sie den [python-Schnellstart](https://developer.microsoft.com/graph/quick-start?platform=option-Python) herunter, um in wenigen Minuten Arbeitscode zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="eddd7-103">Download the [Python quick start](https://developer.microsoft.com/graph/quick-start?platform=option-Python) to get working code in minutes.</span></span>
> - <span data-ttu-id="eddd7-104">Laden oder Klonen Sie das [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span><span class="sxs-lookup"><span data-stu-id="eddd7-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="eddd7-105">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="eddd7-105">Prerequisites</span></span>

<span data-ttu-id="eddd7-106">Bevor Sie mit diesem Lernprogramm beginnen, sollten Sie [python](https://www.python.org/) (mit [PIP](https://pypi.org/project/pip/)) auf dem Entwicklungscomputer installiert haben.</span><span class="sxs-lookup"><span data-stu-id="eddd7-106">Before you start this tutorial, you should have [Python](https://www.python.org/) (with [pip](https://pypi.org/project/pip/)) installed on your development machine.</span></span> <span data-ttu-id="eddd7-107">Wenn Sie nicht über python verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="eddd7-107">If you do not have Python, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="eddd7-108">Dieses Tutorial wurde mit Python-Version 3.7.0 und Django Version 2,1 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="eddd7-108">This tutorial was written with Python version 3.7.0 and Django version 2.1.</span></span> <span data-ttu-id="eddd7-109">Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="eddd7-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="eddd7-110">Feedback</span><span class="sxs-lookup"><span data-stu-id="eddd7-110">Feedback</span></span>

<span data-ttu-id="eddd7-111">Feedback zu diesem Tutorial finden Sie im GitHub- [Repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span><span class="sxs-lookup"><span data-stu-id="eddd7-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp).</span></span>