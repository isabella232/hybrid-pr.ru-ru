---
title: Шаблон гибридного ретранслятора в Azure и Azure Stack Hub
description: Используйте шаблон гибридного ретранслятора в Azure и Azure Stack Hub, чтобы подключиться к пограничным ресурсам, защищенным брандмауэрами.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911151"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="fd1b3-103">Шаблон гибридного ретранслятора</span><span class="sxs-lookup"><span data-stu-id="fd1b3-103">Hybrid relay pattern</span></span>

<span data-ttu-id="fd1b3-104">Узнайте, как подключиться к граничным ресурсам или устройствам, защищенным брандмауэрами, с помощью шаблона гибридной трансляции и Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="fd1b3-105">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="fd1b3-105">Context and problem</span></span>

<span data-ttu-id="fd1b3-106">Пограничные устройства часто находятся за корпоративным брандмауэром или устройством NAT.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="fd1b3-107">Хотя они защищены, у них может не быть возможности обмениваться данными с общедоступным облаком или пограничными устройствами в других корпоративных сетях.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="fd1b3-108">Возможно, понадобится предоставить пользователям безопасный доступ к определенным портам и функциональным возможностям в общедоступном облаке.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="fd1b3-109">Решение</span><span class="sxs-lookup"><span data-stu-id="fd1b3-109">Solution</span></span>

