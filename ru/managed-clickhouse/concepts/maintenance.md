# Техническое обслуживание

Под техническим обслуживанием понимается:

* автоматическая установка обновлений и исправлений СУБД для хостов (в т. ч. для выключенных кластеров);
* изменение класса хостов и объема хранилища;
* другие сервисные работы.

Изменение мажорной версии СУБД не включено в техническое обслуживание. Подробнее о переходе между мажорными версиями см. в разделе [{#T}](../operations/cluster-version-update.md).

## Окно обслуживания {#maintenance-window}

Предпочтительное время технического обслуживания можно задать при [создании кластера](../operations/cluster-create.md) или [изменении его настроек](../operations/update.md):

* **Произвольное время** (по умолчанию) — разрешает проведение технического обслуживания в любое время.
* **По расписанию** — укажите предпочтительное время начала обслуживания: нужные день недели и час дня по UTC. Например, можно выбрать время, когда кластер наименее загружен.

## Порядок обслуживания {#maintenance-order}

Порядок технического обслуживания кластеров {{ mch-name }} определяется количеством [шардов](sharding.md) и входящих в них хостов:

* В кластерах из одного хоста техническое обслуживание проходит единственный хост {{ CH }}. Поэтому если во время технического обслуживания потребуется его перезагрузка, такой кластер станет недоступным.
* Если кластер состоит из нескольких хостов {{ CH }}, входящих в единственный шард, [хосты-реплики](replication.md) последовательно проходят техническое обслуживание. Порядок хостов в очереди определяется случайным образом. Если во время технического обслуживания потребуется перезагрузка хоста, он станет недоступным на это время. Если вы используете для доступа к кластеру FQDN или IP-адрес хоста {{ CH }}, такой кластер может стать недоступным. Чтобы обеспечить бесперебойную работу приложения, подключайтесь к кластеру через [специальный FQDN](../operations/connect.md#auto), всегда указывающий на доступный хост.
* Если кластер состоит из нескольких шардов, то техническое обслуживание проходит пошардово в порядке возрастания номера шарда. При этом сначала на обслуживание выводятся хосты одного шарда, затем двух, четырех и так далее, но не более 10. Техническое обслуживание хостов происходит так же как и в кластере из одного шарда. Если вы используете для доступа к шарду кластера FQDN или IP-адрес хоста {{ CH }}, такой шард может стать недоступным. Чтобы обеспечить бесперебойную работу приложения, подключайтесь к шарду через [специальный FQDN](../operations/connect.md#auto), всегда указывающий на доступный хост в шарде.