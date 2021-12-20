## MassTransit и RabbitMQ

MassTransit и RabbitMQ используются для интеграции с другими системами ПИК. Наша система принимает и/или отправляет
сообщения.

### Получение сообщения от внешней системы

1. Внешняя система шлет сообщение в RabbitMQ:

<details>
  <summary>Пример сообщения</summary>

```json
{
  "messageSendDate": "2021-09-28T15:28:12.0516079+03:00",
  "messageId": "6e35282e-533a-43ac-ac2b-be085377d754",
  "messageDateTimeUtc": "2021-09-28T12:28:12.0516089Z",
  "messageUniqueId": "6e35282e-533a-43ac-ac2b-be085377d754",
  "batchMessageTotal": 1,
  "batchMessageNumber": 1,
  "processUniqueId": "65c71179-0bdf-4c51-8389-961b36d5c344",
  "destinationAddress": "rabbitmq://test.ru/pik/PIK.ESB.Messages.Implementation.BOP:IdpTeam",
  "messageType": [
    "urn:message:PIK.ESB.Messages.Implementation.BOP:IdpTeam",
    "urn:message:PIK.ESB.Messages.BOP:IIdpTeam",
    "urn:message:PIK.ESB.Messages.Implementation:Message",
    "urn:message:PIK.ESB.Messages:IMessage"
  ],
  "message": {
    "id": 37035,
    "guid": "ff483d42-a045-4eb8-a2ba-99a4b6e07d28",
    "objectId": 14659,
    "livingComplexId": 77,
    "roleId": 39,
    "employeeId": 23890,
    "employeeGuid": "71345723-99b7-11e6-aa10-001ec9d8cb21",
    "globalId": "7552f2a8-e52e-4540-9b09-855000b92482",
    "status": "Утверждено",
    "dateStart": "2021-09-25T00:00:00",
    "individualId": 23657,
    "created": "2021-09-28T15:28:12.033",
    "modified": "2021-09-28T15:28:12.033",
    "bopId": "3cef3fc8-edd2-40de-9831-5a2e521c2b39",
    "messageSendDate": "2021-09-28T15:28:12.0516079+03:00",
    "messageId": "6e35282e-533a-43ac-ac2b-be085377d754",
    "messageDateTimeUtc": "2021-09-28T12:28:12.0516089Z",
    "messageUniqueId": "6e35282e-533a-43ac-ac2b-be085377d754",
    "batchMessageTotal": 1,
    "batchMessageNumber": 1,
    "processUniqueId": "65c71179-0bdf-4c51-8389-961b36d5c344"
  }
}
```

</details>

| RabbitMQ для feature стендов | RabbitMQ для test стенда | RabbitMQ для prod |
| --- | --- | --- |
| https://rabbit.dev.svc.pik-digital.ru/#/ | https://rabbit.test.svc.pik-digital.ru/#/| https://rabbit.svc.pik-digital.ru/#/ |

2. Автоматически создаются exchange c названиями `PIK.ESB.Messages.Implementation.BOP:IdpTeam`
   и `PIK.ESB.Messages.BOP:IIdpTeam`, которые связаны друг с другом. Названия exchange можно найти в сообщении в
   поле `messageType`.

