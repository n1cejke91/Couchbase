## Описание
Этот проект разворачивает кластер Couchbase версии `3.9` с тремя узлами в Docker. После развертывания создается база данных (bucket) `test_bucket` и заполняется тестовыми данными. Также проводится проверка отказоустойчивости.

## Запуск кластера

1. **Создать `docker-compose.yml`**:

    ```yaml
    version: '3.9'
    services:
      couchbase1:
        image: couchbase/server:community
        container_name: couchbase1
        hostname: couchbase1
        volumes:
          - ./node1:/opt/couchbase/var
        ports:
          - 8091:8091
          - 8092:8092
          - 8093:8093
          - 11210:11210
      couchbase2:
        image: couchbase/server:community
        container_name: couchbase2
        hostname: couchbase2
        volumes:
          - ./node2:/opt/couchbase/var
        ports:
          - 8094:8091
      couchbase3:
        image: couchbase/server:community
        container_name: couchbase3
        hostname: couchbase3
        volumes:
          - ./node3:/opt/couchbase/var
        ports:
          - 8095:8091
    ```

2. **Запустить контейнеры:**

    ```sh
    docker-compose up -d
    ```

## Настройка кластера

1. Открыть браузер и перейти на [http://localhost:8091](http://localhost:8091).
2. Создать новый кластер, задав `Admin` в качестве логина и `password` в качестве пароля.
3. Выбрать сервисы `Data`, `Query`, `Index`.

### Добавление узлов в кластер:

```sh
    docker exec -it couchbase1 couchbase-cli server-add -c couchbase1:8091 -u Admin -p password \
      --server-add=couchbase2:8091 --server-add-username=Admin --server-add-password=password
    
    docker exec -it couchbase1 couchbase-cli server-add -c couchbase1:8091 -u Admin -p password \
      --server-add=couchbase3:8091 --server-add-username=Admin --server-add-password=password

    docker exec -it couchbase1 couchbase-cli rebalance -c couchbase1:8091 -u Admin -p password
```

## Создание базы данных и тестовых данных

### Создание bucket `test_bucket`
```sh
    docker exec -it couchbase1 couchbase-cli bucket-create -c couchbase1:8091 -u Admin -p password \
      --bucket=test_bucket --bucket-type=couchbase --bucket-ramsize=256
```

### Добавление тестовых данных через REST API
```sh
    curl -X POST -u Admin:password http://localhost:8091/pools/default/buckets/test_bucket/docs/item1 -d '{
        "name": "Item 1",
        "category": "Electronics",
        "price": 100
    }'

    curl -X POST -u Admin:password http://localhost:8091/pools/default/buckets/test_bucket/docs/item2 -d '{
        "name": "Item 2",
        "category": "Books",
        "price": 20
    }'

    curl -X POST -u Admin:password http://localhost:8091/pools/default/buckets/test_bucket/docs/item3 -d '{
        "name": "Item 3",
        "category": "Furniture",
        "price": 300
    }'
```

### Проверка наличия данных
```sh
    curl -u Admin:password http://localhost:8091/pools/default/buckets/test_bucket/docs/item1
```

## Проверка отказоустойчивости

### Остановка одного узла:
```sh
    docker stop couchbase2
    docker exec -it couchbase1 couchbase-cli rebalance -c couchbase1:8091 -u Admin -p password
```

### Проверка доступности данных:
```sh
    curl -u Admin:password http://localhost:8091/pools/default/buckets/test_bucket/docs/item1
```

### Запуск узла обратно и повторное балансирование:
```sh
    docker start couchbase2
    docker exec -it couchbase1 couchbase-cli rebalance -c couchbase1:8091 -u Admin -p password
```

## Заключение
Запущен кластер Couchbase с тестовыми данными, готовый к дальнейшей настройке и использованию.

