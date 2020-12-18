---
title: Развертывание группы доступности SQL Server 2016 в Azure и Azure Stack Hub
description: Из этой статьи вы узнаете, как развернуть группу доступности SQL Server 2016 в Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0f857515a44ece7f967ade3dee8f493481709851
ms.sourcegitcommit: c890f2c5e5e5f9f93c921f02dd1a6ca5026d5289
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/10/2020
ms.locfileid: "97091808"
---
# <a name="deploy-a-sql-server-2016-availability-group-across-two-azure-stack-hub-environments"></a>Развертывание группы доступности SQL Server 2016 в двух средах Azure Stack Hub

В этой статье описано, как автоматически развернуть базовый высокодоступный (HA) кластер SQL Server 2016 Enterprise с сайтом асинхронного аварийного восстановления (DR) в двух средах Azure Stack Hub. Подробные сведения об SQL Server 2016 и высоком уровне доступности см. в статье [Группы доступности Always On: решение для обеспечения высокой доступности и аварийного восстановления](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

В этом решении показано, как создать среду, после чего вы выполните в ней следующие действия:

> [!div class="checklist"]
> - оркестрация развертывания в двух экземплярах Azure Stack Hub;
> - использование Docker для предотвращения возникновения проблем с зависимостями с помощью профилей API Azure;
> - развертывание основных высокодоступных кластеров SQL Server 2016 Enterprise с сайта аварийного восстановления.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub — это расширение Azure. Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Только это гибридное облако позволяет создавать и развертывать гибридные приложения где угодно.  
> 
> В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений. Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.

## <a name="architecture-for-sql-server-2016"></a>Архитектура для SQL Server 2016

![Высокодоступный кластер SQL Server 2016 в Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Предварительные требования для SQL Server 2016

- Две подключенные интегрированные системы Azure Stack Hub. Это развертывание не работает с пакетом средств разработки Azure Stack (ASDK). Дополнительные сведения об Azure Stack Hub см. [здесь](https://azure.microsoft.com/overview/azure-stack/).
- Подписка клиента в каждом экземпляре Azure Stack Hub.
  - **Запомните или запишите идентификатор каждой подписки и конечной точки Azure Resource Manager для каждого экземпляра Azure Stack Hub.**
- Субъект-служба Azure Active Directory (Azure AD) с разрешениями для подписки клиента в каждом экземпляре Azure Stack Hub. Вам может потребоваться создать два субъекта-службы, если экземпляры Azure Stack Hub развертываются в разных клиентах Azure AD. См. сведения о создании субъекта-службы для Azure Stack Hub в руководстве по [предоставлению приложениям доступа к Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).
  - **Запомните или запишите идентификатор приложения каждого субъекта-службы, секрет клиента и имя клиента (xxxxx.onmicrosoft.com).**
- Для SQL Server 2016 Enterprise выполняется синдикация со всеми экземплярами Marketplace в Azure Stack Hub. Сведения о синдикации Marketplace см. в [руководстве по загрузке элементов Marketplace в Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Убедитесь, что в организации есть соответствующие лицензии SQL**.
- Приложение [Docker для Windows](https://docs.docker.com/docker-for-windows/), установленное на локальном компьютере.

## <a name="get-the-docker-image"></a>Получение образа Docker

Использование образов Docker для каждого развертывания позволяет устранить проблемы с зависимостями разных версий Azure PowerShell.

1. Убедитесь, что Docker для Windows использует контейнеры Windows.
2. Выполните следующий скрипт в командной строке с повышенными привилегиями для получения контейнера Docker с помощью скриптов развертывания.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Развертывание группы доступности

1. После извлечения образа контейнера запустите его.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. После запуска контейнера откроется терминал PowerShell с повышенными привилегиями в контейнере. Измените каталоги, чтобы получить скрипт развертывания.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Запустите развертывание. Укажите учетные данные и имена ресурсов. HA относится к Azure Stack Hub, где будет развернут высокодоступный (HA) кластер. DR относится к Azure Stack Hub, где будет развернут кластер аварийного восстановления (DR).

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

4. Введите `Y`, чтобы разрешить установку поставщика NuGet для запуска устанавливаемых модулей профиля API 2018-03-01-hybrid.

5. Дождитесь завершения развертывания ресурса.

6. После завершения развертывания ресурсов аварийного восстановления выйдите из контейнера.

      ```powershell
      exit
      ```

7. Проверьте развертывание, просмотрев ресурсы на портале для каждого экземпляра Azure Stack Hub. Подключитесь к одному из экземпляров SQL в высокодоступной среде и проверьте группу доступности с использованием SQL Server Management Studio (SSMS).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Дальнейшие действия

- SQL Server Management Studio позволяет выполнить отработку отказа кластера вручную. См. статью [Выполнение принудительного перехода на другой ресурс вручную для группы доступности Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017).
- Ознакомьтесь с дополнительными сведениями о приложениях гибридного облака. См. документацию по [решениям гибридного облака](/azure-stack/user/).
- Используйте собственные данные или измените код на основе примера на сайте [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
