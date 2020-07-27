---
title: Развертывание группы доступности SQL Server 2016 в Azure и Azure Stack Hub
description: Из этой статьи вы узнаете, как развернуть группу доступности SQL Server 2016 в Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 85b859457b9b54a973c5fc23329b927212b60a07
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477088"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="28175-103">Развертывание группы доступности SQL Server 2016 в Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="28175-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="28175-104">В этой статье описано, как автоматически развернуть базовый высокодоступный (HA) кластер SQL Server 2016 Enterprise с сайтом асинхронного аварийного восстановления (DR) в двух средах Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28175-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="28175-105">Подробные сведения об SQL Server 2016 и высоком уровне доступности см. в статье [Группы доступности Always On: решение для обеспечения высокой доступности и аварийного восстановления](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="28175-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="28175-106">В этом решении показано, как создать среду, после чего вы выполните в ней следующие действия:</span><span class="sxs-lookup"><span data-stu-id="28175-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="28175-107">оркестрация развертывания в двух экземплярах Azure Stack Hub;</span><span class="sxs-lookup"><span data-stu-id="28175-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="28175-108">использование Docker для предотвращения возникновения проблем с зависимостями с помощью профилей API Azure;</span><span class="sxs-lookup"><span data-stu-id="28175-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="28175-109">развертывание основных высокодоступных кластеров SQL Server 2016 Enterprise с сайта аварийного восстановления.</span><span class="sxs-lookup"><span data-stu-id="28175-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="28175-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="28175-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="28175-111">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="28175-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="28175-112">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Только это гибридное облако позволяет создавать и развертывать гибридные приложения где угодно.</span><span class="sxs-lookup"><span data-stu-id="28175-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="28175-113">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="28175-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="28175-114">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="28175-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="28175-115">Архитектура для SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="28175-115">Architecture for SQL Server 2016</span></span>

![Высокодоступный кластер SQL Server 2016 в Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="28175-117">Предварительные требования для SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="28175-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="28175-118">Две подключенные интегрированные системы Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28175-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="28175-119">Это развертывание не работает с пакетом средств разработки Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="28175-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="28175-120">Дополнительные сведения об Azure Stack Hub см. [здесь](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="28175-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="28175-121">Подписка клиента в каждом экземпляре Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28175-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="28175-122">**Запомните или запишите идентификатор каждой подписки и конечной точки Azure Resource Manager для каждого экземпляра Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="28175-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="28175-123">Субъект-служба Azure Active Directory (Azure AD) с разрешениями для подписки клиента в каждом экземпляре Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28175-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="28175-124">Вам может потребоваться создать два субъекта-службы, если экземпляры Azure Stack Hub развертываются в разных клиентах Azure AD.</span><span class="sxs-lookup"><span data-stu-id="28175-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="28175-125">См. сведения о создании субъекта-службы для Azure Stack Hub в руководстве по [предоставлению приложениям доступа к Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="28175-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="28175-126">**Запомните или запишите идентификатор приложения каждого субъекта-службы, секрет клиента и имя клиента (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="28175-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="28175-127">Для SQL Server 2016 Enterprise выполняется синдикация со всеми экземплярами Marketplace в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28175-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="28175-128">Сведения о синдикации Marketplace см. в [руководстве по загрузке элементов Marketplace в Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="28175-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="28175-129">**Убедитесь, что в организации есть соответствующие лицензии SQL**.</span><span class="sxs-lookup"><span data-stu-id="28175-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="28175-130">Приложение [Docker для Windows](https://docs.docker.com/docker-for-windows/), установленное на локальном компьютере.</span><span class="sxs-lookup"><span data-stu-id="28175-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="28175-131">Получение образа Docker</span><span class="sxs-lookup"><span data-stu-id="28175-131">Get the Docker image</span></span>

<span data-ttu-id="28175-132">Использование образов Docker для каждого развертывания позволяет устранить проблемы с зависимостями разных версий Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="28175-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="28175-133">Убедитесь, что Docker для Windows использует контейнеры Windows.</span><span class="sxs-lookup"><span data-stu-id="28175-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="28175-134">Выполните следующий скрипт в командной строке с повышенными привилегиями для получения контейнера Docker с помощью скриптов развертывания.</span><span class="sxs-lookup"><span data-stu-id="28175-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="28175-135">Развертывание группы доступности</span><span class="sxs-lookup"><span data-stu-id="28175-135">Deploy the availability group</span></span>

1. <span data-ttu-id="28175-136">После извлечения образа контейнера запустите его.</span><span class="sxs-lookup"><span data-stu-id="28175-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="28175-137">После запуска контейнера откроется терминал PowerShell с повышенными привилегиями в контейнере.</span><span class="sxs-lookup"><span data-stu-id="28175-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="28175-138">Измените каталоги, чтобы получить скрипт развертывания.</span><span class="sxs-lookup"><span data-stu-id="28175-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="28175-139">Запустите развертывание.</span><span class="sxs-lookup"><span data-stu-id="28175-139">Run the deployment.</span></span> <span data-ttu-id="28175-140">Укажите учетные данные и имена ресурсов.</span><span class="sxs-lookup"><span data-stu-id="28175-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="28175-141">HA относится к Azure Stack Hub, где будет развернут высокодоступный (HA) кластер.</span><span class="sxs-lookup"><span data-stu-id="28175-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="28175-142">DR относится к Azure Stack Hub, где будет развернут кластер аварийного восстановления (DR).</span><span class="sxs-lookup"><span data-stu-id="28175-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="28175-143">Введите `Y`, чтобы разрешить установку поставщика NuGet для запуска устанавливаемых модулей профиля API 2018-03-01-hybrid.</span><span class="sxs-lookup"><span data-stu-id="28175-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="28175-144">Дождитесь завершения развертывания ресурса.</span><span class="sxs-lookup"><span data-stu-id="28175-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="28175-145">После завершения развертывания ресурсов аварийного восстановления выйдите из контейнера.</span><span class="sxs-lookup"><span data-stu-id="28175-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="28175-146">Проверьте развертывание, просмотрев ресурсы на портале для каждого экземпляра Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28175-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="28175-147">Подключитесь к одному из экземпляров SQL в высокодоступной среде и проверьте группу доступности с использованием SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="28175-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="28175-149">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="28175-149">Next steps</span></span>

- <span data-ttu-id="28175-150">SQL Server Management Studio позволяет выполнить отработку отказа кластера вручную.</span><span class="sxs-lookup"><span data-stu-id="28175-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="28175-151">См. статью [Выполнение принудительного перехода на другой ресурс вручную для группы доступности Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017).</span><span class="sxs-lookup"><span data-stu-id="28175-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="28175-152">Ознакомьтесь с дополнительными сведениями о приложениях гибридного облака.</span><span class="sxs-lookup"><span data-stu-id="28175-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="28175-153">См. документацию по [решениям гибридного облака](https://aka.ms/azsdevtutorials).</span><span class="sxs-lookup"><span data-stu-id="28175-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="28175-154">Используйте собственные данные или измените код на основе примера на сайте [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="28175-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
