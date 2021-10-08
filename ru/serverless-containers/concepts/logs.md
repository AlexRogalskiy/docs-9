# Логи контейнера

Контейнер пишет логи в {{ cloud-logging-full-name }}, в [лог-группу по умолчанию](../../logging/concepts/log-group.md) для каталога, в котором находится контейнер.

Существует два вида логов:
* автоматические — логи запросов к контейнерам;
* пользовательские — логи, которые пишет пользовательский код в стандартный вывод (stdout) и стандартный вывод ошибок (stderr).

Подробнее о работе с логами в [документации {{ cloud-logging-full-name }}](../../logging/).