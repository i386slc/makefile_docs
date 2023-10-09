# Удаление образов

Вы можете удалить образы контейнеров из реестра контейнеров.

{% hint style="warning" %}
Удаление образов контейнеров является деструктивным действием и не может быть отменено. Чтобы восстановить удаленный образ контейнера, необходимо пересобрать и повторно загрузить его.
{% endhint %}

## Сбор мусора

Удаление образа контейнера на самоуправляемых экземплярах не освобождает место для хранения, а лишь помечает образ как подходящий для удаления. Чтобы фактически удалить образы контейнеров, на которые нет ссылок, и освободить место для хранения, администраторы должны запустить <mark style="color:purple;">сбор мусора</mark>.

На GitLab.com последняя версия Container Registry включает автоматический онлайн-сборщик мусора. Для получения дополнительной информации см. эту [публикацию в блоге](https://about.gitlab.com/blog/2021/10/25/gitlab-com-container-registry-update/). В этой новой версии реестра контейнеров следующие элементы автоматически планируются к удалению через 24 часа, если на них нет ссылок:

* Слои, на которые не ссылается какой-либо манифест образа.
* Манифесты образов, которые не имеют тегов и на которые не ссылается другой манифест (например, образы с несколькими архитектурами).

## Использование интерфейса GitLab

Чтобы удалить образы контейнеров с помощью пользовательского интерфейса GitLab:

1. На левой боковой панели выберите **Search or go to** и найдите свой проект или группу.
2. Для:
   * группы выберите **Operate > Container Registry**
   * проекта выберите **Deploy > Container Registry**
3. На странице **Container Registry** вы можете выбрать то, что хотите удалить, одним из следующих способов:
   * Удаление всего репозитория и всех содержащихся в нем тегов путем выбора красного значка корзины **Trash**.
   * Перейдите к хранилищу и удалите теги по отдельности или массово, выбрав красный значок корзины **Trash** рядом с тегом, который вы хотите удалить.
4. В диалоговом окне выберите **Remove tag**

## Использование API GitLab

Вы можете использовать API для автоматизации процесса удаления образов контейнеров. Дополнительные сведения см. в следующих конечных точках:

* [Delete a Registry repository](https://docs.gitlab.com/ee/api/container\_registry.html#delete-registry-repository)
* [Delete an individual Registry repository tag](https://docs.gitlab.com/ee/api/container\_registry.html#delete-a-registry-repository-tag)
* [Delete Registry repository tags in bulk](https://docs.gitlab.com/ee/api/container\_registry.html#delete-registry-repository-tags-in-bulk)

## Использование GitLab CI/CD

{% hint style="info" %}
GitLab CI/CD не предоставляет встроенного способа удаления образов контейнеров. В этом примере используется сторонний инструмент под названием [reg](https://github.com/genuinetools/reg), который взаимодействует с API реестра GitLab. Для получения помощи по этому стороннему инструменту см. [очередь вопросов для reg](https://github.com/genuinetools/reg/issues).
{% endhint %}

В следующем примере определены два этапа: **build** и **clean**. Задание **build\_image** создает образ контейнера для ветки, а задание **delete\_image** удаляет его. Исполняемый файл **reg** загружается и используется для удаления образа контейнера, соответствующего <mark style="color:purple;">предопределенной переменной CI/CD</mark> `$CI_PROJECT_PATH:$CI_COMMIT_REF_SLUG`.

Чтобы использовать этот пример, измените переменную **IMAGE\_TAG** в соответствии со своими потребностями.

```yaml
stages:
  - build
  - clean

build_image:
  image: docker:20.10.16
  stage: build
  services:
    - docker:20.10.16-dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - branches
  except:
    - main

delete_image:
  before_script:
    - curl --fail --show-error --location "https://github.com/genuinetools/reg/releases/download/v$REG_VERSION/reg-linux-amd64" --output ./reg
    - echo "$REG_SHA256  ./reg" | sha256sum -c -
    - chmod a+x ./reg
  image: curlimages/curl:7.86.0
  script:
    - ./reg rm -d --auth-url $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $IMAGE_TAG
  stage: clean
  variables:
    IMAGE_TAG: $CI_PROJECT_PATH:$CI_COMMIT_REF_SLUG
    REG_SHA256: ade837fc5224acd8c34732bf54a94f579b47851cc6a7fd5899a98386b782e228
    REG_VERSION: 0.16.1
  only:
    - branches
  except:
    - main
```

{% hint style="info" %}
Вы можете загрузить последнюю версию **reg** со [страницы выпусков](https://github.com/genuinetools/reg/releases), а затем обновить пример кода, изменив переменные **REG\_SHA256** и **REG\_VERSION**, определенные в задании **delete\_image**.
{% endhint %}

## Использование политики очистки

Вы можете создать [политику очистки](umenshenie-khranilisha-reestra-konteinerov.md#politika-ochistki) для каждого проекта, чтобы гарантировать регулярное удаление старых тегов и образов из реестра контейнеров.
