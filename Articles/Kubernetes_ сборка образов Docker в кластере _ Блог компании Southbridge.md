Kubernetes: сборка образов Docker в кластере / Блог компании Southbridge

Чтобы собирать образы Docker в контейнере и при этом обойтись без Docker, можно использовать kaniko. Давайте узнаем, как запускать kaniko локально и в кластере Kubernetes.

![image](../_resources/add7e4c335d844b29131d33bfc41068a.jpg)  
_Дальше будет многабукаф_

Допустим, решили вы собрать образы Docker в кластере Kubernetes (ну вот надо). Чем это удобно, рассмотрим на реальном примере, так нагляднее.

Еще мы поговорим о Docker внутри Docker и о его альтернативе — kaniko, с которым можно собирать образы Docker, не используя Docker. Наконец, мы узнаем, как настроить сборку образов в кластере Kubernetes.

Общее описание Kubernetes есть в книге ["Kubernetes in Action" ("Kubernetes в действии")](https://www.thenativeweb.io/blog/2018-04-05-15-17-recommended-reading-kubernetes-in-action/).

  

### Реальный пример

У нас в the native web есть немало приватных образов Docker, которые нужно где-то хранить. Вот мы и реализовали приватный [Docker Hub](https://hub.docker.com/). В общедоступном [Docker Hub](https://hub.docker.com/) есть две функции, которые нас особенно заинтересовали.

Во-первых, мы хотели создать очередь, которая будет асинхронно собирать образы Docker в Kubernetes. Во-вторых, реализовать отправку собранных образов в приватный [реестр Docker](https://docs.docker.com/registry/).

Обычно для реализации этих функций используется напрямую Docker CLI:

  

    $ docker build ...
    $ docker push ...

Но в кластере Kubernetes у нас размещаются контейнеры на базе маленьких и элементарных образов Linux, в которых Docker по умолчанию не содержится. Если теперь мы хотим использовать Docker (например, `docker build...`) в контейнере, нужно что-то вроде Docker внутри Docker.

  

### Что не так с Docker внутри Docker?

Чтобы собирать образы контейнера в Docker, нам нужен запущенный Docker-демон в контейнере, то есть Docker внутри Docker. Docker-демон — это виртуализированная среда, а контейнер в Kubernetes виртуализирован сам по себе. То есть, если хотим запустить Docker-демон в контейнере, нужно использовать вложенную виртуализацию. Для этого запускаем контейнер в привилегированном режиме — чтобы получить доступ к хост-системе. Но при этом возникают проблемы с безопасностью: например, приходится работать с разными файловыми системами (хоста и контейнера) или использовать кэш сборки из хост-системы. Вот почему мы и не хотели трогать Docker внутри Docker.

  

### Знакомство с kaniko

Не Docker внутри Docker одним… Есть еще одно решение — [kaniko](https://github.com/GoogleContainerTools/kaniko). Это инструмент, написанный на [Go](https://golang.org/), он собирает образы контейнеров из Dockerfile без Docker. Затем отправляет их в указанный [реестр Docker](https://docs.docker.com/registry/). Рекомендуется настроить kaniko — использовать готовый [образ-executor](https://console.cloud.google.com/gcr/images/kaniko-project/GLOBAL/executor?pli=1), который можно запустить как контейнер Docker или контейнер в Kubernetes.

Только учтите, что kaniko пока находится в разработке и поддерживает не все команды Dockerfile, например `--chownflag` для команды `COPY`.

  

### Запуск kaniko

Если хотите запустить kaniko, нужно указать для контейнера kaniko несколько аргументов. Сначала вставьте Dockerfile со всеми его зависимостями в контейнер kaniko. Локально (в Docker) для этого используется параметр `-v <путь_в_хосте>:<путь_в_контейнере>`, а в Kubernetes есть [вольюмы](https://kubernetes.io/docs/concepts/storage/volumes/).

Вставив Dockerfile с зависимостями в контейнер kaniko, добавьте аргумент `--context`, он укажет путь к прикрепленному каталогу (внутри контейнера). Следующий аргумент — `--dockerfile`. Он указывает путь к Dockerfile (включая имя). Еще один важный аргумент `--destination` с полным URL к реестру Docker (включая имя и тег образа).

  

### Локальный запуск

Kaniko запускается несколькими способами. Например, на локальном компьютере с помощью Docker (чтобы не возиться с кластером Kubernetes). Запустите kaniko следующей командой:

  

    $ docker run \
      -v $(pwd):/workspace \
      gcr.io/kaniko-project/executor:latest \
      --dockerfile=<path-to-dockerfile> \
      --context=/workspace \
      --destination=<repo-url-with-image-name>:<tag>

Если в реестре Docker включена аутентифиакация, kaniko сначала должен войти. Для этого подключите локальный файл Docker `config.jsonfile` с учетными данными для реестра Docker к контейнеру kaniko с помощью следующей команды:

  

    $ docker run \
      -v $(pwd):/workspace \
      -v ~/.docker/config.json:/kaniko/.docker/config.json \
      gcr.io/kaniko-project/executor:latest \
      --dockerfile=<path-to-dockerfile> \
      --context=/workspace \
      --destination=<repo-url-with-image-name>:<tag>

  

### Запуск в Kubernetes

В примере мы хотели запустить kaniko в кластере Kubernetes. А еще нам нужно было что-то вроде очереди для сборки образов. Если при сборке или отправке образа в реестр Docker случится сбой, будет неплохо, если процесс станет автоматически запускаться снова. Для этого в Kubernetes существует [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) (задание). Настройте `backoffLimit`, указав, как часто процесс должен повторять попытки.

Проще всего внедрить Dockerfile с зависимостями в контейнер kaniko с помощью объекта [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) (в нашем примере он называется `kaniko-workspace`). Он будет привязан к контейнеру как каталог, и в `kaniko-workspace` уже должны быть все данные. Допустим, в другом контейнере уже есть Dockerfile с зависимостями в каталоге `/my-build` в `kaniko-workspace`.

Не забывайте, что в AWS беда с PersistentVolumeClaim. Если создать PersistentVolumeClaim в AWS, он появится только на одном узле в кластере AWS и будет доступен только там. (upd: на самом деле при создании PVC будет создан RDS вольюм в случайной зоне доступности вашего кластера. Соответственно, этот вольюм будет доступен всем машинам в этой зоне. Kubernetes сам контролирует, чтобы под, использующий данный PVC, был запущен на ноде в зоне доступности RDS вольюма. – прим.пер.) Так что, если вы запустите Job kaniko и это задание окажется на другом узле, оно не запустится, ведь PersistentVolumeClaim недоступен. Будем надеяться, что скоро Amazon Elastic File System будет доступна в Kubernetes, и проблема исчезнет. (upd: EFS в Kubernetes поддерживается с помощью [storage provisioner](https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs). – прим.пер.)

Ресурс задания для сборки образов Docker обычно выглядит так:

  

    apiVersion: batch/v1
    kind: Job
    metadata:
      name: build-image
    spec:
      template:
        spec:
          containers:
          - name: build-image
            image: gcr.io/kaniko-project/executor:latest
            args:
              - "--context=/workspace/my-build"
              - "--dockerfile=/workspace/my-build/Dockerfile"
              - "--destination=<repo-url-with-image-name>:<tag>"
            volumeMounts:
            - name: workspace
              mountPath: /workspace
          volumes:
          - name: workspace
            persistentVolumeClaim:
              claimName: kaniko-workspace
          restartPolicy: Never
      backoffLimit: 3

Если целевой реестр Docker требует аутентификации, передайте файл `config.json` с учетными данными в контейнер kaniko. Проще всего подключить [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) к контейнеру, где уже есть файл `config.json`. Здесь PersistentVolumeClaim будет подключен не как каталог, а скорее как файл в пути `/kaniko/.docker/config.json` в контейнере kaniko:

  

    apiVersion: batch/v1
    kind: Job
    metadata:
      name: build-image
    spec:
      template:
        spec:
          containers:
          - name: build-image
            image: gcr.io/kaniko-project/executor:latest
            args:
              - "--context=/workspace/my-build"
              - "--dockerfile=/workspace/my-build/Dockerfile"
              - "--destination=<repo-url-with-image-name>:<tag>"
            volumeMounts:
            - name: config-json
              mountPath: /kaniko/.docker/config.json
              subPath: config.json
            - name: workspace
              mountPath: /workspace
          volumes:
            - name: config-json
              persistentVolumeClaim:
                claimName: kaniko-credentials
            - name: workspace
              persistentVolumeClaim:
                claimName: kaniko-workspace
          restartPolicy: Never
      backoffLimit: 3

Если хотите проверить статус выполняющегося задания сборки, используйте `kubectl`. Чтобы отфильтровать статус по `stdout`, выполните команду:

  

    $ kubectl get job build-image -o go-template='{{(index .status.conditions 0).type}}'

  

### Итоги

Из статьи вы узнали, когда Docker внутри Docker не подходит для сборки образов Docker в Kubernetes. Получили представление о kaniko — альтернативе Docker внутри Docker, с которой собираются образы Docker без Docker. А еще научились писать ресурсы Job, чтобы собирать образы Docker в Kubernetes. И, наконец, увидели, как узнать статус выполняющегося задания.