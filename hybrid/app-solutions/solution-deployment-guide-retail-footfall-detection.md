---
title: Развертывание решения определения посещаемости на основе искусственного интеллекта в Azure и Azure Stack Hub
description: Узнайте, как развернуть решение обнаружения посещения с использованием ИИ для анализа трафика посетителей в розничных магазинах с помощью Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895376"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="25c6f-103">Развертывание решения определения посещаемости на основе искусственного интеллекта с использованием Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="25c6f-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="25c6f-104">В статье описывается, как развернуть решение с использованием ИИ, которое создает полезные сведения на основе реальных действий, используя Azure, Azure Stack Hub и комплект SDK для искусственного интеллекта Пользовательского визуального распознавания.</span><span class="sxs-lookup"><span data-stu-id="25c6f-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="25c6f-105">В этом решении вы узнаете, как выполнять следующие задачи:</span><span class="sxs-lookup"><span data-stu-id="25c6f-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="25c6f-106">Развертывание пакетов облачных приложений (CNAB) на пограничных устройствах.</span><span class="sxs-lookup"><span data-stu-id="25c6f-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="25c6f-107">Развертывание приложения, охватывающего границы облака.</span><span class="sxs-lookup"><span data-stu-id="25c6f-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="25c6f-108">Использование комплекта SDK для искусственного интеллекта Пользовательского визуального распознавания для вывода на пограничных устройствах.</span><span class="sxs-lookup"><span data-stu-id="25c6f-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="25c6f-109">![Схема основных аспектов проектирования гибридных приложений](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="25c6f-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="25c6f-110">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="25c6f-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="25c6f-111">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Это решение позволяет использовать единственное гибридное облако, с помощью которого можно создавать и развертывать гибридные приложения в любой точке мира.</span><span class="sxs-lookup"><span data-stu-id="25c6f-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="25c6f-112">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="25c6f-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="25c6f-113">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="25c6f-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="25c6f-114">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="25c6f-114">Prerequisites</span></span>

