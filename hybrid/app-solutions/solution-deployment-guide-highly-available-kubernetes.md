---
title: Развертывание высокодоступного кластера Kubernetes в Azure Stack Hub
description: Узнайте, как развертывать решение кластера Kubernetes, чтобы обеспечить высокий уровень доступности, с помощью Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911925"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="46710-103">Развертывание кластера Kubernetes с высоким уровнем доступности в Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="46710-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="46710-104">В этой статье показано, как создать высокодоступную среду кластера Kubernetes, развернутую на нескольких экземплярах Azure Stack Hub в разных физических расположениях.</span><span class="sxs-lookup"><span data-stu-id="46710-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="46710-105">С помощью этого руководства по развертыванию решения вы узнаете, как выполнять следующие задачи:</span><span class="sxs-lookup"><span data-stu-id="46710-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="46710-106">Скачивание и подготовка обработчика AKS</span><span class="sxs-lookup"><span data-stu-id="46710-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="46710-107">Подключение к вспомогательной виртуальной машине обработчика AKS</span><span class="sxs-lookup"><span data-stu-id="46710-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="46710-108">Развертывание кластера Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="46710-109">Подключение к кластеру Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="46710-110">Подключение Azure Pipelines к кластеру Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="46710-111">Настройка мониторинга</span><span class="sxs-lookup"><span data-stu-id="46710-111">Configure monitoring</span></span>
> - <span data-ttu-id="46710-112">Развертывание приложения</span><span class="sxs-lookup"><span data-stu-id="46710-112">Deploy application</span></span>
> - <span data-ttu-id="46710-113">Автомасштабирование приложения</span><span class="sxs-lookup"><span data-stu-id="46710-113">Autoscale application</span></span>
> - <span data-ttu-id="46710-114">Настройка диспетчера трафика</span><span class="sxs-lookup"><span data-stu-id="46710-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="46710-115">Обновление Kubernetes в Службе контейнеров Azure (AKS)</span><span class="sxs-lookup"><span data-stu-id="46710-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="46710-116">Масштабирование Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="46710-117">![Экран "Основные аспекты проектирования гибридных приложений"](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="46710-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="46710-118">Microsoft Azure Stack Hub — это расширение Azure.</span><span class="sxs-lookup"><span data-stu-id="46710-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="46710-119">Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Это решение позволяет использовать единственное гибридное облако, с помощью которого можно создавать и развертывать гибридные приложения в любой точке мира.</span><span class="sxs-lookup"><span data-stu-id="46710-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="46710-120">В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений.</span><span class="sxs-lookup"><span data-stu-id="46710-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="46710-121">Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.</span><span class="sxs-lookup"><span data-stu-id="46710-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="46710-122">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="46710-122">Prerequisites</span></span>

<span data-ttu-id="46710-123">Прежде чем приступить к работе с этим руководством по развертыванию, не забудьте выполнить следующие действия:</span><span class="sxs-lookup"><span data-stu-id="46710-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="46710-124">Ознакомьтесь со статьей [Шаблон кластера Kubernetes с высоким уровнем доступности](pattern-highly-available-kubernetes.md).</span><span class="sxs-lookup"><span data-stu-id="46710-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="46710-125">Просмотрите содержимое [вспомогательного репозитория GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), в котором можно найти дополнительные ресурсы, упоминаемые в этой статье.</span><span class="sxs-lookup"><span data-stu-id="46710-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="46710-126">Подключите для вашей учетной записи доступ к [пользовательскому порталу Azure Stack Hub](/azure-stack/user/azure-stack-use-portal) с по меньшей мере [разрешениями участника](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="46710-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="46710-127">Скачивание и подготовка обработчика AKS</span><span class="sxs-lookup"><span data-stu-id="46710-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="46710-128">Обработчик AKS — это двоичный файл, который можно использовать на любом узле Windows или Linux для достижения конечных точек Azure Resource Manager в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="46710-129">В этом руководстве описано, как развертывать новую виртуальную машину Linux (или Windows) в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="46710-130">Она будет использоваться позже, когда обработчик AKS развернет кластеры Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="46710-131">Для развертывания кластера Kubernetes в Azure Stack Hub с помощью обработчика AKS можно также использовать существующую виртуальную машину на Windows или Linux.</span><span class="sxs-lookup"><span data-stu-id="46710-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="46710-132">Пошаговые инструкции и требования для обработчика AKS описаны здесь:</span><span class="sxs-lookup"><span data-stu-id="46710-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="46710-133">[Установка обработчика AKS на Linux в Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (или же используя [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="46710-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="46710-134">Обработчик AKS — это вспомогательное средство для развертывания и эксплуатации (неуправляемых) кластеров Kubernetes (в Azure и Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="46710-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="46710-135">О сведениях и отличиях обработчика AKS в Azure Stack Hub можно почитать здесь:</span><span class="sxs-lookup"><span data-stu-id="46710-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="46710-136">Что собой представляет обработчик AKS в Azure Stack Hub?</span><span class="sxs-lookup"><span data-stu-id="46710-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="46710-137">[Использование обработчика AKS в Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (на GitHub)</span><span class="sxs-lookup"><span data-stu-id="46710-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="46710-138">Пример среды будет использовать Terraform для автоматизации развертывания виртуальной машины обработчика AKS.</span><span class="sxs-lookup"><span data-stu-id="46710-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="46710-139">[Сведения и код можно найти во вспомогательном репозитории GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="46710-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="46710-140">После выполнения этого шага в Azure Stack Hub появится новая группа ресурсов, содержащая вспомогательную виртуальную машину обработчика AKS, а также связанные ресурсы:</span><span class="sxs-lookup"><span data-stu-id="46710-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Ресурсы виртуальной машины для обработчика AKS в Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="46710-142">Если обработчик AKS необходимо развернуть в отключенной от Интернета автономной среде, ознакомьтесь со сведениями в разделе об [экземплярах Azure Stack Hub, отключенных от Интернета ](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances).</span><span class="sxs-lookup"><span data-stu-id="46710-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="46710-143">На следующем шаге мы будем использовать только что развернутую виртуальную машину обработчика AKS для развертывания кластера Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="46710-144">Подключение к вспомогательной виртуальной машине обработчика AKS</span><span class="sxs-lookup"><span data-stu-id="46710-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="46710-145">Сначала необходимо подключиться к ранее созданной вспомогательной виртуальной машине обработчика AKS.</span><span class="sxs-lookup"><span data-stu-id="46710-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="46710-146">У виртуальной машины должен быть общедоступный IP-адрес и доступ через SSH (порт 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="46710-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Экран "Страница "Обзор" виртуальной машины обработчика AKS"](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="46710-148">Для подключения к виртуальной машине Linux с помощью SSH можно использовать любое средство на ваш выбор, включая MobaXterm, puTTY или PowerShell на Windows 10.</span><span class="sxs-lookup"><span data-stu-id="46710-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="46710-149">Подключившись, выполните команду `aks-engine`.</span><span class="sxs-lookup"><span data-stu-id="46710-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="46710-150">Ознакомьтесь с [поддерживаемыми версиями обработчика AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions), чтобы узнать больше об обработчике AKS и версиях Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![Экран "Пример командной строки aks-engine"](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="46710-152">Развертывание кластера Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="46710-153">Сама вспомогательная виртуальная машина обработчика AKS еще не создала кластер Kubernetes в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="46710-154">Создание кластера является первым действием, которое необходимо выполнить на вспомогательной виртуальной машине обработчика AKS.</span><span class="sxs-lookup"><span data-stu-id="46710-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="46710-155">Пошаговые инструкции описаны здесь:</span><span class="sxs-lookup"><span data-stu-id="46710-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="46710-156">Развертывание кластера Kubernetes с обработчиком AKS в Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="46710-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="46710-157">В результате выполнения команды `aks-engine deploy` и подготовительных действий, описанных на предыдущих шагах, вы получите полнофункциональный кластер Kubernetes, развернутый в пространстве клиента первого экземпляра Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="46710-158">Сам кластер состоит из таких компонентов Azure IaaS, как виртуальные машины, подсистемы балансировки нагрузки, виртуальные сети, диски и т. д.</span><span class="sxs-lookup"><span data-stu-id="46710-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Экран "Портал Azure Stack Hub с компонентами Azure IaaS кластера"](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="46710-160">Azure Load Balancer (конечная точка API K8s)</span><span class="sxs-lookup"><span data-stu-id="46710-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="46710-161">Рабочие узлы (пул агентов)</span><span class="sxs-lookup"><span data-stu-id="46710-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="46710-162">Главные узлы</span><span class="sxs-lookup"><span data-stu-id="46710-162">Master Nodes</span></span>

<span data-ttu-id="46710-163">Теперь кластер настроен и работает. На следующем шаге мы попробуем к нему подключиться.</span><span class="sxs-lookup"><span data-stu-id="46710-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="46710-164">Подключение к кластеру Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="46710-165">Теперь к ранее созданному кластеру Kubernetes можно подключиться либо через протокол SSH (с помощью ключа SSH, указанного в составе развертывания), либо используя `kubectl` (рекомендуется).</span><span class="sxs-lookup"><span data-stu-id="46710-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="46710-166">Программа командой строки Kubernetes `kubectl` для Windows, Linux и macOS доступна [здесь](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="46710-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="46710-167">Она уже предварительно установлена и настроена на главных узлах нашего кластера.</span><span class="sxs-lookup"><span data-stu-id="46710-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Экран "Выполнение kubectl на главном узле"](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="46710-169">Главный узел не рекомендуется использовать как инсталляционный сервер для административных задач.</span><span class="sxs-lookup"><span data-stu-id="46710-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="46710-170">Конфигурация `kubectl` хранится в `.kube/config` на главных узлах, а также на виртуальной машине обработчика AKS.</span><span class="sxs-lookup"><span data-stu-id="46710-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="46710-171">Вы можете скопировать конфигурацию на компьютер администратора с подключением к кластеру Kubernetes, а затем выполнить на нем команду `kubectl`.</span><span class="sxs-lookup"><span data-stu-id="46710-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="46710-172">Позже для настройки подключения службы в Azure Pipelines можно также использовать файл `.kube/config`.</span><span class="sxs-lookup"><span data-stu-id="46710-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="46710-173">Защитите эти файлы, поскольку они содержат учетные данные для вашего кластера Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="46710-174">У злоумышленника с доступом к файлу появится достаточно сведений, чтобы получить права администратора.</span><span class="sxs-lookup"><span data-stu-id="46710-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="46710-175">Все действия, которые выполняются с помощью начального файла `.kube/config`, выполняются под учетной записью администратора кластера.</span><span class="sxs-lookup"><span data-stu-id="46710-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="46710-176">Попробуйте теперь выполнить различные команды, используя `kubectl` для проверки состояния кластера.</span><span class="sxs-lookup"><span data-stu-id="46710-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="46710-177">У Kubernetes есть собственная модель _ *управления доступом на основе ролей (RBAC)* \*, которая позволяет создавать детальные определения и привязки ролей.</span><span class="sxs-lookup"><span data-stu-id="46710-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="46710-178">Этот способ управления доступом к кластеру является более предпочтительным, чем передача разрешений администратора кластера.</span><span class="sxs-lookup"><span data-stu-id="46710-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="46710-179">Подключение Azure Pipelines к кластерам Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="46710-180">Чтобы подключить Azure Pipelines к ранее развернутому кластеру Kubernetes, необходим файл kube config (`.kube/config`), упомянутый на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="46710-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="46710-181">Подключитесь к одному из главных узлов кластера Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="46710-182">Скопируйте содержимое файла `.kube/config`.</span><span class="sxs-lookup"><span data-stu-id="46710-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="46710-183">Перейдите в раздел Azure DevOps > "Параметры проекта" > "Подключения к службе", чтобы создать новое подключение службы Kubernetes (используйте в качестве способа проверки подлинности KubeConfig)</span><span class="sxs-lookup"><span data-stu-id="46710-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="46710-184">Для Azure Pipelines (или его агентов сборки) требуется доступ к API Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="46710-185">При наличии подключения к Интернету из Azure Pipelines к кластеру Kubernetes в Azure Stack Hub необходимо развернуть локальный агент сборки Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="46710-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="46710-186">Процесс развертывания локальных агентов для Azure Pipelines можно выполнить или в Azure Stack Hub, или на компьютере с сетевым подключением ко всем необходимым конечным точкам управления.</span><span class="sxs-lookup"><span data-stu-id="46710-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="46710-187">См. подробные сведения здесь:</span><span class="sxs-lookup"><span data-stu-id="46710-187">See the details here:</span></span>

* <span data-ttu-id="46710-188">[Агенты Azure Pipelines](/azure/devops/pipelines/agents/agents) для [Windows](/azure/devops/pipelines/agents/v2-windows) или [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="46710-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="46710-189">В разделе шаблонов [Рекомендации по развертыванию (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) содержится поток решений, который поможет вам понять, когда нужно использовать локальные агенты и когда агенты, размещенные Майкрософт:</span><span class="sxs-lookup"><span data-stu-id="46710-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="46710-190">[![Экран "Поток решений для локальных агентов"](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="46710-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="46710-191">В этом примере решения топология включает в себя локальный агент сборки на каждом экземпляре Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="46710-192">Агент может получить доступ и к конечным точкам управления Azure Stack Hub, так и к конечным точкам API кластера Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="46710-193">[![Экран "Только исходящий трафик"](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="46710-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="46710-194">Эта модель соответствует общему нормативному требованию, которое заключается в том, чтобы из решения приложения поступали только исходящие подключения.</span><span class="sxs-lookup"><span data-stu-id="46710-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="46710-195">Настройка мониторинга</span><span class="sxs-lookup"><span data-stu-id="46710-195">Configure monitoring</span></span>

<span data-ttu-id="46710-196">Для мониторинга контейнеров в решении можно использовать [Azure Monitor](/azure/azure-monitor/) для контейнеров.</span><span class="sxs-lookup"><span data-stu-id="46710-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="46710-197">Это позволит указать для Azure Monitor кластер Kubernetes, развернутый обработчиком AKS в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="46710-198">Есть два способа включить Azure Monitor для кластера.</span><span class="sxs-lookup"><span data-stu-id="46710-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="46710-199">Для обоих способов понадобится настроить в Azure рабочую область Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="46710-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="46710-200">[Метод 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) использует диаграмму Helm</span><span class="sxs-lookup"><span data-stu-id="46710-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="46710-201">[Метод 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two), относящийся к спецификации кластера обработчика AKS</span><span class="sxs-lookup"><span data-stu-id="46710-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="46710-202">В примере топологии используется "Метод 1", который позволяет автоматизировать процесс и упростить установку обновлений.</span><span class="sxs-lookup"><span data-stu-id="46710-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="46710-203">Для следующего шага на вашем компьютере должна быть рабочая область Azure Log Analytics (ее идентификатор и ключ), `Helm` (версия 3) и `kubectl`.</span><span class="sxs-lookup"><span data-stu-id="46710-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="46710-204">Helm — это диспетчер пакетов Kubernetes, представленный в виде двоичного файла, который выполняется на macOS, Windows и Linux.</span><span class="sxs-lookup"><span data-stu-id="46710-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="46710-205">Его можно скачать здесь: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm полагается на файл конфигурации Kubernetes, используемый для команды `kubectl`.</span><span class="sxs-lookup"><span data-stu-id="46710-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="46710-206">Выполнив эту команду, можно установить агент Azure Monitor в кластере Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="46710-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="46710-207">Агент Operations Management Suite (OMS) в кластере Kubernetes будет передавать данные мониторинга в рабочую область Azure Log Analytics (используя исходящий протокол HTTPS).</span><span class="sxs-lookup"><span data-stu-id="46710-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="46710-208">Теперь вы можете использовать Azure Monitor, чтобы получить более подробные аналитические сведения о кластерах Kubernetes в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="46710-209">Приведенная схема отлично отображает возможности аналитики, которую можно автоматически развернуть в кластерах вашего приложения.</span><span class="sxs-lookup"><span data-stu-id="46710-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="46710-210">[![Экран "Кластеры Azure Stack Hub в Azure Monitor"](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="46710-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="46710-211">[![Экран "Сведения о кластере Azure Monitor"](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="46710-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="46710-212">Если Azure Monitor не отображает данные Azure Stack Hub, убедитесь, что вы выполнили инструкции по [добавлению решения Azure Monitor для контейнеров в рабочую область Azure Log Analytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).</span><span class="sxs-lookup"><span data-stu-id="46710-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="46710-213">Развертывание приложения</span><span class="sxs-lookup"><span data-stu-id="46710-213">Deploy the application</span></span>

<span data-ttu-id="46710-214">Перед установкой нашего примера приложения следует выполнить еще один шаг по настройке контроллера объекта ingress на основе nginx в нашем кластере Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="46710-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="46710-215">Контроллер объекта ingress используется в качестве подсистемы балансировки нагрузки уровня 7 для маршрутизации трафика в кластере на основе узла, пути или протокола.</span><span class="sxs-lookup"><span data-stu-id="46710-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="46710-216">Входящий трафик Nginx доступный в виде диаграммы Helm.</span><span class="sxs-lookup"><span data-stu-id="46710-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="46710-217">Подробные инструкции см. в разделе о [диаграммах Helm в репозитории GitHub](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="46710-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="46710-218">Пример приложения также упаковывается как диаграмма Helm, аналогично [агенту наблюдения Azure](#configure-monitoring) с предыдущего шага.</span><span class="sxs-lookup"><span data-stu-id="46710-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="46710-219">Таким образом, развертывать приложение на нашем кластере Kubernetes довольно просто.</span><span class="sxs-lookup"><span data-stu-id="46710-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="46710-220">[Файлы диаграммы Helm можно найти во вспомогательном репозитории GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="46710-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="46710-221">Пример приложения — это трехуровневое приложение, развернутое в кластере Kubernetes на каждом из двух экземпляров Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="46710-222">Приложение использует базу данных MongoDB.</span><span class="sxs-lookup"><span data-stu-id="46710-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="46710-223">Дополнительные сведения о том, как получить данные, реплицируемые в нескольких экземплярах, см. в разделе [Рекомендации по данным и хранилищу](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="46710-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="46710-224">Развернув диаграмму Helm для приложения, вы увидите, что все три уровня приложения представленные в качестве развертываний и наборов с отслеживанием состояния (для базы данных) с одним объектом pod:</span><span class="sxs-lookup"><span data-stu-id="46710-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="46710-225">На стороне службы можно найти контроллер объекта ingress на основе nginx, а также его общедоступный IP-адрес:</span><span class="sxs-lookup"><span data-stu-id="46710-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="46710-226">"Внешний IP-адрес" — это наша "конечная точка приложения".</span><span class="sxs-lookup"><span data-stu-id="46710-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="46710-227">С его помощью пользователи будут подключаться, чтобы открыть приложение. Кроме того, мы будем использовать этот IP в качестве конечной точки на шаге [Настройки диспетчера трафика](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="46710-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="46710-228">Автомасштабирование приложения</span><span class="sxs-lookup"><span data-stu-id="46710-228">Autoscale the application</span></span>
<span data-ttu-id="46710-229">При необходимости вы можете настроить [средство горизонтального автомасштабирования объекта pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/), с помощью которого можно увеличивать или уменьшать масштаб на основе определенных метрик, например загрузки ЦП.</span><span class="sxs-lookup"><span data-stu-id="46710-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="46710-230">Следующая команда создаст средство горизонтального автомасштабирования объекта pod, которое обслуживает от 1 до 10 реплик Pod, управляемых с помощью веб-развертывания оценок.</span><span class="sxs-lookup"><span data-stu-id="46710-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="46710-231">HPA будет увеличивать и уменьшать число реплик (с помощью развертывания) для поддержания средней загрузки ЦП во всех объектах Pod на уровне 80 %.</span><span class="sxs-lookup"><span data-stu-id="46710-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="46710-232">Текущий статус средства автомасштабирования можно проверить, выполнив команду:</span><span class="sxs-lookup"><span data-stu-id="46710-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="46710-233">Настройка диспетчера трафика</span><span class="sxs-lookup"><span data-stu-id="46710-233">Configure Traffic Manager</span></span>

<span data-ttu-id="46710-234">Для распределения трафика между двумя (или больше) развертываниями приложения используется [диспетчер трафика Azure](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="46710-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="46710-235">Диспетчер трафика Azure — это подсистема балансировки нагрузки в Azure, основанная на DNS.</span><span class="sxs-lookup"><span data-stu-id="46710-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="46710-236">Диспетчер трафика использует DNS для направления клиентских запросов к наиболее подходящей конечной точке службы в зависимости от метода маршрутизации трафика и работоспособности конечных точек.</span><span class="sxs-lookup"><span data-stu-id="46710-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="46710-237">Вместо Диспетчера трафика Azure можно также использовать другие глобальные решения для балансировки нагрузки, размещенные локально.</span><span class="sxs-lookup"><span data-stu-id="46710-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="46710-238">В примере сценария мы будем использовать Диспетчер трафика Azure, чтобы распределять трафик между двумя экземплярами нашего приложения.</span><span class="sxs-lookup"><span data-stu-id="46710-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="46710-239">Они могут выполняться на экземплярах Azure Stack Hub в одинаковых или различных расположениях:</span><span class="sxs-lookup"><span data-stu-id="46710-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![Экран "Локальный диспетчер трафика"](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="46710-241">Мы настроим Диспетчер трафика в Azure так, чтобы он указывал на два разных экземпляра нашего приложения:</span><span class="sxs-lookup"><span data-stu-id="46710-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="46710-242">[![Экран "Профиль конечной точки диспетчера графика"](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="46710-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="46710-243">Как видите, две конечные точки указывают на два экземпляра развернутого приложения, упоминаемого в [предыдущем разделе](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="46710-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="46710-244">На этом этапе:</span><span class="sxs-lookup"><span data-stu-id="46710-244">At this point:</span></span>
- <span data-ttu-id="46710-245">Создана инфраструктура Kubernetes, включая контроллер объекта ingress.</span><span class="sxs-lookup"><span data-stu-id="46710-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="46710-246">Развернуты кластеры между двумя экземплярами Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="46710-247">Настроен мониторинг.</span><span class="sxs-lookup"><span data-stu-id="46710-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="46710-248">Диспетчер трафика Azure будет распределять нагрузку трафика между двумя экземплярами Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="46710-249">Помимо этой инфраструктуры был автоматически развернут пример трехуровневого приложения с использованием диаграмм Helm.</span><span class="sxs-lookup"><span data-stu-id="46710-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="46710-250">Решение теперь настроено и доступно для пользователей!</span><span class="sxs-lookup"><span data-stu-id="46710-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="46710-251">Есть еще несколько рекомендаций по работе, которые следует рассмотреть после развертывания, и о которых подробнее рассказано в следующих двух разделах.</span><span class="sxs-lookup"><span data-stu-id="46710-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="46710-252">Обновление Kubernetes в Службе контейнеров Azure (AKS)</span><span class="sxs-lookup"><span data-stu-id="46710-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="46710-253">При обновлении кластера Kubernetes обратите внимание на следующие темы:</span><span class="sxs-lookup"><span data-stu-id="46710-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="46710-254">Обновление кластера Kubernetes — это сложная операция Дня 2, которую можно выполнить с помощью обработчика AKS.</span><span class="sxs-lookup"><span data-stu-id="46710-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="46710-255">Дополнительные сведения см. в разделе [Обновление кластера Kubernetes в Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="46710-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="46710-256">Обработчик AKS позволяет обновлять кластеры до более новых версий Kubernetes и базовых образов ОС.</span><span class="sxs-lookup"><span data-stu-id="46710-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="46710-257">Дополнительные сведения см. в разделе [Шаги для обновления к более новой версии Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="46710-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="46710-258">Кроме того, до новых версий базовых образов ОС можно обновить только основные узлы.</span><span class="sxs-lookup"><span data-stu-id="46710-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="46710-259">Дополнительные сведения см. в разделе [Шаги для обновления образа ОС](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="46710-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="46710-260">Более новые базовые образы ОС содержат обновления безопасности и ядра.</span><span class="sxs-lookup"><span data-stu-id="46710-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="46710-261">Ответственность за отслеживание доступности более новых версий Kubernetes и образов ОС лежит на операторе кластера.</span><span class="sxs-lookup"><span data-stu-id="46710-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="46710-262">Оператор должен планировать и выполнять эти обновления с помощью обработчика AKS.</span><span class="sxs-lookup"><span data-stu-id="46710-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="46710-263">Базовые образы ОС необходимо скачать с Azure Stack Hub Marketplace, используя оператор Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="46710-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="46710-264">Масштабирование Kubernetes</span><span class="sxs-lookup"><span data-stu-id="46710-264">Scale Kubernetes</span></span>

<span data-ttu-id="46710-265">Масштабирование является еще одной операцией Дня 2, которой можно управлять с помощью обработчика AKS.</span><span class="sxs-lookup"><span data-stu-id="46710-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="46710-266">Команда scale использует тот же файл конфигурации кластера (apimodel.json) в выходном каталоге, как входные данные для нового развертывания Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="46710-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="46710-267">Обработчик AKS выполняет операцию масштабирования для указанного пула агентов.</span><span class="sxs-lookup"><span data-stu-id="46710-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="46710-268">После завершения операции масштабирования обработчик AKS обновляет определение кластера в том же файле apimodel.json.</span><span class="sxs-lookup"><span data-stu-id="46710-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="46710-269">Определение кластера отражает новое число узлов, чтобы соответствовать обновленной текущей конфигурации кластера.</span><span class="sxs-lookup"><span data-stu-id="46710-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="46710-270">Масштабирование кластера Kubernetes в Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="46710-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="46710-271">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="46710-271">Next steps</span></span>

- <span data-ttu-id="46710-272">Узнайте больше об [аспектах проектирования гибридных приложений](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="46710-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="46710-273">Изучите и предложите улучшения для [кода в этом примере на сайте GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="46710-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>