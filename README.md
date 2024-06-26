# Описание
Кластер состоит из 3-х Docker-контейнеров: Master, Slave и Arbiter. Скрипт-агент написан на Python и запускается во всех контейнерах.
Когда появляются потери сетевой связности Slave -> Master и это подтверждается от Arbiter, то Slave промоутится до мастера. Но если у Slave нет связи ни с одним из них, то промоут не происходит.

Сам промоут создаёт триггер-файл "/tmp/promote_me". Slave проверяет связь Arbiter -> Master и Slave -> Master ежесекундно. Master проверяет связь Master -> Slave и Master -> Arbiter каждые 5 секунд. Но при отсутствия обеих связей происходит смена политик по умолчанию на DROP, тем самым происходит блокировка всех входящих подключений на Master через iptables.

# Запуск
Запуск кластера: ```docker compose up```
Запуск скрипта для тестирования кластера ```python writer.py```

# Тестирование кластера
Тест №1: умирает Slave (в середине теста Writer на хосте стопит контейнер ```docker compose stop pg-slave```).
При ```synchronous_commit = off```: без потерь.
При ```synchronous_commit = remote_apply```: без потерь.

Тест №2: умирает Master (в середине теста Writer на хосте стопит контейнер ```docker compose stop pg-master```).
При ```synchronous_commit = off```: 24 записи были потеряны.
При ```synchronous_commit = remote_apply```: без потерь.

# Вывод
При использовании synchronous_commit = remote_apply потери данных отсутствуют, даже если Slave или Master недоступны, но при synchronous_commit = off потери данных возможны, если Master становится недоступным.
