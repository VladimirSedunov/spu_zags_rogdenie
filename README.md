<h1 align="center">Проект по тестированию процесса учета сведений ЕГР ЗАГС о государственной регистрации рождения застрахованных лиц содержащиеся в Едином государственном реестре записей актов гражданского состояния</h1>




<hr>

## Проект реализован с использованием
Python = Pytest = Selenium = Selene = Selenoid = Allure Report = Jenkins = Telegram = Java = IBM WAS

![](/design/icons/Python.png)&emsp;![](/design/icons/Pytest.png)&emsp;![](/design/icons/Selenium.png)&emsp;![](/design/icons/Selene.png)&emsp;![](/design/icons/Selenoid.png)&emsp;![](/design/icons/Allure_Report.png)&emsp;![](/design/icons/Jenkins.png)&emsp;![](/design/icons/Telegram.png)&emsp;![](/design/icons/Java.png)&emsp;![ ](/design/icons/WAS.png)

<hr>

## Основные понятия и функции тестируемого процесса

Пакет с отчетностью ФНС «за НР» состоит из документов:
* –	Документ «Расчёт по страховым взносам» – VO_RASCHOPS.
* –	Унифицированный протокол проверки - УПП.

Пакет с отчетностью ФНС для ФЛ/ИП/КФХ «за себя» состоит из документов:
* –	Документ «Расчёт по страховым взносам» – VO_RASCHOPS.
* –	Документ «Уплата страхователем страховых взносов» – VO_SVEDUPLSV.
* –	Унифицированный протокол проверки – УПП.

Основные функции:
Подсистема СПУ должна обеспечить выполнение следующих функций:
* –	Прием и обработку пакета отчетности ФНС по наемным рабочим с учетом УПП; 
* –	Прием и обработку пакета отчетности по КФХ за себя / КФХ «за члена КФХ» с учетом VO_SVEDUPLSV и УПП;
* –	Прием и обработку пакета отчетности по ИП/ФЛ «за себя» с учетом VO_SVEDUPLSV и УПП;
* –	Прием и обработку пакета отчетности по ИП АУСН с учетом УПП;
* –	Автоматическое формирование стажа по отчетности ИП/ФЛ/ИП АУСН;
* –	Автоматический расчет уплат по ИП/ФЛ/КФХ «за себя»;
* –	Формирование протокола разнесения сведений.

Документы, участвующие в технологическом процессе:
* Входящие документы по процессу:
  * Расчет по СВ в части начислений по СВ на ОПС	(VO_RASCHOPS)
  * Уплата страхователями СВ на ОПС для целей ведения ПУ	(VO_SVEDUPLSV)
  * Унифицированный протокол проверки	(УНИФИЦИРОВАННЫЙ_ПРОТОКОЛ_ПРОВЕРКИ)
* Исходящие документы по процессу:
  * Протокол разнесения сведений пакета с расчетом и уплатами	(ПРС VO_RASCHOPS)
<hr>

## Основной шаблон выполнения шагов тест-кейса:
* Подготовка База Данных (удаление сведений по Расчопс, Налпров, Сведупл по указанным организациям за все периоды, выполнение скриптов UPDATE организаций)
* Чтение файла КП (контрольные примеры) с набором тест-кейсов (чек-лист), настройками и скриптами для сверки в формате Excel.
* Укладка реестра:
  * Распаковка zip-файла во временный каталог tmp с учётом кириллицы в названиях файлов
  * -
  * подготовка файла описателя VO_SVEDUPLSV в формате json (корректировка полей) из шаблонного файла
  * отправка опияателя (POST-запрос) и получение значения id из response
  * отправка VO_SVEDUPLSV (файл в формате XML) на URL, зависящий от id, и получение значения documentId из response в формате JSON
  * -
  * подготовка файла описателя УПП в формате json (корректировка полей) из шаблонного файла
  * отправка описателя УПП (POST-запрос) и получение значения id из response
  * отправка УПП (файл в формате XML) на URL, зависящий от id, и получение значения documentId из response в формате JSON
  * -
  * подготовка файла описателя VO_RASCHOPS в формате json (корректировка полей) из шаблонного файла
  * отправка описателя VO_RASCHOPS (POST-запрос) и получение значения id из response
  * отправка VO_RASCHOPS (файл в формате XML) на URL, зависящий от id, и получение значения documentId из response в формате JSON
  * -
  * корректировка множества полей файла в формате SOAP из zip-архива ("квиток") и сохранение в папке TMP в формате XML
  * отправка квитка через MQClient с помощью приложения и библиотек JAVA
* Ожидание окончания процесса укладки реестра (выполнение в цикле пяти различных SQL-запросов к базе данных)
* Создание отчёта в Excel о расхождениях между ожидаемым и фактическим результатом
<hr>
 
## Фрагмент КП для ручного тестирования.
Тест-кейсов очень много, из них отобраны 103 теста для автоматизации.
![](/design/images/КП.PNG)
<hr>

## Файл КП для автотестов:
* /data/Реестр_ЗАГС_о_РОЖДЕНИИ_ЕГР ЗАГС/КП автотест ЗАГС-РЖ.xlsx

<hr>

## Как реализованы проверки в автотестах

Проверки в автотестах осуществляются после каждой укладки реестра. Обычно в тесте есть одна такая проверка, но возможно любое количество таких проверок.

Проверка представляет из себя сверку результатов выполнения заданных SQL-скриптов с контрольными (далее - Эталонами).
Контрольные результаты находятся в КП на Листах с соответствующими названиями.
Например, название Листа "T4TC2_404_1" означает, что здесь находятся Эталоны для теста T4TC2 (сам тест называется test_T4TC2), 404 - условный номер Альбома форматов (т.е. схем XSD), 1 - номер по порядку проверки в тесте.

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

![](/design/images/REPORT_T3TC1_4010.png)

Расхождения реальных значений и эталонных выделены розовым цветом.

Кроме того, Лист, предшествующий Листу с таблицами, называется "Лог_1" ("Лог_2" и т.д.) и содержит сводную информацию о расхождениях:

![](/design/images/Лог_1.png)

Если расхождений нет, этот Лист выглядит так:

![](/design/images/Лог_2.PNG)
<hr>

## Формирование Эталонов

Автотестирование может быть выполнено в режиме создания Эталонов.
В этом случае по результатам работы выбранного набора тестов формируются Листы с выборками по SQL-запросам, ячейки автоматически раскрашиваются в соответствии с таблицей "Раскраска",
Листы компонуются в один файл Excel.

Тестировщику остаётся проверить выборки и заменить Листы в файле КП.
<hr>

## Запуск автотестов выполняется на сервере Jenkins

Командная строка:

pytest ${SELECT_TESTS} ${SUMMARY_DIFF_LOG} --bd_name=${bd_name}

### Параметры сборки

* SELECT_TESTS - Параметр определяет выбранную группу тестов.
* bd_name - База данных (тестовый стенд)
* SUMMARY_DIFF_LOG - Параметр задаёт необходимость прикрепления к Allure-отчёту итоговой информации о расхождениях

![](/design/images/jenkins1.png)

### 5. Результат запуска сборки можно посмотреть в отчёте Allure Report

![](/design/images/jenkins2.png)

<hr>

## Настроено автоматическое оповещение о результатах сборки Jenkins в Telegram-бот

![](/design/images/telegram_bot.PNG)

<hr>

## Примечание:

Проект не может быть выложен на публичный ресурс по соображениям конфиденциальности.

