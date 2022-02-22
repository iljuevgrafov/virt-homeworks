# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

```
Dockerfile манифест
FROM centos
COPY ./elasticinstallscript /home/elasticinstallscript
COPY ./elasticsearch.yml /home/elasticsearch.yml
RUN /bin/bash -c "chmod +x /home/elasticinstallscript && /home/elasticinstallscript"

Bash-script elasticinstallscript

#!/bin/bash
useradd elasticsearch 
cd /home
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.0.0-linux-x86_64.tar.gz
tar -xzvf elasticsearch-8.0.0-linux-x86_64.tar.gz
cp /home/elasticsearch.yml ./elasticsearch-8.0.0/config/elasticsearch.yml
chown -R elasticsearch /var/lib
chown -R 1000:1000 /var/log/elasticsearch/
chown -R 1000:1000 /home/elasticsearch-8.0.0
cd elasticsearch-8.0.0
su elasticsearch ./bin/elasticsearch
```
```
Ссылка на репозиторий
https://hub.docker.com/repository/docker/iljuevgrafov/centoselastic
```
```
ответ `elasticsearch` на запрос пути `/` в json виде

{
    "name": "netology_test",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "oLZ5wEW4SXqMzGlO1l3tMw",
    "version": {
        "number": "8.0.0",
        "build_flavor": "default",
        "build_type": "tar",
        "build_hash": "1b6a7ece17463df5ff54a3e1302d825889aa1161",
        "build_date": "2022-02-03T16:47:57.507843096Z",
        "build_snapshot": false,
        "lucene_version": "9.0.0",
        "minimum_wire_compatibility_version": "7.17.0",
        "minimum_index_compatibility_version": "7.0.0"
    },
    "tagline": "You Know, for Search"
}
```

Далее мы будем работать с данным экземпляром elasticsearch.

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.
```
{
    "ind-1": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "ind-1",
                "creation_date": "1645471827576",
                "number_of_replicas": "0",
                "uuid": "XBBBDqTsQL-hWP8RRTHZFw",
                "version": {
                    "created": "8000099"
                }
            }
        }
    },
    "ind-2": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "2",
                "provided_name": "ind-2",
                "creation_date": "1645471851549",
                "number_of_replicas": "1",
                "uuid": "SISlKF2ETA6OhpCCWb_MSA",
                "version": {
                    "created": "8000099"
                }
            }
        }
    },
    "ind-3": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "4",
                "provided_name": "ind-3",
                "creation_date": "1645471866318",
                "number_of_replicas": "2",
                "uuid": "l0uzg_koTd-H43yWpJQuhQ",
                "version": {
                    "created": "8000099"
                }
            }
        }
    }
}
```

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?
```
Потому что в кластере одна нода, невозможно распередлить шарды по нодам и они попадают в состояние unassigned
```

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.
```
API запрос
{
    "type": "fs",
    "settings": {
        "location": "/home/elasticsearch-8.0.0/snapshots"
    }
}

Результат вызова
{
    "acknowledged": true
}
```

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
