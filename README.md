<h1 align="center">Проект по тестированию процесса учета сведений ЕГР ЗАГС о государственной регистрации рождения застрахованных лиц содержащиеся в Едином государственном реестре записей актов гражданского состояния</h1>

<hr>

## Проект реализован с использованием
Python = Pytest = Selenium = Selene = Selenoid = Allure Report = Jenkins = Telegram = Java = IBM WAS

![](/design/icons/Python.png)&emsp;![](/design/icons/Pytest.png)&emsp;![](/design/icons/Selenium.png)&emsp;![](/design/icons/Selene.png)&emsp;![](/design/icons/Selenoid.png)&emsp;![](/design/icons/Allure_Report.png)&emsp;![](/design/icons/Jenkins.png)&emsp;![](/design/icons/Telegram.png)&emsp;![](/design/icons/Java.png)&emsp;![ ](/design/icons/WAS.png)

<hr>

## Основные функции тестируемого процесса

* –	Прием перечня сведений о государственной регистрации рождения;
* –	Проверка перечня сведений о государственной регистрации рождения;
* –	Обработка перечня сведений о государственной регистрации рождения в подсистеме СПУ;
* –	Регистрация нового ИЛС ЗЛ в подсистеме СПУ;
* –	Поиск СНИЛС родителей для нового ИЛС ЗЛ;
* –	Сохранение информации по родителям для нового ИЛС ЗЛ.

<hr>

## Реализованные автотесты с разбивкой по группам

* Тест 1: укладка в БД + бизнес-проверки.
* Тест 3: В ЕХД, другие подсистемы (ПТК СПУ, ВИО, УТП)
* Тест 5: статус АЗ = 01 (исходные сведения)					
* Тест 7: статус АЗ = 02 (запись восстановлена), 04 (запись восстановлена по решению суда)					
* Тест 8: статус АЗ = 07 (изменение сведений)					
* Тест 9: Принятие решений по ошибкам в режиме "Поиск сведений по номеру АЗ" (загрузить документы, найти все АЗ в АРМе)					
* Тест 10: Проактивный СНИЛС (отправка квитанции, сохранение следа в БД)					
* Тест 11: Версии (автоматически)					

<hr>

