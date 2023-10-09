# Аутентификация

{% hint style="warning" %}
В GitLab 16.0 и более поздних версиях <mark style="color:purple;">внешняя авторизация</mark> предотвращает доступ токенов личного доступа и токенов развертывания к реестрам контейнеров и пакетов и влияет на всех пользователей, которые используют эти токены для доступа к реестрам. Вы можете отключить внешнюю авторизацию, если хотите использовать токены личного доступа и развертывать токены с реестрами контейнеров или пакетов.
{% endhint %}

Для аутентификации в реестре контейнеров вы можете использовать:

* [Personal access token](https://docs.gitlab.com/ee/user/profile/personal\_access\_tokens.html).
* [Deploy token](https://docs.gitlab.com/ee/user/project/deploy\_tokens/index.html).
* [Project access token](https://docs.gitlab.com/ee/user/project/settings/project\_access\_tokens.html).
* [Group access token](https://docs.gitlab.com/ee/user/group/settings/group\_access\_tokens.html).

Все эти методы аутентификации требуют минимальной области действия:

* Для доступа на чтение (**pull**) — `read_registry`.
* Для доступа на запись (**push**) это будут `write_registry` и `read_registry`.

Для аутентификации выполните команду входа `docker login`. Например:

```bash
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
```

## Использование GitLab CI/CD для аутентификации

Чтобы использовать CI/CD для аутентификации в реестре контейнеров, вы можете использовать:

* Переменную `CI_REGISTRY_USER` CI/CD. Эта переменная имеет доступ для чтения и записи к реестру контейнеров и действительна только для одного задания. Его пароль также создается автоматически и назначается `CI_REGISTRY_PASSWORD`.

```bash
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
```

* [CI job token](https://docs.gitlab.com/ee/ci/jobs/ci\_job\_token.html).

```bash
docker login -u $CI_REGISTRY_USER -p $CI_JOB_TOKEN $CI_REGISTRY
```

* Токен развертывания ([deploy token](https://docs.gitlab.com/ee/user/project/deploy\_tokens/index.html#gitlab-deploy-token)) с минимальной областью действия:
  * Для доступа на чтение (pull) `read_registry`.
  * Для доступа на запись (push) используйте `write_registry`.

```bash
docker login -u $CI_DEPLOY_USER -p $CI_DEPLOY_PASSWORD $CI_REGISTRY
```

* Токен личного доступа ([personal access token](https://docs.gitlab.com/ee/user/profile/personal\_access\_tokens.html)) с минимальным объемом:
  * Для доступа на чтение (pull) `read_registry`.
  * Для доступа на запись (push) используйте `write_registry`.

```bash
docker login -u <username> -p <access_token> $CI_REGISTRY
```
