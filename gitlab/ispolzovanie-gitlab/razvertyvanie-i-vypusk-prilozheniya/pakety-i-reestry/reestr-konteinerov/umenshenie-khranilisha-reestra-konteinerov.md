# Уменьшение хранилища реестра контейнеров

Реестры контейнеров могут со временем увеличиваться в размерах, если вы не управляете использованием реестра. Например, если вы добавите большое количество образов или тегов:

* Получение списка доступных тегов или образов становится медленнее.
* Они занимают много места на сервере.

Вам следует удалить ненужные образы и теги и настроить [политику очистки](umenshenie-khranilisha-reestra-konteinerov.md#politika-ochistki) для автоматического управления использованием реестра контейнеров.

## Проверка использования хранилища реестра контейнеров

На странице **Usage Quotas** (**Settings > Usage Quotas > Storage**) отображается использование хранилища для пакетов. На этой странице рассказывается об использовании реестра контейнеров, который доступен только на GitLab.com. Измерение использования возможно только в новой версии GitLab Container Registry, поддерживаемой базой данных метаданных, которая [доступна на GitLab.com](https://gitlab.com/groups/gitlab-org/-/epics/5523) начиная с `GitLab 15.7`. Информацию о запланированной доступности самоуправляемых экземпляров см. в [epic 5521](https://gitlab.com/groups/gitlab-org/-/epics/5521).

## Политика очистки

> * В GitLab 13.2 [переименовано](https://gitlab.com/gitlab-org/gitlab/-/issues/218737) из "expiration policy" в "cleanup policy".
> * [Требуемые разрешения](https://gitlab.com/gitlab-org/gitlab/-/issues/350682) изменены с разработчика **devloper** на сопровождающего **maintainer** в GitLab 15.0.

Политика очистки — это запланированное задание, которое можно использовать для удаления тегов из реестра контейнеров. Для проекта, в котором оно определено, теги, соответствующие шаблону регулярного выражения, удаляются. Нижележащие слои и изображения остаются.

Чтобы удалить нижележащие слои и изображения, не связанные ни с какими тегами, администраторы могут использовать <mark style="color:purple;">сбор мусора</mark> с ключом `-m`.

### Включить политику очистки

Вы можете запускать политики очистки для всех проектов, за исключением следующих:

* Для самоуправляемых экземпляров GitLab проект должен быть создан в GitLab 12.8 или более поздней версии. Однако администратор может включить политику очистки для всех проектов (даже созданных до GitLab 12.8) в <mark style="color:purple;">настройках приложения GitLab</mark>, установив для **container\_expiration\_policies\_enable\_historic\_entries** значение `true`. Альтернативно вы можете выполнить следующую команду в <mark style="color:purple;">консоли Rails</mark>:

```
ApplicationSetting.last.update(container_expiration_policies_enable_historic_entries: true)
```

Включение политик очистки во всех проектах может повлиять на производительность, особенно если вы используете [внешний реестр](umenshenie-khranilisha-reestra-konteinerov.md#ispolzovanie-s-vneshnimi-reestrami-konteinerov).

{% hint style="warning" %}
По соображениям производительности включенные политики очистки автоматически отключаются для проектов на GitLab.com, у которых нет образа контейнера.
{% endhint %}

### Как работает политика очистки

Политика очистки собирает все теги в реестре контейнеров и исключает теги до тех пор, пока не останутся единственные теги, которые вы хотите удалить.

Политика очистки ищет изображения по имени тега. Поддержка полного сопоставления пути отслеживается в выпуске [281071](https://gitlab.com/gitlab-org/gitlab/-/issues/281071).

Политика очистки:

1. Собирает все теги для данного репозитория в список.
2. Исключает тег с именем **latest**.
3. Оценивает **name\_regex** (теги с истекающим сроком действия), исключая несовпадающие имена.
4. Исключает любые теги, соответствующие значению **name\_regex\_keep** (теги, которые необходимо сохранить).
5. Исключает любые теги, которые не имеют манифеста (не являются частью параметров пользовательского интерфейса).
6. Упорядочивает оставшиеся теги по **created\_date**.
7. Исключает N тегов на основе значения **keep\_n** (количество сохраняемых тегов).
8. Исключает теги, более поздние, чем значение **old\_than** (интервал срока действия).
9. Удаляет оставшиеся теги в списке из реестра контейнеров.

{% hint style="warning" %}
На GitLab.com время выполнения политики очистки ограничено. Некоторые теги могут остаться в реестре контейнеров после выполнения политики. При следующем запуске политики оставшиеся теги будут включены. Для удаления всех тегов может потребоваться несколько запусков.
{% endhint %}

{% hint style="warning" %}
Самоуправляемые установки GitLab поддерживают сторонние реестры контейнеров, соответствующие спецификации [Docker Registry HTTP API V2](https://docs.docker.com/registry/spec/api/). Однако эта спецификация не включает операцию удаления тега. Поэтому GitLab использует обходной путь для удаления тегов при взаимодействии со сторонними реестрами контейнеров. Дополнительную информацию см. в выпуске [15737](https://gitlab.com/gitlab-org/gitlab/-/issues/15737). Из-за возможных вариантов реализации этот обходной путь не гарантирует одинаковой предсказуемой работы со всеми сторонними реестрами. Если вы используете реестр контейнеров GitLab, этот обходной путь не требуется, поскольку мы реализовали специальную операцию удаления тега. В этом случае можно ожидать, что политики очистки будут последовательными и предсказуемыми.
{% endhint %}

#### Пример рабочего процесса политики очистки

Взаимодействие между правилами сохранения и удаления для политики очистки может быть сложным. Например, в проекте с такой конфигурацией политики очистки:

* **Keep the most recent**: 1 тег на имя образа
* **Keep tags matching**: `production-.*`
* **Remove tags older than**: 7 дней
* **Remove tags matching**: `.*`

И репозиторий-контейнер с этими тегами:

* `latest`, опубликовано 2 часа назад
* `production-v44`, опубликовано 3 дня назад
* `production-v43`, опубликовано 6 дней назад
* `production-v42`, опубликовано 11 дней назад
* `dev-v44`, опубликовано 2 дня назад
* `dev-v43`, опубликовано 5 дней назад
* `dev-v42`, опубликовано 10 дней назад
* `v44`, опубликовано вчера
* `v43`, опубликовано 12 дней назад
* `v42`, опубликовано 20 дней назад

В этом примере теги, которые будут удалены при следующем запуске очистки, — это `dev-v42`, `v43` и `v42`. Вы можете интерпретировать правила как применимые с таким приоритетом:

* Правила **сохранения** имеют наивысший приоритет. Теги необходимо сохранять, если они соответствуют **какому-либо** правилу.
  * Необходимо сохранять тег **latest**, поскольку последние теги сохраняются **всегда**.
  * Теги `production-v44`, `production-v43` и `production-v42` необходимо сохранить, поскольку они соответствуют правилу **Keep tags matching**.
  * Тег `v44` необходимо сохранить, поскольку он является самым последним и соответствует правилу **Keep the most recent**.
* Правила **удаления** имеют более низкий приоритет, а теги удаляются только в том случае, если **все** правила совпадают. Для тегов, не соответствующих каким-либо правилам сохранения (`dev-44`, `dev-v43`, `dev-v42`, `v43` и `v42`):
  * `dev-44` и `dev-43` **не соответствуют** тегам **Remove tags older than** и сохраняются.
  * `dev-v42`, `v43` и `v42` соответствуют правилам **Remove tags older than** и **Remove tags matching**, поэтому эти три тега можно удалить.

### Создание политики очистки

Вы можете создать политику очистки в [API](umenshenie-khranilisha-reestra-konteinerov.md#ispolzuite-api-politiki-ochistki) или пользовательском интерфейсе.

Чтобы создать политику очистки в пользовательском интерфейсе:

1. Для вашего проекта перейдите в **Settings > Packages and registries**.
2. В разделе **Cleanup policies** выберите **Set cleanup rules**.
3. Заполните поля:

<table><thead><tr><th width="234">Поле</th><th>Описание</th></tr></thead><tbody><tr><td><strong>Toggle</strong></td><td>Включите или отключите политику</td></tr><tr><td><strong>Run cleanup</strong></td><td>Как часто должна выполняться политика</td></tr><tr><td><strong>Keep the most recent</strong></td><td>Сколько тегов всегда сохранять для каждого образа</td></tr><tr><td><strong>Keep tags matching</strong></td><td>Шаблон регулярного выражения, определяющий, какие теги сохранять. Тег <strong>latest</strong> всегда сохраняется. Для всех тегов используйте <code>.*</code>. См. другие <a href="umenshenie-khranilisha-reestra-konteinerov.md#primery-shablonov-regulyarnykh-vyrazhenii">примеры шаблонов регулярных выражений</a>.</td></tr><tr><td><strong>Remove tags older than</strong></td><td>Удаляйте только теги старше X дней.</td></tr><tr><td><strong>Remove tags matching</strong></td><td>Шаблон регулярного выражения, определяющий, какие теги следует удалить. Это значение не может быть пустым. Для всех тегов используйте <code>.*</code>. См. другие <a href="umenshenie-khranilisha-reestra-konteinerov.md#primery-shablonov-regulyarnykh-vyrazhenii">примеры шаблонов регулярных выражений</a>.</td></tr></tbody></table>

4. Выберите **Save**

Политика выполняется с выбранным вами запланированным интервалом.

{% hint style="info" %}
Если вы отредактируете политику и снова выберите **Save**, интервал будет сброшен.
{% endhint %}

### Примеры шаблонов регулярных выражений

Политики очистки используют шаблоны регулярных выражений, чтобы определить, какие теги следует сохранить или удалить, как в пользовательском интерфейсе, так и в API.

Шаблоны регулярных выражений автоматически заключаются в привязки `\A` и `\Z`. Поэтому вам не нужно включать токены `\A`, `\Z`, `^` или `$` в шаблоны регулярных выражений.

Вот несколько примеров шаблонов регулярных выражений, которые вы можете использовать:

* Соответствует всем тегам:

```regex
.*
```

Этот шаблон является значением по умолчанию для регулярного выражения срока действия.

* Соответствует тегам, начинающимся с буквы **v**:

```regex
v.+
```

* Соответствует только тегу с именем **main**:

```regex
main
```

* Соответствует тегам, которые имеют имена или начинаются с **release**:

```regex
release.*
```

* Соответствует тегам, которые начинаются с буквы **v**, называются **main** или начинаются с **release**:

```regex
(?:v.+|main|release.*)
```

### Установите ограничения очистки для экономии ресурсов

> * [Представлено](https://gitlab.com/gitlab-org/gitlab/-/issues/288812) в GitLab 13.9 <mark style="color:purple;">с флагом</mark> **container\_registry\_expiration\_policies\_throttling**. По умолчанию отключено.
> * [Включено по умолчанию](https://gitlab.com/groups/gitlab-org/-/epics/2270) в GitLab 14.9.
> * [Удален](https://gitlab.com/gitlab-org/gitlab/-/merge\_requests/84996) флаг функции **container\_registry\_expiration\_policies\_throttling** в GitLab 15.0.

Политики очистки выполняются в фоновом режиме. Этот процесс сложен, и в зависимости от количества удаляемых тегов его завершение может занять некоторое время.

Чтобы предотвратить нехватку ресурсов сервера, вы можете использовать следующие настройки приложения:

* **container\_registry\_expiration\_policies\_worker\_capacity**: максимальное количество рабочих процессов очистки, работающих одновременно. Это значение должно быть больше или равно `0`. Следует начать с меньшего числа и увеличивать его после мониторинга ресурсов, используемых фоновыми рабочими процессами. Чтобы удалить все воркеры и не выполнять политики очистки, установите для этого параметра значение `0`. Значение по умолчанию — `4`.
* **container\_registry\_delete\_tags\_service\_timeout**: максимальное время (в секундах), которое может потребоваться процессу очистки для удаления пакета тегов. Значение по умолчанию — `250`.
* **container\_registry\_cleanup\_tags\_service\_max\_list\_size**: максимальное количество тегов, которые можно удалить за одно выполнение. Дополнительные теги необходимо удалить при другом исполнении. Вам следует начать с небольшого числа и увеличивать его после проверки правильности удаления образов контейнеров. Значение по умолчанию — `200`.
* **container\_registry\_expiration\_policies\_caching**: включить или отключить кэширование меток времени создания тегов во время выполнения политик. Кэшированные метки времени хранятся в <mark style="color:purple;">Redis</mark>. Включено по умолчанию.

Для самоуправляемых экземпляров эти настройки можно обновить в <mark style="color:purple;">консоли Rails</mark>:

```
  ApplicationSetting.last.update(container_registry_expiration_policies_worker_capacity: 3)
```

Они также доступны в <mark style="color:purple;">зоне администратора</mark>:

1. На левой боковой панели выберите **Search or go to**.
2. Выберите **Admin Area**.
3. На левой боковой панели выберите **Settings > CI/CD**.
4. Разверните **Container Registry**.

### Используйте API политики очистки

Вы можете устанавливать, обновлять и отключать политики очистки с помощью API GitLab.

Примеры:

* Выберите все теги, сохраните хотя бы 1 тег для каждого образа, очистите все теги старше 14 дней, запускайте один раз в месяц, сохраните все образы с именем **main** и включите политику:

```bash
curl --request PUT --header 'Content-Type: application/json;charset=UTF-8' \
     --header "PRIVATE-TOKEN: <your_access_token>" \
     --data-binary '{"container_expiration_policy_attributes":{"cadence":"1month","enabled":true,"keep_n":1,"older_than":"14d","name_regex":".*","name_regex_keep":".*-main"}}' \
     "https://gitlab.example.com/api/v4/projects/2"

```

Допустимые значения **cadence** при использовании API:

* `1d` (каждый день)
* `7d` (каждую неделю)
* `14d` (каждые 2 недели)
* `1month` (каждый месяц)
* `3month` (каждые 3 месяца)

Допустимые значения для **keep\_n** (количество тегов, сохраняемых на имя образа) при использовании API:

* 1
* 5
* 10
* 25
* 50
* 100

Допустимые значения для **old\_than** (дней до автоматического удаления тегов) при использовании API:

* `7d`
* `14d`
* `30d`
* `90d`

Дополнительную информацию см. в документации API: <mark style="color:purple;">Редактировать API проекта</mark>.

### Использование с внешними реестрами контейнеров

При использовании <mark style="color:purple;">внешнего реестра контейнеров</mark> выполнение политики очистки в проекте может иметь некоторые риски для производительности. Если в проекте применяется политика удаления тысяч тегов, фоновые задания GitLab могут быть зарезервированы или полностью завершиться сбоем. Для проектов, созданных до GitLab 12.8, следует включать политики очистки контейнеров только в том случае, если количество очищаемых тегов минимально.

## Дополнительные возможности сокращения объема хранилища реестра контейнеров

Вот несколько других вариантов, которые вы можете использовать для уменьшения объема хранилища реестра контейнеров, используемого вашим проектом:

* Используйте [пользовательский интерфейс](udalenie-obrazov.md#ispolzovanie-interfeisa-gitlab) GitLab для удаления отдельных тегов образов или всего репозитория, содержащего все теги.
* Используйте API для <mark style="color:purple;">удаления отдельных тегов образов</mark>.
* Используйте API, чтобы <mark style="color:purple;">удалить весь репозиторий реестра контейнеров, содержащий все теги</mark>.
* Используйте API для <mark style="color:purple;">массового удаления тегов хранилища реестра</mark>.

## Устранение неполадок с политиками очистки

### Что-то пошло не так при обновлении политики очистки

Если вы видите это сообщение об ошибке `Something went wrong while updating the cleanup policy.`, проверьте шаблоны регулярных выражений, чтобы убедиться, что они действительны.

GitLab использует [синтаксис RE2](https://github.com/google/re2/wiki/Syntax) для регулярных выражений в политике очистки. Вы можете протестировать их с помощью [тестера регулярных выражений regex101](https://regex101.com/), используя версию **Golang**. Просмотрите некоторые распространенные [примеры шаблонов регулярных выражений](umenshenie-khranilisha-reestra-konteinerov.md#primery-shablonov-regulyarnykh-vyrazhenii).

### Политика очистки не удаляет теги

Причины этого могут быть разные:

* В GitLab 13.6 и более ранних версиях при запуске политики очистки вы можете ожидать, что она удалит теги, но это не так. Это происходит, когда политика очистки сохраняется без редактирования значения в поле **Remove tags matching**. Это поле имеет значение `.*`, выделенное серым цветом, в качестве заполнителя. Если в поле явно не введено `.*` (или другой шаблон регулярного выражения), передается нулевое значение `nil`. Это значение предотвращает сопоставление сохраненной политики очистки с какими-либо тегами. В качестве обходного пути измените политику очистки. В поле **Remove tags matching** введите `.*` и сохраните. Это значение указывает, что все теги должны быть удалены.
* Если вы используете самоуправляемые экземпляры GitLab и у вас более 1000 тегов в репозитории контейнеров, вы можете столкнуться с [проблемой истечения срока действия токена реестра контейнеров](https://gitlab.com/gitlab-org/gitlab/-/issues/288814) с ошибкой контекста авторизации: недопустимый токен в журналах (`error authorizing context: invalid token`). Чтобы это исправить, есть два обходных пути:
  * Если вы используете GitLab 13.9 или новее, вы можете [установить ограничения для политики очистки](umenshenie-khranilisha-reestra-konteinerov.md#ustanovite-ogranicheniya-ochistki-dlya-ekonomii-resursov). Это ограничивает выполнение очистки по времени и позволяет избежать ошибки токена с истекшим сроком действия.
  * Продлите срок действия токенов аутентификации реестра контейнеров. По умолчанию это 5 минут. Вы можете установить собственное значение, запустив `ApplicationSetting.last.update(container_registry_token_expire_delay: <integer>)` в консоли Rails, где `<integer>` — желаемое количество минут. Для справки: на GitLab.com задержка истечения срока действия установлена на 15 минут. Если вы увеличите это значение, вы увеличите время, необходимое для отзыва разрешений.

Альтернативно вы можете создать список тегов для удаления и использовать этот список для удаления тегов. Чтобы создать список и удалить теги:

1. Запустите следующий сценарий оболочки. Команда непосредственно перед циклом `for` гарантирует, что `list_o_tags.out` всегда повторно инициализируется при запуске цикла. После выполнения этой команды имена всех тегов записываются в файл `list_o_tags.out`:

```bash
# Получите список всех тегов в определенном репозитории контейнеров
# с учетом [pagination](../../../api/rest/index.md#pagination)
echo -n "" > list_o_tags.out; for i in {1..N}; do curl --header 'PRIVATE-TOKEN: <PAT>' "https://gitlab.example.com/api/v4/projects/<Project_id>/registry/repositories/<container_repo_id>/tags?per_page=100&page=${i}" | jq '.[].name' | sed 's:^.\(.*\).$:\1:' >> list_o_tags.out; done
```

Если у вас есть доступ к консоли Rails, вы можете ввести следующие команды, чтобы получить список тегов, ограниченный датой:

```go
output = File.open( "/tmp/list_o_tags.out","w" )
Project.find(<Project_id>).container_repositories.find(<container_repo_id>).tags.each do |tag|
  output << tag.name + "\n" if tag.created_at < 1.month.ago
end;nil
output.close
```

Этот набор команд создает файл `/tmp/list_o_tags.out`, в котором перечислены все теги с датой `created_at` старше одного месяца.

2. Удалите все теги, которые вы хотите сохранить, из файла `list_o_tags.out`. Например, вы можете использовать **sed** для анализа файла и удаления тегов.

#### Linux

```bash
# Удалить тег `latest` из файла
sed -i '/latest/d' list_o_tags.out

# Удалить первые N тегов из файла
sed -i '1,Nd' list_o_tags.out

# Удалить теги, начинающиеся с `Av` из файла
sed -i '/^Av/d' list_o_tags.out

# Удалить теги, заканчивающиеся на `_v3` из файла
sed -i '/_v3$/d' list_o_tags.out
```

#### macOS

```bash
# Удалить тег `latest` из файла
sed -i .bak '/latest/d' list_o_tags.out

# Удалить первые N тегов из файла
sed -i .bak '1,Nd' list_o_tags.out

# Удалить теги, начинающиеся с `Av` из файла
sed -i .bak '/^Av/d' list_o_tags.out

# Удалить теги, заканчивающиеся на `_v3` из файла
sed -i .bak '/_v3$/d' list_o_tags.out

```

3. Дважды проверьте файл `list_o_tags.out`, чтобы убедиться, что он содержит только те теги, которые вы хотите удалить.
4. Запустите этот сценарий оболочки, чтобы удалить теги в файле `list_o_tags.out`:

```bash
# пройдите по list_o_tags.out, чтобы удалить по одному тегу за раз
while read -r LINE || [[ -n $LINE ]]; do
    echo ${LINE};
    curl --request DELETE --header 'PRIVATE-TOKEN: <PAT>' "https://gitlab.example.com/api/v4/projects/<Project_id>/registry/repositories/<container_repo_id>/tags/${LINE}";
    sleep 0.1;
    echo;
done < list_o_tags.out > delete.logs
```
