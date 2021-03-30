---
title: Развертывание гибридного приложения с локальными данными, масштабируемого в разных облаках
description: Узнайте, как развернуть приложение, использующее локальные данные и выполняющее масштабирование в нескольких облаках с помощью Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0989859fd68847932d3e69defee59740a2bffd44
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895403"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="f8f53-103">Развертывание гибридного приложения с локальными данными, масштабируемого в разных облаках</span><span class="sxs-lookup"><span data-stu-id="f8f53-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="f8f53-104">В этом руководстве по решению показано, как развернуть гибридное приложение, которое охватывает Azure и Azure Stack Hub и использует один источник локальных данных.</span><span class="sxs-lookup"><span data-stu-id="f8f53-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="f8f53-105">Используя гибридное облачное решение, можно объединить преимущества соответствия частного облака с масштабируемостью общедоступного.</span><span class="sxs-lookup"><span data-stu-id="f8f53-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="f8f53-106">Ваши разработчики могут воспользоваться преимуществами экосистемы разработчиков Майкрософт и применить свои навыки работы в облаке и локальных средах.</span><span class="sxs-lookup"><span data-stu-id="f8f53-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="f8f53-107">Обзор и предположения</span><span class="sxs-lookup"><span data-stu-id="f8f53-107">Overview and assumptions</span></span>

<span data-ttu-id="f8f53-108">В этом руководстве описано, как настроить рабочий процесс, который позволяет разработчикам развернуть веб-приложение в общедоступном и частном облаках.</span><span class="sxs-lookup"><span data-stu-id="f8f53-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="f8f53-109">Это приложение сможет получить доступ к маршрутизируемой сети, недоступной из Интернета и размещенной в частном облаке.</span><span class="sxs-lookup"><span data-stu-id="f8f53-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="f8f53-110">Такие веб-приложения отслеживаются, и при резком увеличении трафика программа изменяет записи DNS, чтобы перенаправить трафик в общедоступное облако.</span><span class="sxs-lookup"><span data-stu-id="f8f53-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="f8f53-111">Когда трафик опускается до уровня, предшествующего пику, он направляется обратно в частное облако.</span><span class="sxs-lookup"><span data-stu-id="f8f53-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="f8f53-112">В рамках этого руководства рассматриваются следующие задачи:</span><span class="sxs-lookup"><span data-stu-id="f8f53-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f8f53-113">Развертывание сервера базы данных SQL Server, подключенного с помощью гибридного подключения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="f8f53-114">Подключение веб-приложения в глобальной среде Azure к гибридной сети.</span><span class="sxs-lookup"><span data-stu-id="f8f53-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="f8f53-115">Настройка DNS для масштабирования в нескольких облаках.</span><span class="sxs-lookup"><span data-stu-id="f8f53-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="f8f53-116">Настройка SSL-сертификатов для масштабирования в нескольких облаках.</span><span class="sxs-lookup"><span data-stu-id="f8f53-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="f8f53-117">Настройка и развертывание веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="f8f53-118">Создание и настройка профиля диспетчера трафика для масштабирования в нескольких облаках.</span><span class="sxs-lookup"><span data-stu-id="f8f53-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="f8f53-119">Настройка мониторинга и оповещений Application Insights для большего объема трафика.</span><span class="sxs-lookup"><span data-stu-id="f8f53-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="f8f53-120">Настройка автоматического переключения трафика между глобальной средой Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="f8f53-121">![Схема основных аспектов проектирования гибридных приложений](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="f8f53-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="f8f53-122">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="f8f53-123">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Это решение позволяет использовать единственное гибридное облако, с помощью которого можно создавать и развертывать гибридные приложения в любой точке мира.</span><span class="sxs-lookup"><span data-stu-id="f8f53-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="f8f53-124">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="f8f53-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="f8f53-125">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="f8f53-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="f8f53-126">Предположения</span><span class="sxs-lookup"><span data-stu-id="f8f53-126">Assumptions</span></span>

<span data-ttu-id="f8f53-127">В этом учебнике также предполагается, что вы знакомы с глобальной средой Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="f8f53-128">Если вы хотите узнать больше, прежде чем начать работу с руководством, прочтите следующие статьи:</span><span class="sxs-lookup"><span data-stu-id="f8f53-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- <span data-ttu-id="f8f53-129">[Введение в Azure](https://azure.microsoft.com/overview/what-is-azure/).</span><span class="sxs-lookup"><span data-stu-id="f8f53-129">[Introduction to Azure](https://azure.microsoft.com/overview/what-is-azure/)</span></span>
- [<span data-ttu-id="f8f53-130">Обзор Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview)

<span data-ttu-id="f8f53-131">В этом руководстве также предполагается, что у вас есть подписка Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="f8f53-132">Если у вас еще нет подписки, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.</span><span class="sxs-lookup"><span data-stu-id="f8f53-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="f8f53-133">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="f8f53-133">Prerequisites</span></span>

