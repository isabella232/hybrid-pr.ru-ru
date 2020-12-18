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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Развертывание кластера Kubernetes с высоким уровнем доступности в Azure Stack Hub

В этой статье показано, как создать высокодоступную среду кластера Kubernetes, развернутую на нескольких экземплярах Azure Stack Hub в разных физических расположениях.

С помощью этого руководства по развертыванию решения вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> - Скачивание и подготовка обработчика AKS
> - Подключение к вспомогательной виртуальной машине обработчика AKS
> - Развертывание кластера Kubernetes
> - Подключение к кластеру Kubernetes
> - Подключение Azure Pipelines к кластеру Kubernetes
> - Настройка мониторинга
> - Развертывание приложения
> - Автомасштабирование приложения
> - Настройка диспетчера трафика
> - Обновление Kubernetes в Службе контейнеров Azure (AKS)
> - Масштабирование Kubernetes

> [!Tip]  
> ![Экран "Основные аспекты проектирования гибридных приложений"](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub — это расширение Azure. Azure Stack Hub обеспечивает гибкость и высокую скорость внедрения инноваций облачных вычислений в локальной среде. Это решение позволяет использовать единственное гибридное облако, с помощью которого можно создавать и развертывать гибридные приложения в любой точке мира.  
> 
> В руководстве по [проектированию гибридных приложений](overview-app-design-considerations.md) перечислены основные аспекты качественного программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность), которые следует учитывать при разработке, развертывании и использовании гибридных приложений. Эти рекомендации помогут оптимизировать разработку гибридных приложений и предотвратить появление проблем с рабочими средами.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к работе с этим руководством по развертыванию, не забудьте выполнить следующие действия:

