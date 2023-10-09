# build и push образо

Прежде чем вы сможете создавать и отправлять образы контейнеров, вы должны пройти [аутентификацию](autentifikaciya.md) в реестре контейнеров.

## Использование команд Docker

Вы можете использовать команды Docker для создания и отправки образов контейнеров в реестр контейнеров:

1. Выполните [аутентификацию](autentifikaciya.md) с помощью реестра контейнеров.
2. Запустите команду Docker для сборки или отправки. Например:

* чтобы собрать образ:

```bash
docker build -t registry.example.com/group/project/image .
```

* что залить образ:

```bash
docker push registry.example.com/group/project/image
```

## Настройка файла `.gitlab-ci.yml`

Вы можете настроить файл `.gitlab-ci.yml` для создания и отправки образов контейнеров в реестр контейнеров.

* Если несколько заданий требуют аутентификации, поместите команду аутентификации в файл `before_script`.
* Перед сборкой используйте `docker build --pull` для получения изменений в базовых образах. Это займет немного больше времени, но зато гарантирует актуальность вашего образа.
* Перед каждым запуском докера `docker run` выполняйте явный запрос докера `docker pull`, чтобы получить только что созданный образ. Этот шаг особенно важен, если вы используете несколько раннеров, которые локально кэшируют образы. Если вы используете Git SHA в теге образа, каждое задание уникально, и у вас никогда не должно быть устаревшего образа. Тем не менее, все еще возможно иметь устаревший образ, если вы пересобираете данный коммит после изменения зависимости.
* Не делайте сборку непосредственно на основе тега `latest`, поскольку одновременно может выполняться несколько заданий.

## Использование GitLab CI/CD

Вы можете использовать <mark style="color:purple;">GitLab CI/CD</mark> для создания и отправки образов контейнеров в реестр контейнеров. Вы можете использовать CI/CD для тестирования, сборки и развертывания проекта из созданного вами образа контейнера.

### Использлвание образа контейнера Docker-in-Docker из вашего реестра контейнеров

Вы можете использовать собственные образы контейнеров для Docker-in-Docker.

1. Настройте <mark style="color:purple;">Docker-in-Docker</mark>.
2. Обновите образ `image` и службу `service`, чтобы они указывали на ваш реестр.
3. Добавьте псевдоним службы <mark style="color:purple;">alias</mark>.

Ваш `.gitlab-ci.yml` должен выглядеть примерно так:

```yaml
build:
  image: $CI_REGISTRY/group/project/docker:20.10.16
  services:
    - name: $CI_REGISTRY/group/project/docker:20.10.16-dind
      alias: docker
  stage: build
  script:
    - docker build -t my-docker-image .
    - docker run my-docker-image /script/to/run/tests
```

Если вы забудете установить псевдоним службы, образ контейнера не сможет найти службу **dind**, и появится ошибка, подобная следующей:

```
error during connect: Get http://docker:2376/v1.39/info: dial tcp: lookup docker on 192.168.0.1:53: no such host
```

### Использование образа контейнера Docker-in-Docker с прокси-сервером зависимостей

Вы можете использовать свои собственные образы контейнеров с Dependency Proxy.

1. Настройте <mark style="color:purple;">Docker-in-Docker</mark>.
2. Обновите образ `image` и службу `service`, чтобы они указывали на ваш реестр.
3. Добавьте псевдоним службы <mark style="color:purple;">alias</mark>.

Ваш `.gitlab-ci.yml` должен выглядеть примерно так:

```yaml
build:
  image: ${CI_DEPENDENCY_PROXY_GROUP_IMAGE_PREFIX}/docker:20.10.16
  services:
    - name: ${CI_DEPENDENCY_PROXY_GROUP_IMAGE_PREFIX}/docker:18.09.7-dind
      alias: docker
  stage: build
  script:
    - docker build -t my-docker-image .
    - docker run my-docker-image /script/to/run/tests
```

Если вы забудете установить псевдоним службы, образ контейнера не сможет найти службу **dind**, и появится ошибка, подобная следующей:

```
error during connect: Get http://docker:2376/v1.39/info: dial tcp: lookup docker on 192.168.0.1:53: no such host
```

## Примеры реестра контейнеров с GitLab CI/CD

Если вы используете Docker-in-Docker на своих раннерах, ваш файл `.gitlab-ci.yml` должен выглядеть примерно так:

```yaml
build:
  image: docker:20.10.16
  stage: build
  services:
    - docker:20.10.16-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY/group/project/image:latest .
    - docker push $CI_REGISTRY/group/project/image:latest
```

Вы можете использовать <mark style="color:purple;">переменные CI/CD</mark> в файле `.gitlab-ci.yml`. Например:

```bash
build:
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
```

В этом примере `$CI_REGISTRY_IMAGE` преобразуется в адрес реестра, привязанного к этому проекту. `$CI_COMMIT_REF_NAME` преобразуется в имя ветки или тега, которое может содержать косую черту. Теги изображений не могут содержать косую черту. Используйте `$CI_COMMIT_REF_SLUG` в качестве тега изображения. Вы можете объявить переменную `$IMAGE_TAG`, объединив `$CI_REGISTRY_IMAGE` и `$CI_COMMIT_REF_NAME`, чтобы сэкономить на вводе текста в разделе сценария.

В этом примере задачи разделены на 4 этапа конвейера, включая два теста, которые выполняются параллельно. Сборка **build** сохраняется в реестре контейнеров и используется на последующих этапах, при необходимости загружая образ контейнера. Изменения в основном файле **main** также помечаются как последние **latest** и развертываются с использованием сценария развертывания для конкретного приложения:

```yaml
default:
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

stages:
  - build
  - test
  - release
  - deploy

variables:
  # Use TLS https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#tls-enabled
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"
  CONTAINER_TEST_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  CONTAINER_RELEASE_IMAGE: $CI_REGISTRY_IMAGE:latest

build:
  stage: build
  script:
    - docker build --pull -t $CONTAINER_TEST_IMAGE .
    - docker push $CONTAINER_TEST_IMAGE

test1:
  stage: test
  script:
    - docker pull $CONTAINER_TEST_IMAGE
    - docker run $CONTAINER_TEST_IMAGE /script/to/run/tests

test2:
  stage: test
  script:
    - docker pull $CONTAINER_TEST_IMAGE
    - docker run $CONTAINER_TEST_IMAGE /script/to/run/another/test

release-image:
  stage: release
  script:
    - docker pull $CONTAINER_TEST_IMAGE
    - docker tag $CONTAINER_TEST_IMAGE $CONTAINER_RELEASE_IMAGE
    - docker push $CONTAINER_RELEASE_IMAGE
  only:
    - main

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
  environment: production
```

{% hint style="info" %}
В этом примере явно вызывается `docker pull`. Если вы предпочитаете неявно извлекать образ контейнера с помощью `image:` и использовать исполнитель <mark style="color:purple;">Docker</mark> или <mark style="color:purple;">Kubernetes</mark>, убедитесь, что для параметра **pull\_policy** установлено значение **always**.
{% endhint %}
