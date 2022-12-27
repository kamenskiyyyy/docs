# Типы дисков в {{ mos-name }}


{{ mos-name }} позволяет использовать сетевые и локальные диски для организации хранилища кластеров баз данных. Сетевые диски реализованы на базе сетевых блоков — виртуальных дисков в инфраструктуре {{ yandex-cloud }}. Локальные диски физически размещаются в серверах кластера.

{% include [storage-type](../../_includes/mdb/mos/storage-type.md) %}

## Особенности хранилища на локальных SSD-дисках {#local-storage-features}

* Хранилище на локальных SSD-дисках в кластере из одного хоста не обеспечивает отказоустойчивости: при отказе диска данные теряются безвозвратно. Чтобы обеспечить отказоустойчивость, создавайте кластеры из трех и более хостов, настраивайте [шардирование и репликацию](scalability-and-resilience.md) индексов.
* Кластер с таким хранилищем тарифицируется, даже если он остановлен. Подробнее — в [правилах тарификации](../pricing.md).

## Особенности хранилища на нереплицируемых SSD-дисках {#network-nrd-storage-features}

{% include [nrd-storage-details](../../_includes/mdb/nrd-storage-details.md) %}

## Выбор типа хранилища при создании кластера {#storage-type-selection}

Количество хостов с ролью `DATA`, которые можно создать вместе с кластером {{ OS }}, зависит от выбранного типа хранилища:

* При использовании хранилища на локальных SSD-дисках (`local-ssd`) или на нереплицируемых SSD-дисках (`network-ssd-nonreplicated`) вы можете создать кластер из трех или более хостов (минимум три хоста необходимо, чтобы обеспечить отказоустойчивость).
* При использовании хранилища на сетевых HDD-дисках (`network-hdd`) или сетевых SSD-дисках (`network-ssd`) вы можете добавить любое количество хостов в пределах [текущей квоты](./limits.md).

Подробнее об ограничениях на количество хостов в кластере см. в разделе [Квоты и лимиты](./limits.md).
