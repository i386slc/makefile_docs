# PyPI

## Пакеты PyPI в реестре пакетов

> История версий
>
> * [Представлено](https://gitlab.com/gitlab-org/gitlab/-/issues/208747) в GitLab 12.10.
> * [Перенесено](https://gitlab.com/gitlab-org/gitlab/-/issues/221259) с GitLab Premium на GitLab Free в версии 13.3.

Публикуйте пакеты PyPI в реестре пакетов вашего проекта. Затем устанавливайте пакеты всякий раз, когда вам нужно использовать их в качестве зависимости.

Реестр пакетов работает с:

* [pip](https://pypi.org/project/pip/)
* [twine](https://pypi.org/project/twine/)

Документацию по конкретным конечным точкам API, которые используют клиенты **pip** и **twine**, см. в [документации по PyPI API](https://docs.gitlab.com/ee/api/packages/pypi.html).

Узнайте, как [собрать пакет PyPI](https://docs.gitlab.com/ee/user/packages/workflows/build\_packages.html#pypi).

## Аутентификация в реестре пакетов

Прежде чем вы сможете опубликовать в реестре пакетов, вы должны пройти аутентификацию.

Для этого вы можете использовать:

* [Токен личного доступа](https://docs.gitlab.com/ee/user/profile/personal\_access\_tokens.html) с областью действия, установленной на API.
* [Токен развертывания](https://docs.gitlab.com/ee/user/project/deploy\_tokens/index.html) с областью действия, установленной на **read\_package\_registry**, **write\_package\_registry** или на обе.
* [Токен задания CI](https://docs.gitlab.com/ee/user/packages/pypi\_repository/#authenticate-with-a-ci-job-token).

Не используйте методы аутентификации, отличные от описанных здесь. Недокументированные методы аутентификации могут быть удалены в будущем.

### Аутентификация с помощью личного токена доступа

Чтобы пройти аутентификацию с помощью личного токена доступа, отредактируйте файл `~/.pypirc` и добавьте:

```ini
[distutils]
index-servers =
    gitlab

[gitlab]
repository = https://gitlab.example.com/api/v4/projects/<project_id>/packages/pypi
username = <your_personal_access_token_name>
password = <your_personal_access_token>
```

`<project_id>` — это либо путь проекта в [кодировке URL](https://docs.gitlab.com/ee/api/rest/index.html#namespaced-path-encoding) (например, `group%2Fproject`), либо идентификатор проекта (например, **42**).

### Аутентификация с помощью токена развертывания

Чтобы пройти аутентификацию с помощью токена развертывания, отредактируйте файл `~/.pypirc` и добавьте:

```ini
[distutils]
index-servers =
    gitlab

[gitlab]
repository = https://gitlab.example.com/api/v4/projects/<project_id>/packages/pypi
username = <deploy token username>
password = <deploy token>
```

`<project_id>` — это либо [URL-кодированный](https://docs.gitlab.com/ee/api/rest/index.html#namespaced-path-encoding) путь проекта (например, `group%2Fproject`), либо идентификатор проекта (например, **42**).

### Аутентификация с помощью токена задания CI

> [Представлено](https://gitlab.com/gitlab-org/gitlab/-/issues/202012) в GitLab 13.4.

Для работы с командами PyPI в [GitLab CI/CD](https://docs.gitlab.com/ee/ci/index.html) вы можете использовать **CI\_JOB\_TOKEN** вместо токена личного доступа или токена развертывания.

Например:

```yaml
image: python:latest

run:
  script:
    - pip install build twine
    - python -m build
    - TWINE_PASSWORD=${CI_JOB_TOKEN} TWINE_USERNAME=gitlab-ci-token python -m twine upload --repository-url ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/pypi dist/*
```

Вы также можете использовать **CI\_JOB\_TOKEN** в файле `~/.pypirc`, который вы регистрируете в GitLab:

```ini
[distutils]
index-servers =
    gitlab

[gitlab]
repository = https://gitlab.example.com/api/v4/projects/${env.CI_PROJECT_ID}/packages/pypi
username = gitlab-ci-token
password = ${env.CI_JOB_TOKEN}
```

### Аутентификация для доступа к пакетам в группе

Следуйте приведенным выше инструкциям для типа токена, но используйте URL-адрес группы вместо URL-адреса проекта:

```
https://gitlab.example.com/api/v4/groups/<group_id>/-/packages/pypi
```

## Опубликовать пакет PyPI

Предварительные требования:

* Вы должны [аутентифицироваться в реестре пакетов](pypi.md#autentifikaciya-v-reestre-paketov).
* Ваша [строка версии должна быть действительной](pypi.md#ubedites-chto-vasha-stroka-versii-deistvitelna).
* Максимально допустимый размер пакета составляет **5 ГБ**.
* Вы не можете загружать одну и ту же версию пакета несколько раз. Если вы попытаетесь, вы получите ошибку `400 Bad Request`.
* Пакеты PyPI публикуются с использованием вашего идентификатора проекта **projectID**.
* Если ваш проект находится в группе, пакеты PyPI, опубликованные в реестре вашего проекта, также доступны в реестре на уровне группы (см. [Установка на уровне группы](pypi.md#ustanovka-s-urovnya-gruppy)).

Затем вы можете [опубликовать пакет с помощью twine](pypi.md#publikaciya-paketa-pypi-s-pomoshyu-twine).

### Убедитесь, что ваша строка версии действительна

Если ваша строка версии (например, `0.0.1`) недействительна, она будет отклонена. GitLab использует следующее регулярное выражение для проверки строки версии.

```regex
\A(?:
    v?
    (?:([0-9]+)!)?                                                 (?# epoch)
    ([0-9]+(?:\.[0-9]+)*)                                          (?# release segment)
    ([-_\.]?((a|b|c|rc|alpha|beta|pre|preview))[-_\.]?([0-9]+)?)?  (?# pre-release)
    ((?:-([0-9]+))|(?:[-_\.]?(post|rev|r)[-_\.]?([0-9]+)?))?       (?# post release)
    ([-_\.]?(dev)[-_\.]?([0-9]+)?)?                                (?# dev release)
    (?:\+([a-z0-9]+(?:[-_\.][a-z0-9]+)*))?                         (?# local version)
)\z}xi
```

Вы можете поэкспериментировать с регулярным выражением и попробовать свои версии строк с помощью этого [редактора регулярных выражений](https://rubular.com/r/FKM6d07ouoDaFV).

Дополнительные сведения о регулярном выражении см. в [этой документации](https://www.python.org/dev/peps/pep-0440/#appendix-b-parsing-version-strings-with-regular-expressions).

### Публикация пакета PyPI с помощью twine

Чтобы опубликовать пакет PyPI, выполните команду, например:

```bash
python3 -m twine upload --repository gitlab dist/*
```

Это сообщение указывает на то, что пакет был успешно опубликован:

```
Uploading distributions to https://gitlab.example.com/api/v4/projects/<your_project_id>/packages/pypi
Uploading mypypipackage-0.0.1-py3-none-any.whl
100%|███████████████████████████████████████████████████████████████████████████████████████████| 4.58k/4.58k [00:00<00:00, 10.9kB/s]
Uploading mypypipackage-0.0.1.tar.gz
100%|███████████████████████████████████████████████████████████████████████████████████████████| 4.24k/4.24k [00:00<00:00, 11.0kB/s]
```

Чтобы просмотреть опубликованный пакет, перейдите на страницу **Packages and registries** вашего проекта.

Если вы не использовали файл `.pypirc` для определения источника репозитория, вы можете опубликовать в репозитории встроенную аутентификацию:

```
TWINE_PASSWORD=<personal_access_token or deploy_token>\
TWINE_USERNAME=<username or deploy_token_username>\
python3 -m twine upload\
--repository-url https://gitlab.example.com/api/v4/projects/<project_id>/packages/pypi\
dist/*
```

Если вы не выполнили шаги, описанные на этой странице, убедитесь, что ваш пакет был правильно собран и что вы [создали пакет PyPI с помощью инструментов setuptools](https://packaging.python.org/tutorials/packaging-projects/).

Затем вы можете загрузить свой пакет с помощью следующей команды:

```bash
python -m twine upload --repository <source_name> dist/<package_file>
```

* `<package_file>` — это имя файла вашего пакета, оканчивающееся на `.tar.gz` или `.whl`.
* `<source_name>` — это имя источника, используемое во время установки.

### Публикация пакетов с тем же именем или версией

Вы не можете опубликовать пакет, если пакет с таким же именем и версией уже существует. Сначала необходимо [удалить существующий пакет](https://docs.gitlab.com/ee/user/packages/package\_registry/reduce\_package\_registry\_storage.html#delete-a-package). Если вы попытаетесь опубликовать один и тот же пакет более одного раза, возникнет ошибка `400 Bad Request`.

## Установка пакета PyPI

В [GitLab 14.2 и более поздних версиях](https://gitlab.com/gitlab-org/gitlab/-/issues/233413), если пакет PyPI не найден в реестре пакетов, запрос перенаправляется на [pypi.org](https://pypi.org/).

Администраторы могут отключить это поведение в [настройках непрерывной интеграции](https://docs.gitlab.com/ee/user/admin\_area/settings/continuous\_integration.html).

{% hint style="danger" %}
При использовании параметра **--index-url** не указывайте порт, если это порт по умолчанию, например **80** для URL-адреса, начинающегося с **http**, или **443** для URL-адреса, начинающегося с **https**.
{% endhint %}

### Установка на уровне проекта

Чтобы установить последнюю версию пакета, используйте следующую команду:

```bash
pip install\
--index-url https://<personal_access_token_name>:<personal_access_token>@gitlab.example.com/api/v4/projects/<project_id>/packages/pypi/simple\
--no-deps <package_name>
```

* `<package_name>` — это название пакета.
* `<personal_access_token_name>` — это название токена личного доступа с областью действия **read\_api**.
* `<personal_access_token>` — это токен личного доступа с областью действия **read\_api**.
* `<project_id>` — это либо [URL-адрес](https://docs.gitlab.com/ee/api/rest/index.html#namespaced-path-encoding) проекта (например, `group%2Fproject`), либо идентификатор проекта (например, **42**).

В этих командах вы можете использовать `--extra-index-url` вместо `--index-url`. Однако использование `--extra-index-url` делает вас уязвимыми для атак с путаницей зависимостей, потому что он проверяет наличие пакета в репозитории PyPi перед проверкой пользовательского репозитория. `--extra-index-url` добавляет указанный URL-адрес в качестве дополнительного реестра, который клиент проверяет на наличие пакета. `--index-url` указывает клиенту проверять наличие пакета только по указанному URL-адресу.

Если вы следовали руководству и хотите установить пакет **MyPyPiPackage**, вы можете запустить:

```bash
pip install mypypipackage --no-deps\
--index-url https://<personal_access_token_name>:<personal_access_token>@gitlab.example.com/api/v4/projects/<your_project_id>/packages/pypi/simple
```

Это сообщение указывает на то, что пакет был успешно установлен:

```
Looking in indexes: https://<personal_access_token_name>:****@gitlab.example.com/api/v4/projects/<your_project_id>/packages/pypi/simple
Collecting mypypipackage
  Downloading https://gitlab.example.com/api/v4/projects/<your_project_id>/packages/pypi/files/d53334205552a355fee8ca35a164512ef7334f33d309e60240d57073ee4386e6/mypypipackage-0.0.1-py3-none-any.whl (1.6 kB)
Installing collected packages: mypypipackage
Successfully installed mypypipackage-0.0.1
```

### Установка с уровня группы

Чтобы установить последнюю версию пакета из группы, используйте следующую команду:

```bash
pip install\
--index-url https://<personal_access_token_name>:<personal_access_token>@gitlab.example.com/api/v4/groups/<group_id>/-/packages/pypi/simple\
--no-deps <package_name>
```

В этой команде:

* `<package_name>` — это название  пакета.
* `<personal_access_token_name>` — это название токена личного доступа с областью действия read\_api.
* `<personal_access_token>` — это токен личного доступа с областью действия read\_api.
* `<group_id>` — это идентификатор группы.

В этих командах вы можете использовать `--extra-index-url` вместо `--index-url`. Однако использование `--extra-index-url` делает вас уязвимыми для атак с путаницей зависимостей, потому что он проверяет наличие пакета в репозитории PyPi перед проверкой пользовательского репозитория. `--extra-index-url` добавляет указанный URL-адрес в качестве дополнительного реестра, который клиент проверяет на наличие пакета. `--index-url` указывает клиенту проверять наличие пакета только по указанному URL-адресу.

Если вы следуете руководству и хотите установить пакет **MyPyPiPackage**, вы можете запустить:

```bash
pip install mypypipackage --no-deps\
--index-url https://<personal_access_token_name>:<personal_access_token>@gitlab.example.com/api/v4/groups/<your_group_id>/-/packages/pypi/simple
```

### Имена пакетов

GitLab ищет пакеты, использующие [нормализованные имена Python (PEP-503)](https://www.python.org/dev/peps/pep-0503/#normalized-names). Символы `-`, `_` и `.` все обрабатываются одинаково, а повторяющиеся символы удаляются.

Запрос `pip install` для `my.package` ищет пакеты, которые соответствуют любому из трех символов, таких как `my-package`, `my_package` и `my....package`.

## Использование requirements.txt

Если вы хотите, чтобы **pip** имел доступ к вашему личному реестру, добавьте параметр `--extra-index-url` вместе с URL-адресом вашего реестра в файл `requirements.txt`.

```python
--extra-index-url https://gitlab.example.com/api/v4/projects/<project_id>/packages/pypi/simple
package-name==1.0.0
```

Если это частный реестр, вы можете пройти аутентификацию несколькими способами. Например:

* Используя ваш файл `requirements.txt`:

```python
--extra-index-url https://__token__:<your_personal_token>@gitlab.example.com/api/v4/projects/<project_id>/packages/pypi/simple
package-name==1.0.0
```

* Используя файл `~/.netrc`:

```
machine gitlab.example.com
login __token__
password <your_personal_token>
```

## Исправление проблем

Для повышения производительности команда **pip** кэширует файлы, связанные с пакетом. **Pip** не удаляет данные сам по себе. Кэш увеличивается по мере установки новых пакетов. Если у вас возникли проблемы, очистите кеш с помощью этой команды:

```bash
pip cache purge
```

### Несколько параметров index-url или extra-index-url

Вы можете определить несколько параметров `index-url` и `extra-index-url`.

Если вы используете одно и то же доменное имя (например, `gitlab.example.com`) несколько раз с разными токенами аутентификации, **pip** может не найти ваши пакеты. Эта проблема связана с тем, как **pip** [регистрирует и хранит ваши токены](https://github.com/pypa/pip/pull/10904#issuecomment-1126690115) во время выполнения команд.

Чтобы обойти эту проблему, вы можете использовать [токен группового развертывания](https://docs.gitlab.com/ee/user/project/deploy\_tokens/index.html) с областью действия **read\_package\_registry** из общей родительской группы для всех проектов или групп, на которые нацелены значения **index-url** и **extra-index-url**.

## Поддерживаемые команды CLI

Репозиторий GitLab PyPI поддерживает следующие команды CLI:

* `twine upload`: Загрузить пакет в реестр.
* `pip install`: установите пакет PyPI из реестра.
