---
title: Шаблон обучения модели машинного обучения на пограничных устройствах
description: Узнайте, как обучить модель машинного обучения на пограничном устройстве с помощью служб Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911096"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="4e0a2-103">Шаблон обучения модели машинного обучения на пограничных устройствах</span><span class="sxs-lookup"><span data-stu-id="4e0a2-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="4e0a2-104">Создавайте переносимые модели машинного обучения ML на основе данных, которые существуют только в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="4e0a2-105">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="4e0a2-105">Context and problem</span></span>

<span data-ttu-id="4e0a2-106">Многим организациям хотелось бы получить доступ к полезным сведениям из локальных или устаревших данных с помощью средств, понятных для их специалистов по обработке и анализу данных.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="4e0a2-107">Служба [Машинное обучение Azure](/azure/machine-learning/) предоставляет облачные средства для обучения, настройки и развертывания моделей машинного обучения и глубокого обучения.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="4e0a2-108">Но некоторые данные слишком велики для отправки в облако или их нельзя отправить в облако из-за нормативных требований.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="4e0a2-109">С помощью этого шаблона специалисты по обработке и анализу данных могут использовать Машинное обучение Azure для обучения моделей на основе локальных данных и вычислений.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="4e0a2-110">Решение</span><span class="sxs-lookup"><span data-stu-id="4e0a2-110">Solution</span></span>