<span data-ttu-id="f8f53-134">Перед началом работы с этим решением убедитесь, что выполнены следующие требования.</span><span class="sxs-lookup"><span data-stu-id="f8f53-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="f8f53-135">Пакет средств разработки Azure Stack (ASDK) или подписка на интегрированную систему Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="f8f53-136">Чтобы развернуть ASDK, следуйте инструкциям в разделе [Развертывание пакета SDK для Azure Stack](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="f8f53-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install).</span></span>
- <span data-ttu-id="f8f53-137">В вашей установке Azure Stack Hub должны быть установлены следующие компоненты:</span><span class="sxs-lookup"><span data-stu-id="f8f53-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="f8f53-138">Служба приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-138">The Azure App Service.</span></span> <span data-ttu-id="f8f53-139">Обратитесь к своему оператору Azure Stack Hub, чтобы развернуть и настроить Службу приложений Azure в своей среде.</span><span class="sxs-lookup"><span data-stu-id="f8f53-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="f8f53-140">Для работы с этим руководством требуется, чтобы в Службе приложений была доступна по крайней мере одна (1) выделенная рабочая роль.</span><span class="sxs-lookup"><span data-stu-id="f8f53-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="f8f53-141">Образ Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="f8f53-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="f8f53-142">Windows Server 2016 с образом Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="f8f53-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="f8f53-143">Соответствующие планы и предложения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="f8f53-144">Доменное имя для веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-144">A domain name for your web app.</span></span> <span data-ttu-id="f8f53-145">Если у вас нет доменного имени, его можно приобрести у поставщика доменов, таких как GoDaddy, Bluehost и InMotion.</span><span class="sxs-lookup"><span data-stu-id="f8f53-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="f8f53-146">SSL-сертификат для вашего домена из доверенного центра сертификации, например LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="f8f53-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="f8f53-147">Веб-приложение, которое обменивается данными с Базой данных SQL Server и поддерживает Application Insights.</span><span class="sxs-lookup"><span data-stu-id="f8f53-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="f8f53-148">Вы можете загрузить пример приложения [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) с GitHub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="f8f53-149">Гибридная сеть между виртуальной сетью Azure и виртуальной сетью Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="f8f53-150">Дополнительные инструкции см. в статье [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md) (Настройка подключения к гибридному облаку с помощью Azure и Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="f8f53-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="f8f53-151">Гибридный конвейер непрерывной интеграции и развертывания (CI/CD) с частным агентом сборки в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="f8f53-152">Подробные сведения см. в статье о [настройке идентификатора гибридного облака для приложений Azure и Azure Stack Hub](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="f8f53-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="f8f53-153">Развертывание базы данных SQL Server, подключенной с помощью гибридного подключения</span><span class="sxs-lookup"><span data-stu-id="f8f53-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="f8f53-154">Войдите на портал пользователя Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="f8f53-155">На **панели мониторинга** выберите **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="f8f53-157">В **Marketplace** выберите **Вычисления**, а затем выберите **Дополнительно**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="f8f53-158">В разделе **Дополнительно** выберите образ **Free SQL Server License: SQL Server 2017 Developer on Windows Server** (Бесплатная лицензия на SQL Server: SQL Server 2017 Developer на базе Windows Server 2016).</span><span class="sxs-lookup"><span data-stu-id="f8f53-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Выбор образа виртуальной машины на пользовательском портале Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="f8f53-160">В разделе **Free SQL Server License: SQL Server 2017 Developer on Windows Server** (Бесплатная лицензия на SQL Server: SQL Server 2017 Developer на базе Windows Server) выберите **Создать**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="f8f53-161">Выберите **Основные сведения > Настройка базовых параметров**, укажите **имя** виртуальной машины, **имя пользователя** для сопоставления безопасности SQL Server и **пароль** для сопоставления безопасности.</span><span class="sxs-lookup"><span data-stu-id="f8f53-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="f8f53-162">В раскрывающемся списке **Подписка** выберите подписку для развертывания.</span><span class="sxs-lookup"><span data-stu-id="f8f53-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="f8f53-163">Для параметра **Группа ресурсов** выберите значение **Choose existing** (Выбрать существующую) и поместите виртуальную машину в ту же группу ресурсов, что и веб-приложение Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Настройка основных параметров виртуальной машины на пользовательском портале Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="f8f53-165">В разделе **Размер** нужно выбрать размер виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="f8f53-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="f8f53-166">В этом руководстве мы рекомендуем A2_Standard или DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="f8f53-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="f8f53-167">В разделе **Настройки > Настроить дополнительные возможности** настройте следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="f8f53-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="f8f53-168">**Учетная запись хранения**. Создайте учетную запись, если необходимо.</span><span class="sxs-lookup"><span data-stu-id="f8f53-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="f8f53-169">**Виртуальная сеть.**</span><span class="sxs-lookup"><span data-stu-id="f8f53-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="f8f53-170">Убедитесь, что виртуальная машина SQL Server развернута в той же виртуальной сети, что и VPN-шлюзы.</span><span class="sxs-lookup"><span data-stu-id="f8f53-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="f8f53-171">**Общедоступный IP-адрес**. Оставьте параметры по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="f8f53-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="f8f53-172">**Группа безопасности сети**. (NSG).</span><span class="sxs-lookup"><span data-stu-id="f8f53-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="f8f53-173">Создайте NSG.</span><span class="sxs-lookup"><span data-stu-id="f8f53-173">Create a new NSG.</span></span>
   - <span data-ttu-id="f8f53-174">**Расширения и мониторинг**. Оставьте параметры по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="f8f53-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="f8f53-175">**Учетная запись хранения для диагностики**. Создайте учетную запись, если необходимо.</span><span class="sxs-lookup"><span data-stu-id="f8f53-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="f8f53-176">Нажмите кнопку **ОК**, чтобы сохранить конфигурацию.</span><span class="sxs-lookup"><span data-stu-id="f8f53-176">Select **OK** to save your configuration.</span></span>

     ![Настройка дополнительных функций виртуальной машины на пользовательском портале Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="f8f53-178">В разделе **Настройки SQL Server** настройте следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="f8f53-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="f8f53-179">Для параметра **Подключение SQL** выберите **Общедоступный (Интернет)** .</span><span class="sxs-lookup"><span data-stu-id="f8f53-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="f8f53-180">Для параметра **Порт** оставьте значение по умолчанию **1433**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="f8f53-181">Для параметра **Проверка подлинности SQL** выберите значение **Включить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="f8f53-182">При включении параметра проверки подлинности SQL он должен быть автоматически заполнен сведениями "SQLAdmin", настроенными в разделе **Основные сведения**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="f8f53-183">Для остальных параметров оставьте значения по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="f8f53-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="f8f53-184">Щелкните **ОК**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-184">Select **OK**.</span></span>

     ![Настройка параметров SQL Server на пользовательском портале Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="f8f53-186">В разделе **Сводка** проверьте конфигурацию ВМ, а затем нажмите кнопку **ОК**, чтобы начать развертывание.</span><span class="sxs-lookup"><span data-stu-id="f8f53-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Сводка конфигурации на пользовательском портале Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="f8f53-188">Создание виртуальной машины может занять некоторое время.</span><span class="sxs-lookup"><span data-stu-id="f8f53-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="f8f53-189">Просмотреть состояние виртуальных машин можно в разделе **Виртуальные машины**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Состояние виртуальных машин на пользовательском портале Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="f8f53-191">Создание веб-приложений в Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f8f53-192">Служба приложений Azure упрощает запуск веб-приложения и управление им.</span><span class="sxs-lookup"><span data-stu-id="f8f53-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="f8f53-193">Так как Azure Stack Hub согласуется с Azure, Службу приложений можно запускать в обеих средах.</span><span class="sxs-lookup"><span data-stu-id="f8f53-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="f8f53-194">Вы будете использовать службу приложений для размещения приложения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="f8f53-195">Создание веб-приложений</span><span class="sxs-lookup"><span data-stu-id="f8f53-195">Create web apps</span></span>

1. <span data-ttu-id="f8f53-196">Создайте веб-приложение в Azure, следуя инструкциям в разделе [Создание плана службы приложений](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="f8f53-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="f8f53-197">Веб-приложение должно находиться в той же подписке и группе ресурсов, что и ваша гибридная сеть.</span><span class="sxs-lookup"><span data-stu-id="f8f53-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="f8f53-198">Повторите предыдущий шаг (1) в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="f8f53-199">Добавление маршрута для Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="f8f53-200">Служба приложений в Azure Stack Hub должна поддерживать маршрутизацию из общедоступного сегмента Интернета, чтобы пользователи могли получить доступ к приложению.</span><span class="sxs-lookup"><span data-stu-id="f8f53-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="f8f53-201">Если вы можете получить доступ к Azure Stack Hub из Интернета, запишите общедоступный IP-адрес или URL-адрес веб-приложения Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="f8f53-202">Если вы используете ASDK, вы можете [настроить статическое сопоставление NAT](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal), чтобы предоставить службу приложений за пределами виртуального окружения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="f8f53-203">Подключение веб-приложения в среде Azure к гибридной сети</span><span class="sxs-lookup"><span data-stu-id="f8f53-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="f8f53-204">Чтобы обеспечить подключение между веб-интерфейсом в Azure и Базой данных SQL Server в Azure Stack Hub, веб-приложение должно быть подключено к гибридной сети между Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="f8f53-205">Чтобы настроить подключение, необходимо сделать следующее:</span><span class="sxs-lookup"><span data-stu-id="f8f53-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="f8f53-206">настроить подключение "точка — сеть";</span><span class="sxs-lookup"><span data-stu-id="f8f53-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="f8f53-207">настроить веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="f8f53-207">Configure the web app.</span></span>
- <span data-ttu-id="f8f53-208">изменить шлюз локальной сети в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="f8f53-209">Настройка подключения "точка — сеть" в виртуальной сети Azure</span><span class="sxs-lookup"><span data-stu-id="f8f53-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="f8f53-210">Шлюз виртуальной сети в гибридной сети на стороне Azure должен разрешить подключения "точка — сеть" для интеграции со Службой приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="f8f53-211">На портале Azure перейдите на страницу шлюза виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="f8f53-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="f8f53-212">В разделе **Параметры** выберите **Конфигурация "точка-сеть"** .</span><span class="sxs-lookup"><span data-stu-id="f8f53-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Параметр "точка — сеть" в настройках шлюза виртуальной сети Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="f8f53-214">Выберите **Настроить сейчас**, чтобы настроить подключение "точка — сеть".</span><span class="sxs-lookup"><span data-stu-id="f8f53-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Запуск конфигурации "точка — сеть" в настройках шлюза виртуальной сети Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="f8f53-216">На странице конфигурации **Точка — сеть** в поле **Пул адресов** введите диапазон частных IP-адресов, который вы хотите использовать.</span><span class="sxs-lookup"><span data-stu-id="f8f53-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="f8f53-217">Указанный диапазон не должен перекрывать другие диапазоны адресов, которые уже используются в подсетях в глобальной системе Azure или компонентах Azure Stack Hub гибридной сети.</span><span class="sxs-lookup"><span data-stu-id="f8f53-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="f8f53-218">В разделе **Тип туннеля** снимите флажок **IKEv2 VPN**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="f8f53-219">Нажмите кнопку **Сохранить**, чтобы завершить настройку подключения "точка — сеть".</span><span class="sxs-lookup"><span data-stu-id="f8f53-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Параметры "точка — сеть" в настройках шлюза виртуальной сети Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="f8f53-221">Интеграция приложения Службы приложений Azure с гибридной сетью</span><span class="sxs-lookup"><span data-stu-id="f8f53-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="f8f53-222">Чтобы подключить приложение к виртуальной сети Azure, следуйте инструкциям по [включению интеграции с виртуальной сетью, требуемой шлюзом](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="f8f53-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="f8f53-223">Перейдите к разделу **Параметры** для плана службы приложений, в котором размещено веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="f8f53-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="f8f53-224">В разделе **Параметры** выберите **Сеть**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-224">In **Settings**, select **Networking**.</span></span>

    ![Конфигурация сети для плана службы приложений](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="f8f53-226">В разделе **Интеграция виртуальной сети** выберите **Щелкните здесь для управления**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Управление интеграцией виртуальной сети для плана службы приложений](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="f8f53-228">Выберите виртуальную сеть, которую требуется настроить.</span><span class="sxs-lookup"><span data-stu-id="f8f53-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="f8f53-229">В разделе **IP-АДРЕСА, ПЕРЕНАПРАВЛЕННЫЕ В ВИРТУАЛЬНУЮ СЕТЬ** введите диапазон IP-адресов для виртуальной сети Azure, виртуальной сети Azure Stack Hub и адресных пространств "точка — сеть".</span><span class="sxs-lookup"><span data-stu-id="f8f53-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="f8f53-230">Нажмите кнопку **Сохранить**, чтобы проверить и сохранить настройки.</span><span class="sxs-lookup"><span data-stu-id="f8f53-230">Select **Save** to validate and save these settings.</span></span>

    ![Диапазоны IP-адресов для маршрутизации в интеграции виртуальной сети](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="f8f53-232">Дополнительные сведения об интеграции службы приложений с виртуальными сетями Azure см. в статье [Интеграция приложения с виртуальной сетью Azure](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="f8f53-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="f8f53-233">Настройка виртуальной сети Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="f8f53-234">Для шлюза локальной сети в виртуальной сети Azure Stack Hub необходимо настроить маршрутизацию трафика из диапазона адресов "точка — сеть" Службы приложений.</span><span class="sxs-lookup"><span data-stu-id="f8f53-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="f8f53-235">На портале Azure Stack Hub выберите **Шлюз локальной сети**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-235">In the Azure Stack Hub portal, go to **Local network gateway**.</span></span> <span data-ttu-id="f8f53-236">В разделе **Параметры** выберите пункт **Конфигурация**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-236">Under **Settings**, select **Configuration**.</span></span>

    ![Параметр конфигурации шлюза в настройках шлюза локальной сети Azure Stack Hub](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="f8f53-238">В поле **Адресное пространство** введите диапазон адресов для подключения "точка — сеть" для шлюза виртуальной сети в Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Адресное пространство "точка — сеть" в настройках шлюза локальной сети Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="f8f53-240">Нажмите кнопку **Сохранить**, чтобы проверить и сохранить конфигурацию.</span><span class="sxs-lookup"><span data-stu-id="f8f53-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="f8f53-241">Настройка DNS для масштабирования в нескольких облаках</span><span class="sxs-lookup"><span data-stu-id="f8f53-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="f8f53-242">Благодаря правильной настройке DNS для облачных приложений пользователи могут получать доступ к глобальной среде Azure и экземплярам Azure Stack Hub веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="f8f53-243">Конфигурация DNS для этого руководства также позволяет диспетчеру трафика Azure перенаправлять трафик при изменении нагрузки.</span><span class="sxs-lookup"><span data-stu-id="f8f53-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="f8f53-244">Для управления Azure DNS в этом руководстве используется Azure DNS, иначе домены Службы приложений не будут работать.</span><span class="sxs-lookup"><span data-stu-id="f8f53-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="f8f53-245">Создание дочерних доменов</span><span class="sxs-lookup"><span data-stu-id="f8f53-245">Create subdomains</span></span>

<span data-ttu-id="f8f53-246">Так как диспетчер трафика использует имена DNS CNAME, дочерний домен необходим для правильной маршрутизации трафика в конечные точки.</span><span class="sxs-lookup"><span data-stu-id="f8f53-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="f8f53-247">Подробные сведения о сопоставлении записей DNS и доменных имен с использованием диспетчера трафика см. в [этой статье](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="f8f53-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="f8f53-248">Для конечной точки Azure вы создадите дочерний домен, с помощью которого пользователи смогут получать доступ к веб-приложению.</span><span class="sxs-lookup"><span data-stu-id="f8f53-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="f8f53-249">Для этого руководства можно использовать **app.northwind.com**, но это значение необходимо настроить на основе вашего собственного домена.</span><span class="sxs-lookup"><span data-stu-id="f8f53-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="f8f53-250">Кроме того, необходимо создать дочерний домен с записью A для конечной точки Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="f8f53-251">Вы можете использовать **azurestack.northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="f8f53-252">Настройка личного домена в Azure</span><span class="sxs-lookup"><span data-stu-id="f8f53-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="f8f53-253">Добавьте имя узла **app.northwind.com** в веб-приложении Azure путем [сопоставления записи CNAME со Службой приложений Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="f8f53-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="f8f53-254">Настройка личных доменов в Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="f8f53-255">Добавьте имя узла **azurestack.northwind.com** в веб-приложении Azure Stack Hub путем [сопоставления записи A со Службой приложений Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="f8f53-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="f8f53-256">Используйте маршрутизируемый через Интернет IP-адрес для приложения Службы приложений.</span><span class="sxs-lookup"><span data-stu-id="f8f53-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="f8f53-257">Добавьте имя узла **app.northwind.com** в веб-приложении Azure Stack Hub путем [сопоставления записи CNAME со Службой приложений Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="f8f53-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="f8f53-258">Используйте имя узла, которое вы настроили на предыдущем шаге (1), как целевой объект для записи CNAME.</span><span class="sxs-lookup"><span data-stu-id="f8f53-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="f8f53-259">Настройка SSL-сертификатов для масштабирования в нескольких облаках</span><span class="sxs-lookup"><span data-stu-id="f8f53-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="f8f53-260">Очень важно обеспечить безопасность конфиденциальных данных, собранных веб-приложением, как при передаче в Базу данных SQL, так и при хранении в ней.</span><span class="sxs-lookup"><span data-stu-id="f8f53-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="f8f53-261">Вы настроите веб-приложения Azure и Azure Stack Hub для использования SSL-сертификатов для всего входящего трафика.</span><span class="sxs-lookup"><span data-stu-id="f8f53-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="f8f53-262">Добавление SSL в Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f8f53-263">Добавление SSL в Azure:</span><span class="sxs-lookup"><span data-stu-id="f8f53-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="f8f53-264">Убедитесь, что получаемый SSL-сертификат является допустимым для созданного дочернего домена.</span><span class="sxs-lookup"><span data-stu-id="f8f53-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="f8f53-265">(Можно воспользоваться групповыми сертификатами.)</span><span class="sxs-lookup"><span data-stu-id="f8f53-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="f8f53-266">На портале Azure следуйте инструкциям из разделов о **подготовке веб-приложения** и **привязке SSL-сертификата** статьи [Руководство по передаче и привязыванию SSL-сертификата в Службе приложений Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="f8f53-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="f8f53-267">Выберите значение **SNI-based SSL** (SSL на основе SNI) в качестве **типа SSL**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="f8f53-268">Перенаправьте весь трафик в HTTPS-порт.</span><span class="sxs-lookup"><span data-stu-id="f8f53-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="f8f53-269">Следуйте инструкциям в разделе **Принудительное использование HTTPS** статьи [Руководство. Привязывание существующего настраиваемого SSL-сертификата к веб-приложениям Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="f8f53-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="f8f53-270">Чтобы добавить SSL в Azure Stack Hub, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="f8f53-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="f8f53-271">Повторите шаги 1–3, которые использовались для Azure, с помощью портала Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="f8f53-272">Настройка и развертывание веб-приложения</span><span class="sxs-lookup"><span data-stu-id="f8f53-272">Configure and deploy the web app</span></span>

<span data-ttu-id="f8f53-273">Вы измените код приложения, чтобы обеспечить передачу данных телеметрии в правильный экземпляр Application Insights, и настроите веб-приложения с использованием правильных строк подключения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="f8f53-274">Дополнительные сведения о службе Application Insights см. в статье [Что такое Azure Application Insights?](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="f8f53-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="f8f53-275">Добавление Application Insights</span><span class="sxs-lookup"><span data-stu-id="f8f53-275">Add Application Insights</span></span>

1. <span data-ttu-id="f8f53-276">Откройте веб-приложение в Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="f8f53-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="f8f53-277">[Добавьте Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) в проект, чтобы передать данные телеметрии, с помощью которых Application Insights создает оповещения при увеличении или уменьшении веб-трафика.</span><span class="sxs-lookup"><span data-stu-id="f8f53-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="f8f53-278">Настройка динамических строк подключения</span><span class="sxs-lookup"><span data-stu-id="f8f53-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="f8f53-279">Каждый экземпляр веб-приложения будет использовать для подключения к Базе данных SQL разные методы.</span><span class="sxs-lookup"><span data-stu-id="f8f53-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="f8f53-280">Приложение в Azure использует частный IP-адрес виртуальной машины SQL Server, а приложение в Azure Stack Hub — общедоступный IP-адрес виртуальной машины SQL Server.</span><span class="sxs-lookup"><span data-stu-id="f8f53-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="f8f53-281">В интегрированной системе Azure Stack Hub общедоступный IP-адрес не должен маршрутизироваться через Интернет.</span><span class="sxs-lookup"><span data-stu-id="f8f53-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="f8f53-282">В ASDK общедоступный IP-адрес не поддерживает маршрутизацию за пределы ASDK.</span><span class="sxs-lookup"><span data-stu-id="f8f53-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="f8f53-283">С помощью переменных Среды службы приложений вы можете передать другую строку подключения каждому экземпляру приложения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="f8f53-284">Откройте приложение в Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="f8f53-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="f8f53-285">Откройте файл Startup.cs и найдите такой блок кода:</span><span class="sxs-lookup"><span data-stu-id="f8f53-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="f8f53-286">Замените предыдущий блок кода следующим кодом, в котором используется строка подключения, определенная в файле *appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="f8f53-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="f8f53-287">Настройка параметров Службы приложений</span><span class="sxs-lookup"><span data-stu-id="f8f53-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="f8f53-288">Создайте строки подключения для Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="f8f53-289">Эти строки должны быть одинаковыми за исключением используемых IP-адресов.</span><span class="sxs-lookup"><span data-stu-id="f8f53-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="f8f53-290">В Azure и Azure Stack Hub добавьте соответствующую строку подключения [в качестве параметра приложения](/azure/app-service/web-sites-configure) в веб-приложении, используя `SQLCONNSTR\_` в качестве префикса имени.</span><span class="sxs-lookup"><span data-stu-id="f8f53-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="f8f53-291">**Сохраните** параметры веб-приложения и перезагрузите приложение.</span><span class="sxs-lookup"><span data-stu-id="f8f53-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="f8f53-292">Включение автоматического масштабирования в глобальной среде Azure</span><span class="sxs-lookup"><span data-stu-id="f8f53-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="f8f53-293">При создании веб-приложения в Среде службы приложений запускается один его экземпляр.</span><span class="sxs-lookup"><span data-stu-id="f8f53-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="f8f53-294">Его масштаб можно автоматически горизонтально увеличить, добавляя экземпляры, чтобы предоставить приложению дополнительные вычислительные ресурсы.</span><span class="sxs-lookup"><span data-stu-id="f8f53-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="f8f53-295">Аналогичным образом можно автоматически горизонтально уменьшить масштаб и уменьшить число экземпляров, необходимых для приложения.</span><span class="sxs-lookup"><span data-stu-id="f8f53-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="f8f53-296">Чтобы настроить горизонтальное увеличение и уменьшение масштаба, необходим план службы приложений.</span><span class="sxs-lookup"><span data-stu-id="f8f53-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="f8f53-297">Если у вас нет плана, создайте его, прежде чем выполнять дальнейшие действия.</span><span class="sxs-lookup"><span data-stu-id="f8f53-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="f8f53-298">Включение автоматического развертывания</span><span class="sxs-lookup"><span data-stu-id="f8f53-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="f8f53-299">На портале Azure найдите план службы приложений для сайтов, которые нужно горизонтально масштабировать, а затем выберите **Увеличить масштаб (план службы приложений)** .</span><span class="sxs-lookup"><span data-stu-id="f8f53-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Масштабирование службы приложений Azure](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="f8f53-301">Выберите **Включить автомасштабирование**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-301">Select **Enable autoscale**.</span></span>

    ![Включение автомасштабирования в Службе приложений Azure](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="f8f53-303">Введите имя для параметра **Имя параметра автомасштабирования**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="f8f53-304">Для правила автомасштабирования **по умолчанию** выберите параметр **Масштабировать на основе метрики**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="f8f53-305">Задайте для параметра **Ограничения экземпляров** значения **Минимум: 1**, **Максимум: 10** и **По умолчанию: 1**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Конфигурация автомасштабирования в Службе приложений Azure](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="f8f53-307">Выберите **+Add a rule** (+Добавление правила).</span><span class="sxs-lookup"><span data-stu-id="f8f53-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="f8f53-308">В разделе **Источник метрики** выберите **Current Resource** (Текущий ресурс).</span><span class="sxs-lookup"><span data-stu-id="f8f53-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="f8f53-309">Используйте следующие критерии и действия для правила.</span><span class="sxs-lookup"><span data-stu-id="f8f53-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="f8f53-310">Критерии</span><span class="sxs-lookup"><span data-stu-id="f8f53-310">Criteria</span></span>

1. <span data-ttu-id="f8f53-311">В разделе **Агрегат времени** выберите **Среднее**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="f8f53-312">В разделе **Имя метрики** выберите **Процент ЦП**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="f8f53-313">В разделе **Оператор** выберите **больше**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="f8f53-314">Для параметра **Пороговое значение** задайте значение **50**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="f8f53-315">Для параметра **Длительность** задайте значение **10**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="f8f53-316">Действие</span><span class="sxs-lookup"><span data-stu-id="f8f53-316">Action</span></span>

1. <span data-ttu-id="f8f53-317">В разделе **Операция** выберите **Увеличить счетчик на**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="f8f53-318">Для параметра **Число экземпляров** задайте значение **2**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="f8f53-319">Задайте для параметра **Восстановление** значение **5**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="f8f53-320">Выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-320">Select **Add**.</span></span>

5. <span data-ttu-id="f8f53-321">Выберите **+ Add a rule** (+ Добавление правила).</span><span class="sxs-lookup"><span data-stu-id="f8f53-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="f8f53-322">В разделе **Источник метрики** выберите **Current Resource** (Текущий ресурс).</span><span class="sxs-lookup"><span data-stu-id="f8f53-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="f8f53-323">Текущий ресурс будет содержать имя или GUID плана службы приложений, а раскрывающиеся списки **Тип ресурса** и **Ресурс** станут недоступными.</span><span class="sxs-lookup"><span data-stu-id="f8f53-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="f8f53-324">Включение автоматического горизонтального уменьшения масштаба</span><span class="sxs-lookup"><span data-stu-id="f8f53-324">Enable automatic scale in</span></span>

<span data-ttu-id="f8f53-325">После уменьшения трафика веб-приложение Azure может автоматически сократить число активных экземпляров для снижения затрат.</span><span class="sxs-lookup"><span data-stu-id="f8f53-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="f8f53-326">Это действие требует меньше ресурсов, чем развертывание, благодаря чему влияние на пользователей приложения сводится к минимуму.</span><span class="sxs-lookup"><span data-stu-id="f8f53-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="f8f53-327">Перейдите к условию горизонтального увеличения масштаба **По умолчанию**, а затем выберите **+ Add a rule** (Добавить правило).</span><span class="sxs-lookup"><span data-stu-id="f8f53-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="f8f53-328">Используйте следующие критерии и действия для правила.</span><span class="sxs-lookup"><span data-stu-id="f8f53-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="f8f53-329">Критерии</span><span class="sxs-lookup"><span data-stu-id="f8f53-329">Criteria</span></span>

1. <span data-ttu-id="f8f53-330">В разделе **Агрегат времени** выберите **Среднее**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="f8f53-331">В разделе **Имя метрики** выберите **Процент ЦП**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="f8f53-332">В разделе **Оператор** выберите **меньше**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="f8f53-333">Для параметра **Пороговое значение** задайте значение **30**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="f8f53-334">Для параметра **Длительность** задайте значение **10**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="f8f53-335">Действие</span><span class="sxs-lookup"><span data-stu-id="f8f53-335">Action</span></span>

1. <span data-ttu-id="f8f53-336">В разделе **Операция** выберите **Уменьшить счетчик на**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="f8f53-337">Задайте для параметра **Число экземпляров** значение **1**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="f8f53-338">Задайте для параметра **Восстановление** значение **5**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="f8f53-339">Выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="f8f53-340">Создание профиля диспетчера трафика и настройка его для масштабирования в нескольких облаках</span><span class="sxs-lookup"><span data-stu-id="f8f53-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="f8f53-341">Создайте на портале Azure профиль Диспетчера трафика, а затем настройте конечные точки, чтобы включить масштабирование в нескольких облаках.</span><span class="sxs-lookup"><span data-stu-id="f8f53-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="f8f53-342">Создание профиля диспетчера трафика</span><span class="sxs-lookup"><span data-stu-id="f8f53-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="f8f53-343">Выберите **Создать ресурс**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="f8f53-344">Выберите **Сети**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-344">Select **Networking**.</span></span>
3. <span data-ttu-id="f8f53-345">Выберите **Профиль диспетчера трафика** и настройте следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="f8f53-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="f8f53-346">В поле **Имя** введите имя профиля.</span><span class="sxs-lookup"><span data-stu-id="f8f53-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="f8f53-347">Это имя **должно** быть уникальным в пределах зоны trafficmanager.net. Оно используется для создания DNS-имени (например, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="f8f53-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="f8f53-348">Для параметра **Метод маршрутизации** выберите **Взвешенный**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="f8f53-349">Из списка **Подписка** выберите подписку, в которой необходимо создать этот профиль.</span><span class="sxs-lookup"><span data-stu-id="f8f53-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="f8f53-350">В разделе **Группа ресурсов** создайте группу ресурсов, в которую следует поместить этот профиль.</span><span class="sxs-lookup"><span data-stu-id="f8f53-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="f8f53-351">Из списка **Расположение группы ресурсов** выберите расположение группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="f8f53-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="f8f53-352">Этот параметр задает расположение группы ресурсов и не влияет на профиль диспетчера трафика, который развернут глобально.</span><span class="sxs-lookup"><span data-stu-id="f8f53-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="f8f53-353">Нажмите кнопку **создания**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-353">Select **Create**.</span></span>

    ![Создание профиля диспетчера трафика](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="f8f53-355">Когда глобальное развертывание профиля диспетчера трафика завершится, он появится в списке ресурсов для группы ресурсов, в которой он создан.</span><span class="sxs-lookup"><span data-stu-id="f8f53-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="f8f53-356">Добавление конечных точек диспетчера трафика</span><span class="sxs-lookup"><span data-stu-id="f8f53-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="f8f53-357">Найдите профиль диспетчера трафика, который вы создали.</span><span class="sxs-lookup"><span data-stu-id="f8f53-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="f8f53-358">(Если вы перешли в группу ресурсов для профиля, выберите профиль.)</span><span class="sxs-lookup"><span data-stu-id="f8f53-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="f8f53-359">В колонке **Профиль диспетчера трафика** в разделе **Параметры** щелкните **Конечные точки**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="f8f53-360">Выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-360">Select **Add**.</span></span>

4. <span data-ttu-id="f8f53-361">В разделе **Добавить конечную точку** используйте следующие параметры для Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="f8f53-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="f8f53-362">В поле **Тип** выберите **Внешняя конечная точка**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="f8f53-363">Укажите **имя** конечной точки.</span><span class="sxs-lookup"><span data-stu-id="f8f53-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="f8f53-364">Для параметра **Полное доменное имя или IP-адрес** укажите внешний URL-адрес веб-приложения Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="f8f53-365">Для параметра **Вес** оставьте значение по умолчанию **1**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="f8f53-366">В результате весь трафик будет поступать в эту конечную точку, если она работоспособна.</span><span class="sxs-lookup"><span data-stu-id="f8f53-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="f8f53-367">Оставьте флажок **Добавить как отключенный** снятым.</span><span class="sxs-lookup"><span data-stu-id="f8f53-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="f8f53-368">Нажмите кнопку **ОК**, чтобы сохранить конечную точку Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="f8f53-369">Далее нужно настроить конечную точку Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="f8f53-370">В разделе **Профиль диспетчера трафика** выберите **Конечные точки**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="f8f53-371">Щелкните **+Добавить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-371">Select **+Add**.</span></span>
3. <span data-ttu-id="f8f53-372">В разделе **Добавить конечную точку** используйте следующие параметры для Azure:</span><span class="sxs-lookup"><span data-stu-id="f8f53-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="f8f53-373">В раскрывающемся списке **Тип** выберите **Конечная точка Azure**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="f8f53-374">Укажите **имя** конечной точки.</span><span class="sxs-lookup"><span data-stu-id="f8f53-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="f8f53-375">В раскрывающемся списке **Тип целевого ресурса** выберите **Служба приложений**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="f8f53-376">В разделе **Целевой ресурс** щелкните **Выберите службу приложений**, чтобы отобразился список веб-приложений, размещенных в этой подписке.</span><span class="sxs-lookup"><span data-stu-id="f8f53-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="f8f53-377">В колонке **Ресурсы** выберите службу приложений, которую требуется добавить в качестве первой конечной точки.</span><span class="sxs-lookup"><span data-stu-id="f8f53-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="f8f53-378">Для параметра **Вес** выберите значение **2**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="f8f53-379">В результате весь трафик будет передаваться в эту конечную точку, если основная конечная точка неработоспособна или если у вас настроено правило либо оповещение, которое при активации перенаправляет трафик.</span><span class="sxs-lookup"><span data-stu-id="f8f53-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="f8f53-380">Оставьте флажок **Добавить как отключенный** снятым.</span><span class="sxs-lookup"><span data-stu-id="f8f53-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="f8f53-381">Нажмите кнопку **ОК**, чтобы сохранить конечную точку Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="f8f53-382">После настройки обе конечные точки отобразятся в разделе **Профиль диспетчера трафика**, если выбрать **Конечные точки**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="f8f53-383">В примере на следующем снимке экрана показано две конечные точки со сведениями о состоянии и конфигурации каждой из них.</span><span class="sxs-lookup"><span data-stu-id="f8f53-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Конечные точки в профиле Диспетчера трафика](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="f8f53-385">Настройка мониторинга и оповещений Application Insights в Azure</span><span class="sxs-lookup"><span data-stu-id="f8f53-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="f8f53-386">Azure Application Insights позволяет отслеживать приложение и отправлять оповещения на основе настраиваемых условий</span><span class="sxs-lookup"><span data-stu-id="f8f53-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="f8f53-387">(например, при недоступности приложения, возникновении ошибок или проблемах с производительностью).</span><span class="sxs-lookup"><span data-stu-id="f8f53-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="f8f53-388">Для создания оповещений вы будете использовать метрики Azure Application Insights.</span><span class="sxs-lookup"><span data-stu-id="f8f53-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="f8f53-389">Когда эти оповещения активируются, экземпляр веб-приложений автоматически переключится с Azure Stack Hub на Azure для горизонтального увеличения масштаба, а затем обратно на Azure Stack Hub для горизонтального уменьшения масштаба.</span><span class="sxs-lookup"><span data-stu-id="f8f53-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="f8f53-390">Создание оповещения из метрик</span><span class="sxs-lookup"><span data-stu-id="f8f53-390">Create an alert from metrics</span></span>

<span data-ttu-id="f8f53-391">В рамках этого руководства перейдите к группе ресурсов на портале Azure, а затем выберите экземпляр Application Insights, чтобы открыть **Application Insights**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="f8f53-393">С помощью этого представления вы создадите оповещения о горизонтальном увеличении и уменьшении масштаба.</span><span class="sxs-lookup"><span data-stu-id="f8f53-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="f8f53-394">Создание оповещения о развертывании</span><span class="sxs-lookup"><span data-stu-id="f8f53-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="f8f53-395">В меню **Настройка** выберите **Оповещения (классические)** .</span><span class="sxs-lookup"><span data-stu-id="f8f53-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="f8f53-396">Выберите **Добавить оповещение метрики (классическое)** .</span><span class="sxs-lookup"><span data-stu-id="f8f53-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="f8f53-397">В разделе **Добавление правила** настройте следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="f8f53-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="f8f53-398">Для параметра **Имя** введите **Burst into Azure Cloud** (Повышение в облако Azure).</span><span class="sxs-lookup"><span data-stu-id="f8f53-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="f8f53-399">Поле **Описание** заполнять необязательно.</span><span class="sxs-lookup"><span data-stu-id="f8f53-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="f8f53-400">В разделе **Источник** > **Оповещения включены** выберите **Метрики**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="f8f53-401">В разделе **Критерии** выберите свою подписку, группу ресурсов для профиля диспетчера трафика, а также имя профиля диспетчера трафика для ресурса.</span><span class="sxs-lookup"><span data-stu-id="f8f53-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="f8f53-402">Для параметра **Метрика** выберите **Частота запросов**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="f8f53-403">Для параметра **Условие** выберите **больше**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="f8f53-404">Для параметра **Пороговое значение** введите **2**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="f8f53-405">Для параметра **Период** выберите **За последние 5 минут**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="f8f53-406">В разделе **Уведомить по**:</span><span class="sxs-lookup"><span data-stu-id="f8f53-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="f8f53-407">Установите флажок **Участники, читатели и владельцы электронной почты**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="f8f53-408">Введите адрес электронной почты в разделе **Дополнительные адреса электронной почты администратора**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="f8f53-409">В строке меню выберите **Сохранить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="f8f53-410">Создание оповещения о горизонтальном уменьшении масштаба</span><span class="sxs-lookup"><span data-stu-id="f8f53-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="f8f53-411">В меню **Настройка** выберите **Оповещения (классические)** .</span><span class="sxs-lookup"><span data-stu-id="f8f53-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="f8f53-412">Выберите **Добавить оповещение метрики (классическое)** .</span><span class="sxs-lookup"><span data-stu-id="f8f53-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="f8f53-413">В разделе **Добавление правила** настройте следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="f8f53-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="f8f53-414">В поле **Имя** введите **Scale back into Azure Stack Hub** (Свернуть обратно в Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="f8f53-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="f8f53-415">Поле **Описание** заполнять необязательно.</span><span class="sxs-lookup"><span data-stu-id="f8f53-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="f8f53-416">В разделе **Источник** > **Оповещения включены** выберите **Метрики**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="f8f53-417">В разделе **Критерии** выберите свою подписку, группу ресурсов для профиля диспетчера трафика, а также имя профиля диспетчера трафика для ресурса.</span><span class="sxs-lookup"><span data-stu-id="f8f53-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="f8f53-418">Для параметра **Метрика** выберите **Частота запросов**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="f8f53-419">Для параметра **Условие** выберите **меньше**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="f8f53-420">Для параметра **Пороговое значение** введите **2**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="f8f53-421">Для параметра **Период** выберите **За последние 5 минут**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="f8f53-422">В разделе **Уведомить по**:</span><span class="sxs-lookup"><span data-stu-id="f8f53-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="f8f53-423">Установите флажок **Участники, читатели и владельцы электронной почты**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="f8f53-424">Введите адрес электронной почты в разделе **Дополнительные адреса электронной почты администратора**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="f8f53-425">В строке меню выберите **Сохранить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="f8f53-426">На следующем снимке экрана показаны оповещения для развертывания и свертывания.</span><span class="sxs-lookup"><span data-stu-id="f8f53-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Оповещения Application Insights (классическое приложение)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="f8f53-428">Перенаправление трафика между Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f8f53-429">Вы можете настроить для трафика веб-приложения автоматическое переключение или переключение вручную между Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="f8f53-430">Настройка переключения вручную между Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f8f53-431">По достижении на веб-сайте пороговых значений, которые вы настроили, вы получите оповещение.</span><span class="sxs-lookup"><span data-stu-id="f8f53-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="f8f53-432">Для перенаправления трафика в Azure вручную сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="f8f53-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="f8f53-433">На портале Azure выберите профиль диспетчера трафика.</span><span class="sxs-lookup"><span data-stu-id="f8f53-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Конечные точки Диспетчера трафика на портале Azure](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="f8f53-435">Выберите **Конечные точки**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="f8f53-436">Выберите **Конечная точка Azure**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="f8f53-437">В разделе **Состояние** выберите **Включено** и нажмите **Сохранить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Включение конечной точки Azure на портале Azure](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="f8f53-439">На странице **Конечные точки** профиля диспетчера трафика выберите **Внешняя конечная точка**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="f8f53-440">В разделе **Состояние** выберите **Отключено** и нажмите **Сохранить**.</span><span class="sxs-lookup"><span data-stu-id="f8f53-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Отключение конечной точки Azure Stack Hub на портале Azure](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="f8f53-442">После настройки конечных точек трафик приложения передается в веб-приложение развертывания Azure, а не в веб-приложение Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Изменение конечных точек в трафике веб-приложения Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="f8f53-444">Для возвращения потока обратно в Azure Stack Hub используйте предыдущие шаги, чтобы:</span><span class="sxs-lookup"><span data-stu-id="f8f53-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="f8f53-445">включить конечную точку Azure Stack Hub;</span><span class="sxs-lookup"><span data-stu-id="f8f53-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="f8f53-446">отключить конечную точку Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="f8f53-447">Настройка автоматического переключения между Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f8f53-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f8f53-448">Вы также можете использовать мониторинг Application Insights, если приложение выполняется в [бессерверной](https://azure.microsoft.com/overview/serverless-computing/) среде, предоставляемой Функциями Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="f8f53-449">В этом случае можно настроить Application Insights, чтобы использовать веб-перехватчик, который вызывает приложение-функцию.</span><span class="sxs-lookup"><span data-stu-id="f8f53-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="f8f53-450">Это приложение автоматически включает или отключает конечную точку в ответ на оповещение.</span><span class="sxs-lookup"><span data-stu-id="f8f53-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="f8f53-451">Следуйте приведенным ниже инструкциям в качестве руководства для настройки автоматического переключения трафика.</span><span class="sxs-lookup"><span data-stu-id="f8f53-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="f8f53-452">Создайте приложение-функцию Azure.</span><span class="sxs-lookup"><span data-stu-id="f8f53-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="f8f53-453">Создайте функцию, активируемую HTTP-запросом.</span><span class="sxs-lookup"><span data-stu-id="f8f53-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="f8f53-454">Импортируйте пакеты SDK для Resource Manager, веб-приложений и диспетчера трафика.</span><span class="sxs-lookup"><span data-stu-id="f8f53-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="f8f53-455">Разработайте код, чтобы:</span><span class="sxs-lookup"><span data-stu-id="f8f53-455">Develop code to:</span></span>

   - <span data-ttu-id="f8f53-456">пройти проверку подлинности в подписке Azure;</span><span class="sxs-lookup"><span data-stu-id="f8f53-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="f8f53-457">использовать параметр, который переключает конечные точки диспетчера трафика, чтобы направить трафик в Azure или Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f8f53-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="f8f53-458">Сохраните код и добавьте URL-адрес приложения-функции с соответствующими параметрами в раздел **Веб-перехватчик** с параметрами правила генерации оповещений Application Insights.</span><span class="sxs-lookup"><span data-stu-id="f8f53-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="f8f53-459">Трафик будет автоматически перенаправлен, когда сработает оповещение Application Insights.</span><span class="sxs-lookup"><span data-stu-id="f8f53-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f8f53-460">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="f8f53-460">Next steps</span></span>

- <span data-ttu-id="f8f53-461">Дополнительные сведения о шаблонах для облака Azure см. в статье [Конструктивные шаблоны облачных решений](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="f8f53-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
