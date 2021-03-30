---
title: Перенаправление трафика с помощью географически распределенного приложения, созданного с использованием Azure и Azure Stack Hub
description: Из этой статьи вы узнаете, как с помощью Azure и Azure Stack Hub создать географически распределенное приложение, которое перенаправляет трафик в определенные конечные точки.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895437"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="f00dc-103">Перенаправление трафика с помощью географически распределенного приложения, созданного с использованием Azure и Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f00dc-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f00dc-104">Узнайте, как в шаблоне географически распределенного приложения направлять трафик к определенным конечным точкам на основе различных метрик.</span><span class="sxs-lookup"><span data-stu-id="f00dc-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="f00dc-105">Создав профиль диспетчера трафика с маршрутизацией по географическим данным и конфигурацией конечных точек, можно направлять данные к определенным конечным точкам с учетом региональных требований, корпоративных и международных стандартов и ваших потребностей по обработке данных.</span><span class="sxs-lookup"><span data-stu-id="f00dc-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="f00dc-106">В этом решении показано, как создать среду, после чего вы выполните в ней следующие действия:</span><span class="sxs-lookup"><span data-stu-id="f00dc-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f00dc-107">создание географически распределенного приложения;</span><span class="sxs-lookup"><span data-stu-id="f00dc-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="f00dc-108">использование диспетчера трафика для приложения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="f00dc-109">Использование шаблона географически распределенных приложений</span><span class="sxs-lookup"><span data-stu-id="f00dc-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="f00dc-110">Шаблон географически распределенного приложения позволяет использовать приложение в нескольких регионах.</span><span class="sxs-lookup"><span data-stu-id="f00dc-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="f00dc-111">Вы можете по умолчанию использовать общедоступное облако, но некоторым пользователям может потребоваться хранить данные в домашнем регионе.</span><span class="sxs-lookup"><span data-stu-id="f00dc-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="f00dc-112">Также можно направлять пользователей в подходящее облако на основании их требований.</span><span class="sxs-lookup"><span data-stu-id="f00dc-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="f00dc-113">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="f00dc-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="f00dc-114">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="f00dc-114">Scalability considerations</span></span>