<span data-ttu-id="4e0a2-111">В шаблоне обучения на пограничном устройстве используется виртуальная машина, работающая в Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="4e0a2-112">Виртуальная машина регистрируется в качестве целевого объекта вычислений в Azure ML, что обеспечивает ей доступ к данным, доступным только в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="4e0a2-113">В этом случае данные хранятся в хранилище BLOB-объектов Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="4e0a2-114">После обучения модели она регистрируется в Azure ML, помещается в контейнер и добавляется в Реестр контейнеров Azure для развертывания.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="4e0a2-115">Для этой итерации шаблона виртуальная машина для обучения Azure Stack Hub должна быть доступна через общедоступный Интернет.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="4e0a2-116">[![Обучение модели ML в пограничной архитектуре](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="4e0a2-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="4e0a2-117">Вот как работает этот шаблон:</span><span class="sxs-lookup"><span data-stu-id="4e0a2-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="4e0a2-118">Виртуальная машина Azure Stack Hub развертывается и регистрируется в качестве целевого объекта вычислений для Azure ML.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="4e0a2-119">В Azure ML создается эксперимент, в котором в качестве целевого объекта вычислений используется виртуальная машина Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="4e0a2-120">Как только модель будет обучена, она будет зарегистрирована и помещена в контейнер.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="4e0a2-121">Теперь модель можно развернуть в расположениях, которые находятся в локальной среде или в облаке.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="4e0a2-122">Components</span><span class="sxs-lookup"><span data-stu-id="4e0a2-122">Components</span></span>

<span data-ttu-id="4e0a2-123">Это решение использует следующие компоненты.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-123">This solution uses the following components:</span></span>

| <span data-ttu-id="4e0a2-124">Уровень</span><span class="sxs-lookup"><span data-stu-id="4e0a2-124">Layer</span></span> | <span data-ttu-id="4e0a2-125">Компонент</span><span class="sxs-lookup"><span data-stu-id="4e0a2-125">Component</span></span> | <span data-ttu-id="4e0a2-126">Описание</span><span class="sxs-lookup"><span data-stu-id="4e0a2-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="4e0a2-127">Azure</span><span class="sxs-lookup"><span data-stu-id="4e0a2-127">Azure</span></span> | <span data-ttu-id="4e0a2-128">Машинное обучение Azure</span><span class="sxs-lookup"><span data-stu-id="4e0a2-128">Azure Machine Learning</span></span> | <span data-ttu-id="4e0a2-129">[Машинное обучение Azure](/azure/machine-learning/) координирует обучение модели ML.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="4e0a2-130">Реестр контейнеров Azure</span><span class="sxs-lookup"><span data-stu-id="4e0a2-130">Azure Container Registry</span></span> | <span data-ttu-id="4e0a2-131">Azure ML упаковывает модель в контейнер и сохраняет ее в [Реестре контейнеров Azure](/azure/container-registry/) для развертывания.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="4e0a2-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="4e0a2-132">Azure Stack Hub</span></span> | <span data-ttu-id="4e0a2-133">Служба приложений</span><span class="sxs-lookup"><span data-stu-id="4e0a2-133">App Service</span></span> | <span data-ttu-id="4e0a2-134">[Azure Stack Hub со Службой приложений](/azure-stack/operator/azure-stack-app-service-overview) предоставляет основу для компонентов на пограничном устройстве.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="4e0a2-135">Службы вычислений</span><span class="sxs-lookup"><span data-stu-id="4e0a2-135">Compute</span></span> | <span data-ttu-id="4e0a2-136">Для обучения модели ML используется виртуальная машина Azure Stack Hub под управлением Ubuntu с Docker.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="4e0a2-137">Память</span><span class="sxs-lookup"><span data-stu-id="4e0a2-137">Storage</span></span> | <span data-ttu-id="4e0a2-138">Частные данные могут размещаться в хранилище BLOB-объектов Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="4e0a2-139">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="4e0a2-139">Issues and considerations</span></span>

<span data-ttu-id="4e0a2-140">Во время выбора варианта реализации этого решения необходимо учитывать приведенные ниже моменты.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="4e0a2-141">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="4e0a2-141">Scalability</span></span>

<span data-ttu-id="4e0a2-142">Чтобы обеспечить масштабирование решения, в Azure Stack Hub необходимо создать виртуальную машину соответствующего размера для обучения.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="4e0a2-143">Доступность</span><span class="sxs-lookup"><span data-stu-id="4e0a2-143">Availability</span></span>

<span data-ttu-id="4e0a2-144">Убедитесь, что скрипты обучения и виртуальная машина Azure Stack Hub имеют доступ к локальным данным, используемым для обучения.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="4e0a2-145">Управляемость</span><span class="sxs-lookup"><span data-stu-id="4e0a2-145">Manageability</span></span>

<span data-ttu-id="4e0a2-146">Убедитесь, что модели и эксперименты зарегистрированы надлежащим образом, имеют подходящую версию и помечены тегами во избежание путаницы во время развертывания модели.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="4e0a2-147">Безопасность</span><span class="sxs-lookup"><span data-stu-id="4e0a2-147">Security</span></span>

<span data-ttu-id="4e0a2-148">Этот шаблон позволяет Azure ML получать доступ к возможным конфиденциальным данным в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="4e0a2-149">Убедитесь, что учетная запись, используемая для подключения по протоколу SSH к виртуальной машине Azure Stack Hub, имеет надежный пароль, а скрипты обучения не сохраняют или не передают данные в облако.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="4e0a2-150">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="4e0a2-150">Next steps</span></span>

<span data-ttu-id="4e0a2-151">Дополнительные сведения по темам, описанным в этой статье:</span><span class="sxs-lookup"><span data-stu-id="4e0a2-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="4e0a2-152">Общие сведения о ML и связанные разделы см. в [документации по Azure ML](/azure/machine-learning).</span><span class="sxs-lookup"><span data-stu-id="4e0a2-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="4e0a2-153">Дополнительные сведения о создании, хранении образов и управлении ими см. в [документации по Реестру контейнеров Azure](/azure/container-registry/).</span><span class="sxs-lookup"><span data-stu-id="4e0a2-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="4e0a2-154">Дополнительные сведения о поставщике ресурсов и способах развертывания см. в статье [Обзор Службы приложений Azure в Azure Stack](/azure-stack/operator/azure-stack-app-service-overview).</span><span class="sxs-lookup"><span data-stu-id="4e0a2-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="4e0a2-155">См. [рекомендации по проектированию гибридных приложений и ответы на дополнительные вопросы](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="4e0a2-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="4e0a2-156">См. сведения обо [всех продуктах и решениях Azure Stack](/azure-stack).</span><span class="sxs-lookup"><span data-stu-id="4e0a2-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="4e0a2-157">Когда вы будете готовы протестировать пример решения, продолжите работу с [руководством по развертыванию обучения модели ML на пограничном устройстве](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="4e0a2-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="4e0a2-158">В этом руководстве содержатся пошаговые инструкции по развертыванию и тестированию компонентов.</span><span class="sxs-lookup"><span data-stu-id="4e0a2-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
