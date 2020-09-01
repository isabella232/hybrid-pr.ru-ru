---
title: Развертывание приложения, которое выполняет масштабирование в нескольких облаках в Azure и Azure Stack Hub
description: Узнайте, как развертывать приложение, которое выполняет масштабирование в нескольких облаках, используя Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886821"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="fa7ad-103">Развертывание приложения, которое выполняет масштабирование в нескольких облаках с помощью Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fa7ad-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fa7ad-104">Из этой статьи вы узнаете, как создать решение в нескольких облаках, чтобы обеспечить активируемый вручную процесс переключения с веб-приложения, размещенного в Azure Stack Hub, на веб-приложение, размещенное в Azure, с автоматическим масштабированием с помощью диспетчера трафика.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="fa7ad-105">Это позволит вам получить гибкую и масштабируемую облачную служебную программу для работы под нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="fa7ad-106">Возможно, что при использовании этого шаблона клиент не будет готов к запуску приложения в общедоступном облаке.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="fa7ad-107">Тем не менее для бизнеса экономически нецелесообразно поддерживать требуемую емкость в своей локальной среде для обработки пиков спроса на приложение.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="fa7ad-108">Ваш клиент может использовать преимущества, связанные с эластичностью общедоступного облака, в локальном решении.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="fa7ad-109">В этом решении показано, как создать среду, после чего вы выполните в ней следующие действия:</span><span class="sxs-lookup"><span data-stu-id="fa7ad-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="fa7ad-110">Создадите многоузловое веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="fa7ad-111">Настроите процесс непрерывного развертывания и управление им.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="fa7ad-112">Опубликуете веб-приложение в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="fa7ad-113">Создадите выпуск.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-113">Create a release.</span></span>
> - <span data-ttu-id="fa7ad-114">Узнаете, как отслеживать развертывания и управлять ими.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="fa7ad-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="fa7ad-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="fa7ad-116">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="fa7ad-117">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Только это гибридное облако позволяет создавать и развертывать гибридные приложения где угодно.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="fa7ad-118">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="fa7ad-119">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="fa7ad-120">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="fa7ad-120">Prerequisites</span></span>

- <span data-ttu-id="fa7ad-121">Подписка Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-121">Azure subscription.</span></span> <span data-ttu-id="fa7ad-122">При необходимости для начала создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="fa7ad-123">Используйте систему с Azure Stack Hub или разверните Пакет средств разработки Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="fa7ad-124">Инструкции по установке Azure Stack Hub см. в статье [Установка ASDK](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="fa7ad-125">Скрипт автоматизации, используемый после развертывания ASDK, можно найти по адресу [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="fa7ad-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="fa7ad-126">Выполнение этой установки может занять несколько часов.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="fa7ad-127">Разверните службы PaaS [Службы приложений](/azure-stack/operator/azure-stack-app-service-deploy.md) в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="fa7ad-128">[Создайте план и предложения](/azure-stack/operator/service-plan-offer-subscription-overview.md) в рамках среды Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="fa7ad-129">[Создайте подписку клиента](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) в рамках среды Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="fa7ad-130">Создайте веб-приложение в подписке клиента.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="fa7ad-131">Запишите новый URL-адрес веб-приложения для дальнейшего использования.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="fa7ad-132">Разверните виртуальную машину Azure Pipelines в подписке клиента.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="fa7ad-133">Требуется виртуальная машина Windows Server 2016 с .NET 3.5.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="fa7ad-134">Эта виртуальная машина будет создана в подписке клиента Azure Stack Hub в качестве частного агента сборки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="fa7ad-135">[Windows Server 2016 с образом виртуальной машины SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) доступен в Azure Stack Hub Marketplace.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="fa7ad-136">Если этот образ недоступен, обратитесь к оператору Azure Stack Hub, чтобы убедиться, что он добавлен в среду.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="fa7ad-137">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="fa7ad-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="fa7ad-138">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="fa7ad-138">Scalability</span></span>

