Тестовое задание
============
Задача:
Написать скрипт для рассылки сообщений через vk API

В таблице могут распологаться до 100 миллионов записей (на них и будем тестировать)
Таблица - id,name
Скрипт должен запускаться параллельно самому себе и не отправлять сообщения одному и томуже по несколько раз.
Сообщение должно быть "Привет %name% как дела?" где %name% имя пользователя

Ограничения vk API
Скрипту можно передавать не более 100 идентификаторов пользователей.
Не более 3х вызовов в секунду.



Приступим.

Порассуждаем логически - можно отправить до 100 ИДшников, но текст у всех должен зависеть от имени.
Делаем выборку - group by name и узнаем сколько каких имено одинаковых.

Все это запихиваем в мескэш, и делаем команду скрипту брать по 50к обьектов.
Обновляем мемкэш (уменьшаем количество на 50к и присваиваем номер итерации) После этого проводжим все тяжелые операции (выборка из БД, перебор обьектов.)
У нас есть 50к или меньше обьектов которым нужно отправить одинаковый текст (имя то одинаковое)
Забиваем их все в массив, и на каждом сотом элементе делаем отправку и заносим все успешные ИД в мемкэш. После 3х отправок сверяем время, если прошло меньше секунды то ждем 1 секунду. и обнорвляем БД.
Если скрипт упал/остановился/вырубили электричество итп, то некотоыре ИД уже получили сообщения, но в базу еще не занесены. Они остаются в мемкэше и при запуске скрипта, или работе параллельного скрипта мы их заносим в базу. Никаких повторов, высокая скорость работы, многопоточность.

PROFIT

По поводу логирования - Класс логов присваивается базовому классу. Его можно переопределить. В моем случае я все это заношу в файл посредством system('echo >>.... тк быстрее всего работает с большими файлами.
