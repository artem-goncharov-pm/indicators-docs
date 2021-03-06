﻿##############################################################################
RISK-1-20. Неоприлюднення тендерної документації замовником.
##############################################################################

***************
Суть індикатора
***************

Замовником оприлюднено оголошення про проведення процедури закупівлі без оприлюднення тендерної документації.
Даний індикатор виявляє ситуації, коли на момент визначення переможця замовником тендерна документація не опублікована.

************************************
Законодавче обґрунтування індикатора
************************************

Відсутність тендерної документації є порушенням частини 1 статті 10 Закону України "Про публічні закупівлі".

********************************
Підстава для розробки індикатора
********************************

Цей індикатор було розроблено, оскільки система електронних закупівель не передбачає перевірки наявності тендерної документації.

*********************************
Методологія розрахунку індикатора
*********************************


Етап існування процедури
========================
Індикатор розраховується, коли процедура знаходиться на етапі *тендерингу*.


Рівень розрахунку
=================
Індикатор розраховується на рівні *тендера*.

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

Розрахунок індикатора вимикається, коли процедура переходить в статус "Завершена".

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
- ``data.documents``
- ``data.documents.author``
- ``data.documents.format``
- ``data.documents.datePublished``

Формула розрахунку
==================

Якщо в json-документі, що відповідає процедурі, для жодного об'єкта  відсутній блок ``data.awards``, де  виконується умова ``data.awards.status = 'active'``, індикатор приймає значення ``-2``. Розрахунок завершується.

Якщо в json-документі, що відповідає процедурі, присутній блок ``data.awards``, де хоча б в одному об'єкті виконується умова ``data.awards.status = 'active'``, перехлдмо на наступний крок.

1. До уваги беруться усі об'єкти з блоку ``data.documents`` для яких виконуються наступні умови:

  а) ``data.documents.author`` *не дорівнює* ``'auction'``

  б) ``data.documents.format`` *не дорівнює* ``'application/pkcs7-signature'``

2. Серед документів, що були взяті до уваги, обраховується найраніша дата публікації, тобто *мінімальна* з дат ``data.documents.datePublished``

3. До уваги беруться усі об'єкти з блоку ``data.awards``, для яких виконується умова ``data.awards.status = 'active'``

4. Для кожного взятого до уваги об'єкта з блоку ``data.awards`` виконуються наступні дії:
 
  а) порівнюються дати ``data.awards.date`` та дата, визначена в п.2. 

  б) якщо дата, визначена в п.2, більша за дату ``data.awards.date``, то індикатор приймає значення ``1``

5. Якщо на момент розрахунку блок ``data.documents`` відсутній, або в ньому нема жодного документу, що може бути взятий до розгляду відповідно до п.1, то індикатор приймає значення ``1``

Фактори, що впливають на неточність розрахунку
==============================================

1. Індикатор може бути порахований неточно у випадках, коли замовники в окремих сферах господарювання і організації, що не є замовниками, помилково визначають себе в системі як загальні замовники.

2. Індикатор може бути порахований неточно у випадках, коли замовником неправильно визначено тип процедури.
