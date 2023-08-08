# PyP

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

* Вы должны [аутентифицироваться в реестре пакетов](pyp.md#autentifikaciya-v-reestre-paketov).
* Ваша [строка версии должна быть действительной](pyp.md#ubedites-chto-vasha-stroka-versii-deistvitelna).
* Максимально допустимый размер пакета составляет **5 ГБ**.
* Вы не можете загружать одну и ту же версию пакета несколько раз. Если вы попытаетесь, вы получите ошибку `400 Bad Request`.
* Пакеты PyPI публикуются с использованием вашего идентификатора проекта **projectID**.
* Если ваш проект находится в группе, пакеты PyPI, опубликованные в реестре вашего проекта, также доступны в реестре на уровне группы (см. Установка на уровне группы).

Затем вы можете опубликовать пакет с помощью twine.

### Убедитесь, что ваша строка версии действительна

Если ваша строка версии (например, 0.0.1) недействительна, она будет отклонена. GitLab использует следующее регулярное выражение для проверки строки версии.

### Публикация пакета PyPI с помощью twine

### Публикация пакетов с тем же именем или версией

## Установка пакета PyPI

### Установка на уровне проекта

### Установка с уровня группы

### Имена пакетов

## Использование requirements.txt

## Исправление проблем

### Несколько параметров index-url или extra-index-url

## Поддерживаемые команды CLI
