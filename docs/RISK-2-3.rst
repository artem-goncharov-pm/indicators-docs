﻿#######################################################################################
RISK-2-3. Переможцем закупівлі обрано учасника, у якого відсутні документи у тендерній пропозиції.
#######################################################################################

***************
Суть індикатора
***************

Замовником обрано переможцем процедури закупівлі учасника, яким у складі тендерної пропозиції не подано жодного документа, які вимагалися тендерною документацією

************************************
Законодавче обґрунтування індикатора
************************************

Свідчить про ймовірне порушення вимог частини першої статті 30 Закону в частині невідхилення замовником тендерної пропозиції учасника-переможця через її невідповідність умовам тендерної документації.

********************************
Підстава для розробки індикатора
********************************

Цей індикатор було розроблено, оскільки система електронних закупівель не передбачає перевірки наявності тендерної документації.

*********************************
Методологія розрахунку індикатора
*********************************

Досліджуються процедури закупівель відкриті торги, під час проведення яких замовник оприлюднив протокол розгляду тендерних пропозицій та повідомлення про намір укласти договір про закупівлю, на предмет завантаження учасником, якого у подальшому визнано переможцем процедури закупівель, документів (електронних файлів) у складі тендерної пропозиції. 
У разі не завантаження учасником, якого у подальшому визнано переможцем процедури закупівель, у складі тендерної пропозиції жодного документа (електронного файлу), окрім  електронного цифрового підпису (файл з розширенням *.p7s) спрацьовує індикатор ризику.


Етап існування процедури
========================
Індикатор розраховується, коли процедура знаходиться на етапі *тендерингу*.



Рівень розрахунку
=================
Індикатор розраховується на рівні *лоту*.

Джерела даних для розрахунку
============================

Для розрахунку індикатора вікористовуються наступні джерела даних:

- API модуля тендеринга електронної системи закупівель

Типи процедур
=============

Індикатор розраховується для наступних типів процедур:

- ``aboveThresholdUA`` - *відкриті торги*
- ``aboveThresholdEU`` - *відкриті торги з публікацією англійською мовою*

Типи замовників
===============

Індикатор розраховується для загальних замовників (``general``) та (``special``).


Стадії процедур
===============

Подія, що вмикає розрахунок індикатора
--------------------------------------

Подія, що вмикає розрахунок індикатора - перехід процедури у статус ``active.qualification``.

Подія, що вимикає розрахунок індикатора
---------------------------------------

Розрахунок індикатора вимикається, коли процедура переходить в статус ``complete``.

Статуси процедур
----------------

Виходячи з подій, що вмикають та вимикають розрахунок індикатора, маємо наступні умови розрахунку:

- Індикатор розраховується на наступні статуси процедур:
  
   - ``active.qualification``
   - ``active.awarded``

Частота розрахунку
==================

Індикатор розраховується при будь-якій зміні json-документа, що відповідає процедурі, якщо присутні всі умови для його розрахунку.

Окрім цього індикатор перераховується раз на добу незалежно від змін у json-документі, що відповідає процедурі, якщо присутні всі умови для його розрахунку.

Поля для розрахунку
===================

Для розрахунку індикатора використовуються наступні поля з API модуля тендеринга:

- ``data.awards``
- ``data.awards.status``
- ``data.awards.date``
- ``data.awards.bid_id``
- ``data.awards.lotID``
- ``data.bids.documents``
- ``data.bids.documents.datePublished``

Робота з датами
===============
Усі дати конвертуються до місцевої часової зони, враховуючи зимовий/літній час. Після конвертації залишаємо лише дату, відкидаючи час.

Кількість днів від Дати1 до Дати2 розраховуємо так: розрахунок починаємо від наступного дня від Дати1 и закінчуємо Датою2, тобто Дату1 в розрахунок не включаємо, а Дату2 в розрахунок включаємо.


Формула розрахунку
==================

Якщо попереднє розраховане значення індикатора дорівнює ``1``, то індикатор приймає значення ``1``. Розрахунок завершується.

Якщо в json-документі, що відповідає процедурі, немає жодного блоку ``data.awards``, де  ``data.awards.status = 'active'``, індикатор приймає значення ``-2``

Якщо в json-документі, що відповідає процедурі, присутній хоча б один блок ``data.awards``, де  ``data.awards.status = 'active'``, переходимо на наступний крок.

1. До уваги беруться усі об'єкти з блоку ``data.awards``, для яких виконується умова ``data.awards.status = 'active'``

2. Для кожного взятого до уваги об'єкта з блоку ``data.awards`` виконуються наступні дії:
 
  а) для процедури типу ``aboveThresholdUA`` порівнюються дати ``data.awards.date`` та *мінімальна* з дат ``data.bids.documents``, при умові ``data.awards.bid_id = bids.id``
  
  б) для процедури типу ``aboveThresholdEU`` порівнюються дати ``data.awards.date`` та *мінімальна* з дат документів зібраних з 3-х блоків: ``data.bids.documents``, ``data.bids.eligibilityDocuments`` та ``data.bids.financialDocuments``, при умові ``data.awards.bid_id = bids.id``

  в) якщо порівнювана дата ``data.bids.documents`` більша за дату ``data.awards.date``, то індикатор приймає значення ``1`` для лота ``data.awards.lotID``

5. Для процедур типу ``aboveThresholdUA``: якщо блок ``data.bids.documents``, що належить до об'єкту ``data.bids``, пов'язаного з даним об'єктом ``data.awards`` через ``data.awards.bid_id = bids.id``, відсутній на момент розрахунку, то індикатор приймає значення ``1`` для лота ``data.awards.lotID``

  5.1. Для процедур типу ``aboveThresholdEU``: якщо блоки ``data.bids.documents``, ``data.bids.eligibilityDocuments`` та ``data.bids.financialDocuments``, що належать до об'єкту ``data.bids``, пов'язаного з даним об'єктом ``data.awards`` через ``data.awards.bid_id = bids.id``, відсутні на момент розрахунку, то індикатор приймає значення ``1`` для лота ``data.awards.lotID``

6. Для процедур типу ``aboveThresholdUA``: якщо блок ``data.bids.documents``, що належить до об'єкту ``data.bids``, пов'язаного з даним об'єктом ``data.awards`` через ``data.awards.bid_id = bids.id``, має лише ``data.bids.documents.documentOf = 'lot'``, у яких ``data.bids.documents.relatedItem !=data.awards.lotID``, то індикатор приймає значення ``1`` для лота ``data.awards.lotID``.

  6.1. Для процедур типу ``aboveThresholdEU``: якщо блоки ``data.bids.documents``, ``data.bids.eligibilityDocuments`` та ``data.bids.financialDocuments``, що належать до об'єкту ``data.bids``, пов'язаного з даним об'єктом ``data.awards`` через ``data.awards.bid_id = bids.id``, мають лише ``data.bids.(documents/financialDocuments/eligibilityDocuments).documentOf = 'lot'``, у яких ``data.bids.(documents/financialDocuments/eligibilityDocuments).relatedItem !=data.awards.lotID``, то індикатор приймає значення ``1`` для лота ``data.awards.lotID``.

7. В інших випадках індикатор дорівнює ``0``.

Фактори, що впливають на неточність розрахунку
==============================================

1. Індикатор може бути порахований неточно у випадках, коли замовники в окремих сферах господарювання і організації, що не є замовниками, помилково визначають себе в системі як загальні замовники.

2. Індикатор може бути порахований неточно у випадках, коли замовником неправильно визначено тип процедури.
