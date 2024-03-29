---
title: Шаблон решения для масштабирования в разных облаках (локальные данные) для Azure Stack Hub
description: Узнайте, как создать масштабируемое приложение, использующее локальные данные, в Azure и Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281250"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Шаблон решения для масштабирования в разных облаках (локальные данные)

Узнайте, как создать гибридное приложение с поддержкой Azure и Azure Stack Hub. Этот шаблон также показывает, как использовать один источник локальных данных для обеспечения соответствия.

## <a name="context-and-problem"></a>Контекст и проблема

Многие организации собирают и хранят значительный объем конфиденциальных данных своих клиентов. Часто им запрещено хранить конфиденциальные данные в общедоступном облаке согласно корпоративным или государственным нормативным требованиям. Эти организации также хотят воспользоваться преимуществами масштабируемости общедоступного облака. Общедоступное облако может справляться с сезонными пиковыми нагрузками трафика, что позволяет клиентам платить только за требуемое оборудование по мере необходимости.

## <a name="solution"></a>Решение

Решение обеспечивает преимущества соответствия частного облака, сочетая их с масштабируемостью общедоступного облака. Гибридное облако Azure и Azure Stack Hub обеспечивает согласованную работу для разработчиков. Такая согласованность позволяет им применять свои навыки в общедоступных облаках и локальных средах.

Используя руководство по развертыванию решения, можно развернуть идентичное веб-приложение в общедоступном и частном облаке. Вы также можете получить доступ к маршрутизируемой сети, недоступной из Интернета и размещенной в частном облаке. В веб-приложениях отслеживается нагрузка. При значительном увеличении трафика программа управляет записями DNS, чтобы перенаправить трафик в общедоступное облако. При снижении трафика записи DNS обновляются, чтобы направить трафик обратно в частное облако.

