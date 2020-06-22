---
title: Шаблон решения для масштабирования в разных облаках в Azure Stack Hub
description: Узнайте, как создать масштабируемое приложение в Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911137"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="0e41b-103">Шаблон масштабирования в нескольких облаках</span><span class="sxs-lookup"><span data-stu-id="0e41b-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="0e41b-104">Вы можете автоматически добавлять ресурсы в существующее приложение в соответствии с увеличением нагрузки.</span><span class="sxs-lookup"><span data-stu-id="0e41b-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="0e41b-105">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="0e41b-105">Context and problem</span></span>

<span data-ttu-id="0e41b-106">Приложение не может увеличить емкость, чтобы удовлетворить непредвиденный рост потребностей в ресурсах.</span><span class="sxs-lookup"><span data-stu-id="0e41b-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="0e41b-107">Недостаток масштабируемости приводит к тому, что пользователи не могут обратиться к приложению в периоды пиковой нагрузки.</span><span class="sxs-lookup"><span data-stu-id="0e41b-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="0e41b-108">Приложение может обслуживать фиксированное число пользователей.</span><span class="sxs-lookup"><span data-stu-id="0e41b-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="0e41b-109">Международным компаниям требуются безопасные, надежные и доступные облачные приложения.</span><span class="sxs-lookup"><span data-stu-id="0e41b-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="0e41b-110">Очень важно своевременно реагировать на увеличение нагрузки и использовать для этого подходящую инфраструктуру.</span><span class="sxs-lookup"><span data-stu-id="0e41b-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="0e41b-111">Предприятия прилагают много усилий, чтобы сбалансировать затраты и обслуживание с обеспечением защиты, хранения бизнес-данных и их доступности в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="0e41b-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="0e41b-112">Возможно, вы не сможете запускать приложение в общедоступном облаке.</span><span class="sxs-lookup"><span data-stu-id="0e41b-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="0e41b-113">Тем не менее для бизнеса экономически нецелесообразно поддерживать требуемую емкость в своей локальной среде для обработки пиков спроса на приложение.</span><span class="sxs-lookup"><span data-stu-id="0e41b-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="0e41b-114">Благодаря этому шаблону можно воспользоваться эластичностью общедоступного облака в локальном решении.</span><span class="sxs-lookup"><span data-stu-id="0e41b-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="0e41b-115">Решение</span><span class="sxs-lookup"><span data-stu-id="0e41b-115">Solution</span></span>

<span data-ttu-id="0e41b-116">Шаблон масштабирования в нескольких облаках расширяет возможности приложения, расположенного в локальном облаке, с помощью ресурсов общедоступного облака.</span><span class="sxs-lookup"><span data-stu-id="0e41b-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="0e41b-117">Шаблон активируется при увеличении или уменьшении нагрузки, соответственно добавляя или удаляя ресурсы в облаке.</span><span class="sxs-lookup"><span data-stu-id="0e41b-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="0e41b-118">Эти ресурсы обеспечивают избыточность, высокую доступность и географическую маршрутизацию.</span><span class="sxs-lookup"><span data-stu-id="0e41b-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Шаблон масштабирования в нескольких облаках](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="0e41b-120">Этот шаблон применяется только к компонентам приложения без отслеживания состояния.</span><span class="sxs-lookup"><span data-stu-id="0e41b-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="0e41b-121">Components</span><span class="sxs-lookup"><span data-stu-id="0e41b-121">Components</span></span>

<span data-ttu-id="0e41b-122">Шаблон масштабирования в разных облаках состоит из следующих компонентов.</span><span class="sxs-lookup"><span data-stu-id="0e41b-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="0e41b-123">За пределами облака</span><span class="sxs-lookup"><span data-stu-id="0e41b-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="0e41b-124">Диспетчер трафика</span><span class="sxs-lookup"><span data-stu-id="0e41b-124">Traffic Manager</span></span>

