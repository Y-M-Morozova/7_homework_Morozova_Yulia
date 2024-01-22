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
