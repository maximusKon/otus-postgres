1. Нет. Так как уровень изоляции транзакции read committed, а изменения в первой сессии ещё не были закомичены.
2. Да, так как теперь изменения закомичены.
3. Нет. Так как уровень изоляции транзакции repeatable read, а изменения в первой сессии ещё не были закомичены.
4. Нет, так как хоть изменения и закомичены, но на уровне изоляции repeatable read мы так же защищены от фантомного чтения
5. Теперь транзакция закоммичена и нам доступны данные, закоммиченные первой транзакцией