<span data-ttu-id="25c6f-115">Прежде чем приступить к работе с этим руководством по развертыванию, не забудьте выполнить следующие действия:</span><span class="sxs-lookup"><span data-stu-id="25c6f-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="25c6f-116">Ознакомьтесь с разделом [Шаблон определения посещаемости](pattern-retail-footfall-detection.md).</span><span class="sxs-lookup"><span data-stu-id="25c6f-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="25c6f-117">Получите пользовательский доступ к Пакету средств разработки Azure Stack (ASDK) или экземпляру интегрированной системы Azure Stack Hub, используя:</span><span class="sxs-lookup"><span data-stu-id="25c6f-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="25c6f-118">[Службу приложений Azure в установленном поставщике ресурсов Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview).</span><span class="sxs-lookup"><span data-stu-id="25c6f-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview) installed.</span></span> <span data-ttu-id="25c6f-119">Для доступа к экземпляру Azure Stack Hub требуется доступ оператора. Можно также обратиться к администратору для установки.</span><span class="sxs-lookup"><span data-stu-id="25c6f-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="25c6f-120">Подписка на предложение, которое предоставляет квоту на Службу приложений и службу хранилища.</span><span class="sxs-lookup"><span data-stu-id="25c6f-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="25c6f-121">Для создания предложения необходим доступ оператора.</span><span class="sxs-lookup"><span data-stu-id="25c6f-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="25c6f-122">Получите доступ к подписке Azure.</span><span class="sxs-lookup"><span data-stu-id="25c6f-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="25c6f-123">Если у вас еще нет подписки Azure, [подпишитесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем начинать работу.</span><span class="sxs-lookup"><span data-stu-id="25c6f-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="25c6f-124">Создайте два субъекта-службы в каталоге:</span><span class="sxs-lookup"><span data-stu-id="25c6f-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="25c6f-125">один, настроенный для использования с ресурсами Azure, с доступом в области подписки Azure;</span><span class="sxs-lookup"><span data-stu-id="25c6f-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="25c6f-126">и еще один, настроенный для использования с ресурсами Azure Stack Hub, с доступом в области подписки Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25c6f-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="25c6f-127">Дополнительные сведения о создании субъектов-служб и авторизации доступа см. в статье [Использование удостоверения приложения для доступа к ресурсам](/azure-stack/operator/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="25c6f-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals).</span></span> <span data-ttu-id="25c6f-128">Если вы предпочитаете использовать Azure CLI, ознакомьтесь со статьей [Создание субъекта-службы Azure с помощью Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span><span class="sxs-lookup"><span data-stu-id="25c6f-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="25c6f-129">Разверните Azure Cognitive Services в Azure или Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25c6f-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="25c6f-130">Сначала [узнайте больше о Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="25c6f-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="25c6f-131">Затем перейдите к статье [Развертывание Azure Cognitive Services в Azure Stack](/azure-stack/user/azure-stack-solution-template-cognitive-services), чтобы развернуть Cognitive Services в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25c6f-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="25c6f-132">Сначала необходимо зарегистрироваться для доступа к предварительной версии.</span><span class="sxs-lookup"><span data-stu-id="25c6f-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="25c6f-133">Клонируйте или скачайте ненастроенный комплект SDK для искусственного интеллекта Пользовательского визуального распознавания Azure.</span><span class="sxs-lookup"><span data-stu-id="25c6f-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="25c6f-134">Дополнительные сведения см. на странице со сведениями о [концепции комплекта SDK для искусственного интеллекта](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="25c6f-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="25c6f-135">Зарегистрируйтесь для использования учетной записи Power BI.</span><span class="sxs-lookup"><span data-stu-id="25c6f-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="25c6f-136">Ключ подписки API Распознавания лиц Azure Cognitive Services и URL-адрес конечной точки.</span><span class="sxs-lookup"><span data-stu-id="25c6f-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="25c6f-137">На странице [Пробная версия Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) можно получить бесплатную пробную версию.</span><span class="sxs-lookup"><span data-stu-id="25c6f-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="25c6f-138">Можно также следовать инструкциям в статье [Create a Cognitive Services resource using the Azure portal](/azure/cognitive-services/cognitive-services-apis-create-account) (Создание ресурса Cognitive Services на портале Azure).</span><span class="sxs-lookup"><span data-stu-id="25c6f-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="25c6f-139">Установите следующие ресурсы для разработки:</span><span class="sxs-lookup"><span data-stu-id="25c6f-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="25c6f-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="25c6f-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [<span data-ttu-id="25c6f-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="25c6f-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="25c6f-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="25c6f-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="25c6f-143">Porter используется для развертывания облачных приложений с помощью предоставленных манифестов пакета CNAB.</span><span class="sxs-lookup"><span data-stu-id="25c6f-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="25c6f-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="25c6f-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - <span data-ttu-id="25c6f-145">[Azure IoT Tools для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).</span><span class="sxs-lookup"><span data-stu-id="25c6f-145">[Azure IoT Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)</span></span>
  - [<span data-ttu-id="25c6f-146">Расширение Python для Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="25c6f-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="25c6f-147">Python</span><span class="sxs-lookup"><span data-stu-id="25c6f-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="25c6f-148">Развертывание гибридного облачного приложения</span><span class="sxs-lookup"><span data-stu-id="25c6f-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="25c6f-149">Сначала с помощью Porter CLI создайте набор учетных данных, а затем разверните облачное приложение.</span><span class="sxs-lookup"><span data-stu-id="25c6f-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="25c6f-150">Клонируйте или скачайте пример кода решения по ссылке https://github.com/azure-samples/azure-intelligent-edge-patterns.</span><span class="sxs-lookup"><span data-stu-id="25c6f-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="25c6f-151">Porter создаст набор учетных данных, которые будут автоматизировать развертывание приложения.</span><span class="sxs-lookup"><span data-stu-id="25c6f-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="25c6f-152">Перед выполнением команды создания учетных данных убедитесь, что у вас есть следующее:</span><span class="sxs-lookup"><span data-stu-id="25c6f-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="25c6f-153">Субъект-служба для доступа к ресурсам Azure, включая идентификатор субъекта-службы, ключ и DNS клиента.</span><span class="sxs-lookup"><span data-stu-id="25c6f-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="25c6f-154">Идентификатор вашей подписки Azure.</span><span class="sxs-lookup"><span data-stu-id="25c6f-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="25c6f-155">Субъект-служба для доступа к ресурсам Azure Stack Hub, включая идентификатор субъекта-службы, ключ и DNS клиента.</span><span class="sxs-lookup"><span data-stu-id="25c6f-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="25c6f-156">Идентификатор вашей подписки Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25c6f-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="25c6f-157">Ключ API Распознавания лиц Azure Cognitive Services и URL-адрес конечной точки ресурса.</span><span class="sxs-lookup"><span data-stu-id="25c6f-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="25c6f-158">Запустите процесс создания учетных данных Porter и следуйте инструкциям на экране.</span><span class="sxs-lookup"><span data-stu-id="25c6f-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="25c6f-159">Для выполнения Porter также требуется набор параметров.</span><span class="sxs-lookup"><span data-stu-id="25c6f-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="25c6f-160">Создайте текстовый файл параметров и введите приведенные ниже пары "имя — значение".</span><span class="sxs-lookup"><span data-stu-id="25c6f-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="25c6f-161">Обратитесь к администратору Azure Stack Hub, если вам нужна помощь по какому-либо из требуемых значений.</span><span class="sxs-lookup"><span data-stu-id="25c6f-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="25c6f-162">Значение `resource suffix` используется для того, чтобы у ресурсов развертывания были уникальные имена в Azure.</span><span class="sxs-lookup"><span data-stu-id="25c6f-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="25c6f-163">Это должна быть уникальная строка из букв и цифр, не длиннее 8 символов.</span><span class="sxs-lookup"><span data-stu-id="25c6f-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="25c6f-164">Сохраните текстовый файл и запишите его путь.</span><span class="sxs-lookup"><span data-stu-id="25c6f-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="25c6f-165">Теперь вы готовы к развертыванию гибридного облачного приложения с помощью Porter.</span><span class="sxs-lookup"><span data-stu-id="25c6f-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="25c6f-166">Выполните команду установки и понаблюдайте, как ресурсы развертываются в Azure и Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="25c6f-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="25c6f-167">После завершения развертывания запишите следующие значения:</span><span class="sxs-lookup"><span data-stu-id="25c6f-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="25c6f-168">строка подключения камеры;</span><span class="sxs-lookup"><span data-stu-id="25c6f-168">The camera's connection string.</span></span>
    - <span data-ttu-id="25c6f-169">строка подключения учетной записи хранилища изображений;</span><span class="sxs-lookup"><span data-stu-id="25c6f-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="25c6f-170">имена групп ресурсов.</span><span class="sxs-lookup"><span data-stu-id="25c6f-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="25c6f-171">Подготовка комплекта SDK для искусственного интеллекта Пользовательского визуального распознавания</span><span class="sxs-lookup"><span data-stu-id="25c6f-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="25c6f-172">Затем настройте комплект SDK для искусственного интеллекта Пользовательского визуального распознавания, как показано в этом [кратком руководстве](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="25c6f-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="25c6f-173">Кроме того, необходимо настроить и протестировать камеру, используя строку подключения, полученную на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="25c6f-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="25c6f-174">Развертывание приложения для камеры</span><span class="sxs-lookup"><span data-stu-id="25c6f-174">Deploy the camera app</span></span>

<span data-ttu-id="25c6f-175">С помощью Porter CLI создайте набор учетных данных, а затем разверните приложение камеры.</span><span class="sxs-lookup"><span data-stu-id="25c6f-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="25c6f-176">Porter создаст набор учетных данных, которые будут автоматизировать развертывание приложения.</span><span class="sxs-lookup"><span data-stu-id="25c6f-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="25c6f-177">Перед выполнением команды создания учетных данных убедитесь, что у вас есть следующее:</span><span class="sxs-lookup"><span data-stu-id="25c6f-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="25c6f-178">Субъект-служба для доступа к ресурсам Azure, включая идентификатор субъекта-службы, ключ и DNS клиента.</span><span class="sxs-lookup"><span data-stu-id="25c6f-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="25c6f-179">Идентификатор вашей подписки Azure.</span><span class="sxs-lookup"><span data-stu-id="25c6f-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="25c6f-180">Строка подключения учетной записи хранилища изображений, полученная при развертывании облачного приложения.</span><span class="sxs-lookup"><span data-stu-id="25c6f-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="25c6f-181">Запустите процесс создания учетных данных Porter и следуйте инструкциям на экране.</span><span class="sxs-lookup"><span data-stu-id="25c6f-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="25c6f-182">Для выполнения Porter также требуется набор параметров.</span><span class="sxs-lookup"><span data-stu-id="25c6f-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="25c6f-183">Создайте текстовый файл параметров и введите следующий текст.</span><span class="sxs-lookup"><span data-stu-id="25c6f-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="25c6f-184">Обратитесь к администратору Azure Stack Hub, если вы не знаете какие-либо из требуемых значений.</span><span class="sxs-lookup"><span data-stu-id="25c6f-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="25c6f-185">Значение `deployment suffix` используется для того, чтобы у ресурсов развертывания были уникальные имена в Azure.</span><span class="sxs-lookup"><span data-stu-id="25c6f-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="25c6f-186">Это должна быть уникальная строка из букв и цифр, не длиннее 8 символов.</span><span class="sxs-lookup"><span data-stu-id="25c6f-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="25c6f-187">Сохраните текстовый файл и запишите его путь.</span><span class="sxs-lookup"><span data-stu-id="25c6f-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="25c6f-188">Теперь вы готовы к развертыванию приложения камеры с помощью Porter.</span><span class="sxs-lookup"><span data-stu-id="25c6f-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="25c6f-189">Выполните команду установки и понаблюдайте, как будет создано развертывание IoT Edge.</span><span class="sxs-lookup"><span data-stu-id="25c6f-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="25c6f-190">Убедитесь, что развертывание камеры завершено, просмотрев кадры с камеры по адресу `https://<camera-ip>:3000/`, где `<camara-ip>` является IP-адресом камеры.</span><span class="sxs-lookup"><span data-stu-id="25c6f-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="25c6f-191">Этот шаг может занять до 10 минут.</span><span class="sxs-lookup"><span data-stu-id="25c6f-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="25c6f-192">Настройка Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="25c6f-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="25c6f-193">Теперь, когда данные передаются в Azure Stream Analytics с камеры, необходимо вручную авторизовать их для взаимодействия с Power BI.</span><span class="sxs-lookup"><span data-stu-id="25c6f-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="25c6f-194">На портале Azure откройте **Все ресурсы**, а также задание *process-footfall\[ваш_суффикс\]* .</span><span class="sxs-lookup"><span data-stu-id="25c6f-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="25c6f-195">В разделе **Топология задания** области задания Stream Analytics выберите вариант **Выходные данные**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="25c6f-196">Выберите приемник выходных данных **traffic-output**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="25c6f-197">Щелкните **Обновить авторизацию** и войдите в учетную запись Power BI.</span><span class="sxs-lookup"><span data-stu-id="25c6f-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Обновление запроса авторизации в Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="25c6f-199">Сохраните параметры вывода.</span><span class="sxs-lookup"><span data-stu-id="25c6f-199">Save the output settings.</span></span>

6. <span data-ttu-id="25c6f-200">Перейдите на панель **Обзор** и выберите **Запуск**, чтобы начать отправку данных в Power BI.</span><span class="sxs-lookup"><span data-stu-id="25c6f-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="25c6f-201">Выберите время начала создания выходных данных задания **Сейчас**, а затем — **Запуск**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="25c6f-202">Вы можете просматривать состояние задания на панели уведомлений.</span><span class="sxs-lookup"><span data-stu-id="25c6f-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="25c6f-203">Создание панели мониторинга Power BI</span><span class="sxs-lookup"><span data-stu-id="25c6f-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="25c6f-204">После успешного выполнения задания перейдите на сайт [Power BI](https://powerbi.com/) и войдите с помощью рабочей или учебной учетной записи.</span><span class="sxs-lookup"><span data-stu-id="25c6f-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="25c6f-205">Если запрос задания Stream Analytics выдает результаты, то созданный набор данных *footfall-dataset* находится на вкладке **Наборы данных**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="25c6f-206">В рабочей области Power BI выберите **+ Создать**, чтобы создать панель мониторинга с именем *Анализ посещаемости*.</span><span class="sxs-lookup"><span data-stu-id="25c6f-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="25c6f-207">В верхней части окна щелкните **Добавить плитку**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="25c6f-208">Затем выберите **Пользовательские данные потоковой передачи** и нажмите кнопку **Далее**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="25c6f-209">Выберите **footfall-dataset** в разделе **Ваши наборы данных**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="25c6f-210">Выберите **Карта** из раскрывающегося списка **Тип визуализации** и добавьте **age** в раздел **Поля**.</span><span class="sxs-lookup"><span data-stu-id="25c6f-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="25c6f-211">Нажмите кнопку **Далее**, чтобы ввести имя для плитки, а затем выберите **Применить** для создания плитки.</span><span class="sxs-lookup"><span data-stu-id="25c6f-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="25c6f-212">При необходимости можно добавить дополнительные поля и карты.</span><span class="sxs-lookup"><span data-stu-id="25c6f-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="25c6f-213">Тестирование решения</span><span class="sxs-lookup"><span data-stu-id="25c6f-213">Test Your Solution</span></span>

<span data-ttu-id="25c6f-214">Обратите внимание, как изменяются данные в картах, созданных в Power BI, когда перед камерой проходят разные люди.</span><span class="sxs-lookup"><span data-stu-id="25c6f-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="25c6f-215">Отображение вывода после записи может занять до 20 секунд.</span><span class="sxs-lookup"><span data-stu-id="25c6f-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="25c6f-216">Удаление решения</span><span class="sxs-lookup"><span data-stu-id="25c6f-216">Remove Your Solution</span></span>

<span data-ttu-id="25c6f-217">Если вы хотите удалить решение, выполните следующие команды с помощью Porter, используя те же файлы параметров, которые были созданы для развертывания:</span><span class="sxs-lookup"><span data-stu-id="25c6f-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="25c6f-218">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="25c6f-218">Next steps</span></span>

- <span data-ttu-id="25c6f-219">Узнайте больше об [аспектах проектирования гибридных приложений](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="25c6f-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="25c6f-220">Изучите и предложите улучшения для [кода в этом примере на сайте GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="25c6f-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