[![Шаблон решения для масштабирования в разных облаках (локальные данные)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Components

Это решение использует следующие компоненты.

| Уровень | Компонент | Описание |
|----------|-----------|-------------|
| Azure | Служба приложений Azure | [Служба приложений Azure](/azure/app-service/) позволяет создавать и размещать веб-приложения, приложения RESTful API и Функции Azure. Все на любом языке программирования без необходимости управления инфраструктурой. |
| | Виртуальная сеть Azure| [Виртуальная сеть (VNET) Azure](/azure/virtual-network/virtual-networks-overview) — это стандартный строительный блок для вашей частной сети в Azure. Виртуальная сеть позволяет ресурсам Azure различных типов (например, виртуальным машинам) обмениваться данными друг с другом через локальные сети и через Интернет. В решении также демонстрируется использование дополнительных сетевых компонентов:<br>— подсети приложения и шлюза;<br>— локальный сетевой шлюз;<br>— шлюз виртуальной сети, который выступает в качестве подключения VPN-шлюза типа "сеть — сеть";<br>— общедоступный IP-адрес;<br>— VPN-подключение "точка — сеть";<br>— Azure DNS для размещения доменов DNS и предоставления разрешения имен. |
| | диспетчер трафика Azure; | [Диспетчер трафика Azure](/azure/traffic-manager/traffic-manager-overview) — это балансировщик нагрузки трафика на основе DNS. Он позволяет управлять распределением пользовательского трафика между конечными точками службы в разных центрах обработки данных. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) — это расширяемая служба управления производительностью приложений. Она предназначена для веб-разработчиков, создающих приложения и управляющих ими на нескольких платформах.|
| | Функции Azure | [Функции Azure](/azure/azure-functions/) позволяют выполнять код в бессерверной среде без необходимости создавать виртуальную машину или публиковать веб-приложение. |
| | Автомасштабирование Azure | [Автомасштабирование](/azure/azure-monitor/platform/autoscale-overview) — это встроенная функция Облачных служб, виртуальных машин и веб-приложений. Эта функция позволяет приложениям эффективно работать при изменении нагрузки. Приложения могут адаптироваться к пикам трафика, уведомляя вас об изменении метрик и при необходимости выполняя масштабирование. |
| Azure Stack Hub | Вычисления IaaS | Azure Stack Hub позволяет использовать модель приложения, портал самообслуживания и API так же, как в Azure. Azure Stack Hub IaaS поддерживает широкий спектр технологий с открытым кодом для согласованного развертывания в гибридном облаке. В примере решения используется виртуальная машина Windows Server для SQL Server.|
| | Служба приложений Azure | Как и в веб-приложении Azure, в решении используется [Служба приложений Azure в Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) для размещения веб-приложения. |
| | Сеть | Виртуальная сеть Azure Stack Hub работает точно так же, как и виртуальная сеть Azure. В ней используются множество тех же сетевых компонентов, включая пользовательские имена узлов.
| Azure DevOps Services | Регистрация | Выполните быструю настройку непрерывной интеграции для сборки, тестирования и развертывания. Дополнительные сведения см. в статье [Quickstart: Sign up, sign in to Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops) (Краткое руководство. Регистрация и вход в Azure DevOps). |
| | Azure Pipelines | Используйте [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) для непрерывной интеграции и поставки. Azure Pipelines позволяет управлять размещенными агентами сборок и выпусков, а также определениями. |
| | Репозиторий кода | Используйте несколько репозиториев кода, чтобы оптимизировать конвейер разработки, в том числе имеющиеся репозитории кода в GitHub, BitBucket, Dropbox, OneDrive и Azure Repos. |

## <a name="issues-and-considerations"></a>Проблемы и рекомендации

Во время выбора варианта реализации этого решения необходимо учитывать приведенные ниже моменты.

### <a name="scalability"></a>Масштабируемость

Azure и Azure Stack Hub удовлетворяют требованиям современных глобально распределенных компаний.

#### <a name="hybrid-cloud-without-the-hassle"></a>Гибридное облако без трудностей

Корпорация Майкрософт предлагает непревзойденную интеграцию локальных ресурсов с Azure Stack Hub и Azure в едином решении. Эта интеграция устраняет трудности управления многоточечными решениями и сочетанием поставщиков облачных служб. При масштабировании в разных облаках можно легко организовать доступ к Azure, чтобы воспользоваться всеми возможностями этой облачной инфраструктуры. Просто подключите Azure Stack Hub к Azure, используя конфигурацию "выход в облако", и ваши данные и приложения будут доступны в Azure, когда это потребуется.

- Устраните необходимость в создании и обслуживании дополнительного сайта аварийного восстановления.
- Сэкономьте время и деньги, устранив необходимость в резервном копировании на магнитную ленту и перейдя в Azure, где срок хранения данных составляет до 99 лет.
- С легкостью переносите рабочие нагрузки Hyper-V, Physical (предварительная версия) и VMware (предварительная версия) в Azure, чтобы воспользоваться экономичностью и эластичностью облака.
- Запускайте в Azure отчеты или аналитику с ресурсоемкими вычислениями в реплицированной копии локального ресурса без ущерба для рабочих нагрузок.
- Войдите в облако и выполняйте локальные рабочие нагрузки в Azure, используя при необходимости более крупные шаблоны вычислений. Гибридная среда предоставляет необходимые возможности, когда это необходимо.
- В Azure можно быстро создавать многоуровневые среды разработки и даже реплицировать актуальные рабочие данные в среду разработки и тестирования для синхронизации практически в реальном времени.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Экономия масштабирования в нескольких облаках с помощью Azure Stack Hub

Ключевым преимуществом выхода в облако является экономичность. Вы платите за дополнительные ресурсы, только когда в них возникает потребность. Вам больше не нужно оплачивать неиспользуемый запас емкости или пытаться спрогнозировать пиковые нагрузки и колебания.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Снижение высокого уровня нагрузки в облаке

Масштабирование в разных облаках позволяет сглаживать рост нагрузки. Нагрузка распределяется за счет перемещения основных приложений в общедоступное облако, что позволяет освободить локальные ресурсы для приложений, критически важных для бизнеса. Приложение можно разместить в частном облаке, а при росте нагрузки осуществить выход в общедоступное облако.

### <a name="availability"></a>Доступность

Глобальное развертывание сопровождается рядом трудностей, которые могут быть обусловлены разными возможностями подключения и региональными законодательными требованиями. Разработчики могут разработать только одно приложение, а затем развертывать его, руководствуясь различными соображениями и различными требованиями. Разверните приложение в общедоступном облаке Azure, а его дополнительные экземпляры или компоненты – локально. Вы можете управлять трафиком между всеми экземплярами с помощью Azure.

### <a name="manageability"></a>Управляемость

#### <a name="a-single-consistent-development-approach"></a>Единый, согласованный подход к разработке

Azure и Azure Stack Hub позволяют использовать в организации согласованный набор средств разработки. Такая согласованность упрощает реализацию непрерывной интеграции и разработки (CI/CD). Многие приложения и службы, развертываемые в Azure или Azure Stack Hub, взаимозаменяемы и могут эффективно выполняться в любом из этих расположений.

С помощью гибридного конвейера CI/CD можно сделать следующее:

- инициировать новую сборку при фиксации кода в вашем репозитории;
- Автоматически развернуть созданный код в Azure для пользовательского приемочного тестирования.
- как только ваш код прошел тестирование, автоматически развернуть его в Azure Stack Hub.

### <a name="a-single-consistent-identity-management-solution"></a>Единое решение целостного управления идентификацией

Azure Stack Hub работает с Azure Active Directory и службами федерации Active Directory (AD FS). Azure Stack Hub может работать с Azure AD в сценариях с подключением. Для сред без подключения ADFS можно использовать в качестве отключенного решения. Субъекты-службы используются для предоставления доступа к приложениям, позволяя развертывать или настраивать ресурсы с помощью Azure Resource Manager.

### <a name="security"></a>Безопасность

#### <a name="ensure-compliance-and-data-sovereignty"></a>Обеспечение соответствия и независимости данных

Azure Stack Hub позволяет запускать одну и ту же службу в нескольких странах так же, как при использовании общедоступного облака. Развертывание одного приложения в центрах обработки данных в каждой стране позволяет обеспечить соблюдение требований и независимость данных. Это позволяет хранить персональные данные пользователей на территории их стран.

#### <a name="azure-stack-hub---security-posture"></a>Состояние безопасности в Azure Stack Hub

Уровень безопасности можно обеспечить только с помощью надежного непрерывного процесса обслуживания. По этой причине корпорация Майкрософт вкладывает средства в механизм оркестрации, который с легкостью применяет исправления и обновления для всей инфраструктуры.

К компонентам, поставляемым партнерами (изготовителями оборудования) Azure Stack Hub, например узлам жизненного цикла оборудования и работающему на них ПО, корпорация Майкрософт предъявляет те же требования в отношении уровня безопасности, что и к своим продуктам. Это сотрудничество гарантирует, что Azure Stack Hub предусматривает единый и надежный уровень безопасности во всей инфраструктуре. Клиенты, в свою очередь, могут создавать и защищать рабочие нагрузки приложений.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Использование субъектов-служб с помощью PowerShell, интерфейса командной строки и портала Azure

Чтобы предоставить доступ к ресурсам скрипту или приложению, вы можете настроить для вашего приложения идентификатор и выполнить для него проверку подлинности с помощью собственных учетных данных. Такое удостоверение называется субъектом-службой и позволяет сделать следующее:

- Назначить для удостоверения приложения разрешения, которые отличаются от ваших разрешений и ограничиваются только требованиями приложения.
- Использовать сертификат для аутентификации при выполнении автоматического сценария.

См. сведения о создании субъекта-службы и использовании сертификата для учетных данных в руководстве по [использованию удостоверения приложения для доступа к ресурсам](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Когда следует использовать этот шаблон

- Моя организация уже использует подход DevOps или намерена применить его в ближайшем будущем.
- Я хочу реализовать методы CI/CD для своей реализации Azure Stack Hub и общедоступного облака.
- Я хочу объединить конвейеры CI/CD, размещенные в облачных и локальных средах.
- Мне нужна возможность непрерывной разработки приложений в облачной или локальной службе.
- Я хочу единообразно применять навыки разработки для облачных и локальных приложений.
- Я использую Azure, но у меня есть разработчики, работающие в локальном облаке Azure Stack Hub.
- В моих локальных приложениях наблюдаются пиковые нагрузки при сезонных, циклических или произвольных колебаниях спроса.
- У меня есть локальные компоненты и я хочу использовать облако для их эффективного масштабирования.
- Мне нужна взможность масштабирования в облако, но при этом приложение должно выполняться преимущественно в локальной среде.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения по темам, описанным в этой статье:

- Обзор использования этого шаблона см. в видеоролике о [динамическом масштабировании приложений между центрами обработки данных и общедоступным облаком](https://www.youtube.com/watch?v=2lw8zOpJTn0).
- См. [рекомендации по проектированию гибридных приложений и ответы на дополнительные вопросы](overview-app-design-considerations.md).
- В этом шаблоне используется семейство продуктов Azure Stack, включая Azure Stack Hub. См. сведения обо [всех продуктах и решениях Azure Stack](/azure-stack).

Когда вы будете готовы протестировать пример решения, продолжите работу с руководством по [развертыванию решения для масштабирования в разных облаках (локальные данные)](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data). В этом руководстве содержатся пошаговые инструкции по развертыванию и тестированию компонентов.