## Основной шаблон выполнения шагов тест-кейса:
* Подготовка Базы Данных (удаление сведений по организациям и СНИЛС, заливка в БД исходных сведений по СНИЛС).
* Чтение файла КП (контрольные примеры) с набором тест-кейсов (чек-лист), настройками и скриптами для сверки в формате Excel.
* Открытие браузера и Авторизация в интерфейсе ПО АРМ СПУ (опционально)
* Укладка перечня РЖЗГС:
  * Распаковка zip-файла во временный каталог tmp с учётом кириллицы в названиях файлов
  * Отправка формы РЖЗГС в формате XML (PUT-запрос) и получение параметров из responce
  * корректировка полей "квитка" (файла в формате SOAP и сохранение в папке TMP в формате XML
  * отправка квитка через MQClient с помощью приложения и библиотек JAVA
* Ожидание окончания процесса укладки реестра (выполнение в цикле SQL-запроса к базе данных)
* Удаление записей в БД (опционально)
* Различные манипуляции в интерфейсе АРМ СПУ (опционально)
* Создание отчёта в Excel о расхождениях между ожидаемым и фактическим результатом (возможно до 15 и более Листов в Отчёте по количеству значимых шагов теста)
<hr>
 
## Файл КП для ручного тестирования, по которому были созданы автотесты.
* /data/Реестр_ЗАГС_о_РОЖДЕНИИ_ЕГР ЗАГС/КП_ЗАГС-РЖ.xlsx

## Файл КП для автотестов:
* /data/Реестр_ЗАГС_о_РОЖДЕНИИ_ЕГР ЗАГС/КП автотест ЗАГС-РЖ.xlsx

<hr>

## Как реализованы проверки в автотестах

Проверки в автотестах осуществляются после каждой укладки реестра. Обычно в тесте есть одна такая проверка, но возможно любое количество таких проверок.

Проверка представляет из себя сверку результатов выполнения заданных SQL-скриптов с контрольными (далее - Эталонами).
Контрольные результаты находятся в КП на Листах с соответствующими названиями.
Например, название Листа "T1TC4_1" означает, что здесь находятся Эталоны для теста T1TC4 (сам тест называется test_T1TC4), 1 - номер по порядку проверки в тесте.

Содержимое Листа представляет собой набор таблиц следующего содержания:
* Левая верхняя ячейка - уникальный код SQL-запроса
* Справа оn неё - сам SQL-запрос
* Ниже запроса находится строка с перечнем полей таблицы/таблиц базы данных, получаемых при запросе
* Ещё ниже - строки с ожидаемыми (контрольными) результатами SQL-запроса
 
![](/design/images/Эталон.png)

Тип проверки значения в каждой ячейки задаётся её цветом. В этом проекте используются цвета:
* Желтый -	проверяется точное совпадение значений в ячейке и соответствующего значения в выборке, получаемой в результате работы теста.
* ТемноКрасный -	текущая дата или текущий TimeStamp. Разрешается отклонение от текущего времени сервера на несколько минут.
* Фиолетовый -	максимальная дата (9999-12-31) или максимальный TimeStamp (23:59:59 9999-12-31 23:59:59)
* Голубой	- проверка осуществляется особым образом, контрольные значения задаются в тесте, либо формируются в POST-запросах.
* НетЗаливки	- проверка значения не проводится

Кроме того, в отчёте используется Розовый цвет для окраски ячеек с фактическими результатами теста, с расхождениями относительно Эталона.

Таблица цветов находится на Листе "Легенда" и считывается в процессе тестирования.
![](/design/images/Легенда.png)

Таблица соответствия SQL-запросов и их уникальных кодов находится на Листе "Эталон_Скрипты" и считывается в процессе тестирования.
![](/design/images/Эталон_Скрипты.png)

Тип проверки задаётся таблицей "Раскраска" на Листе "Легенда" и считывается в процессе тестирования.

Для каждого SQL-запроса через указание его уникального кода задаются цвета, в которые раскрашиваются ячейки Эталонов.
![](/design/images/Раскраска.png)

**В процессе тестирования** на этапе сверки последовательно выполняются SQL-запросы из таблицы "Эталон_Скрипты". Для каждой полученной таким образом выборки
выполняется сверка значений с Эталонными таблицами в соответствии с цветом ячеек и формируется файл Отчёта, который представляет собой набор Листов, каждый из которых
содержит набор эталонных таблиц и следующих за каждой из них таблицы с фактической выборкой:

![](/design/images/REPORT_regress_ZAGS_RG_T3TC1.png)

Расхождения реальных значений и эталонных выделены розовым цветом.

Кроме того, Лист, предшествующий Листу с таблицами, называется "Лог_1" ("Лог_2" и т.д.) и содержит сводную информацию о расхождениях:

![](/design/images/Лог_1.png)

Если расхождений нет, этот Лист выглядит так:

![](/design/images/Лог_2.png)
<hr>

## Формирование Эталонов

Автотестирование может быть выполнено в режиме создания Эталонов.
В этом случае по результатам работы выбранного набора тестов формируются Листы с выборками по SQL-запросам, ячейки автоматически раскрашиваются в соответствии с таблицей "Раскраска",
Листы компонуются в один файл Excel.

Тестировщику остаётся проверить выборки и заменить Листы в файле КП.
<hr>

## Запуск автотестов выполняется на сервере Jenkins

Командная строка:

pytest ${SELECT_TESTS} --browser=${browser} --browser_version=${browser_version} --window-size=${window_size} --bd_name=${bd_name}

### Параметры сборки

* SELECT_TESTS - Параметр определяет выбранную группу тестов.
* browser (default: chrome). Браузер chrome или firefox.
* browser_version (default: 96.0). Версия браузера на Selenoid.
* window_size (default: 1920x1080). Размер окна браузера.
* bd_name - База данных (тестовый стенд)

![](/design/images/jenkins1.png)

### 5. Результат запуска сборки можно посмотреть в отчёте Allure Report

![](/design/images/jenkins2.pn)

<hr>

## Настроено автоматическое оповещение о результатах сборки Jenkins в Telegram-бот

![](/design/images/telegram_bot.PNG)

<hr>

## Примечание:

Проект не может быть выложен на публичный ресурс по соображениям конфиденциальности.

