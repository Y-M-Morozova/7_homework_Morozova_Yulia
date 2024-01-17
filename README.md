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

    
   
