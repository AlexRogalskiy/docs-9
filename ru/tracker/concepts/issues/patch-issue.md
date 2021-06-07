# Редактировать задачу

Запрос позволяет внести изменения в задачу.

{% note warning %}

Статус задачи может быть изменен только с помощью запроса [<q>Выполнить переход в статус</q>](new-transition.md).

{% endnote %}

## Формат запроса {#section_rnm_x4j_p1b}

Перед выполнением запроса [получите доступ к API](../access.md).

Чтобы отредактировать задачу, используйте HTTP-запрос с методом `PATCH`. Параметры запроса передаются в его теле в формате JSON.

```json
PATCH /{{ ver }}/issues/<issue-id>Host: {{ host }}
Authorization: OAuth <OAuth-токен>
{{ org-id }}
{
   Тело запроса в формате JSON
}
```

{% include [headings](../../../_includes/tracker/api/headings.md) %}

#### Параметры запроса {#req-get-params}

**Ресурс**

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  \<issue-id\> | Идентификатор или ключ задачи. | Число/Строка.

**Дополнительные параметры**

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  version | Версия задачи. Изменения вносятся только в текущую версию задачи. | Число.

#### Параметры тела запроса {#req-body-params}

**Дополнительные параметры**

Параметр | Описание | Тип данных
----- | ----- | -----
summary | Название задачи. | Строка.
[parent](#req-parent) | Родительская задача. | Объект или строка.
description | Описание задачи. | Строка.
[sprint](#req-sprint) | Блок с информацией о спринтах. | Массив объектов или строк.
[type](#req-type) | Тип задачи. | Объект, строка (если передается ключ типа задачи) или число (если передается идентификатор типа задачи).
[priority](#req-priority) | Приоритет задачи. | Объект, строка (если передается ключ приоритета) или число (если передается идентификатор приоритета).
[followers](#req-followers) | Идентификаторы или логины наблюдателей задачи. | Массив объектов или строк.

**Поля объекта** `parent` {#req-parent}

Параметр | Описание | Тип данных
----- | ----- | -----
id | Идентификатор родительской задачи. | Строка.
key | Ключ родительской задачи. | Строка.

**Поля объекта** `sprint` {#req-sprint}

Параметр | Описание | Тип данных
----- | ----- | -----
id | Идентификатор спринта. Информацию о спринте можно получить при помощи [запроса](../../get-sprints.md). | Число.

**Поля объекта** `type` {#req-type}

Параметр | Описание | Тип данных
----- | ----- | -----
id | Идентификатор типа задачи. | Строка.
key | Ключ типа задачи. | Строка.

**Поля объекта** `priority` {#req-priority}

Параметр | Описание | Тип данных
----- | ----- | -----
id | Идентификатор приоритета. | Строка.
key | Ключ приоритета. | Строка.

**Поля объекта** `followers` {#req-followers}

Параметр | Описание | Тип данных
----- | ----- | -----
id | Идентификатор сотрудника. | Строка.

#### Формат тела запроса {#req-body-format}

В теле запроса передается JSON-объект с идентификаторами изменяемых полей задачи и их значениями.

- Чтобы добавить или удалить значение из массива, используйте команды `add` и `remove`:

  - ```json
        {
            "followers": { "add": ["<id сотрудника1>", "<id сотрудника2>"] }
        }
    ```

{% note info %}

Команда `add` добавляет новые значения в массив. Чтобы перезаписать массив (удалить старые значения и добавить новые), используйте команду `set`.

{% endnote %}

- Чтобы обнулить значение поля, укажите значение `null`. Чтобы обнулить массив, используйте пустой массив `[]`. Отдельные значения в массиве можно изменить с помощью команд `target` и `replacement`:

  - `{"followers": null}`
  - ```json
      {
        "followers": {
          "replace": [
              {"target": "<идентификатор1>", "replacement": "<идентификатор2>"},
              {"target": "<идентификатор3>", "replacement": "<идентификатор4>"}]
        }
      }
      ```

- Например, чтобы изменить тип задачи на <q>Ошибка</q>, используйте один из способов:

  - `{"type": 1}`
  - `{"type": "bug"}`
  - ```json
      {
          "type": { "id": "1" }
      }
      ```
  - ```json
      {
          "type": { "name": "Ошибка" }
      }
      ```
  - ```json
      {
          "type": {"set": "bug"}
      }
      ```

> Пример 1: Изменить название, описание, тип и приоритет задачи.
> 
> - Используется HTTP-метод PATCH.
> - Редактируется задача TEST-1.
> - Новый тип задачи: <q>Ошибка</q>.
> - Новый приоритет задачи: <q>Низкий</q>.
>
> ```
> PATCH /v2/issues/TEST-1
> Host: {{ host }}
> Authorization: OAuth <OAuth-токен>
> {{ org-id }}
> 
> {
>     "summary": "Новое название задачи",
>     "description": "Новое описание задачи",
>     "type": {
>         "id": "1",
>         "key": "bug"
>         },
>     "priority": {
>         "id": "2",
>         "key": "minor"
>         }
> }
> ```

> Пример 2: Изменить родительскую задачу, добавить в спринты, добавить наблюдателей.
> 
> - Используется HTTP-метод PATCH.
> - Редактируется задача TEST-1.
> - Новая родительская задача: TEST-2.
> - Задача добавляется в спринты с идентификаторами 3 и 2. Спринты должны быть на разных досках. 
> - Добавлены наблюдатели с логинами `userlogin-1` и `userlogin-2`.
> 
> ```
> PATCH /v2/issues/TEST-1
> Host: {{ host }}
> Authorization: OAuth <OAuth-токен>
> {{ org-id }}
> 
> {
>     "parent": {
>         "key": "TEST-2"},
>     "sprint": [{"id": "3"}, {"id": "2"}],
>     "followers": {
>         "add": ["userlogin-1", "userlogin-2"]
>         }
> }
> ```

## Формат ответа {#section_xc3_53j_p1b}

{% list tabs %}

- Запрос выполнен успешно

  В случае успешного выполнения запроса API возвращает ответ с кодом `200 OK`.

  Тело ответа содержит информацию об отредактированной задаче в формате JSON.

    ```json
    {
        "self": "{{ host }}/v2/issues/TREK-9844",
        "id": "593cd211ef7e8a332414f2a7",
        "key": "TREK-9844",
        "version": 7,
        "lastCommentUpdatedAt": "2017-07-18T13:33:44.291+0000",
        "summary": "subtask",
        "parent": {
            "self": "{{ host }}/v2/issues/JUNE-2",
            "id": "593cd0acef7e8a332414f28e",
            "key": "JUNE-2",
            "display": "Task"
            },
        "aliases": [
                "JUNE-3"
            ],
    
        "updatedBy": {
            "self": "{{ host }}/v2/users/1120000000016876",
            "id": "<id сотрудника>",
            "display": "<отображаемое имя сотрудника>"
            },
        "description": "<#<html><head></head><body><div>test</div><div>&nbsp;</div><div>&nbsp;</div> </body></html>#>",    
        "sprint": [
                {
            "self": "{{ host }}/v2/sprints/5317",
            "id": "5317",
            "display": "спринт1"
                }
            ],
        "type": {
            "self": "{{ host }}/v2/issuetypes/2",
            "id": "2",
            "key": "task",
            "display": "Задача"
            },
        "priority": {
            "self": "{{ host }}/v2/priorities/2",
            "id": "2",
            "key": "normal",
            "display": "Средний"
            },
    
        "createdAt": "2017-06-11T05:16:01.339+0000",
        "followers": [
            {
            "self": "{{ host }}/v2/users/1120000000016876",
            "id": "<id сотрудника>",
            "display": "<отображаемое имя сотрудника>"
            }
            ],
        "createdBy": {
            "self": "{{ host }}/v2/users/1120000000049224",
            "id": "<id сотрудника>",
            "display": "<отображаемое имя сотрудника>"
            },
        "votes": 0,
        "assignee": {
            "self": "{{ host }}/v2/users/1120000000049224",
            "id": "<id сотрудника>",
            "display": "<отображаемое имя сотрудника>"
            },
        "queue": {
            "self": "{{ host }}/v2/queues/TREK",
            "id": "111",
            "key": "TREK",
            "display": "Трек"
            },
        "updatedAt": "2017-07-18T13:33:44.291+0000",
        "status": {
            "self": "{{ host }}/v2/statuses/1",
            "id": "1",
            "key": "open",
            "display": "Открыт"
            },
        "previousStatus": {
            "self": "{{ host }}/v2/statuses/2",
            "id": "2",
            "key": "resolved",
            "display": "Решен"
            },
        "favorite": false
    }
    ```

  #### Параметры ответа {#answer-params}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Адрес ресурса API, который содержит информацию о задаче. | Строка.
  id | Идентификатор задачи. | Строка.
  key | Ключ задачи. | Строка.
  version | Версия задачи. Каждое изменение параметров задачи увеличивает номер версии. | Число.
  lastCommentUpdatedAt | Дата и время последнего добавленного комментария. | Строка.
  summary | Название задачи. | Строка.
  [parent](#parent) | Объект с информацией о родительской задаче. | Объект.
  aliases | Массив с информацией об альтернативных ключах задачи. | Массив строк.
  [updatedBy](#updated-by) | Объект с информацией о последнем сотруднике, изменявшим задачу. | Объект.
  description | Описание задачи. | Строка.
  [sprint](#sprint) | Массив объектов с информацией о спринте. | Массив объектов.
  [type](#type) | Объект с информацией о типе задачи. | Объект.
  [priority](#priority) | Объект с информацией о приоритете. | Объект.
  createdAt | Дата и время создания задачи. | Строка.
  [followers](#followers) | Массив объектов с информацией о наблюдателях задачи. | Массив объектов.
  [createdBy](#created-by) | Объект с информацией о создателе задачи. | Объект.
  votes | Количество голосов за задачу. | Число.
  [assignee](#assignee) | Объект с информацией об исполнителе задачи. | Объект.
  [queue](#queue) | Объект с информацией об очереди задачи. | Объект.
  updatedAt | Дата и время последнего обновления задачи. | Строка.
  [status](#status) | Объект с информацией о статусе задачи. | Объект.
  [previousStatus](#previous-status) | Объект с информацией о предыдущем статусе задачи. | Объект.
  favorite | Признак избранной задачи:<ul><li>`true` — пользователь добавил задачу в избранное;</li><li>`false` — задача не добавлена в избранное.</li></ul> | Логический.

  **Поля объекта** `parent` {#parent}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на задачу. | Строка.
  id | Идентификатор задачи. | Строка.
  key | Ключ задачи. | Строка.
  display | Отображаемое название задачи. | Строка.

  **Поля объекта** `updatedBy` {#updated-by}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на пользователя. | Строка.
  id | Идентификатор пользователя. | Строка.
  display | Отображаемое имя пользователя. | Строка.

  **Поля объектов массива** `sprint` {#sprint}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на спринт. | Строка.
  id | Идентификатор спринта. | Строка.
  display | Отображаемое название спринта. | Строка.

  **Поля объекта** `type` {#type}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на тип задачи. | Строка.
  id | Идентификатор типа задачи. | Строка.
  key | Ключ типа задачи. | Строка.
  display | Отображаемое название типа задачи. | Строка.

  **Поля объекта** `priority` {#priority}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на тип приоритета. | Строка.
  id | Идентификатор приоритета. | Строка.
  key | Ключ приоритета. | Строка.
  display | Отображаемое название приоритета. | Строка.

  **Поля объектов массива** `followers` {#followers}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на пользователя. | Строка.
  id | Идентификатор пользователя. | Строка.
  display | Отображаемое имя пользователя. | Строка.

  **Поля объекта** `createdBy` {#created-by}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на пользователя. | Строка.
  id | Идентификатор пользователя. | Строка.
  display | Отображаемое имя пользователя. | Строка.

  **Поля объекта** `assignee` {#assignee}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на пользователя. | Строка.
  id | Идентификатор пользователя. | Строка.
  display | Отображаемое имя пользователя. | Строка.

  **Поля объекта** `queue` {#queue}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на очередь. | Строка.
  id | Идентификатор очереди. | Строка.
  key | Ключ очереди. | Строка.
  display | Отображаемое название очереди. | Строка.

  **Поля объекта** `status` {#status}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на статус. | Строка.
  id | Идентификатор статуса. | Строка.
  key | Ключ статуса. | Строка.
  display | Отображаемое название статуса. | Строка.

  **Поля объекта** `previousStatus` {#previous-status}

  Параметр | Описание | Тип данных
  ----- | ----- | -----
  self | Ссылка на статус. | Строка.
  id | Идентификатор статуса. | Строка.
  key | Ключ статуса. | Строка.
  display | Отображаемое название статуса. | Строка.

- Запрос выполнен с ошибкой

  Если запрос не был успешно обработан, API возвращает ответ с кодом ошибки:

    401
    :   Пользователь не авторизован. Проверьте, были ли выполнены действия, описанные в разделе [{#T}](../access.md).

    403
    :   У вас не хватает прав на выполнение этого действия. Наличие прав можно перепроверить в интерфейсе {{ tracker-name }} — для выполнения действия при помощи API и через интерфейс требуются одинаковые права.

    404
    :   Запрошенный объект не был найден. Возможно, вы указали неверное значение идентификатора или ключа объекта.

    409
    :   При редактировании объекта возник конфликт. Возможно, ошибка возникла из-за неправильно указанной версии изменений.

{% endlist %}
