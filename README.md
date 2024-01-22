<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>


<div align=center><h3>Домашнее задание №7 по теме: «Блокировки PostgreSQL»</h3></div>  

***

<br/><br/>

**I. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.**

1. На моей ранее созданной ВМ в ЯО устанавливаю устанавливаю PostgreSQL 15 версии, подключаюсь к нему, для тестов создаю БД ``locks``, подключаюсь, проверяю соответствующие параметры: ``log_lock_waits`` , ``deadlock_timeout`` , изменяю их согласно условию задания:
    ```sql
    alter system set log_lock_waits = on;
    alter system set deadlock_timeout = 200;
    ```
    
    обновляю конфигурацию кластера, снова проверяю, все ок:

    ![1_1](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/5740c0dc-4228-4b27-b582-e286e667cbf0)

<br/><br/>

2. Для того, чтобы воспроизвести ситуацию, при которой в журнале появятся такие сообщения, сначала сделаю таблицу ``accounts`` , вставлю три строки:

    ![1_2](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/064853aa-fb67-4c78-8b6a-828f180f5327)

     открываю еще один терминал ssh , подключаюсь к БД locks, селект из таблицы  ``accounts`` - ок:

      ![1_3](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/f0dd5411-eb96-446c-919a-779fb1397e5d)

 
   в первом терминале начинаю транзакцию и пытаюсь обновить строку:

    ![1_22_1](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/484491de-7fe5-49a8-9450-a6073a8dd002)

   во втором терминале тоже открываю транзакцию и тоже пытаюсь обновить эту же строку и подвисаю:

    ![1_22_2](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/970a4708-b19f-4204-b192-b6caf073061e)

   жду более 200 мс и первом терминале завершаю транзакцию и смотрю номер обслуживающего процесса:  

   ![1_22_3](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/d8ebee3a-cc75-4906-b464-b1ede00b4d07)

    во втором терминале зависания уже нет, завершаю транзакцию и тоже смотрю номер обслуживающего процесса:

   ![1_22_4](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/e91b2e93-1e6a-489f-b4a1-9dc09d01234a)

   для наглядности логов, открываю третий терминал, в нем смотрю лог и да, информация про блокировки сессий с соответсвующими номерами и операторами  в лог. файле есть:

   ![1_22_5](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/c6ed31a9-dc75-4800-b523-2b39fd4a6019)

**Таким образом делаю вывод, что я настроила postgres так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд.**
   
<br/><br/>

**II.Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.**

1. Открываю 3 терминала(ssh) и в первом создаю представление над pg_locks ``locks_v``:

    ```sql
    CREATE VIEW locks_v AS
    SELECT pid,
       locktype,
       CASE locktype
         WHEN 'relation' THEN relation::REGCLASS::text
         WHEN 'virtualxid' THEN virtualxid::text
         WHEN 'transactionid' THEN transactionid::text
         WHEN 'tuple' THEN relation::REGCLASS::text||':'||tuple::text
       END AS lockid,
       mode,
       granted
    FROM pg_locks;
    ```

    ![2_view](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/35da1266-96ff-4475-8943-d207f2b4bad8)

2. во всех 3х терминалах идентифицирую транзакции и сессии, начинаю транзакцию и пытаюсь обновить одну и ту же строку командой: ``UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;``

   Первая транзакция обновляет и ожидаемо блокирует строку:

    ![2_1](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/d878c467-3143-448e-8d45-c2603e647f2e)
  
   вторая транзакция пытается тоже обновить ту же самую строку и ожидаемо подвисает:

    ![2_2](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/4033c09b-cd79-4517-8bc1-e689d651694d)


   и третья тоже самое обновляет и так же подвисает:

    ![2_3](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/328819ed-7e9e-4b36-9490-d5da7febfb54)
   
 3. Смотрю блокировки транзакций с помощью созданного представления ``locks_v`` над ``pg_locks``.

    Первая транзакция:  

    ![2_7](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/5f5aa3f7-f4e8-4309-96db-af8047b97f29)

    вижу блокировки типа ``relation`` для таблицы ``accounts`` и первичного ключа таблицы ``accounts_pkey`` в режиме ``RowExclusiveLock`` - устанавливается на изменяемое отношение,
    <br>так же вижу блокировки типов ``virtualxid`` (виртуальный идентификатор транзакции) и ``transactionid``(индентификатор транзакции) в режиме ``ExclusiveLock`` - удерживаются каждой транзакцией для самой себя.

    Вторая транзакция:

    ![2_7_2](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/9101854f-9adb-406f-8663-5223d2405a02)

    вижу, что если сравнить с блокировками первой транзакции, вторая транзакция еще ожидает получение блокировки типа ``transactionid`` в режиме ``ShareLock`` для первой транзакции(``granted = f``),
    так же вижу, что удерживается блокировка типа ``tuple`` для обновляемой строки.

    Третья транзакция:

    ![2_7_3](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/82892287-1bb7-42a8-9ae0-7667302dec83)

    вижу, что третья транзакция так же ожидает получение блокировки типа tuple для обновляемой строки(так же ``granted = f``).

    Общую картину текущих ожиданий можно увидеть в представлении pg_stat_activity. Для удобства можно добавить и информацию о блокирующих процессах:

    ```sql
     SELECT pid, wait_event_type, wait_event, pg_blocking_pids(pid) 
     FROM pg_stat_activity 
     WHERE backend_type = 'client backend';
    ```

    ![2_7_7](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/7215b2f4-10c7-4c75-91be-b996921bc736)

    Получается своеобразная «очередь», в которой есть первый (тот, кто удерживает блокировку версии строки - у меня это сессия 1481) и все остальные, выстроившиеся за первым(это сессии сначала 1875, а затем 1926).