- Ознакомьтесь со статьей [Шаблон кластера Kubernetes с высоким уровнем доступности](pattern-highly-available-kubernetes.md).
- Просмотрите содержимое [вспомогательного репозитория GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), в котором можно найти дополнительные ресурсы, упоминаемые в этой статье.
- Подключите для вашей учетной записи доступ к [пользовательскому порталу Azure Stack Hub](/azure-stack/user/azure-stack-use-portal) с по меньшей мере [разрешениями участника](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>Скачивание и подготовка обработчика AKS

Обработчик AKS — это двоичный файл, который можно использовать на любом узле Windows или Linux для достижения конечных точек Azure Resource Manager в Azure Stack Hub. В этом руководстве описано, как развертывать новую виртуальную машину Linux (или Windows) в Azure Stack Hub. Она будет использоваться позже, когда обработчик AKS развернет кластеры Kubernetes.

> [!NOTE]
> Для развертывания кластера Kubernetes в Azure Stack Hub с помощью обработчика AKS можно также использовать существующую виртуальную машину на Windows или Linux.

Пошаговые инструкции и требования для обработчика AKS описаны здесь:

* [Установка обработчика AKS на Linux в Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (или же используя [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

Обработчик AKS — это вспомогательное средство для развертывания и эксплуатации (неуправляемых) кластеров Kubernetes (в Azure и Azure Stack Hub).

О сведениях и отличиях обработчика AKS в Azure Stack Hub можно почитать здесь:

* [Что собой представляет обработчик AKS в Azure Stack Hub?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Использование обработчика AKS в Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (на GitHub)

Пример среды будет использовать Terraform для автоматизации развертывания виртуальной машины обработчика AKS. [Сведения и код можно найти во вспомогательном репозитории GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

После выполнения этого шага в Azure Stack Hub появится новая группа ресурсов, содержащая вспомогательную виртуальную машину обработчика AKS, а также связанные ресурсы:

![Ресурсы виртуальной машины для обработчика AKS в Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Если обработчик AKS необходимо развернуть в отключенной от Интернета автономной среде, ознакомьтесь со сведениями в разделе об [экземплярах Azure Stack Hub, отключенных от Интернета ](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances).

На следующем шаге мы будем использовать только что развернутую виртуальную машину обработчика AKS для развертывания кластера Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Подключение к вспомогательной виртуальной машине обработчика AKS

Сначала необходимо подключиться к ранее созданной вспомогательной виртуальной машине обработчика AKS.

У виртуальной машины должен быть общедоступный IP-адрес и доступ через SSH (порт 22/TCP).

![Экран "Страница "Обзор" виртуальной машины обработчика AKS"](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Для подключения к виртуальной машине Linux с помощью SSH можно использовать любое средство на ваш выбор, включая MobaXterm, puTTY или PowerShell на Windows 10.

```console
ssh <username>@<ipaddress>
```

Подключившись, выполните команду `aks-engine`. Ознакомьтесь с [поддерживаемыми версиями обработчика AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions), чтобы узнать больше об обработчике AKS и версиях Kubernetes.

![Экран "Пример командной строки aks-engine"](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Развертывание кластера Kubernetes

Сама вспомогательная виртуальная машина обработчика AKS еще не создала кластер Kubernetes в Azure Stack Hub. Создание кластера является первым действием, которое необходимо выполнить на вспомогательной виртуальной машине обработчика AKS.

Пошаговые инструкции описаны здесь:

* [Развертывание кластера Kubernetes с обработчиком AKS в Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

В результате выполнения команды `aks-engine deploy` и подготовительных действий, описанных на предыдущих шагах, вы получите полнофункциональный кластер Kubernetes, развернутый в пространстве клиента первого экземпляра Azure Stack Hub. Сам кластер состоит из таких компонентов Azure IaaS, как виртуальные машины, подсистемы балансировки нагрузки, виртуальные сети, диски и т. д.

![Экран "Портал Azure Stack Hub с компонентами Azure IaaS кластера"](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure Load Balancer (конечная точка API K8s)
2) Рабочие узлы (пул агентов)
3) Главные узлы

Теперь кластер настроен и работает. На следующем шаге мы попробуем к нему подключиться.

## <a name="connect-to-the-kubernetes-cluster"></a>Подключение к кластеру Kubernetes

Теперь к ранее созданному кластеру Kubernetes можно подключиться либо через протокол SSH (с помощью ключа SSH, указанного в составе развертывания), либо используя `kubectl` (рекомендуется). Программа командой строки Kubernetes `kubectl` для Windows, Linux и macOS доступна [здесь](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Она уже предварительно установлена и настроена на главных узлах нашего кластера.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Экран "Выполнение kubectl на главном узле"](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Главный узел не рекомендуется использовать как инсталляционный сервер для административных задач. Конфигурация `kubectl` хранится в `.kube/config` на главных узлах, а также на виртуальной машине обработчика AKS. Вы можете скопировать конфигурацию на компьютер администратора с подключением к кластеру Kubernetes, а затем выполнить на нем команду `kubectl`. Позже для настройки подключения службы в Azure Pipelines можно также использовать файл `.kube/config`.

> [!IMPORTANT]
> Защитите эти файлы, поскольку они содержат учетные данные для вашего кластера Kubernetes. У злоумышленника с доступом к файлу появится достаточно сведений, чтобы получить права администратора. Все действия, которые выполняются с помощью начального файла `.kube/config`, выполняются под учетной записью администратора кластера.

Попробуйте теперь выполнить различные команды, используя `kubectl` для проверки состояния кластера.

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
> У Kubernetes есть собственная модель _ *управления доступом на основе ролей (RBAC)* *, которая позволяет создавать детальные определения и привязки ролей. Этот способ управления доступом к кластеру является более предпочтительным, чем передача разрешений администратора кластера.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Подключение Azure Pipelines к кластерам Kubernetes

Чтобы подключить Azure Pipelines к ранее развернутому кластеру Kubernetes, необходим файл kube config (`.kube/config`), упомянутый на предыдущем шаге.

* Подключитесь к одному из главных узлов кластера Kubernetes.
* Скопируйте содержимое файла `.kube/config`.
* Перейдите в раздел Azure DevOps > "Параметры проекта" > "Подключения к службе", чтобы создать новое подключение службы Kubernetes (используйте в качестве способа проверки подлинности KubeConfig)

> [!IMPORTANT]
> Для Azure Pipelines (или его агентов сборки) требуется доступ к API Kubernetes. При наличии подключения к Интернету из Azure Pipelines к кластеру Kubernetes в Azure Stack Hub необходимо развернуть локальный агент сборки Azure Pipelines.

Процесс развертывания локальных агентов для Azure Pipelines можно выполнить или в Azure Stack Hub, или на компьютере с сетевым подключением ко всем необходимым конечным точкам управления. См. подробные сведения здесь:

* [Агенты Azure Pipelines](/azure/devops/pipelines/agents/agents) для [Windows](/azure/devops/pipelines/agents/v2-windows) или [Linux](/azure/devops/pipelines/agents/v2-linux)

В разделе шаблонов [Рекомендации по развертыванию (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) содержится поток решений, который поможет вам понять, когда нужно использовать локальные агенты и когда агенты, размещенные Майкрософт:

[![Экран "Поток решений для локальных агентов"](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

В этом примере решения топология включает в себя локальный агент сборки на каждом экземпляре Azure Stack Hub. Агент может получить доступ и к конечным точкам управления Azure Stack Hub, так и к конечным точкам API кластера Kubernetes.

[![Экран "Только исходящий трафик"](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Эта модель соответствует общему нормативному требованию, которое заключается в том, чтобы из решения приложения поступали только исходящие подключения.

## <a name="configure-monitoring"></a>Настройка мониторинга

Для мониторинга контейнеров в решении можно использовать [Azure Monitor](/azure/azure-monitor/) для контейнеров. Это позволит указать для Azure Monitor кластер Kubernetes, развернутый обработчиком AKS в Azure Stack Hub.

Есть два способа включить Azure Monitor для кластера. Для обоих способов понадобится настроить в Azure рабочую область Log Analytics.

* [Метод 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) использует диаграмму Helm
* [Метод 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two), относящийся к спецификации кластера обработчика AKS

В примере топологии используется "Метод 1", который позволяет автоматизировать процесс и упростить установку обновлений.

Для следующего шага на вашем компьютере должна быть рабочая область Azure Log Analytics (ее идентификатор и ключ), `Helm` (версия 3) и `kubectl`.

Helm — это диспетчер пакетов Kubernetes, представленный в виде двоичного файла, который выполняется на macOS, Windows и Linux. Его можно скачать здесь: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm полагается на файл конфигурации Kubernetes, используемый для команды `kubectl`.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Выполнив эту команду, можно установить агент Azure Monitor в кластере Kubernetes:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Агент Operations Management Suite (OMS) в кластере Kubernetes будет передавать данные мониторинга в рабочую область Azure Log Analytics (используя исходящий протокол HTTPS). Теперь вы можете использовать Azure Monitor, чтобы получить более подробные аналитические сведения о кластерах Kubernetes в Azure Stack Hub. Приведенная схема отлично отображает возможности аналитики, которую можно автоматически развернуть в кластерах вашего приложения.

[![Экран "Кластеры Azure Stack Hub в Azure Monitor"](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Экран "Сведения о кластере Azure Monitor"](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Если Azure Monitor не отображает данные Azure Stack Hub, убедитесь, что вы выполнили инструкции по [добавлению решения Azure Monitor для контейнеров в рабочую область Azure Log Analytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).

## <a name="deploy-the-application"></a>Развертывание приложения

Перед установкой нашего примера приложения следует выполнить еще один шаг по настройке контроллера объекта ingress на основе nginx в нашем кластере Kubernetes. Контроллер объекта ingress используется в качестве подсистемы балансировки нагрузки уровня 7 для маршрутизации трафика в кластере на основе узла, пути или протокола. Входящий трафик Nginx доступный в виде диаграммы Helm. Подробные инструкции см. в разделе о [диаграммах Helm в репозитории GitHub](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

Пример приложения также упаковывается как диаграмма Helm, аналогично [агенту наблюдения Azure](#configure-monitoring) с предыдущего шага. Таким образом, развертывать приложение на нашем кластере Kubernetes довольно просто. [Файлы диаграммы Helm можно найти во вспомогательном репозитории GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

Пример приложения — это трехуровневое приложение, развернутое в кластере Kubernetes на каждом из двух экземпляров Azure Stack Hub. Приложение использует базу данных MongoDB. Дополнительные сведения о том, как получить данные, реплицируемые в нескольких экземплярах, см. в разделе [Рекомендации по данным и хранилищу](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

Развернув диаграмму Helm для приложения, вы увидите, что все три уровня приложения представленные в качестве развертываний и наборов с отслеживанием состояния (для базы данных) с одним объектом pod:

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

На стороне службы можно найти контроллер объекта ingress на основе nginx, а также его общедоступный IP-адрес:

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

"Внешний IP-адрес" — это наша "конечная точка приложения". С его помощью пользователи будут подключаться, чтобы открыть приложение. Кроме того, мы будем использовать этот IP в качестве конечной точки на шаге [Настройки диспетчера трафика](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Автомасштабирование приложения
При необходимости вы можете настроить [средство горизонтального автомасштабирования объекта pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/), с помощью которого можно увеличивать или уменьшать масштаб на основе определенных метрик, например загрузки ЦП. Следующая команда создаст средство горизонтального автомасштабирования объекта pod, которое обслуживает от 1 до 10 реплик Pod, управляемых с помощью веб-развертывания оценок. HPA будет увеличивать и уменьшать число реплик (с помощью развертывания) для поддержания средней загрузки ЦП во всех объектах Pod на уровне 80 %.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Текущий статус средства автомасштабирования можно проверить, выполнив команду:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Настройка диспетчера трафика

Для распределения трафика между двумя (или больше) развертываниями приложения используется [диспетчер трафика Azure](/azure/traffic-manager/traffic-manager-overview). Диспетчер трафика Azure — это подсистема балансировки нагрузки в Azure, основанная на DNS.

> [!NOTE]
> Диспетчер трафика использует DNS для направления клиентских запросов к наиболее подходящей конечной точке службы в зависимости от метода маршрутизации трафика и работоспособности конечных точек.

Вместо Диспетчера трафика Azure можно также использовать другие глобальные решения для балансировки нагрузки, размещенные локально. В примере сценария мы будем использовать Диспетчер трафика Azure, чтобы распределять трафик между двумя экземплярами нашего приложения. Они могут выполняться на экземплярах Azure Stack Hub в одинаковых или различных расположениях:

![Экран "Локальный диспетчер трафика"](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Мы настроим Диспетчер трафика в Azure так, чтобы он указывал на два разных экземпляра нашего приложения:

[![Экран "Профиль конечной точки диспетчера графика"](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Как видите, две конечные точки указывают на два экземпляра развернутого приложения, упоминаемого в [предыдущем разделе](#deploy-the-application).

На этом этапе:
- Создана инфраструктура Kubernetes, включая контроллер объекта ingress.
- Развернуты кластеры между двумя экземплярами Azure Stack Hub.
- Настроен мониторинг.
- Диспетчер трафика Azure будет распределять нагрузку трафика между двумя экземплярами Azure Stack Hub.
- Помимо этой инфраструктуры был автоматически развернут пример трехуровневого приложения с использованием диаграмм Helm. 

Решение теперь настроено и доступно для пользователей!

Есть еще несколько рекомендаций по работе, которые следует рассмотреть после развертывания, и о которых подробнее рассказано в следующих двух разделах.

## <a name="upgrade-kubernetes"></a>Обновление Kubernetes в Службе контейнеров Azure (AKS)

При обновлении кластера Kubernetes обратите внимание на следующие темы:

- Обновление кластера Kubernetes — это сложная операция Дня 2, которую можно выполнить с помощью обработчика AKS. Дополнительные сведения см. в разделе [Обновление кластера Kubernetes в Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Обработчик AKS позволяет обновлять кластеры до более новых версий Kubernetes и базовых образов ОС. Дополнительные сведения см. в разделе [Шаги для обновления к более новой версии Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Кроме того, до новых версий базовых образов ОС можно обновить только основные узлы. Дополнительные сведения см. в разделе [Шаги для обновления образа ОС](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Более новые базовые образы ОС содержат обновления безопасности и ядра. Ответственность за отслеживание доступности более новых версий Kubernetes и образов ОС лежит на операторе кластера. Оператор должен планировать и выполнять эти обновления с помощью обработчика AKS. Базовые образы ОС необходимо скачать с Azure Stack Hub Marketplace, используя оператор Azure Stack Hub.

## <a name="scale-kubernetes"></a>Масштабирование Kubernetes

Масштабирование является еще одной операцией Дня 2, которой можно управлять с помощью обработчика AKS.

Команда scale использует тот же файл конфигурации кластера (apimodel.json) в выходном каталоге, как входные данные для нового развертывания Azure Resource Manager. Обработчик AKS выполняет операцию масштабирования для указанного пула агентов. После завершения операции масштабирования обработчик AKS обновляет определение кластера в том же файле apimodel.json. Определение кластера отражает новое число узлов, чтобы соответствовать обновленной текущей конфигурации кластера.

- [Масштабирование кластера Kubernetes в Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об [аспектах проектирования гибридных приложений](overview-app-design-considerations.md).
- Изучите и предложите улучшения для [кода в этом примере на сайте GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).