<span data-ttu-id="fd1b3-110">Шаблон гибридного ретранслятора использует Azure Relay для установления туннеля WebSocket между двумя конечными точками, которые не могут напрямую обмениваться данными.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="fd1b3-111">Устройства за пределами локальной среды, которые необходимо подключить к локальной конечной точке, будут подключены к конечной точке в общедоступном облаке.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="fd1b3-112">Эта конечная точка перенаправит трафик на стандартные маршруты по безопасному каналу.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="fd1b3-113">Конечная точка в локальной среде получает трафик и направляет его в нужное место назначения.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![Архитектура шаблона решения для гибридного ретранслятора](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="fd1b3-115">Шаблон гибридного ретранслятора работает следующим образом:</span><span class="sxs-lookup"><span data-stu-id="fd1b3-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="fd1b3-116">Устройство подключается к виртуальной машине в Azure по заранее определенному порту.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="fd1b3-117">Трафик перенаправляется в Azure Relay в Azure.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="fd1b3-118">Виртуальная машина в концентраторе Azure Stack, которая уже установила длительное подключение к Azure Relay, получает трафик и перенаправляет его в место назначения.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="fd1b3-119">Локальная служба или конечная точка обрабатывает запрос.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="fd1b3-120">Components</span><span class="sxs-lookup"><span data-stu-id="fd1b3-120">Components</span></span>

<span data-ttu-id="fd1b3-121">Это решение использует следующие компоненты.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-121">This solution uses the following components:</span></span>

| <span data-ttu-id="fd1b3-122">Уровень</span><span class="sxs-lookup"><span data-stu-id="fd1b3-122">Layer</span></span> | <span data-ttu-id="fd1b3-123">Компонент</span><span class="sxs-lookup"><span data-stu-id="fd1b3-123">Component</span></span> | <span data-ttu-id="fd1b3-124">Описание</span><span class="sxs-lookup"><span data-stu-id="fd1b3-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="fd1b3-125">Azure</span><span class="sxs-lookup"><span data-stu-id="fd1b3-125">Azure</span></span> | <span data-ttu-id="fd1b3-126">Azure</span><span class="sxs-lookup"><span data-stu-id="fd1b3-126">Azure VM</span></span> | <span data-ttu-id="fd1b3-127">Виртуальная машина Azure предоставляет общедоступную конечную точку для локального ресурса.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="fd1b3-128">Ретранслятор Azure</span><span class="sxs-lookup"><span data-stu-id="fd1b3-128">Azure Relay</span></span> | <span data-ttu-id="fd1b3-129">[Azure Relay](/azure/azure-relay/) предоставляет инфраструктуру для обслуживания туннеля и подключения между виртуальной машиной Azure и виртуальной машиной центра Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="fd1b3-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd1b3-130">Azure Stack Hub</span></span> | <span data-ttu-id="fd1b3-131">Службы вычислений</span><span class="sxs-lookup"><span data-stu-id="fd1b3-131">Compute</span></span> | <span data-ttu-id="fd1b3-132">Виртуальная машина Azure Stack Hub предоставляет серверную часть туннеля гибридного ретранслятора.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="fd1b3-133">Память</span><span class="sxs-lookup"><span data-stu-id="fd1b3-133">Storage</span></span> | <span data-ttu-id="fd1b3-134">Кластер обработчика AKS, развернутый в Azure Stack Hub, предоставляет масштабируемый отказоустойчивый механизм для запуска контейнера API "Распознавание лиц".</span><span class="sxs-lookup"><span data-stu-id="fd1b3-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="fd1b3-135">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="fd1b3-135">Issues and considerations</span></span>

<span data-ttu-id="fd1b3-136">Во время выбора варианта реализации этого решения необходимо учитывать приведенные ниже моменты.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="fd1b3-137">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="fd1b3-137">Scalability</span></span>

<span data-ttu-id="fd1b3-138">Этот шаблон разрешает только сопоставление портов 1:1 в клиенте и на сервере.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="fd1b3-139">Например, если порт 80 туннелирован для одной службы в конечной точке Azure, его нельзя использовать для другой службы.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="fd1b3-140">Сопоставления портов следует планировать соответствующим образом.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="fd1b3-141">Azure Relay и виртуальные машины должны быть соответствующим образом масштабированы для управления трафиком.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="fd1b3-142">Доступность</span><span class="sxs-lookup"><span data-stu-id="fd1b3-142">Availability</span></span>

<span data-ttu-id="fd1b3-143">Эти туннели и соединения не являются избыточными.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="fd1b3-144">Для обеспечения высокой доступности может потребоваться реализовать код проверки ошибок.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="fd1b3-145">Кроме того, можно использовать пул виртуальных машин, подключенных к Azure Relay, за пределами подсистемы балансировки нагрузки.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="fd1b3-146">Управляемость</span><span class="sxs-lookup"><span data-stu-id="fd1b3-146">Manageability</span></span>

<span data-ttu-id="fd1b3-147">Это решение может масштабироваться на множество устройств и расположений, что может оказаться неудобным.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="fd1b3-148">Службы Интернета вещей Azure могут автоматически переводить новые расположения и устройства в режим подключения и обновлять их.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="fd1b3-149">Безопасность</span><span class="sxs-lookup"><span data-stu-id="fd1b3-149">Security</span></span>

<span data-ttu-id="fd1b3-150">Этот шаблон, как показано ниже, обеспечивает свободный доступ к порту на внутреннем устройстве с пограничного устройства.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="fd1b3-151">Рассмотрите возможность добавления механизма проверки подлинности в службу на внутреннем устройстве или перед конечной точкой гибридного ретранслятора.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fd1b3-152">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="fd1b3-152">Next steps</span></span>

<span data-ttu-id="fd1b3-153">Дополнительные сведения по темам, описанным в этой статье:</span><span class="sxs-lookup"><span data-stu-id="fd1b3-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="fd1b3-154">В этом шаблоне используется Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="fd1b3-155">Дополнительные сведения см. в [документации по Azure Relay](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="fd1b3-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="fd1b3-156">См. [рекомендации по проектированию гибридных приложений и ответы на дополнительные вопросы](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="fd1b3-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="fd1b3-157">См. сведения обо [всех продуктах и решениях Azure Stack](/azure-stack).</span><span class="sxs-lookup"><span data-stu-id="fd1b3-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="fd1b3-158">Когда вы будете готовы протестировать пример решения, продолжите работу с [руководством по развертыванию решения гибридного ретранслятора](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="fd1b3-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="fd1b3-159">В этом руководстве содержатся пошаговые инструкции по развертыванию и тестированию компонентов.</span><span class="sxs-lookup"><span data-stu-id="fd1b3-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>