<span data-ttu-id="f00dc-115">Создаваемое с помощью этой статьи решение не включает в себя механизмы масштабирования.</span><span class="sxs-lookup"><span data-stu-id="f00dc-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="f00dc-116">Но его можно сочетать с другими локальными или облачными и решениями Azure, чтобы удовлетворить требования к масштабируемости.</span><span class="sxs-lookup"><span data-stu-id="f00dc-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="f00dc-117">Сведения о создании гибридного решения для автоматического масштабирования в нескольких облаках в Azure с помощью диспетчера трафика см. в статье [Развертывание приложения, которое выполняет масштабирование в нескольких облаках с помощью Azure и Azure Stack Hub](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="f00dc-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="f00dc-118">Вопросы доступности</span><span class="sxs-lookup"><span data-stu-id="f00dc-118">Availability considerations</span></span>

<span data-ttu-id="f00dc-119">Как и в отношении масштабируемости, это решение не ориентировано непосредственно на обеспечение доступности.</span><span class="sxs-lookup"><span data-stu-id="f00dc-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="f00dc-120">Но локальные решения и решения Azure можно реализовать, чтобы обеспечить высокий уровень доступности всех используемых компонентов.</span><span class="sxs-lookup"><span data-stu-id="f00dc-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="f00dc-121">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="f00dc-121">When to use this pattern</span></span>

- <span data-ttu-id="f00dc-122">У вашей организации есть международные филиалы, для которых применяются региональные политики безопасности и распространения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="f00dc-123">Каждый из офисов организации собирает данные о сотрудниках, бизнесе и предприятии. Для этого нужно создавать отчеты с соблюдением местных требований (в том числе к срокам подачи).</span><span class="sxs-lookup"><span data-stu-id="f00dc-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="f00dc-124">Высокий уровень масштабируемости можно обеспечить за счет горизонтального масштабирования, при котором несколько приложений развертываются в одном или нескольких регионах.</span><span class="sxs-lookup"><span data-stu-id="f00dc-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="f00dc-125">Планирование топологии</span><span class="sxs-lookup"><span data-stu-id="f00dc-125">Planning the topology</span></span>

<span data-ttu-id="f00dc-126">Прежде чем создавать распределенную топологию, следует собрать следующие сведения:</span><span class="sxs-lookup"><span data-stu-id="f00dc-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="f00dc-127">**Личный домен приложения.** Какое имя личного домена клиенты будут использовать для доступа к приложению?</span><span class="sxs-lookup"><span data-stu-id="f00dc-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="f00dc-128">В нашем примере используется имя личного домена *www\.scalableasedemo.com*.</span><span class="sxs-lookup"><span data-stu-id="f00dc-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="f00dc-129">**Домен диспетчера трафика.** При создании [профиля диспетчера трафика Azure](/azure/traffic-manager/traffic-manager-manage-profiles) нужно выбрать доменное имя.</span><span class="sxs-lookup"><span data-stu-id="f00dc-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="f00dc-130">При регистрации записи домена, используемой диспетчером трафика, к этому имени будет добавлен суффикс *trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="f00dc-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="f00dc-131">В нашем примере приложения используется имя *scalable-ase-demo*,</span><span class="sxs-lookup"><span data-stu-id="f00dc-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="f00dc-132">поэтому полное имя домена, управляемое диспетчером трафика, имеет вид *scalable-ase-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="f00dc-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="f00dc-133">**Стратегия масштабирования приложения.** Определите, будет ли нагрузка распределяться между несколькими Средами службы приложений в одном регионе, в нескольких регионах или же и там, и там.</span><span class="sxs-lookup"><span data-stu-id="f00dc-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="f00dc-134">Решение должно учитывать предполагаемые источники клиентского трафика и возможности масштабируемости серверной инфраструктуры приложения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="f00dc-135">Например, приложение, работающее без отслеживания состояния, можно масштабировать, развернув конфигурацию из нескольких Сред службы приложений в одном регионе, а также в нескольких регионах Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="f00dc-136">Сейчас доступно более 15 глобальных регионов Azure. Поэтому клиенты могут создавать по-настоящему глобальные гипермасштабируемые приложения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="f00dc-137">Для нашего примера приложения создано три Среды службы приложений в одном регионе Azure ("Центрально-южная часть США").</span><span class="sxs-lookup"><span data-stu-id="f00dc-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="f00dc-138">**Соглашение об именовании для Сред службы приложений.** Имя каждой Среды службы приложений должно быть уникальным.</span><span class="sxs-lookup"><span data-stu-id="f00dc-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="f00dc-139">Если используется больше двух Сред службы приложений, для их идентификации желательно использовать определенное соглашение об именовании.</span><span class="sxs-lookup"><span data-stu-id="f00dc-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="f00dc-140">В нашем примере используется простое соглашение об именовании.</span><span class="sxs-lookup"><span data-stu-id="f00dc-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="f00dc-141">Среды службы приложений называются *fe1ase*, *fe2ase* и *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="f00dc-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="f00dc-142">**Соглашение об именовании приложений.** Так как предполагается развертывание нескольких экземпляров приложения, каждому из них необходимо присвоить уникальное имя.</span><span class="sxs-lookup"><span data-stu-id="f00dc-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="f00dc-143">В разных Средах службы приложений для Power Apps можно использовать одни и те же имена.</span><span class="sxs-lookup"><span data-stu-id="f00dc-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="f00dc-144">Так как у каждой Среды службы приложений есть уникальный доменный суффикс, разработчики могут использовать одно и то же имя приложения во всех средах.</span><span class="sxs-lookup"><span data-stu-id="f00dc-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="f00dc-145">Например, разработчик может назвать приложения так: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* и т. д.</span><span class="sxs-lookup"><span data-stu-id="f00dc-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="f00dc-146">В нашем примере каждый экземпляр приложения имеет уникальное имя.</span><span class="sxs-lookup"><span data-stu-id="f00dc-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="f00dc-147">*webfrontend1*, *webfrontend2* и *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="f00dc-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="f00dc-148">![Схема основных аспектов проектирования гибридных приложений](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="f00dc-148">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="f00dc-149">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="f00dc-150">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Это решение позволяет использовать единственное гибридное облако, с помощью которого можно создавать и развертывать гибридные приложения в любой точке мира.</span><span class="sxs-lookup"><span data-stu-id="f00dc-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="f00dc-151">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="f00dc-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="f00dc-152">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="f00dc-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="f00dc-153">Часть 1. Создание географически распределенного приложения</span><span class="sxs-lookup"><span data-stu-id="f00dc-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="f00dc-154">В этом раздел описано, как создать веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="f00dc-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f00dc-155">Создайте и опубликуйте веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="f00dc-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="f00dc-156">Добавьте код в Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="f00dc-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="f00dc-157">Настройте сборку приложения на несколько целевых объектов в облаке.</span><span class="sxs-lookup"><span data-stu-id="f00dc-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="f00dc-158">Организуйте и настройте процесс непрерывного развертывания.</span><span class="sxs-lookup"><span data-stu-id="f00dc-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f00dc-159">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="f00dc-159">Prerequisites</span></span>

<span data-ttu-id="f00dc-160">Требуются подписка Azure и установка Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="f00dc-161">Шаги по созданию географически распределенного приложения</span><span class="sxs-lookup"><span data-stu-id="f00dc-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="f00dc-162">Получение личного домена и настройка DNS</span><span class="sxs-lookup"><span data-stu-id="f00dc-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="f00dc-163">Обновите файл зоны DNS для домена.</span><span class="sxs-lookup"><span data-stu-id="f00dc-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="f00dc-164">Azure AD сможет проверить принадлежность имени личного домена.</span><span class="sxs-lookup"><span data-stu-id="f00dc-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="f00dc-165">Вы можете использовать [Azure DNS](/azure/dns/dns-getstarted-portal) для записей Azure, Microsoft 365 и внешних записей DNS в Azure или добавить запись DNS в [другой регистратор DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="f00dc-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="f00dc-166">Зарегистрируйте личный домен у уполномоченного регистратора.</span><span class="sxs-lookup"><span data-stu-id="f00dc-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="f00dc-167">Войдите в соответствующий регистратор доменных имен.</span><span class="sxs-lookup"><span data-stu-id="f00dc-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="f00dc-168">Для обновления DNS могут потребоваться права администратора.</span><span class="sxs-lookup"><span data-stu-id="f00dc-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="f00dc-169">Обновите файл зоны DNS для соответствующего домена, добавив предоставленную службой Azure AD DNS-запись.</span><span class="sxs-lookup"><span data-stu-id="f00dc-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="f00dc-170">Эта DNS-запись не влияет на поведение работающих систем, например на маршрутизацию почты или веб-хостинг.</span><span class="sxs-lookup"><span data-stu-id="f00dc-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="f00dc-171">Создание и публикация веб-приложения</span><span class="sxs-lookup"><span data-stu-id="f00dc-171">Create web apps and publish</span></span>

<span data-ttu-id="f00dc-172">Настройте гибридный конвейер непрерывной интеграции и непрерывного развертывания (CI/CD), чтобы развернуть веб-приложение в Azure и Azure Stack Hub и включить автоматическую отправку изменений в оба облака.</span><span class="sxs-lookup"><span data-stu-id="f00dc-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="f00dc-173">Вам необходим Azure Stack Hub с подходящими образами, объединенными для запуска (Windows Server и SQL), и развернутой Службой приложений.</span><span class="sxs-lookup"><span data-stu-id="f00dc-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="f00dc-174">Подробнее см. статью [Предварительные условия для развертывания Службы приложений в Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="f00dc-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="f00dc-175">Добавление кода в Azure Repos</span><span class="sxs-lookup"><span data-stu-id="f00dc-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="f00dc-176">Войдите в Visual Studio, используя **учетную запись, которая обладает правами на создание проекта** в Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="f00dc-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="f00dc-177">Процессы CI/CD применимы и к коду приложения, и к коду инфраструктуры.</span><span class="sxs-lookup"><span data-stu-id="f00dc-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="f00dc-178">Используйте [шаблоны Azure Resource Manager](https://azure.microsoft.com/resources/templates/) для частной и облачной разработки.</span><span class="sxs-lookup"><span data-stu-id="f00dc-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Подключение к проекту в Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="f00dc-180">**Клонируйте репозиторий** путем создания и открытия веб-приложения по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="f00dc-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Клонирование репозитория в Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="f00dc-182">Создание развертывания веб-приложений в обоих облаках</span><span class="sxs-lookup"><span data-stu-id="f00dc-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="f00dc-183">Измените файл **WebApplication.csproj**. Выберите `Runtimeidentifier` и добавьте `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="f00dc-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="f00dc-184">(См. документацию по [автономному развертыванию](/dotnet/core/deploying/deploy-with-vs#simpleSelf)).</span><span class="sxs-lookup"><span data-stu-id="f00dc-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Изменение файла проекта веб-приложения в Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="f00dc-186">**Добавьте код в Azure Repos** с помощью Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="f00dc-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="f00dc-187">Подтвердите, что **код приложения** был добавлен в Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="f00dc-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="f00dc-188">Создание определения сборки</span><span class="sxs-lookup"><span data-stu-id="f00dc-188">Create the build definition</span></span>

1. <span data-ttu-id="f00dc-189">**Войдите в Azure Pipelines**, чтобы подтвердить возможность создания определений сборки.</span><span class="sxs-lookup"><span data-stu-id="f00dc-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="f00dc-190">Добавьте код `-r win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="f00dc-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="f00dc-191">Это необходимо, чтобы активировать автономное развертывание с использованием .NET Core.</span><span class="sxs-lookup"><span data-stu-id="f00dc-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Добавление кода в определение сборки в Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="f00dc-193">**Запустите сборку**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-193">**Run the build**.</span></span> <span data-ttu-id="f00dc-194">Процесс [сборки автономного развертывания](/dotnet/core/deploying/deploy-with-vs#simpleSelf) будет публиковать артефакты, которые могут выполняться в Azure и Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="f00dc-195">Использование агента, размещенного в Azure</span><span class="sxs-lookup"><span data-stu-id="f00dc-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="f00dc-196">Использование размещенного агента в Azure Pipelines удобно для создания и развертывания веб-приложений.</span><span class="sxs-lookup"><span data-stu-id="f00dc-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="f00dc-197">Обновление и обслуживание, включая постоянную, непрерывную разработку, тестирование и развертывание, автоматически выполняет Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="f00dc-198">Настройка процесса непрерывного развертывания и управление им</span><span class="sxs-lookup"><span data-stu-id="f00dc-198">Manage and configure the CD process</span></span>

<span data-ttu-id="f00dc-199">Azure DevOps Services предоставляет конвейер с широкими возможностями настройки и управления для выпусков в различные среды, такие как среда развертывания, промежуточная среда, среда для контроля качества и рабочая среда, включая получение утверждений на определенных стадиях.</span><span class="sxs-lookup"><span data-stu-id="f00dc-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="f00dc-200">Создание определения выпуска</span><span class="sxs-lookup"><span data-stu-id="f00dc-200">Create release definition</span></span>

1. <span data-ttu-id="f00dc-201">Нажмите кнопку со знаком **плюса**, чтобы добавить новый выпуск на вкладке **Выпуски** в разделе **Сборка и выпуск** Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="f00dc-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Создание определения выпуска в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="f00dc-203">Примените шаблон развертывания службы приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-203">Apply the Azure App Service Deployment template.</span></span>

   ![Применение шаблона развертывания службы приложений Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="f00dc-205">В разделе **Добавление артефакта** добавьте артефакт для приложения сборки облака Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Добавление артефакта в облачную сборку Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="f00dc-207">На вкладке "Конвейер" щелкните ссылку **Фаза, задача** для используемой среды и задайте значения облачной среды Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Настройка значений облачной среды Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="f00dc-209">Задайте **имя среды** и выберите **подписку Azure** для конечной точки облака Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Выбор подписки Azure для облачной конечной точки Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="f00dc-211">В разделе **Имя Службы приложений** укажите требуемое имя.</span><span class="sxs-lookup"><span data-stu-id="f00dc-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Задание имени службы приложений Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="f00dc-213">Введите "Размещенная среда VS2017" в разделе **Очередь агента** для среды, размещенной в облаке Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Установка очереди агента для облачной среды Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="f00dc-215">В меню развертывания службы приложений Azure выберите допустимый для среды **пакет или папку**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="f00dc-216">Нажмите кнопку **ОК**, чтобы выбрать **расположение папки**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-216">Select **OK** to **folder location**.</span></span>
  
      ![Выбор пакета или папки для Среды службы приложений Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Диалоговое окно выбора папки 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="f00dc-219">Сохраните все изменения и вернитесь к **конвейеру выпуска**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Сохранение изменений в конвейере выпуска в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="f00dc-221">Добавьте новый артефакт, выбрав сборку для приложения Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Добавление нового артефакта для приложения Azure Stack Hub в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="f00dc-223">Добавьте еще одну среду, применив развертывание Службы приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Добавление среды в развертывание службы приложений Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="f00dc-225">Назовите новую среду Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-225">Name the new environment Azure Stack Hub.</span></span>

    ![Среда имен в развертывании службы приложений Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="f00dc-227">Найдите среду Azure Stack Hub на вкладке **Задача**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Среда Azure Stack Hub в Azure DevOps Services в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="f00dc-229">Выберите подписку для конечной точки Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Выбор подписки для конечной точки Azure Stack Hub в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="f00dc-231">Задайте имя веб-приложения Azure Stack Hub в качестве имени службы приложений.</span><span class="sxs-lookup"><span data-stu-id="f00dc-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Задание имени веб-приложения Azure Stack Hub в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="f00dc-233">Выберите агент Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-233">Select the Azure Stack Hub agent.</span></span>

    ![Выбор агента Azure Stack Hub в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="f00dc-235">В разделе "Развертывание Службы приложений Azure" выберите допустимый **пакет или папку** для среды.</span><span class="sxs-lookup"><span data-stu-id="f00dc-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="f00dc-236">Нажмите кнопку **ОК**, чтобы выбрать расположение папки.</span><span class="sxs-lookup"><span data-stu-id="f00dc-236">Select **OK** to folder location.</span></span>

    ![Выбор папки для развертывания Службы приложений Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Диалоговое окно выбора папки 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="f00dc-239">На вкладке "Переменная" добавьте переменную `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, задайте для нее значение **true** и область Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Добавление переменной в развертывание приложения Azure в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="f00dc-241">Выберите значок триггера **непрерывного** развертывания в обоих артефактах и включите триггер **непрерывного** развертывания.</span><span class="sxs-lookup"><span data-stu-id="f00dc-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Установка триггера непрерывного развертывания в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="f00dc-243">Выберите значок условий **перед развертыванием** в среде Azure Stack Hub и задайте триггеру значение **После выпуска**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Установка условий перед развертыванием в Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="f00dc-245">Сохраните все изменения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="f00dc-246">Некоторые параметры для задач могли быть автоматически определены как [переменные среды](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) при создании определения выпуска на основе шаблона.</span><span class="sxs-lookup"><span data-stu-id="f00dc-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="f00dc-247">Эти параметры нельзя изменить в параметрах задачи. Для этого нужно выбрать родительский элемент среды.</span><span class="sxs-lookup"><span data-stu-id="f00dc-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="f00dc-248">Часть 2. Методы обновления веб-приложения</span><span class="sxs-lookup"><span data-stu-id="f00dc-248">Part 2: Update web app options</span></span>

<span data-ttu-id="f00dc-249">[Служба приложений Azure](/azure/app-service/overview) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости.</span><span class="sxs-lookup"><span data-stu-id="f00dc-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Служба приложений Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="f00dc-251">Сопоставьте существующее настраиваемое DNS-имя с веб-приложениями Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="f00dc-252">Для этого используйте **запись CNAME** или **запись A**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="f00dc-253">Сопоставление существующего настраиваемого DNS-имени с веб-приложениями Azure</span><span class="sxs-lookup"><span data-stu-id="f00dc-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="f00dc-254">Используйте CNAME для всех настраиваемых DNS-имен, кроме корневого домена (например, northwind.com).</span><span class="sxs-lookup"><span data-stu-id="f00dc-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="f00dc-255">Сведения о том, как перенести активный веб-сайт и его DNS-имя домена в службу приложений, см. в статье [Перенос активного DNS-имени в службу приложений Azure](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="f00dc-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f00dc-256">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="f00dc-256">Prerequisites</span></span>

<span data-ttu-id="f00dc-257">Для работы с этим решением:</span><span class="sxs-lookup"><span data-stu-id="f00dc-257">To complete this solution:</span></span>

- <span data-ttu-id="f00dc-258">[Создайте приложение службы приложений](/azure/app-service/) или используйте приложение, созданное для работы с другим решением.</span><span class="sxs-lookup"><span data-stu-id="f00dc-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="f00dc-259">Приобретите доменное имя и предоставьте поставщику домена доступ к реестру DNS.</span><span class="sxs-lookup"><span data-stu-id="f00dc-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="f00dc-260">Обновите файл зоны DNS для домена.</span><span class="sxs-lookup"><span data-stu-id="f00dc-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="f00dc-261">Azure AD проверит принадлежность имени личного домена.</span><span class="sxs-lookup"><span data-stu-id="f00dc-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="f00dc-262">Вы можете использовать [Azure DNS](/azure/dns/dns-getstarted-portal) для записей Azure, Microsoft 365 и внешних записей DNS в Azure или добавить запись DNS в [другой регистратор DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="f00dc-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="f00dc-263">Зарегистрируйте личный домен у уполномоченного регистратора.</span><span class="sxs-lookup"><span data-stu-id="f00dc-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="f00dc-264">Войдите в соответствующий регистратор доменных имен.</span><span class="sxs-lookup"><span data-stu-id="f00dc-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="f00dc-265">(Для обновления DNS могут потребоваться права администратора.)</span><span class="sxs-lookup"><span data-stu-id="f00dc-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="f00dc-266">Обновите файл зоны DNS для соответствующего домена, добавив предоставленную службой Azure AD DNS-запись.</span><span class="sxs-lookup"><span data-stu-id="f00dc-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="f00dc-267">Например, чтобы добавить записи DNS для northwindcloud.com и www\.northwindcloud.com, настройте параметры DNS для корневого домена northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="f00dc-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="f00dc-268">Доменное имя можно приобрести через [портал Azure](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="f00dc-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="f00dc-269">Чтобы сопоставить настраиваемое DNS-имя с веб-приложением, его уровень [плана службы приложений](https://azure.microsoft.com/pricing/details/app-service/) должен быть платным (**Общий**, **Базовый**, **Стандартный** или **Премиум**).</span><span class="sxs-lookup"><span data-stu-id="f00dc-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="f00dc-270">Создание и сопоставление записей CNAME и A</span><span class="sxs-lookup"><span data-stu-id="f00dc-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="f00dc-271">Доступ к записям DNS с помощью поставщика домена</span><span class="sxs-lookup"><span data-stu-id="f00dc-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="f00dc-272">С помощью Azure DNS настройте пользовательское DNS-имя для веб-приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="f00dc-273">Дополнительные сведения см. в статье [Использование Azure DNS для указания параметров личного домена для службы Azure](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="f00dc-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="f00dc-274">Войдите на веб-сайт основного поставщика домена.</span><span class="sxs-lookup"><span data-stu-id="f00dc-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="f00dc-275">Найдите страницу управления записями DNS.</span><span class="sxs-lookup"><span data-stu-id="f00dc-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="f00dc-276">Каждый поставщик доменов имеет собственный интерфейс для записей DNS.</span><span class="sxs-lookup"><span data-stu-id="f00dc-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="f00dc-277">Найдите области сайта, обозначенные как **Имя домена**, **DNS** или **Name Server Management** (Управление сервером доменных имен).</span><span class="sxs-lookup"><span data-stu-id="f00dc-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="f00dc-278">Страницу управления записями DNS можно открыть из раздела **Мои домены**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="f00dc-279">Воспользуйтесь ссылкой **Файл зоны**, **Записи DNS** или **Расширенная конфигурация**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="f00dc-280">На снимке экрана ниже показан пример страницы с записями DNS:</span><span class="sxs-lookup"><span data-stu-id="f00dc-280">The following screenshot is an example of a DNS records page:</span></span>

![Пример страницы с записями DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="f00dc-282">На странице регистратора доменных имен выберите команду **"Добавить" или "Создать"** , чтобы создать запись.</span><span class="sxs-lookup"><span data-stu-id="f00dc-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="f00dc-283">Некоторые поставщики имеют разные ссылки для добавления записей различных типов.</span><span class="sxs-lookup"><span data-stu-id="f00dc-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="f00dc-284">Обратитесь к документации поставщика.</span><span class="sxs-lookup"><span data-stu-id="f00dc-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="f00dc-285">Добавьте запись CNAME, чтобы сопоставить поддомен с именем узла по умолчанию для приложения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="f00dc-286">Например, для домена www\.northwindcloud.com добавьте запись CNAME, которая сопоставляет имя с `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="f00dc-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="f00dc-287">После добавления этой записи CNAME страница управления записями DNS выглядит так:</span><span class="sxs-lookup"><span data-stu-id="f00dc-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Переход к приложению Azure на портале](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="f00dc-289">Включение сопоставления записи CNAME в приложении Azure</span><span class="sxs-lookup"><span data-stu-id="f00dc-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="f00dc-290">На новой вкладке войдите на портал Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="f00dc-291">Откройте службы приложений.</span><span class="sxs-lookup"><span data-stu-id="f00dc-291">Go to App Services.</span></span>

3. <span data-ttu-id="f00dc-292">Выберите веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="f00dc-292">Select web app.</span></span>

4. <span data-ttu-id="f00dc-293">В левой области навигации страницы приложения на портале Azure выберите **Личные домены**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="f00dc-294">Щелкните значок **+** рядом с параметром **Добавить имя узла**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="f00dc-295">Введите полное доменное имя, например `www.northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="f00dc-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="f00dc-296">Выберите **Проверить**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-296">Select **Validate**.</span></span>

8. <span data-ttu-id="f00dc-297">Если это указано отдельно, добавьте в DNS-записи регистратора доменных имен дополнительные записи других типов (обычно `A` или `TXT`).</span><span class="sxs-lookup"><span data-stu-id="f00dc-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="f00dc-298">Azure предоставит значения и типы для этих записей:</span><span class="sxs-lookup"><span data-stu-id="f00dc-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="f00dc-299">а.</span><span class="sxs-lookup"><span data-stu-id="f00dc-299">a.</span></span>  <span data-ttu-id="f00dc-300">Запись **A** для сопоставления с IP-адресом приложения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="f00dc-301">b.</span><span class="sxs-lookup"><span data-stu-id="f00dc-301">b.</span></span>  <span data-ttu-id="f00dc-302">Запись типа **TXT** для сопоставления с именем узла по умолчанию приложения, `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="f00dc-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="f00dc-303">Служба приложений использует эту запись только во время настройки для проверки того, что вы являетесь владельцем личного домена.</span><span class="sxs-lookup"><span data-stu-id="f00dc-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="f00dc-304">После проверки удалите запись типа TXT.</span><span class="sxs-lookup"><span data-stu-id="f00dc-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="f00dc-305">Выполните эту задачу на вкладке регистратора домена и повторите проверку, чтобы активировать кнопку **Добавить имя узла**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="f00dc-306">Убедитесь, что для **типа записи имени узла** выбрано значение **CNAME** (www.example.com или любой поддомен).</span><span class="sxs-lookup"><span data-stu-id="f00dc-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="f00dc-307">Выберите **Добавить имя узла**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="f00dc-308">Введите полное доменное имя, например `northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="f00dc-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="f00dc-309">Выберите **Проверить**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-309">Select **Validate**.</span></span> <span data-ttu-id="f00dc-310">Теперь активируется **кнопка добавления**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="f00dc-311">Убедитесь, что для **типа записи имени узла** выбрано значение **Запись А** (example.com).</span><span class="sxs-lookup"><span data-stu-id="f00dc-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="f00dc-312">**Добавление имени узла**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-312">**Add hostname**.</span></span>

    <span data-ttu-id="f00dc-313">Возможно, потребуется некоторое время, чтобы новое имя узла отобразилось на странице **Личные домены** вашего приложения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="f00dc-314">Попробуйте обновить браузер, чтобы обновить данные.</span><span class="sxs-lookup"><span data-stu-id="f00dc-314">Try refreshing the browser to update the data.</span></span>
  
    ![Личные домены](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="f00dc-316">В случае ошибки проверки в нижней части страницы появится соответствующее уведомление.</span><span class="sxs-lookup"><span data-stu-id="f00dc-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Ошибка проверки домена](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="f00dc-318">Описанные выше действия можно повторить для сопоставления доменного имени с подстановочным знаком (\*.northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="f00dc-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="f00dc-319">Это позволяет добавлять любые поддомены к этой службе приложений, не создавая для каждого из них отдельную запись CNAME.</span><span class="sxs-lookup"><span data-stu-id="f00dc-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="f00dc-320">Следуйте инструкциям регистратора по настройке этого параметра.</span><span class="sxs-lookup"><span data-stu-id="f00dc-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="f00dc-321">Проверка в браузере</span><span class="sxs-lookup"><span data-stu-id="f00dc-321">Test in a browser</span></span>

<span data-ttu-id="f00dc-322">Перейдите к DNS-именам, настроенным ранее (например, `northwindcloud.com` или `www.northwindcloud.com`).</span><span class="sxs-lookup"><span data-stu-id="f00dc-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="f00dc-323">Часть 3. Привязка настраиваемого SSL-сертификата</span><span class="sxs-lookup"><span data-stu-id="f00dc-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="f00dc-324">В этой части будут рассматриваться такие темы:</span><span class="sxs-lookup"><span data-stu-id="f00dc-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f00dc-325">привязывание пользовательского SSL-сертификата к Службе приложений;</span><span class="sxs-lookup"><span data-stu-id="f00dc-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="f00dc-326">принудительное использование HTTPS для приложения;</span><span class="sxs-lookup"><span data-stu-id="f00dc-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="f00dc-327">автоматизация привязки SSL-сертификата с помощью скриптов.</span><span class="sxs-lookup"><span data-stu-id="f00dc-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="f00dc-328">Если потребуется, получите SSL-сертификат для клиента на портале Azure и привяжите его к веб-приложению.</span><span class="sxs-lookup"><span data-stu-id="f00dc-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="f00dc-329">Подробные сведения см. в статье о [сертификатах службы приложений](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="f00dc-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f00dc-330">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="f00dc-330">Prerequisites</span></span>

<span data-ttu-id="f00dc-331">Для работы с этим решением:</span><span class="sxs-lookup"><span data-stu-id="f00dc-331">To complete this  solution:</span></span>

- <span data-ttu-id="f00dc-332">[Создайте приложение Службы приложений](/azure/app-service/).</span><span class="sxs-lookup"><span data-stu-id="f00dc-332">[Create an App Service app.](/azure/app-service/)</span></span>
- <span data-ttu-id="f00dc-333">[Сопоставьте настраиваемое DNS-имя с веб-приложением](/azure/app-service/app-service-web-tutorial-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="f00dc-333">[Map a custom DNS name to your web app.](/azure/app-service/app-service-web-tutorial-custom-domain)</span></span>
- <span data-ttu-id="f00dc-334">Получите SSL-сертификат из доверенного центра сертификации и подпишите запрос этим ключом.</span><span class="sxs-lookup"><span data-stu-id="f00dc-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="f00dc-335">Требования к SSL-сертификату</span><span class="sxs-lookup"><span data-stu-id="f00dc-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="f00dc-336">Чтобы сертификат можно было использовать в службе приложений, он:</span><span class="sxs-lookup"><span data-stu-id="f00dc-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="f00dc-337">должен быть подписан доверенным центром сертификации;</span><span class="sxs-lookup"><span data-stu-id="f00dc-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="f00dc-338">должен быть экспортирован в PFX-файл, защищенный паролем;</span><span class="sxs-lookup"><span data-stu-id="f00dc-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="f00dc-339">должен содержать закрытый ключ длиной не менее 2048 бит;</span><span class="sxs-lookup"><span data-stu-id="f00dc-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="f00dc-340">должен содержать все промежуточные сертификаты из цепочки сертификатов.</span><span class="sxs-lookup"><span data-stu-id="f00dc-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="f00dc-341">**Сертификаты с шифрованием на основе эллиптических кривых (ECC)** совместимы со Службой приложений, но не рассматриваются в этой статье.</span><span class="sxs-lookup"><span data-stu-id="f00dc-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="f00dc-342">Обратитесь в центр сертификации за помощью в создании сертификата ECC.</span><span class="sxs-lookup"><span data-stu-id="f00dc-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="f00dc-343">Подготовка веб-приложения</span><span class="sxs-lookup"><span data-stu-id="f00dc-343">Prepare the web app</span></span>

<span data-ttu-id="f00dc-344">Чтобы привязать SSL-сертификат к веб-приложению, ваш [план службы приложений](https://azure.microsoft.com/pricing/details/app-service/) должен относиться к категории **Базовый**, **Стандартный** или **Премиум**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="f00dc-345">Вход в Azure</span><span class="sxs-lookup"><span data-stu-id="f00dc-345">Sign in to Azure</span></span>

1. <span data-ttu-id="f00dc-346">Откройте [портал Azure](https://portal.azure.com/) и перейдите к веб-приложению.</span><span class="sxs-lookup"><span data-stu-id="f00dc-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="f00dc-347">В меню слева выберите **Службы приложений** и щелкните имя нужного приложения Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Выбор веб-приложения на портале Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="f00dc-349">Проверка ценовой категории</span><span class="sxs-lookup"><span data-stu-id="f00dc-349">Check the pricing tier</span></span>

1. <span data-ttu-id="f00dc-350">В области навигации слева на странице веб-приложения перейдите к разделу **Параметры** и выберите **Увеличить масштаб (план службы приложений)** .</span><span class="sxs-lookup"><span data-stu-id="f00dc-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Масштабирование меню в веб-приложении](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="f00dc-352">Убедитесь, что веб-приложение не относится к ценовой категории **Бесплатный** или **Общий**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="f00dc-353">Текущая категория веб-приложения выделена синей рамкой.</span><span class="sxs-lookup"><span data-stu-id="f00dc-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Проверка ценовой категории в веб-приложении](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="f00dc-355">Настраиваемые SSL-сертификаты не поддерживаются в ценовых категориях **Бесплатный** и **Общий**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="f00dc-356">Чтобы повысить уровень, выполните инструкции, приведенные в следующем разделе или на странице **Выбор ценовой категории**, а затем перейдите к разделу [Отправка и привязка SSL-сертификата](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="f00dc-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="f00dc-357">Изменение уровня плана службы приложений</span><span class="sxs-lookup"><span data-stu-id="f00dc-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="f00dc-358">Выберите одну из категорий: **Базовый**, **Стандартный** или **Премиум**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="f00dc-359">Щелкните **Выбрать**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-359">Select **Select**.</span></span>

![Выбор ценовой категории веб-приложения](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="f00dc-361">Операция будет считаться завершенной, когда появится уведомление.</span><span class="sxs-lookup"><span data-stu-id="f00dc-361">The scale operation is complete when notification is displayed.</span></span>

![Уведомление об увеличении масштаба](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="f00dc-363">Привязка сертификата SSL и объединение промежуточных сертификатов</span><span class="sxs-lookup"><span data-stu-id="f00dc-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="f00dc-364">Объедините нескольких сертификатов в цепочку.</span><span class="sxs-lookup"><span data-stu-id="f00dc-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="f00dc-365">**Откройте каждый полученный сертификат** в текстовом редакторе.</span><span class="sxs-lookup"><span data-stu-id="f00dc-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="f00dc-366">Создайте файл для объединенных сертификатов и присвойте ему имя *mergedcertificate.crt*.</span><span class="sxs-lookup"><span data-stu-id="f00dc-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="f00dc-367">В текстовом редакторе скопируйте содержимое каждого сертификата в этот файл.</span><span class="sxs-lookup"><span data-stu-id="f00dc-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="f00dc-368">Порядок сертификатов должен соответствовать порядку в цепочке сертификатов, начиная вашим сертификатом и заканчивая корневым сертификатом.</span><span class="sxs-lookup"><span data-stu-id="f00dc-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="f00dc-369">Это должно выглядеть следующим образом:</span><span class="sxs-lookup"><span data-stu-id="f00dc-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="f00dc-370">Экспорт сертификата в PFX-файл</span><span class="sxs-lookup"><span data-stu-id="f00dc-370">Export certificate to PFX</span></span>

<span data-ttu-id="f00dc-371">Экспортируйте объединенный SSL-сертификат с закрытым ключом, созданным с помощью сертификата.</span><span class="sxs-lookup"><span data-stu-id="f00dc-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="f00dc-372">Файл закрытого ключа создается с помощью OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="f00dc-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="f00dc-373">Чтобы экспортировать сертификат в формат PFX, выполните следующую команду и замените заполнители `<private-key-file>` и `<merged-certificate-file>` на путь к закрытому ключу и объединенному файлу сертификата:</span><span class="sxs-lookup"><span data-stu-id="f00dc-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="f00dc-374">Когда появится приглашение, определите пароль для последующей передачи SSL-сертификата в Службу приложений.</span><span class="sxs-lookup"><span data-stu-id="f00dc-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="f00dc-375">Если вы создали запрос на сертификат с помощью IIS или **Certreq.exe**, установите сертификат на локальный компьютер и [экспортируйте его в PFX-файл](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="f00dc-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="f00dc-376">Отправка SSL-сертификата</span><span class="sxs-lookup"><span data-stu-id="f00dc-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="f00dc-377">На панели навигации слева выберите **Параметры SSL**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="f00dc-378">Выберите **Отправить сертификат**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="f00dc-379">В разделе **PFX-файл сертификата** выберите PFX-файл.</span><span class="sxs-lookup"><span data-stu-id="f00dc-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="f00dc-380">В поле **Пароль сертификата** введите пароль, созданный при экспорте PFX-файла.</span><span class="sxs-lookup"><span data-stu-id="f00dc-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="f00dc-381">Щелкните **Отправить**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-381">Select **Upload**.</span></span>

    ![Отправка SSL-сертификата](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="f00dc-383">Когда Служба приложений завершит передачу сертификата, он появится на странице **Параметры SSL**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Параметры SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="f00dc-385">Привязка SSL-сертификата</span><span class="sxs-lookup"><span data-stu-id="f00dc-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="f00dc-386">В разделе **SSL-привязки** выберите **Добавить привязку**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="f00dc-387">Если сертификат отправлен, но не появился в списке доменных имен в раскрывающемся списке **Имя узла**, попробуйте обновить страницу браузера.</span><span class="sxs-lookup"><span data-stu-id="f00dc-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="f00dc-388">На странице **добавления привязки SSL** в раскрывающихся списках выберите доменное имя для защиты и используемый сертификат.</span><span class="sxs-lookup"><span data-stu-id="f00dc-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="f00dc-389">В разделе **Тип SSL** можно выбрать SSL на основе [**указания имени сервера (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) или IP-адреса.</span><span class="sxs-lookup"><span data-stu-id="f00dc-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="f00dc-390">**SSL на основе SNI**: вы можете добавить несколько привязок SSL на основе SNI.</span><span class="sxs-lookup"><span data-stu-id="f00dc-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="f00dc-391">Этот параметр позволяет использовать несколько SSL-сертификатов для защиты нескольких доменов с одним IP-адресом.</span><span class="sxs-lookup"><span data-stu-id="f00dc-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="f00dc-392">Большинство современных браузеров (включая Internet Explorer, Chrome, Firefox и Opera) поддерживает SNI (более подробную информацию о поддержки браузеров можно найти в статье [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication) (Указание имени сервера)).</span><span class="sxs-lookup"><span data-stu-id="f00dc-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="f00dc-393">**SSL на основе IP-адреса**: вы можете добавить только одну привязку SSL на основе IP-адреса.</span><span class="sxs-lookup"><span data-stu-id="f00dc-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="f00dc-394">Этот параметр позволяет использовать только один SSL-сертификат для защиты выделенного общедоступного IP-адреса.</span><span class="sxs-lookup"><span data-stu-id="f00dc-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="f00dc-395">Чтобы защитить несколько доменов, используйте один и тот же SSL-сертификат.</span><span class="sxs-lookup"><span data-stu-id="f00dc-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="f00dc-396">Это стандартный вариант привязки SSL.</span><span class="sxs-lookup"><span data-stu-id="f00dc-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="f00dc-397">Щелкните **Добавить привязку**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-397">Select **Add Binding**.</span></span>

    ![Добавление привязки SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="f00dc-399">Когда служба приложений завершит передачу сертификата, он появится в разделах **SSL-привязки**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Отправка после завершения привязки SSL](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="f00dc-401">Переназначение записи A для SSL на основе IP-адреса</span><span class="sxs-lookup"><span data-stu-id="f00dc-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="f00dc-402">Если вы не используете в веб-приложении SSL на основе IP-адреса, перейдите к статье [Руководство по передаче и привязыванию SSL-сертификата в Службе приложений Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="f00dc-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="f00dc-403">По умолчанию веб-приложение использует общий общедоступный IP-адрес.</span><span class="sxs-lookup"><span data-stu-id="f00dc-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="f00dc-404">После привязки сертификата с помощью SSL на основе IP-адреса Служба приложений создает выделенный IP-адрес для вашего веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="f00dc-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="f00dc-405">Когда запись A будет сопоставлена с веб-приложением, следует внести в реестр домена новый выделенный IP-адрес.</span><span class="sxs-lookup"><span data-stu-id="f00dc-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="f00dc-406">Он появится на странице **Личный домен**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="f00dc-407">Скопируйте этот [IP-адрес](/azure/app-service/app-service-web-tutorial-custom-domain), а затем измените его в [записи A](/azure/app-service/app-service-web-tutorial-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="f00dc-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="f00dc-408">Тестирование HTTPS</span><span class="sxs-lookup"><span data-stu-id="f00dc-408">Test HTTPS</span></span>

<span data-ttu-id="f00dc-409">В разных браузерах перейдите к `https://<your.custom.domain>`, чтобы убедиться, что веб-приложение обслуживается.</span><span class="sxs-lookup"><span data-stu-id="f00dc-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Переход к веб-приложению](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="f00dc-411">Если возникнет ошибка проверки сертификата, это может быть связано с использованием самозаверяющего сертификата или с пропуском промежуточного сертификата при экспорте в PFX-файл.</span><span class="sxs-lookup"><span data-stu-id="f00dc-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="f00dc-412">Принудительное использование HTTPS</span><span class="sxs-lookup"><span data-stu-id="f00dc-412">Enforce HTTPS</span></span>

<span data-ttu-id="f00dc-413">По умолчанию любой пользователь может получить доступ к вашему веб-приложению по HTTP.</span><span class="sxs-lookup"><span data-stu-id="f00dc-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="f00dc-414">Вы можете перенаправлять все HTTP-запросы на HTTPS-порт.</span><span class="sxs-lookup"><span data-stu-id="f00dc-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="f00dc-415">На странице веб-приложения выберите **Параметры SL**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="f00dc-416">Затем в окне **Только HTTPS** выберите **ВКЛ**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Принудительное использование HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="f00dc-418">По завершении операции перейдите по любому из URL-адресов HTTP, которые указывают на ваше приложение.</span><span class="sxs-lookup"><span data-stu-id="f00dc-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="f00dc-419">Пример:</span><span class="sxs-lookup"><span data-stu-id="f00dc-419">For example:</span></span>

- <span data-ttu-id="f00dc-420"> https://<app_name>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="f00dc-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="f00dc-421">Принудительное применение TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="f00dc-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="f00dc-422">Приложение по умолчанию разрешает применение [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) версии 1.0, которая больше не считается безопасной в соответствии с отраслевыми стандартами, такими как [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard).</span><span class="sxs-lookup"><span data-stu-id="f00dc-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="f00dc-423">Чтобы принудительно применить TLS более поздней версии, выполните следующие инструкции:</span><span class="sxs-lookup"><span data-stu-id="f00dc-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="f00dc-424">На странице веб-приложения в области слева выберите **Параметры SSL**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="f00dc-425">Затем в разделе **Версия TLS** выберите минимальную требуемую версию TLS.</span><span class="sxs-lookup"><span data-stu-id="f00dc-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Принудительное использование TLS 1.1 или 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="f00dc-427">Создание профиля диспетчера трафика</span><span class="sxs-lookup"><span data-stu-id="f00dc-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="f00dc-428">Последовательно выберите **Создать ресурс** > **Сети** > **Профиль диспетчера трафика** > **Создать**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="f00dc-429">В области **Создание профиля диспетчера трафика** сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="f00dc-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="f00dc-430">В поле **Имя** укажите имя профиля.</span><span class="sxs-lookup"><span data-stu-id="f00dc-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="f00dc-431">Оно должно быть уникальным в пределах зоны trafficmanager.net. В результате будет создано DNS-имя trafficmanager.net для доступа к профилю диспетчера трафика.</span><span class="sxs-lookup"><span data-stu-id="f00dc-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="f00dc-432">В списке **Метод маршрутизации** выберите **метод географической маршрутизации**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="f00dc-433">В списке **Подписка** выберите подписку, в которой нужно создать профиль.</span><span class="sxs-lookup"><span data-stu-id="f00dc-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="f00dc-434">В разделе **Группа ресурсов** создайте группу ресурсов, в которую следует поместить этот профиль.</span><span class="sxs-lookup"><span data-stu-id="f00dc-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="f00dc-435">Из списка **Расположение группы ресурсов** выберите расположение группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="f00dc-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="f00dc-436">Этот параметр задает расположение группы ресурсов и не влияет на профиль диспетчера трафика, который развернут глобально.</span><span class="sxs-lookup"><span data-stu-id="f00dc-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="f00dc-437">Нажмите кнопку **создания**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-437">Select **Create**.</span></span>

    7. <span data-ttu-id="f00dc-438">Когда глобальное развертывание профиля диспетчера трафика завершится, он появится в соответствующей группе ресурсов как один из ресурсов.</span><span class="sxs-lookup"><span data-stu-id="f00dc-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Профиль диспетчера трафика в группе ресурсов](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="f00dc-440">Добавление конечных точек диспетчера трафика</span><span class="sxs-lookup"><span data-stu-id="f00dc-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="f00dc-441">На панели поиска на портале выполните поиск по имени **профиля диспетчера трафика**, созданного в предыдущем разделе, и выберите нужный профиль в отображаемых результатах.</span><span class="sxs-lookup"><span data-stu-id="f00dc-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="f00dc-442">В колонке **Профиль диспетчера трафика** в разделе **Параметры** щелкните **Конечные точки**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="f00dc-443">Выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-443">Select **Add**.</span></span>

4. <span data-ttu-id="f00dc-444">Добавьте конечную точку Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="f00dc-445">В поле **Тип** выберите **Внешняя конечная точка**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="f00dc-446">Укажите **имя** для этой конечной точки, желательно имя Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="f00dc-447">В качестве полного доменного имени домена (**FQDN**) укажите внешний URL-адрес веб-приложения Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="f00dc-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="f00dc-448">В разделе "Географическое сопоставление" выберите регион или континент, где расположен ресурс</span><span class="sxs-lookup"><span data-stu-id="f00dc-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="f00dc-449">(например, **Европа**).</span><span class="sxs-lookup"><span data-stu-id="f00dc-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="f00dc-450">В раскрывающемся списке "Страна/Регион" выберите страну, для которой будет применяться эта конечная точка</span><span class="sxs-lookup"><span data-stu-id="f00dc-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="f00dc-451">(например, **Германия**).</span><span class="sxs-lookup"><span data-stu-id="f00dc-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="f00dc-452">Оставьте флажок **Добавить как отключенный** снятым.</span><span class="sxs-lookup"><span data-stu-id="f00dc-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="f00dc-453">Щелкните **ОК**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-453">Select **OK**.</span></span>

12. <span data-ttu-id="f00dc-454">Добавление конечной точки Azure.</span><span class="sxs-lookup"><span data-stu-id="f00dc-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="f00dc-455">В раскрывающемся списке **Тип** выберите **Конечная точка Azure**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="f00dc-456">Укажите **имя** конечной точки.</span><span class="sxs-lookup"><span data-stu-id="f00dc-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="f00dc-457">В раскрывающемся списке **Тип целевого ресурса** выберите **Служба приложений**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="f00dc-458">В разделе **Целевой ресурс** щелкните **Выберите службу приложений**, чтобы отобразился список веб-приложений, размещенных в этой подписке.</span><span class="sxs-lookup"><span data-stu-id="f00dc-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="f00dc-459">В колонке **Ресурс** выберите службу приложений, которую вы хотите добавить в качестве первой конечной точки.</span><span class="sxs-lookup"><span data-stu-id="f00dc-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="f00dc-460">В разделе "Географическое сопоставление" выберите регион или континент, где расположен ресурс</span><span class="sxs-lookup"><span data-stu-id="f00dc-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="f00dc-461">(например, **Северная Америка, Центральная Америка, Карибские о-ва**).</span><span class="sxs-lookup"><span data-stu-id="f00dc-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="f00dc-462">В раскрывающемся списке "Страна/Регион" оставьте пустое значение, чтобы выбрать полностью весь регион, указанный выше.</span><span class="sxs-lookup"><span data-stu-id="f00dc-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="f00dc-463">Оставьте флажок **Добавить как отключенный** снятым.</span><span class="sxs-lookup"><span data-stu-id="f00dc-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="f00dc-464">Щелкните **ОК**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="f00dc-465">Создайте по меньшей мере одну конечную точку с географической областью All (World) (Все (мир)), которая будет конечной точкой по умолчанию для этого ресурса.</span><span class="sxs-lookup"><span data-stu-id="f00dc-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="f00dc-466">Добавленные конечные точки отобразятся в колонке **Профиль диспетчера трафика** с состоянием **В сети**.</span><span class="sxs-lookup"><span data-stu-id="f00dc-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Состояние конечной точки в профиле диспетчера трафика](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="f00dc-468">Предприятия мирового уровня используют возможности географического распределения Azure</span><span class="sxs-lookup"><span data-stu-id="f00dc-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="f00dc-469">Направляя трафик в диспетчер трафика Azure и конечные точки с привязкой к географическим регионам, предприятия мирового уровня обеспечивают соблюдение требований регионального законодательства, соответствие политикам и обеспечивают безопасность данных. Все эти факторы очень важны для успешной работы локальных и зарубежных предприятий и филиалов.</span><span class="sxs-lookup"><span data-stu-id="f00dc-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f00dc-470">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="f00dc-470">Next steps</span></span>

- <span data-ttu-id="f00dc-471">Дополнительные сведения о шаблонах для облака Azure см. в статье [Конструктивные шаблоны облачных решений](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="f00dc-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