<br/><br/>
    
**III.Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?**

1. Для воспроизведения взаимоблокировки(ошибки deadlock) откроем три терминала (ssh) и в каждом будем выполнять команды ``update`` таблицы ``accounts``.

   в первом терминале открываю транзакцию и выполним команду ``UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;``:

    ![3_1_1](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/87eab4da-8e68-4da1-aa69-8a443790ccc5)

   в втором терминале открываю транзакцию и выполним команду ``UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 2;``:    

    ![3_1_2](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/2b6c7977-4470-48eb-9fb6-995227f9257f)


   в третьем терминале открываю транзакцию и выполним команду ``UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 3;``:  

    ![3_1_3](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/6ff645c3-fc0e-4798-8a11-9c6ca23b0772)

    далее в первом терминале выполняю ``UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 2;``:

    ![4_1_1](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/6cc529b2-1e08-43f3-a3d0-743bfcb75d08)

    далее во втором терминале выполняю ``UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 3;``:

    ![4_1_2](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/5e893431-1a02-4582-b699-10aa17c503f4)


    далее в третьем терминале выполняю ``UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;``,
    <br> но так как возникает циклическое ожидание, которое никогда не завершится само по себе, то третья транзакция, не получив доступ к ресурсу, инициирует проверку взаимоблокировки
    и обрывается сервером(вижу ошибку взаимоблокировки ``ERROR:  deadlock detected`` в третьем терминале):   
   ![4_1_3](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/7d71dbc2-55c8-4da4-bc54-11e22c715c10)

    Выполняю ``commit`` для завершения транзакций в терминалах, в первом и втором - транзакции завершаются успешно, третья транзакция не может быть завершена успешно, как смотрели выше - по ошибке             
    взаимоблокировки:

   ![4_итог](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/b38dbf97-1742-4f80-a5ad-2e347ee8d4ac)

2. В данной ситуации да, можно разобраться и постфактум, если изучить лог, открываю файл и вижу, что ошибка ``deadlock`` зафиксирована , так же вижу все операторы до этой ошибки и после, то есть есть возможнсть провести анализ и полностью разобраться постфактум:

   ![5](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/1dfb13b9-9607-4ac1-b0dc-066ed8fa977d)

   ***     
### * Задание со звездочкой *

**Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга? Попробуйте воспроизвести такую ситуацию.**

1.  Взаимоблокировка двух транзакций, выполняющих ``update`` одной и той же таблицы (без оператора ``where``) возможна, хотя в реальной работе она маловероятна, но получить её специально можно.
    <br>Команда ``update`` блокирует строки по мере их обновления и это происходит не одномоментно. Поэтому если одна команда будет обновлять строки в одном порядке, а другая - в другом, то они могут                взаимозаблокироваться.
    <br>Это может произойти, если для команд будут построены разные планы выполнения: например, одна будет читать таблицу последовательно, а другая - по индексу. Проиллюстрирую это с помощью курсоров, поскольку     это дает возможность управлять порядком чтения.

2.  Открываю два терминала (ssh) и в каждом определяю курсоры.
    <br>В первом терминале открываю транзакцию и декларирую курсор ``c1``:

    ```sql
        DECLARE c1 CURSOR FOR
        SELECT * FROM accounts ORDER BY acc_no
        FOR UPDATE;
    ```

    ![6_1](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/fee95cd3-7faa-4699-858e-a94d214a04d6)

     Во втором терминале открываю транзакцию и декларирую курсор ``c2`` , но тут выборка пойдет уже в другую(обратную) сторону - определяем с помощью ``desc``:

    ```sql
        DECLARE c2 CURSOR FOR
        SELECT * FROM accounts ORDER BY acc_no DESC 
        FOR UPDATE;
    ```

    ![6_2](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/deb1e725-868f-4caf-8034-5873949516dd)

  3. Получаем результаты запроса через курсор с помощью ``fetch``
     <br> в первом терминале командой: ``FETCH c1;``, а во втором терминале терминале командой: ``FETCH c2;`` :

     ![7_итого](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/2bbec1b4-1b12-4e4b-a9a5-95be3ac0429e)

     

     

  

    