<span data-ttu-id="0e41b-125">На схеме он находится за пределами группы общедоступного облака, но ему нужна возможность координировать трафик как в локальном центре обработки данных, так и в общедоступном облаке.</span><span class="sxs-lookup"><span data-stu-id="0e41b-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="0e41b-126">Подсистема балансировки обеспечивает высокую доступность приложения, отслеживая конечные точки и обеспечивая при необходимости перераспределение отработки отказа.</span><span class="sxs-lookup"><span data-stu-id="0e41b-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="0e41b-127">Служба доменных имен (DNS)</span><span class="sxs-lookup"><span data-stu-id="0e41b-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="0e41b-128">Служба доменных имен, или DNS, отвечает за преобразование (или разрешение) имени веб-сайта или службы в IP-адрес.</span><span class="sxs-lookup"><span data-stu-id="0e41b-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="0e41b-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="0e41b-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="0e41b-130">Размещенный сервер сборки</span><span class="sxs-lookup"><span data-stu-id="0e41b-130">Hosted build server</span></span>

<span data-ttu-id="0e41b-131">Среда для размещения конвейера сборки.</span><span class="sxs-lookup"><span data-stu-id="0e41b-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="0e41b-132">Ресурсы приложения</span><span class="sxs-lookup"><span data-stu-id="0e41b-132">App resources</span></span>

<span data-ttu-id="0e41b-133">Ресурсы приложения, поддерживающие горизонтальное уменьшение и увеличение масштаба, такие как масштабируемые наборы виртуальных машин и контейнеры.</span><span class="sxs-lookup"><span data-stu-id="0e41b-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="0e41b-134">Имя личного домена</span><span class="sxs-lookup"><span data-stu-id="0e41b-134">Custom domain name</span></span>

<span data-ttu-id="0e41b-135">Используйте имя личного домена для стандартной маски запросов маршрутизации.</span><span class="sxs-lookup"><span data-stu-id="0e41b-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="0e41b-136">Общедоступные IP-адреса</span><span class="sxs-lookup"><span data-stu-id="0e41b-136">Public IP addresses</span></span>

<span data-ttu-id="0e41b-137">Общедоступные IP-адреса используются для передачи входящего трафика через Диспетчер трафика в конечную точку ресурсов общедоступного облачного приложения.</span><span class="sxs-lookup"><span data-stu-id="0e41b-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="0e41b-138">Локальное облако</span><span class="sxs-lookup"><span data-stu-id="0e41b-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="0e41b-139">Размещенный сервер сборки</span><span class="sxs-lookup"><span data-stu-id="0e41b-139">Hosted build server</span></span>

<span data-ttu-id="0e41b-140">Среда для размещения конвейера сборки.</span><span class="sxs-lookup"><span data-stu-id="0e41b-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="0e41b-141">Ресурсы приложения</span><span class="sxs-lookup"><span data-stu-id="0e41b-141">App resources</span></span>

<span data-ttu-id="0e41b-142">Ресурсы приложения, поддерживающие горизонтальное уменьшение и увеличение масштаба, такие как масштабируемые наборы виртуальных машин и контейнеры.</span><span class="sxs-lookup"><span data-stu-id="0e41b-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="0e41b-143">Имя личного домена</span><span class="sxs-lookup"><span data-stu-id="0e41b-143">Custom domain name</span></span>

<span data-ttu-id="0e41b-144">Используйте имя личного домена для стандартной маски запросов маршрутизации.</span><span class="sxs-lookup"><span data-stu-id="0e41b-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="0e41b-145">Общедоступные IP-адреса</span><span class="sxs-lookup"><span data-stu-id="0e41b-145">Public IP addresses</span></span>

<span data-ttu-id="0e41b-146">Общедоступные IP-адреса используются для передачи входящего трафика через Диспетчер трафика в конечную точку ресурсов общедоступного облачного приложения.</span><span class="sxs-lookup"><span data-stu-id="0e41b-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="0e41b-147">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="0e41b-147">Issues and considerations</span></span>

<span data-ttu-id="0e41b-148">При принятии решения о реализации этого шаблона необходимо учитывать следующие моменты.</span><span class="sxs-lookup"><span data-stu-id="0e41b-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="0e41b-149">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="0e41b-149">Scalability</span></span>