<span data-ttu-id="fa7ad-139">Ключевым компонентом масштабирования в нескольких облаках является возможность немедленного масштабирования по запросу в инфраструктуре общедоступного и локального облаков. Это позволяет обеспечить согласованность и надежность служб.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="fa7ad-140">Доступность</span><span class="sxs-lookup"><span data-stu-id="fa7ad-140">Availability</span></span>

<span data-ttu-id="fa7ad-141">Убедитесь, что локально развернутые приложения настроены для обеспечения высокой доступности с помощью конфигурации локального оборудования и развертывания программного обеспечения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="fa7ad-142">Управляемость</span><span class="sxs-lookup"><span data-stu-id="fa7ad-142">Manageability</span></span>

<span data-ttu-id="fa7ad-143">Решения в нескольких облаках обеспечивают простое управление и похожий интерфейс между средами.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="fa7ad-144">Для кроссплатформенного управления рекомендуется использовать PowerShell.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="fa7ad-145">Масштабирование в нескольких облаках</span><span class="sxs-lookup"><span data-stu-id="fa7ad-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="fa7ad-146">Получение личного домена и настройка DNS</span><span class="sxs-lookup"><span data-stu-id="fa7ad-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="fa7ad-147">Обновите файл зоны DNS для домена.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="fa7ad-148">Azure AD проверит принадлежность имени личного домена.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="fa7ad-149">Вы можете использовать [Azure DNS](/azure/dns/dns-getstarted-portal) для записей Azure, Microsoft 365 и внешних записей DNS в Azure или добавить запись DNS в [другой регистратор DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="fa7ad-150">Зарегистрируйте личный домен у уполномоченного регистратора.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="fa7ad-151">Войдите в соответствующий регистратор доменных имен.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="fa7ad-152">Для обновления DNS может потребоваться утвержденный администратор.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="fa7ad-153">Обновите файл зоны DNS для соответствующего домена, добавив предоставленную службой Azure AD DNS-запись.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="fa7ad-154">(DNS-запись не повлияет на маршрутизацию почты или поведение при веб-размещении.)</span><span class="sxs-lookup"><span data-stu-id="fa7ad-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="fa7ad-155">Создание многоузлового веб-приложения по умолчанию в Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fa7ad-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="fa7ad-156">Настройте гибридный конвейер непрерывной интеграции и непрерывного развертывания (CI/CD), чтобы развернуть веб-приложения в Azure и Azure Stack Hub и включить автоматическую отправку изменений в оба облака.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="fa7ad-157">Вам необходим Azure Stack Hub с подходящими образами, объединенными для запуска (Windows Server и SQL), и развернутой Службой приложений.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="fa7ad-158">Дополнительные сведения см. в документации по службе приложений Azure ([Предварительные требования для развертывания Службы приложений в Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="fa7ad-159">Добавление кода в Azure Repos</span><span class="sxs-lookup"><span data-stu-id="fa7ad-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="fa7ad-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="fa7ad-160">Azure Repos</span></span>

1. <span data-ttu-id="fa7ad-161">Войдите в Azure Repos, используя учетную запись, которая обладает правами на создание проекта в Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="fa7ad-162">Гибридные процессы CI/CD применимы и к коду приложения, и к коду инфраструктуры.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="fa7ad-163">Используйте [шаблоны Azure Resource Manager](https://azure.microsoft.com/resources/templates/) для частной и облачной разработки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Подключение к проекту в Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="fa7ad-165">**Клонируйте репозиторий** путем создания и открытия веб-приложения по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Клонирование репозитория в веб-приложение Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="fa7ad-167">Создание автономного развертывания веб-приложения для служб приложений в обоих облаках</span><span class="sxs-lookup"><span data-stu-id="fa7ad-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="fa7ad-168">Измените файл **WebApplication.csproj**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="fa7ad-169">Выберите `Runtimeidentifier` и добавьте `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="fa7ad-170">(См. документацию по [автономному развертыванию](/dotnet/core/deploying/deploy-with-vs#simpleSelf)).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Изменение файла проекта веб-приложения](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="fa7ad-172">Добавьте код в Azure Repos с помощью Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="fa7ad-173">Подтвердите, что код приложения был добавлен в Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="fa7ad-174">Создание определения сборки</span><span class="sxs-lookup"><span data-stu-id="fa7ad-174">Create the build definition</span></span>

1. <span data-ttu-id="fa7ad-175">Войдите в Azure Pipelines, чтобы подтвердить возможность создания определений сборки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="fa7ad-176">Добавьте код **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="fa7ad-177">Это необходимо, чтобы активировать автономное развертывание с использованием .NET Core.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Добавление кода в веб-приложение](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="fa7ad-179">Запустите сборку.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-179">Run the build.</span></span> <span data-ttu-id="fa7ad-180">Процесс [сборки автономного развертывания](/dotnet/core/deploying/deploy-with-vs#simpleSelf) будет публиковать артефакты, которые выполняются в Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="fa7ad-181">Использование агента, размещенного в Azure</span><span class="sxs-lookup"><span data-stu-id="fa7ad-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="fa7ad-182">С помощью агента сборки, размещенного в Azure Pipelines, можно с легкостью создавать и развертывать веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="fa7ad-183">Обновление и обслуживание агента, включая постоянную непрерывную разработку, автоматически выполняет Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="fa7ad-184">Настройка процесса непрерывного развертывания и управление им</span><span class="sxs-lookup"><span data-stu-id="fa7ad-184">Manage and configure the CD process</span></span>

<span data-ttu-id="fa7ad-185">Azure Pipelines и Azure DevOps Services предоставляют конвейер с широкими возможностями настройки и управления для выпусков в различные среды, такие как среда развертывания, промежуточная среда, среда для контроля качества и рабочая среда, включая получение утверждений на определенных стадиях.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="fa7ad-186">Создание определения выпуска</span><span class="sxs-lookup"><span data-stu-id="fa7ad-186">Create release definition</span></span>

1. <span data-ttu-id="fa7ad-187">Нажмите кнопку со знаком **плюса**, чтобы добавить новый выпуск на вкладке **Выпуски** в разделе **Сборка и выпуск** Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Создание определения выпуска](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="fa7ad-189">Примените шаблон развертывания службы приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-189">Apply the Azure App Service Deployment template.</span></span>

   ![Применение шаблона развертывания для Службы приложений Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="fa7ad-191">В разделе **Добавление артефакта** добавьте артефакт для приложения сборки облака Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Добавление артефакта в сборку облака Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="fa7ad-193">На вкладке "Конвейер" щелкните ссылку **Фаза, задача** для используемой среды и задайте значения облачной среды Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Настройка значений среды облака Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="fa7ad-195">Задайте **имя среды** и выберите **подписку Azure** для конечной точки облака Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Выбор подписки Azure для конечной точки облака Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="fa7ad-197">В разделе **Имя Службы приложений** укажите требуемое имя.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Выбор имени Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="fa7ad-199">Введите "Размещенная среда VS2017" в разделе **Очередь агента** для среды, размещенной в облаке Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Настройка очереди агента для среды, размещенной в облаке Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="fa7ad-201">В меню развертывания службы приложений Azure выберите допустимый для среды **пакет или папку**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="fa7ad-202">Нажмите кнопку **ОК**, чтобы выбрать **расположение папки**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-202">Select **OK** to **folder location**.</span></span>
  
      ![Выбор пакета или папки для среды Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Выбор пакета или папки для среды Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="fa7ad-205">Сохраните все изменения и вернитесь к **конвейеру выпуска**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Сохранение изменений в конвейере выпуска](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="fa7ad-207">Добавьте новый артефакт, выбрав сборку для приложения Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Добавление нового артефакта для приложения Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="fa7ad-209">Добавьте еще одну среду, применив развертывание Службы приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Добавление среды в развертывание Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="fa7ad-211">Назовите новую среду Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-211">Name the new environment "Azure Stack".</span></span>

    ![Присвоение имени среде в развертывании Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="fa7ad-213">Найдите среду Azure Stack на вкладке **Задача**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Среда Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="fa7ad-215">Выберите подписку для конечной точки Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Выбор подписки для конечной точки Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="fa7ad-217">Задайте имя веб-приложения Azure Stack в качестве имени службы приложений.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="fa7ad-218">![Присвоение имени веб-приложению Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="fa7ad-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="fa7ad-219">Выберите агент Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-219">Select the Azure Stack agent.</span></span>

    ![Выбор агента Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="fa7ad-221">В разделе "Развертывание Службы приложений Azure" выберите допустимый **пакет или папку** для среды.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="fa7ad-222">Нажмите кнопку **ОК**, чтобы выбрать расположение папки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-222">Select **OK** to folder location.</span></span>

    ![Выбор папки для развертывания Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Выбор папки для развертывания Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="fa7ad-225">В разделе "Переменные" добавьте переменную `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, задайте для нее значение **true** и область Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Добавление переменной в развертывание Службы приложений Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="fa7ad-227">Выберите значок триггера **непрерывного** развертывания в обоих артефактах и включите триггер **непрерывного** развертывания.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Выбор триггера непрерывного развертывания](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="fa7ad-229">Выберите значок условий **перед развертыванием** в среде Azure Stack и задайте триггеру значение **После выпуска**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Выбор условий, выполняемых перед развертыванием](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="fa7ad-231">Сохраните все изменения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="fa7ad-232">Некоторые параметры для задач могли быть автоматически определены как [переменные среды](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) при создании определения выпуска на основе шаблона.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="fa7ad-233">Эти параметры нельзя изменить в параметрах задачи. Для этого нужно выбрать родительский элемент среды.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="fa7ad-234">Публикация в Azure Stack Hub с помощью Visual Studio</span><span class="sxs-lookup"><span data-stu-id="fa7ad-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="fa7ad-235">Создавая конечные точки, сборка Azure DevOps Services может развертывать приложения службы Azure в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="fa7ad-236">Azure Pipelines подключается к агенту сборки, который подключается к Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="fa7ad-237">Войдите в Azure DevOps Services и перейдите на страницу параметров приложения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="fa7ad-238">В разделе **Параметры** выберите **Безопасность**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="fa7ad-239">В списке **Группы VSTS** выберите **Создатели конечных точек**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="fa7ad-240">На вкладке **Члены** выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="fa7ad-241">В разделе **Add users and groups** (Добавление пользователей и групп) введите имя пользователя и выберите этого пользователя из списка.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="fa7ad-242">Щелкните **Save changes** (Сохранить изменения).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="fa7ad-243">В списке **Группы VSTS** выберите **Администраторы конечной точки**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="fa7ad-244">На вкладке **Члены** выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="fa7ad-245">В разделе **Add users and groups** (Добавление пользователей и групп) введите имя пользователя и выберите этого пользователя из списка.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="fa7ad-246">Щелкните **Save changes** (Сохранить изменения).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-246">Select **Save changes**.</span></span>

<span data-ttu-id="fa7ad-247">Теперь, когда сведения о конечной точке существуют, подключение Azure Pipelines к Azure Stack Hub готово к использованию.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="fa7ad-248">Агент сборки в Azure Stack Hub получает инструкции от Azure Pipelines, а затем передает сведения о конечной точке для взаимодействия с Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="fa7ad-249">Разработка сборки приложения</span><span class="sxs-lookup"><span data-stu-id="fa7ad-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="fa7ad-250">Вам необходим Azure Stack Hub с подходящими образами, объединенными для запуска (Windows Server и SQL), и развернутой Службой приложений.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="fa7ad-251">Подробнее см. статью [Предварительные условия для развертывания Службы приложений в Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="fa7ad-252">Используйте такие [шаблоны Azure Resource Manager](https://azure.microsoft.com/resources/templates/), как код веб-приложения из Azure Repos, для развертывания в обоих облаках.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="fa7ad-253">Добавление кода в проект Azure Repos</span><span class="sxs-lookup"><span data-stu-id="fa7ad-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="fa7ad-254">Войдите в Azure Repos, используя учетную запись, которая обладает правами на создание проекта в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="fa7ad-255">**Клонируйте репозиторий** путем создания и открытия веб-приложения по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="fa7ad-256">Создание автономного развертывания веб-приложения для служб приложений в обоих облаках</span><span class="sxs-lookup"><span data-stu-id="fa7ad-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="fa7ad-257">Измените файл **WebApplication.csproj**. Выберите `Runtimeidentifier` и добавьте `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="fa7ad-258">Дополнительные сведения см. в документации [по автономному развертыванию](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="fa7ad-259">Добавьте код в Azure Repos с помощью Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="fa7ad-260">Подтвердите, что код приложения был добавлен в Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="fa7ad-261">Создание определения сборки</span><span class="sxs-lookup"><span data-stu-id="fa7ad-261">Create the build definition</span></span>

1. <span data-ttu-id="fa7ad-262">Войдите в Azure Pipelines, используя учетную запись, с помощью которой можно создать определение сборки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="fa7ad-263">Перейдите на страницу **Build Web Application** (Сборка веб-приложения) для проекта.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="fa7ad-264">Добавьте код **-r win10-x64** в поле **Аргумент**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="fa7ad-265">Это необходимо, чтобы активировать автономное развертывание с использованием .NET Core.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="fa7ad-266">Запустите сборку.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-266">Run the build.</span></span> <span data-ttu-id="fa7ad-267">Процесс [сборки автономного развертывания](/dotnet/core/deploying/deploy-with-vs#simpleSelf) будет публиковать артефакты, которые могут выполняться в Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="fa7ad-268">Использование агента сборки, размещенного в Azure</span><span class="sxs-lookup"><span data-stu-id="fa7ad-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="fa7ad-269">С помощью агента сборки, размещенного в Azure Pipelines, можно с легкостью создавать и развертывать веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="fa7ad-270">Обновление и обслуживание агента, включая постоянную непрерывную разработку, автоматически выполняет Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="fa7ad-271">Настройка процесса непрерывного развертывания</span><span class="sxs-lookup"><span data-stu-id="fa7ad-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="fa7ad-272">Azure Pipelines и Azure DevOps Services предоставляют конвейер с широкими возможностями настройки и управления для выпусков в различные среды, такие как среда разработки, промежуточная среда, среда для контроля качества и рабочая среда.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="fa7ad-273">Этот процесс может охватывать требование утверждений на определенных стадиях жизненного цикла приложения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="fa7ad-274">Создание определения выпуска</span><span class="sxs-lookup"><span data-stu-id="fa7ad-274">Create release definition</span></span>

<span data-ttu-id="fa7ad-275">Создание определения выпуска — это последний шаг в процессе сборки приложения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="fa7ad-276">Определение выпуска используется для создания выпуска и развертывания сборки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="fa7ad-277">Войдите в Azure Pipelines и перейдите к разделу **Сборка и выпуск** для своего проекта.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="fa7ad-278">На вкладке **Выпуски** выберите **[ + ]** , а затем — **Создать определение выпуска**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="fa7ad-279">В разделе **Select a Template** (Выбор шаблона) выберите **Развертывание службы приложений Azure**, а затем — **Применить**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="fa7ad-280">В разделе **Добавление артефакта** в раскрывающемся меню **Источник (определение сборки)** выберите приложение сборки облака Azure.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="fa7ad-281">На вкладке **Конвейер** выберите ссылку **Фаза 1**, **Задача 1**, чтобы **просмотреть задачи среды**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="fa7ad-282">На вкладке **Задачи** укажите Azure в качестве **имени среды** и выберите подписку AzureCloud Traders-Web EP из раскрывающегося списка **Подписка Azure**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="fa7ad-283">Введите **имя службы приложений Azure**. На следующем снимке экрана указано имя `northwindtraders`.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="fa7ad-284">Для фазы агента выберите **Hosted VS2017** (Размещается в VS2017) из раскрывающегося списка **Очередь агентов**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="fa7ad-285">В разделе **Развертывание службы приложений Azure** выберите допустимый для среды **пакет или папку**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="fa7ad-286">В разделе **Select File or Folder** (Выбор файла или папки) нажмите кнопку **ОК**, чтобы выбрать **расположение**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="fa7ad-287">Сохраните все изменения и вернитесь к **конвейеру**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="fa7ad-288">На вкладке **Конвейер** щелкните **Добавить артефакт**, а затем из раскрывающегося списка **Источник (определение сборки)** выберите **NorthwindCloud Traders-Vessel**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="fa7ad-289">В разделе **Select a Template** (Выбор шаблона) добавьте другую среду.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="fa7ad-290">Выберите **Развертывание службы приложений Azure**, а затем щелкните **Применить**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="fa7ad-291">Введите `Azure Stack Hub` в качестве **имени среды**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="fa7ad-292">На вкладке **Задачи** найдите и выберите Azure Stack Tasks.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="fa7ad-293">В раскрывающемся списке **Подписка Azure** выберите **AzureStack Traders-Vessel EP** для конечной точки Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="fa7ad-294">Введите имя веб-приложения Azure Stack Hub в качестве **имени службы приложений**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="fa7ad-295">В разделе **Выбор агентов** выберите **AzureStack -b Douglas Fir** из раскрывающегося списка **Очередь агентов**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="fa7ad-296">В разделе **Развертывание службы приложений Azure** выберите допустимый для среды **пакет или папку**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="fa7ad-297">В разделе **Select File Or Folder** (Выбор файла или папки) нажмите кнопку **OК**, чтобы выбрать **расположение** папки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="fa7ad-298">На вкладке **Переменная** найдите переменную с именем `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="fa7ad-299">Задайте для переменной значение **true**, а для области — **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="fa7ad-300">На вкладке **Конвейер** выберите значок **триггера непрерывного развертывания** для артефакта NorthwindCloud Traders-Web, а для **триггера непрерывного развертывания** задайте значение **Включено**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="fa7ad-301">То же самое сделайте для артефакта **NorthwindCloud Traders-Vessel**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="fa7ad-302">В среде Azure Stack Hub выберите значок **Условия перед развертыванием** и задайте триггеру значение **После выпуска**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="fa7ad-303">Сохраните все изменения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="fa7ad-304">Некоторые параметры для задач выпуска могли быть автоматически определены как [переменные среды](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) при создании определения выпуска на основе шаблона.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="fa7ad-305">Эти параметры невозможно изменить в настройках задачи, но можно изменить в элементах родительской среды.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="fa7ad-306">Создание выпуска</span><span class="sxs-lookup"><span data-stu-id="fa7ad-306">Create a release</span></span>

1. <span data-ttu-id="fa7ad-307">На вкладке **Конвейер** откройте раскрывающийся список **Выпуск** и выберите **Создать выпуск**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="fa7ad-308">Введите описание выпуска, проверьте, выбраны ли правильные артефакты, а затем нажмите **Создать**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="fa7ad-309">Через несколько секунд появится баннер с сообщением о создании выпуска, а имя выпуска отобразится в виде ссылки.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="fa7ad-310">Щелкните ссылку, чтобы просмотреть страницу сводных данных о выпуске.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="fa7ad-311">Откроется страница со сводными данными о выпуске.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="fa7ad-312">На следующем снимке экрана выпуска 2 в разделе **Среды** для **состояния развертывания** для Azure отображается значение "Выполняется", а для Azure Stack Hub — "Успешно".</span><span class="sxs-lookup"><span data-stu-id="fa7ad-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="fa7ad-313">Когда состояние развертывания для среды Azure изменится на "Успешно", появится баннер, указывающий, что выпуск готов к утверждению.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="fa7ad-314">Если развертывание находится в состоянии ожидания или завершилось с ошибкой, отобразится синий значок информации **(i)** .</span><span class="sxs-lookup"><span data-stu-id="fa7ad-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="fa7ad-315">Наведите на него курсор, чтобы увидеть всплывающий элемент со сведениями о причине задержки или сбоя.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="fa7ad-316">Другие представления, такие как список выпусков, также отображают значок, который указывает, что ожидается утверждение.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="fa7ad-317">Всплывающий элемент этого значка содержит имя среды и дополнительные сведения о развертывании.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="fa7ad-318">Администратор может легко увидеть общий ход выполнения выпусков, а также какие выпуски ожидают утверждения.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="fa7ad-319">Мониторинг и отслеживание развертываний</span><span class="sxs-lookup"><span data-stu-id="fa7ad-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="fa7ad-320">На странице сводных данных о **выпуске 2** выберите ссылку **Журналы**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="fa7ad-321">Во время развертывания на этой странице отображается динамический журнал агента.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="fa7ad-322">На панели слева отображается состояние каждой операции в развертывании для каждой среды.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="fa7ad-323">Выберите значок пользователя в столбце **Действие** для утверждения, используемого до развертывания или после него. Так вы сможете узнать, кто одобрил или отклонил развертывание, а также просмотреть соответствующее сообщение.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="fa7ad-324">После завершения развертывания в области справа отображается весь файл журнала.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="fa7ad-325">Выберите любой из **шагов** в области слева, чтобы просмотреть файл журнала для отдельного шага, например **Инициализация задания**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="fa7ad-326">Возможность просмотра отдельных журналов упрощает трассировку и отладку частей общего развертывания.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="fa7ad-327">Вы можете **Сохранить** файлы журнала для этого шага или выбрать **Скачать все журналы как ZIP-файл**.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="fa7ad-328">Откройте вкладку **Сводка**, чтобы увидеть общие сведения о выпуске.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="fa7ad-329">В этом представлении отображаются сведения о сборке, средах, в которые она была развернута, состояние развертывания и другая информация о выпуске.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="fa7ad-330">Выберите ссылку среды (**Azure** или **Azure Stack Hub**), чтобы получить дополнительные сведения об имеющихся развертываниях и развертываниях в состоянии ожидания для определенной среды.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="fa7ad-331">Эти представления можно использовать, чтобы быстро убедиться, что та же сборка была развернута в обеих средах.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="fa7ad-332">Откройте **развернутое рабочее приложение** в своем браузере.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="fa7ad-333">Например, для веб-сайта Службы приложений Azure откройте следующий URL-адрес: `https://[your-app-name\].azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="fa7ad-334">Интеграция Azure и Azure Stack Hub обеспечивает решение, масштабируемое в нескольких облаках</span><span class="sxs-lookup"><span data-stu-id="fa7ad-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="fa7ad-335">Гибкая и надежная служба для множества облаков обеспечивает защиту данных, резервное копирование и избыточность, согласованность и быструю доступность, масштабируемое хранение и распределение, а также геореплицированную маршрутизацию.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="fa7ad-336">Этот процесс с активацией вручную обеспечивает надежное и эффективное переключение нагрузки между размещенными веб-приложениями, обеспечивая моментальную доступность важных данных.</span><span class="sxs-lookup"><span data-stu-id="fa7ad-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fa7ad-337">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="fa7ad-337">Next steps</span></span>

- <span data-ttu-id="fa7ad-338">Дополнительные сведения о шаблонах для облака Azure см. в статье [Конструктивные шаблоны облачных решений](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="fa7ad-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