3. Сообщение шлется в exchange с названием `PIK.ESB.Messages.Implementation.BOP:IdpTeam`. Список exchange находится во
   вкладке `Exchanges` [https://rabbit.dev.svc.pik-digital.ru/#/exchanges](https://rabbit.dev.svc.pik-digital.ru/#/exchanges):

![mr-list](images/exchanges.png)

Чтобы найти exchange, нужно в поле `Filter` ввести полное название или его часть, к примеру `IdpTeam`:

![mr-list](images/find-exchange.png)

4. Сообщение из `PIK.ESB.Messages.Implementation.BOP:IdpTeam` попадает в `PIK.ESB.Messages.BOP:IIdpTeam`, тк они
   связаны. Посмотреть связь можно на странице первого exchange, для этого надо тыкнуть на его название в табличке:

![mr-list](images/open-exchange.png)

Связи можно посмотреть в `Bindings`:

![mr-list](images/implementation-bindings.png)

После чего тыкнуть на `PIK.ESB.Messages.BOP:IIdpTeam`:

![mr-list](images/open-interface-exchange.png)

5. Далее, сообщение из `PIK.ESB.Messages.BOP:IIdpTeam` попадает во все связанные с ней дальше exchange, которые связаны
   с очередями (queue):

![mr-list](images/interface-exchange-bindings.png)

К примеру exchange `tracker-feature-74374` связан с очередью (queue), которая имеет такое же
название `tracker-feature-74374`, как и exchange:

![mr-list](images/tracker-feature-74374-exchange.png)

6. Дальше сообщение из очереди `tracker-feature-74374` попадает слушателю (consumer):

![mr-list](images/tracker-feature-74374-queue.png)

Слушатель запускается на стороне системы, которая хочет получить сообщение. В данном случае, это feature стенд задачи
74374 в ТН. Скрипт слушателя уже не относится к программному обеспечению RabbitMQ и пишется силами разработчиков на
стороне системы, которая хочет получить сообщение.

При запуске слушателя, очередь `tracker-feature-74374` и exchange `tracker-feature-74374` создаются и связываются друг с
другом автоматически. Связь exchange `tracker-feature-74374` с exchange `PIK.ESB.Messages.BOP:IIdpTeam` так же создается
автоматически. Если на момент запуска слушателя exchange `PIK.ESB.Messages.BOP:IIdpTeam` и
`PIK.ESB.Messages.Implementation.BOP:IdpTeam` еще не существуют, то слушатель создает и связывает их друг с другом.

7. Скрипт слушателя в ТН определяет тип сообщения по `messageType` из сообщения

### Схема получения сообщения

![mr-list](images/masstransit-message-schema.png)

### Технические детали MassTransit и RabbitMQ в ТехНадзоре

Классы сообщений для прослушивания и публикации находятся в отдельном
пакете [pik/pik-esb-messages](https://git.pik.ru/php/pik-esb-messages).

Отправка сообщения из ТН осуществляется несколькими способами:

* через консольную команду [masstransit:publish](#masstransit-publish)
* при срабатывании какого-нибудь события (event) в приложении ТН. К примеру, при создании чек-листа через апи,
  срабатывает событие CheckListSaveEvent, слушатель этого события создает работу CheckListPublishJob, в которой
  происходит отправка сообщения. При публикации через консольную команду создается та же работа, что при публикации при
  срабатывании события.
* при вызове api методов, к примеру `POST /issues/statuses/push`, который публикует статусы замечаний.

### Консольная команда `masstransit:publish` для отправки сообщений: <a id="masstransit-publish"/>

Если вызвать `masstransit:publish`, то выбрать тип публикуемой сущности можно через интерактивный выбор прямо в консоли.

Или же можно передать к примеру параметр `--entity=check_lists`, чтобы опубликовать чек-листы. Ниже примеры вызова
команды со всеми возможными `--entity=`:

* `masstransit:publish --entity=check_lists`
* `masstransit:publish --entity=issues`
* `masstransit:publish --entity=issues_attachments`
* `masstransit:publish --entity=works`
* `masstransit:publish --entity=object_types`
* `masstransit:publish --entity=construction_elements`
* `masstransit:publish --entity=check_list_realizations`
* `masstransit:publish --entity=elements`
* `masstransit:publish --entity=issue_categories`
* `masstransit:publish --entity=element_issue_categories`
* `masstransit:publish --entity=issue_statuses`
* `masstransit:publish --entity=schedule_object_meeting`

По умолчанию, отправляются записи, у которых `updated_at` >= последней даты отправки сообщений.

Для отправки всех записей нужно передать параметр `--force`.

Параметр `--from` нужен для отправки записей, у которых `updated_at` >= даты указанной в параметре. Пример:

```bash
masstransit:publish --entity=object_types --from="2020-20-20 20:20:20"
```

### Консольная команда для прослушивания очереди для получения сообщений от других систем ПИК:

```bash
masstransit:consume
```

Активные слушатели указываются в `env` переменной приложения `MASSTRANSIT_ACTIVE_CONSUME_HANDLERS` через запятую без
пробелов (не путать с переменной, которая передается в `review:start`).

Пример: `MASSTRANSIT_ACTIVE_CONSUME_HANDLERS=ExternalUserConsumeHandler,IssueAttachmentConsumeHandler`.

Все возможные слушатели перечислены [ниже](#все-возможные-слушатели).

**Так же через MassTransit происходит получение объектов от МДС. Перед получением всех объектов следует выполнить
команду:**

```bash
import:prepare_before_full_objects
```

Которая деактивирует все МДС объекты и удалит дубли секций и этажей.

### Все возможные слушатели

| AUTH |
| --- |
| ExternalUserConsumeHandler |
| **БИМ** |
| IssueAttachmentConsumeHandler |
| IssueConsumeHandler |
| **БОП** |
| IdpTeamConsumeHandler |
| **ЦРМ** |
| ChangeAppointmentConsumeHandler |
| ChangeStatusAppConsumeHandler |
| SendRemarkConsumeHandler |
| SendRequestGoConsumeHandler |
| **МДС** |
| AccessPlanConsumeHandler |
| ElevatorConsumeHandler |
| FloorConsumeHandler |
| LivingComplexConsumeHandler |
| ObjectCategoryConsumeHandler |
| ObjectConsumeHandler |
| ObjectSeriesConsumeHandler |
| ObjectTypeConsumeHandler |
| RealEstateConsumeHandler |
| RealEstateKindConsumeHandler |
| SectionConsumeHandler |
| **Календарь вызовов** |
| MeetingConsumeHandler |

### Отслеживание процесса публикации и получения сообщений

Публикация и получение сообщений происходит всегда через job, все они логируются в kibana с текстом 'Masstransit log', в
input пишется получаемое сообщение, в output - отправляемое.

Если при выполнении job произошла ошибка, то она пишется в таблицу failed_jobs. Данные в этой таблице хранятся 14 дней.

Так же, почти любые ошибки можно найти в kibana.

Еще ошибка может возникнуть до выполнения самой job, в этом случае ошибки при прослушивании попадают в очередь rabbitmq
c суффиксом _error и так же отправляются в sentry, а ошибки при публикации не шлются в _error очередь.

**Совет: проще и быстрее искать ошибки и прочие логи в kibana.**

### Стенды

По умолчанию на feature стендах не работает прослушивание и отправка сообщений.

Чтобы включить прослушивание, нужно в `review:start` передать два параметра: `RABBITMQ` равное `true` и
`MASSTRANSIT_CONSUME_HANDLERS` равное названиям слушателей через запятую (без пробелов).

Чтобы включить отправку, нужно передать `RABBITMQ` равное `true`.

Как передавать переменные в `review:start` описано [тут](stands.md#стенды).

#### Описание переменных

| Переменная | Описание | Дефолт | Пример |
| --- | --- | --- | --- |
| `MASSTRANSIT_CONSUME_HANDLERS` | Названия слушателей через запятую (без пробелов) | На feature пусто, на остальных [все возможные слушатели](#все-возможные-слушатели) | `ExternalUserConsumeHandler,IssueAttachmentConsumeHandler` |
| `RABBITMQ` | Булево значение, если `true`, то запускается rabbitmq consumer и publisher | На feature равно `false`, т.к. не во всех задачах нужна интеграция по шине rabbitmq | `true` |

#### Описание всех переменных [тут](stands.md#все-переменные-feature-стендов)