<span data-ttu-id="0e41b-150">Ключевым компонентом масштабирования в нескольких облаках является возможность предоставления масштабирования по запросу.</span><span class="sxs-lookup"><span data-stu-id="0e41b-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="0e41b-151">Масштабирование должно осуществляться между общедоступной и локальной облачными инфраструктурами и обеспечивать согласованное надежное обслуживание по запросу.</span><span class="sxs-lookup"><span data-stu-id="0e41b-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="0e41b-152">Доступность</span><span class="sxs-lookup"><span data-stu-id="0e41b-152">Availability</span></span>

<span data-ttu-id="0e41b-153">Убедитесь, что локально развернутые приложения настроены для обеспечения высокой доступности с помощью конфигурации локального оборудования и развертывания программного обеспечения.</span><span class="sxs-lookup"><span data-stu-id="0e41b-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="0e41b-154">Управляемость</span><span class="sxs-lookup"><span data-stu-id="0e41b-154">Manageability</span></span>

<span data-ttu-id="0e41b-155">Шаблон для нескольких облаков обеспечивают простое управление и привычный интерфейс между средами.</span><span class="sxs-lookup"><span data-stu-id="0e41b-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="0e41b-156">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="0e41b-156">When to use this pattern</span></span>

<span data-ttu-id="0e41b-157">Используйте этот шаблон в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="0e41b-157">Use this pattern:</span></span>

- <span data-ttu-id="0e41b-158">Если вам нужно увеличить емкость приложения при непредвиденных или периодических увеличениях нагрузки.</span><span class="sxs-lookup"><span data-stu-id="0e41b-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="0e41b-159">Если вы не хотите инвестировать средства в ресурсы, которые будут использоваться только во время пиковых нагрузок.</span><span class="sxs-lookup"><span data-stu-id="0e41b-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="0e41b-160">Платите только за те ресурсы, которые используете.</span><span class="sxs-lookup"><span data-stu-id="0e41b-160">Pay for what you use.</span></span>

<span data-ttu-id="0e41b-161">Этот шаблон не рекомендуется использовать в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="0e41b-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="0e41b-162">Для решения требуется подключение пользователей через Интернет.</span><span class="sxs-lookup"><span data-stu-id="0e41b-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="0e41b-163">Для вашего предприятия действуют местные нормы, требующие, чтобы исходное подключение поступало от вызова на месте.</span><span class="sxs-lookup"><span data-stu-id="0e41b-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="0e41b-164">В сети возникают обычные узкие места, которые ограничивают эффективность масштабирования.</span><span class="sxs-lookup"><span data-stu-id="0e41b-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="0e41b-165">Ваша среда изолирована от Интернета и ее нельзя подключать к общедоступному облаку.</span><span class="sxs-lookup"><span data-stu-id="0e41b-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="0e41b-166">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="0e41b-166">Next steps</span></span>

<span data-ttu-id="0e41b-167">Дополнительные сведения по темам, описанным в этой статье:</span><span class="sxs-lookup"><span data-stu-id="0e41b-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="0e41b-168">Дополнительные сведения о работе балансировщика нагрузки трафика на основе DNS см. в статье с [общими сведениями о диспетчере трафика Azure](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="0e41b-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="0e41b-169">См. [рекомендации по проектированию гибридных приложений и ответы на дополнительные вопросы](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="0e41b-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="0e41b-170">См. сведения обо [всех продуктах и решениях Azure Stack](/azure-stack).</span><span class="sxs-lookup"><span data-stu-id="0e41b-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="0e41b-171">Когда вы будете готовы протестировать пример решения, продолжите работу с [руководством по развертыванию решения масштабирования в нескольких облаках](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="0e41b-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="0e41b-172">В этом руководстве содержатся пошаговые инструкции по развертыванию и тестированию компонентов.</span><span class="sxs-lookup"><span data-stu-id="0e41b-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="0e41b-173">Вы узнали, как создать решение для работы в нескольких облаках с активируемым вручную процессом переключения с веб-приложения, размещенного в Azure Stack Hub, на веб-приложение, размещенное в Azure.</span><span class="sxs-lookup"><span data-stu-id="0e41b-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="0e41b-174">Вы также узнаете, как использовать автоматическое масштабирование с помощью диспетчера трафика, обеспечивая гибкую и масштабируемую облачную служебную программу для работы под нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="0e41b-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
