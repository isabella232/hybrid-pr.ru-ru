---
title: Настройка идентификатора гибридного облака для приложений Azure и Azure Stack Hub
description: Узнайте, как настроить идентификатор гибридного облака для приложений Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477258"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="f2937-103">Настройка идентификатора гибридного облака для приложений Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f2937-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="f2937-104">Из этой статьи вы узнаете, как настроить идентификатор гибридного облака для приложений Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f2937-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="f2937-105">В Azure и Azure Stack Hub предусмотрено два способа доступа к приложениям.</span><span class="sxs-lookup"><span data-stu-id="f2937-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="f2937-106">Если в Azure Stack Hub есть постоянное подключение к Интернету, можно использовать Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="f2937-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="f2937-107">Если в Azure Stack Hub нет подключения к Интернету, используйте службы федерации Active Directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="f2937-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="f2937-108">Субъекты-службы обеспечивают доступ к приложениям Azure Stack Hub для их развертывания или настройки с помощью Azure Resource Manager в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f2937-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="f2937-109">В этом решении показано, как создать среду, после чего вы выполните в ней следующие действия:</span><span class="sxs-lookup"><span data-stu-id="f2937-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f2937-110">создадите гибридный идентификатор для глобальной платформы Azure и Azure Stack Hub;</span><span class="sxs-lookup"><span data-stu-id="f2937-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="f2937-111">получите маркер для доступа к API Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f2937-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="f2937-112">Для выполнения указанных в этом решении действий необходимо иметь разрешения оператора Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f2937-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="f2937-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="f2937-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="f2937-114">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="f2937-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="f2937-115">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Только это гибридное облако позволяет создавать и развертывать гибридные приложения где угодно.</span><span class="sxs-lookup"><span data-stu-id="f2937-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="f2937-116">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="f2937-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="f2937-117">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="f2937-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="f2937-118">Создание субъекта-службы с доступом для Azure AD с помощью портала</span><span class="sxs-lookup"><span data-stu-id="f2937-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="f2937-119">Если вы развернули Azure Stack Hub с использованием Azure AD в качестве хранилища идентификаторов, субъекты-службы создаются точно так же, как для Azure.</span><span class="sxs-lookup"><span data-stu-id="f2937-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="f2937-120">Подробные сведения об использовании портала см. в статье [Предоставление приложениям доступа к Azure Stack](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity).</span><span class="sxs-lookup"><span data-stu-id="f2937-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="f2937-121">Прежде чем начать, проверьте [необходимые разрешения Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions).</span><span class="sxs-lookup"><span data-stu-id="f2937-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="f2937-122">Создание субъекта-службы для AD FS с помощью PowerShell</span><span class="sxs-lookup"><span data-stu-id="f2937-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="f2937-123">Если вы развернули Azure Stack Hub с использованием AD FS, для создания субъекта-службы, назначения роли для доступа и входа с этим идентификатором можно использовать PowerShell.</span><span class="sxs-lookup"><span data-stu-id="f2937-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="f2937-124">Подробные сведения об использовании PowerShell см. в статье [Предоставление приложениям доступа к Azure Stack](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity).</span><span class="sxs-lookup"><span data-stu-id="f2937-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="f2937-125">Использование API Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f2937-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="f2937-126">В описании решения [API Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use.md) приведены шаги для получения маркера для доступа к API Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f2937-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="f2937-127">Подключение к Azure Stack Hub с помощью PowerShell</span><span class="sxs-lookup"><span data-stu-id="f2937-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="f2937-128">Краткое руководство [Установка PowerShell для Azure Stack](/azure-stack/operator/azure-stack-powershell-install.md) поможет выполнить действия, необходимые для установки Azure PowerShell, и подключиться к установке Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f2937-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f2937-129">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="f2937-129">Prerequisites</span></span>

<span data-ttu-id="f2937-130">Установленная среда Azure Stack Hub с подключением к Azure AD и доступной подпиской.</span><span class="sxs-lookup"><span data-stu-id="f2937-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="f2937-131">Если у вас нет Azure Stack Hub, см. инструкции по установке [Пакета средств разработки Azure Stack (ASDK)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="f2937-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="f2937-132">Подключение к Azure Stack Hub с помощью кода</span><span class="sxs-lookup"><span data-stu-id="f2937-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="f2937-133">Чтобы подключиться к Azure Stack Hub с использованием кода, получите конечную точку аутентификации и конечную точку Graph для Azure Stack Hub с помощью API конечных точек Azure Resource Manager,</span><span class="sxs-lookup"><span data-stu-id="f2937-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="f2937-134">а затем выполните аутентификацию с помощью запросов REST.</span><span class="sxs-lookup"><span data-stu-id="f2937-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="f2937-135">Образец клиентского приложения можно найти на [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="f2937-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="f2937-136">Если пакет Azure SDK для используемого языка не поддерживает профили API Azure, вы не можете использовать его для работы с Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f2937-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="f2937-137">Дополнительные сведения о профилях API Azure см. в статье [Управление профилями версий API](/azure-stack/user/azure-stack-version-profiles.md).</span><span class="sxs-lookup"><span data-stu-id="f2937-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f2937-138">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="f2937-138">Next steps</span></span>

- <span data-ttu-id="f2937-139">Дополнительные сведения о работе с идентификаторами в Azure Stack Hub см. в [этой статье](/azure-stack/operator/azure-stack-identity-architecture.md).</span><span class="sxs-lookup"><span data-stu-id="f2937-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="f2937-140">Дополнительные сведения о шаблонах для облака Azure см. в статье [Конструктивные шаблоны облачных решений](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="f2937-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
