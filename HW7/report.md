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

* Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
* Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга? - да, если в таблице одна строка
