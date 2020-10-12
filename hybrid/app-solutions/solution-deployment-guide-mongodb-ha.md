---
title: Развертывание высокодоступного решения MongoDB в Azure и Azure Stack Hub
description: Узнайте, как развернуть высокодоступное решение MongoDB в Azure и Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852513"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="0840f-103">Развертывание высокодоступного решения MongoDB в Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="0840f-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="0840f-104">В этой статье описано, как автоматически развернуть базовый высокодоступный кластер MongoDB с сайтом аварийного восстановления в двух средах Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0840f-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="0840f-105">Подробные сведения о MongoDB и высоком уровне доступности см. в руководстве по [элементам набора реплик](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="0840f-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="0840f-106">В этом решении вы создадите среду, чтобы затем выполнить в ней следующие действия:</span><span class="sxs-lookup"><span data-stu-id="0840f-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="0840f-107">оркестрация развертывания в двух экземплярах Azure Stack Hub;</span><span class="sxs-lookup"><span data-stu-id="0840f-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="0840f-108">использование Docker для предотвращения возникновения проблем с зависимостями с помощью профилей API Azure;</span><span class="sxs-lookup"><span data-stu-id="0840f-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="0840f-109">развертывание основных высокодоступных кластеров MongoDB с сайта аварийного восстановления.</span><span class="sxs-lookup"><span data-stu-id="0840f-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="0840f-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="0840f-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="0840f-111">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="0840f-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="0840f-112">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Только это гибридное облако позволяет создавать и развертывать гибридные приложения где угодно.</span><span class="sxs-lookup"><span data-stu-id="0840f-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="0840f-113">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="0840f-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="0840f-114">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="0840f-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="0840f-115">Архитектура MongoDB для использования с Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="0840f-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Высокодоступная архитектура MongoDB в Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="0840f-117">Требования к MongoDB для использования с Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="0840f-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="0840f-118">Две подключенные интегрированные системы Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0840f-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="0840f-119">Это развертывание не работает с пакетом средств разработки Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="0840f-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="0840f-120">См. сведения об [Azure Stack Hub](https://azure.microsoft.com/products/azure-stack/hub/).</span><span class="sxs-lookup"><span data-stu-id="0840f-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="0840f-121">Подписка клиента в каждом экземпляре Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0840f-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="0840f-122">**Запомните или запишите идентификатор каждой подписки и конечной точки Azure Resource Manager для каждого экземпляра Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="0840f-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="0840f-123">Субъект-служба Azure Active Directory (Azure AD) с разрешениями для подписки клиента в каждом экземпляре Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0840f-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="0840f-124">Вам может потребоваться создать два субъекта-службы, если экземпляры Azure Stack Hub развертываются в разных клиентах Azure AD.</span><span class="sxs-lookup"><span data-stu-id="0840f-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="0840f-125">Чтобы узнать, как создать субъект-службу для Azure Stack Hub, см. раздел [Использование удостоверения приложения для доступа к ресурсам Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="0840f-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="0840f-126">**Запомните или запишите идентификатор приложения каждого субъекта-службы, секрет клиента и имя клиента (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="0840f-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="0840f-127">Для Ubuntu 16.04 выполняется синдикация со всеми экземплярами Marketplace в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0840f-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="0840f-128">Сведения о синдикации Marketplace см. в [руководстве по загрузке элементов Marketplace в Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="0840f-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="0840f-129">Приложение [Docker для Windows](https://docs.docker.com/docker-for-windows/), установленное на локальном компьютере.</span><span class="sxs-lookup"><span data-stu-id="0840f-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="0840f-130">Получение образа Docker</span><span class="sxs-lookup"><span data-stu-id="0840f-130">Get the Docker image</span></span>

<span data-ttu-id="0840f-131">Использование образов Docker для каждого развертывания позволяет устранить проблемы с зависимостями разных версий Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="0840f-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="0840f-132">Убедитесь, что Docker для Windows использует контейнеры Windows.</span><span class="sxs-lookup"><span data-stu-id="0840f-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="0840f-133">В командной строке выполните следующую команду с повышенными привилегиями, чтобы получить контейнер Docker и скрипты развертывания.</span><span class="sxs-lookup"><span data-stu-id="0840f-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="0840f-134">Развертывание кластеров</span><span class="sxs-lookup"><span data-stu-id="0840f-134">Deploy the clusters</span></span>

1. <span data-ttu-id="0840f-135">После извлечения образа контейнера запустите его.</span><span class="sxs-lookup"><span data-stu-id="0840f-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="0840f-136">После запуска контейнера откроется терминал PowerShell с повышенными привилегиями в контейнере.</span><span class="sxs-lookup"><span data-stu-id="0840f-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="0840f-137">Измените каталоги, чтобы получить скрипт развертывания.</span><span class="sxs-lookup"><span data-stu-id="0840f-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="0840f-138">Запустите развертывание.</span><span class="sxs-lookup"><span data-stu-id="0840f-138">Run the deployment.</span></span> <span data-ttu-id="0840f-139">Укажите учетные данные и имена ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0840f-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="0840f-140">HA относится к Azure Stack Hub, где будет развернут высокодоступный (HA) кластер.</span><span class="sxs-lookup"><span data-stu-id="0840f-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="0840f-141">DR относится к Azure Stack Hub, где будет развернут кластер аварийного восстановления (DR).</span><span class="sxs-lookup"><span data-stu-id="0840f-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. <span data-ttu-id="0840f-142">Введите `Y`, чтобы разрешить установку поставщика NuGet для запуска устанавливаемых модулей профиля API 2018-03-01-hybrid.</span><span class="sxs-lookup"><span data-stu-id="0840f-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="0840f-143">Сначала развертываются высокодоступные ресурсы.</span><span class="sxs-lookup"><span data-stu-id="0840f-143">The HA resources will deploy first.</span></span> <span data-ttu-id="0840f-144">Перейдите к развертыванию и дождитесь его завершения.</span><span class="sxs-lookup"><span data-stu-id="0840f-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="0840f-145">После этого развернутые ресурсы можно проверить на портале Azure Stack Hub (HA).</span><span class="sxs-lookup"><span data-stu-id="0840f-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="0840f-146">Вы можете продолжить развертывать ресурсы аварийного восстановления и решить, хотите ли вы включить инсталляционный сервер в Azure Stack Hub DR для взаимодействия с кластером.</span><span class="sxs-lookup"><span data-stu-id="0840f-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="0840f-147">Дождитесь завершения развертывания ресурса аварийного восстановления.</span><span class="sxs-lookup"><span data-stu-id="0840f-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="0840f-148">После завершения развертывания ресурсов аварийного восстановления выйдите из контейнера.</span><span class="sxs-lookup"><span data-stu-id="0840f-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="0840f-149">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="0840f-149">Next steps</span></span>

- <span data-ttu-id="0840f-150">Если вы включили виртуальную машину инсталляционного сервера в Azure Stack Hub DR, вы можете подключиться по протоколу SSH для взаимодействия с кластером MongoDB, установив Mongo CLI.</span><span class="sxs-lookup"><span data-stu-id="0840f-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="0840f-151">Подробные сведения о взаимодействии с MongoDB с помощью Mongo см. в статье об [оболочке Mongo](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="0840f-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="0840f-152">Подробные сведения о приложениях гибридного облака см. в статье о [гибридных облачных решениях](/azure-stack/user/).</span><span class="sxs-lookup"><span data-stu-id="0840f-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="0840f-153">Измените код на основе примера на сайте [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="0840f-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>