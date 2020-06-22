---
title: Шаблон геораспределенного приложения в Azure Stack Hub
description: Сведения о шаблоне геораспределенного приложения для интеллектуальной границы с использованием Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910885"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="ea510-103">Шаблон геораспределенного приложения</span><span class="sxs-lookup"><span data-stu-id="ea510-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="ea510-104">Узнайте, как предоставлять конечные точки приложений в нескольких регионах и маршрутизировать пользовательский трафик на основе требований к расположению и соответствию.</span><span class="sxs-lookup"><span data-stu-id="ea510-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="ea510-105">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="ea510-105">Context and problem</span></span>

<span data-ttu-id="ea510-106">Организации с широким географическим охватом прилагают много усилий, чтобы обеспечить безопасное и надежное распространение данных и доступа к ним, соблюдая при этом требуемые уровни безопасности, соответствия и производительности для каждого пользователя, расположения и устройства в пределах границ.</span><span class="sxs-lookup"><span data-stu-id="ea510-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="ea510-107">Решение</span><span class="sxs-lookup"><span data-stu-id="ea510-107">Solution</span></span>

<span data-ttu-id="ea510-108">Шаблон географической маршрутизации трафика Azure Stack Hub (геораспределенные приложения) позволяет перенаправлять трафик в определенные конечные точки на основе различных метрик.</span><span class="sxs-lookup"><span data-stu-id="ea510-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="ea510-109">Можно создать диспетчер трафика с географической маршрутизацией и конфигурацией конечных точек, который направляет данные к конечным точкам с учетом региональных требований, корпоративных и международных стандартов и ваших потребностей в данных.</span><span class="sxs-lookup"><span data-stu-id="ea510-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Геораспределенный шаблон](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="ea510-111">Components</span><span class="sxs-lookup"><span data-stu-id="ea510-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="ea510-112">За пределами облака</span><span class="sxs-lookup"><span data-stu-id="ea510-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="ea510-113">Диспетчер трафика</span><span class="sxs-lookup"><span data-stu-id="ea510-113">Traffic Manager</span></span>

<span data-ttu-id="ea510-114">На схеме Диспетчер трафика находится за пределами общедоступного облака, но ему нужно координировать трафик как в локальном центре обработки данных, так и в общедоступном облаке.</span><span class="sxs-lookup"><span data-stu-id="ea510-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="ea510-115">Подсистема балансировки направляет трафик в географические расположения.</span><span class="sxs-lookup"><span data-stu-id="ea510-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="ea510-116">Служба доменных имен (DNS)</span><span class="sxs-lookup"><span data-stu-id="ea510-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="ea510-117">Служба доменных имен, или DNS, отвечает за преобразование (или разрешение) имени веб-сайта или службы в IP-адрес.</span><span class="sxs-lookup"><span data-stu-id="ea510-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="ea510-118">Общедоступное облако</span><span class="sxs-lookup"><span data-stu-id="ea510-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="ea510-119">Конечная точка облака</span><span class="sxs-lookup"><span data-stu-id="ea510-119">Cloud Endpoint</span></span>

<span data-ttu-id="ea510-120">Общедоступные IP-адреса используются для передачи входящего трафика через Диспетчер трафика в конечную точку ресурсов общедоступного облачного приложения.</span><span class="sxs-lookup"><span data-stu-id="ea510-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="ea510-121">Локальные облака</span><span class="sxs-lookup"><span data-stu-id="ea510-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="ea510-122">Локальная конечная точка</span><span class="sxs-lookup"><span data-stu-id="ea510-122">Local endpoint</span></span>

<span data-ttu-id="ea510-123">Общедоступные IP-адреса используются для передачи входящего трафика через Диспетчер трафика в конечную точку ресурсов общедоступного облачного приложения.</span><span class="sxs-lookup"><span data-stu-id="ea510-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="ea510-124">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="ea510-124">Issues and considerations</span></span>

<span data-ttu-id="ea510-125">При принятии решения о реализации этого шаблона необходимо учитывать следующие моменты.</span><span class="sxs-lookup"><span data-stu-id="ea510-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="ea510-126">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="ea510-126">Scalability</span></span>

<span data-ttu-id="ea510-127">При увеличении объема трафика шаблон обеспечивает географическую маршрутизацию трафика, а не масштабирование.</span><span class="sxs-lookup"><span data-stu-id="ea510-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="ea510-128">Тем не менее этот шаблон можно сочетать с другими решениями Azure и локальными решениями.</span><span class="sxs-lookup"><span data-stu-id="ea510-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="ea510-129">Например, этот шаблон можно использовать с шаблоном масштабирования между облаками.</span><span class="sxs-lookup"><span data-stu-id="ea510-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="ea510-130">Доступность</span><span class="sxs-lookup"><span data-stu-id="ea510-130">Availability</span></span>

<span data-ttu-id="ea510-131">Убедитесь, что локально развернутые приложения настроены для обеспечения высокой доступности с помощью конфигурации локального оборудования и развертывания программного обеспечения.</span><span class="sxs-lookup"><span data-stu-id="ea510-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="ea510-132">Управляемость</span><span class="sxs-lookup"><span data-stu-id="ea510-132">Manageability</span></span>

<span data-ttu-id="ea510-133">Данный шаблон обеспечивает простое управление и привычный интерфейс в разных средах.</span><span class="sxs-lookup"><span data-stu-id="ea510-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="ea510-134">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="ea510-134">When to use this pattern</span></span>

- <span data-ttu-id="ea510-135">У моей организации есть международные филиалы, для которых применяются региональные политики безопасности и распространения.</span><span class="sxs-lookup"><span data-stu-id="ea510-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="ea510-136">Каждый из офисов организации собирает данные о сотрудниках, бизнесе и предприятии, для чего нужно создавать отчеты с соблюдением региональных требований в соответствующем часовом поясе.</span><span class="sxs-lookup"><span data-stu-id="ea510-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="ea510-137">Высокой степени масштабируемости в сценариях с высокой нагрузкой можно добиться за счет горизонтального масштабирования, когда несколько приложений развертываются в одном или нескольких регионах.</span><span class="sxs-lookup"><span data-stu-id="ea510-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="ea510-138">Приложения должны быть высокодоступными и реагировать на клиентские запросы даже в случае сбоя в одном из регионов.</span><span class="sxs-lookup"><span data-stu-id="ea510-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ea510-139">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="ea510-139">Next steps</span></span>

<span data-ttu-id="ea510-140">Дополнительные сведения по темам, описанным в этой статье:</span><span class="sxs-lookup"><span data-stu-id="ea510-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="ea510-141">Дополнительные сведения о работе балансировщика нагрузки трафика на основе DNS см. в статье с [общими сведениями о диспетчере трафика Azure](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="ea510-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="ea510-142">См. [рекомендации по проектированию гибридных приложений и ответы на дополнительные вопросы](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="ea510-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="ea510-143">См. сведения обо [всех продуктах и решениях Azure Stack](/azure-stack).</span><span class="sxs-lookup"><span data-stu-id="ea510-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="ea510-144">Когда вы будете готовы протестировать пример решения, продолжите работу с [руководством по развертыванию решения для геораспределенного приложения](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="ea510-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="ea510-145">В этом руководстве содержатся пошаговые инструкции по развертыванию и тестированию компонентов.</span><span class="sxs-lookup"><span data-stu-id="ea510-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="ea510-146">Узнайте, как в шаблоне географически распределенного приложения направлять трафик в определенные конечные точки на основе различных метрик.</span><span class="sxs-lookup"><span data-stu-id="ea510-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="ea510-147">Создав профиль диспетчера трафика с маршрутизацией по географическим данным и конфигурацией конечных точек, можно направлять данные к определенным конечным точкам с учетом региональных требований, корпоративных и международных стандартов и ваших потребностей по обработке данных.</span><span class="sxs-lookup"><span data-stu-id="ea510-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
