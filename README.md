<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>


<div align=center><h3>Домашнее задание №7 по теме: «Блокировки PostgreSQL»</h3></div>  

***

<br/><br/>

**I. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.**

1. На моей ранее созданной ВМ в ЯО устанавливаю устанавливаю PostgreSQL 15 версии, подключаюсь к нему, для тестов создаю БД ``locks``, подключаюсь, проверяю соответствующие параметры: ``log_lock_waits`` , ``deadlock_timeout`` , изменяю их, обновляю конфигурацию, снова проверяю, все ок:

    ![1_1](https://github.com/Y-M-Morozova/7_homework_Morozova_Yulia/assets/153178571/b32600d1-8d9f-4d4d-9ba9-788edf99c385)

<br/><br/>

