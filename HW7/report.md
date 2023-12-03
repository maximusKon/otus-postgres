* Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения. - Сделано
```
11-26 15:14:24.183 UTC [18689] postgres@locks LOG:  process 18689 still waiting for ShareLock on transaction 739 after 200.147 ms
2023-11-26 15:14:24.183 UTC [18689] postgres@locks DETAIL:  Process holding the lock: 18678. Wait queue: 18689.
2023-11-26 15:14:24.183 UTC [18689] postgres@locks CONTEXT:  while updating tuple (0,1) in relation "accounts"
2023-11-26 15:14:24.183 UTC [18689] postgres@locks STATEMENT:  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
```
* Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.
```
 locktype | relation |       mode       | granted |  pid  | wait_for
----------+----------+------------------+---------+-------+----------
 relation | accounts | RowExclusiveLock | t       | 18678 | {}
 relation | accounts | RowExclusiveLock | t       | 18689 | {18678}
 tuple    | accounts | ExclusiveLock    | t       | 18689 | {18678}
 relation | accounts | RowExclusiveLock | t       | 18801 | {18689}
 tuple    | accounts | ExclusiveLock    | f       | 18801 | {18689}
```
Первая - блокировка таблицы accounts при UPDATE
Вторая - то же самое, но транзакция ожидает снятие блокировки процессом 18678
Третья - блокировка на конкретную строку, которую планируется апдейтить
Четвёртая - блокировка таблицы accounts, но ожидающая процесс 18689
Пятая - блокировка на конкретную строку, которую планируется апдейтить, но ещё не вручённая этой транзакции, так как сначала апдейтить будет процесс 18678, иначе будет конфликт.

* Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений? - В каждой транзакции первой командой апдейтим три разные строчки, соответственно 1, 2 и 3. Далее 3-я транзакция пытается апдейтнуть строку 1 - лочится, 1-я транзакция пятается апдейтнуть строку 2 - лочится, 2-я транзакция пытается апдейтнуть строку 3 - сваливаемся в дедлок:
```
ERROR:  deadlock detected
DETAIL:  Process 2097 waits for ShareLock on transaction 751; blocked by process 2183.
Process 2183 waits for ShareLock on transaction 749; blocked by process 1090.
Process 1090 waits for ShareLock on transaction 750; blocked by process 2097.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,15) in relation "accounts"
```
Эта информация также будет записана в лог фаил
* Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга? - сомнительно, ведь первая получит лок на все строчки. Во всяком случае воспроизвести не смог.
