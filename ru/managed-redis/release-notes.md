# История изменений в {{ mrd-full-name }}

{% include [Tags](../_includes/mdb/release-notes-tags.md) %}

## 01.07.2022 {#01.07.2022}

* В {{ TF }} добавлены настройки `replica_priority` и `assign_public_ip`. {{ tag-tf }}
* Исправлена ошибка, из-за которой было недоступно управление окном технического обслуживания с помощью {{ TF }}. {{ tag-tf }}

## 01.06.2022 {#01.06.2022}

* Прекращена поддержка {{ RD }} версий 5.0 и 6.0. Создание кластеров этих версий больше недоступно. Через месяц после выхода версии 7.0 существующие кластеры {{ RD }} 5.0 и 6.0 будут автоматически обновлены до версии 6.2.
* Добавлена возможность изменения настройки `client-output-buffer-limit` normal и pubsub. Подробнее см. в описании конфигурационного файла [redis.conf](https://raw.githubusercontent.com/redis/redis/unstable/redis.conf). {{ tag-con }} {{ tag-cli }} {{ tag-tf }}

## 01.04.2022 {#01.04.2022}

* Добавлена возможность создания кластеров с публичным доступом. Включение или изменение этой настройки доступно на уровне хоста и только для кластеров с включенным TLS. {{ tag-con }}
* Добавлено управление настройкой персистентности. Отключение персистентности увеличивает производительность кластера, но повышает риск потери данных. Подробнее см. в разделе [Персистентность](concepts/replication#persistence). {{ tag-con }} {{ tag-cli }} {{ tag-tf }}