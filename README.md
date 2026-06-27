# Borgbackup
create backup

#### Задача:
1. Настроить стенд с двумя ВМ: backup_server и
client.
2. Настроить удаленный бэкап каталога /etc с сервера client при помощи borgbackup.
3. Запустите стенд на 30 минут. Убедитесь, что резервные копии снимаются.
4. Остановите бэкап, удалите или переместите директорию /etc и восстановите её из бэкапа. Подробности в описании ДЗ в ЛК.
5. Для сдачи ДЗ ожидаем настроенный стенд, логи процесса бэкапа и описание процесса восстановления

#### Решение:

Запустим 2 ВМ (Ub24): bksrv (backup_server) - 192.168.56.17 и cl (client) - 192.168.56.18

Установим на обеих ВМ borgbackup

su -

apt install borgbackup

Создадим системного пользователя borg на bksrv, под которым будут храниться репозитории (/home/borg/otusborg, см далее)

useradd -m borg

passwd borg

Сгенерируем ssh-ключи на cl и скопируем публичный ключ на bksrv (тк используется push-модель)

ssh-keygen -q -t ed25519 -f /home/leo/.ssh/id_ed25519_borg -N "" -C "client-backup-key"

ssh-copy-id -i /home/leo/.ssh/id_ed25519_borg.pub borg@192.168.56.17

В файле /home/borg/.ssh/authorized_keys можно указать команду (добавить в начале строки), которая будет выполняться автоматически при подключении по этому ключу:

command="borg serve --restrict-to-path /home/borg/otusborg",restrict 

Инициализируем репозиторий на cl

borg init -e none borg@192.168.56.17:otusborg

Запускаем бэкап на cl и перенаправим stdout и stderr в лог файл для анализавост

borg create --stats --list borg@192.168.56.17:otusborg::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc > /var/log/borg_backup.log 2>&1

Проверить состояние на сервере

su borg -

borg list /home/borg/otusborg         

Получим

etc-2026-06-26_15:47:29              Fri, 2026-06-26 15:47:36 [35a11a55a42f0ca3313efbf0f54c3c2e24e6470d6e15dd00d137a6cce607a0f6]

При необходимости восстанавливаем бэкап: извлекаем на bksrv и забираем с cl

#bksrv

mkdir /home/borg/otusborg/etc_restore

borg extract /home/borg/otusborg::"etc-2026-06-26_15:47:29"            

#cl

rsync -avz --progress borg@192.168.56.17:/home/borg/otusborg/etc_restore/ /tmp/restored_etc/
