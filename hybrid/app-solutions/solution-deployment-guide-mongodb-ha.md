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
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Развертывание высокодоступного решения MongoDB в Azure и Azure Stack Hub

В этой статье описано, как автоматически развернуть базовый высокодоступный кластер MongoDB с сайтом аварийного восстановления в двух средах Azure Stack Hub. Подробные сведения о MongoDB и высоком уровне доступности см. в руководстве по [элементам набора реплик](https://docs.mongodb.com/manual/core/replica-set-members/).

В этом решении вы создадите среду, чтобы затем выполнить в ней следующие действия:

> [!div class="checklist"]
> - оркестрация развертывания в двух экземплярах Azure Stack Hub;
> - использование Docker для предотвращения возникновения проблем с зависимостями с помощью профилей API Azure;
> - развертывание основных высокодоступных кластеров MongoDB с сайта аварийного восстановления.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub — это расширение Azure. Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Только это гибридное облако позволяет создавать и развертывать гибридные приложения где угодно.  
> 
> В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений. Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Архитектура MongoDB для использования с Azure Stack Hub

![Высокодоступная архитектура MongoDB в Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Требования к MongoDB для использования с Azure Stack Hub

- Две подключенные интегрированные системы Azure Stack Hub. Это развертывание не работает с пакетом средств разработки Azure Stack (ASDK). См. сведения об [Azure Stack Hub](https://azure.microsoft.com/products/azure-stack/hub/).
  - Подписка клиента в каждом экземпляре Azure Stack Hub. 
  - **Запомните или запишите идентификатор каждой подписки и конечной точки Azure Resource Manager для каждого экземпляра Azure Stack Hub.**
- Субъект-служба Azure Active Directory (Azure AD) с разрешениями для подписки клиента в каждом экземпляре Azure Stack Hub. Вам может потребоваться создать два субъекта-службы, если экземпляры Azure Stack Hub развертываются в разных клиентах Azure AD. Чтобы узнать, как создать субъект-службу для Azure Stack Hub, см. раздел [Использование удостоверения приложения для доступа к ресурсам Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).
  - **Запомните или запишите идентификатор приложения каждого субъекта-службы, секрет клиента и имя клиента (xxxxx.onmicrosoft.com).**
- Для Ubuntu 16.04 выполняется синдикация со всеми экземплярами Marketplace в Azure Stack Hub. Сведения о синдикации Marketplace см. в [руководстве по загрузке элементов Marketplace в Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- Приложение [Docker для Windows](https://docs.docker.com/docker-for-windows/), установленное на локальном компьютере.

## <a name="get-the-docker-image"></a>Получение образа Docker

Использование образов Docker для каждого развертывания позволяет устранить проблемы с зависимостями разных версий Azure PowerShell.

1. Убедитесь, что Docker для Windows использует контейнеры Windows.
2. В командной строке выполните следующую команду с повышенными привилегиями, чтобы получить контейнер Docker и скрипты развертывания.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Развертывание кластеров

1. После извлечения образа контейнера запустите его.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. После запуска контейнера откроется терминал PowerShell с повышенными привилегиями в контейнере. Измените каталоги, чтобы получить скрипт развертывания.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Запустите развертывание. Укажите учетные данные и имена ресурсов. HA относится к Azure Stack Hub, где будет развернут высокодоступный (HA) кластер. DR относится к Azure Stack Hub, где будет развернут кластер аварийного восстановления (DR).

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

4. Введите `Y`, чтобы разрешить установку поставщика NuGet для запуска устанавливаемых модулей профиля API 2018-03-01-hybrid.

5. Сначала развертываются высокодоступные ресурсы. Перейдите к развертыванию и дождитесь его завершения. После этого развернутые ресурсы можно проверить на портале Azure Stack Hub (HA).

6. Вы можете продолжить развертывать ресурсы аварийного восстановления и решить, хотите ли вы включить инсталляционный сервер в Azure Stack Hub DR для взаимодействия с кластером.

7. Дождитесь завершения развертывания ресурса аварийного восстановления.

8. После завершения развертывания ресурсов аварийного восстановления выйдите из контейнера.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Дальнейшие действия

- Если вы включили виртуальную машину инсталляционного сервера в Azure Stack Hub DR, вы можете подключиться по протоколу SSH для взаимодействия с кластером MongoDB, установив Mongo CLI. Подробные сведения о взаимодействии с MongoDB с помощью Mongo см. в статье об [оболочке Mongo](https://docs.mongodb.com/manual/mongo/).
- Подробные сведения о приложениях гибридного облака см. в статье о [гибридных облачных решениях](/azure-stack/user/).
- Измените код на основе примера на сайте [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).