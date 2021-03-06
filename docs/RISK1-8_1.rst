﻿###############################################################################
RISK-1-8-1. Несвоєчасне укладання замовником договору про закупівлю за результатами проведення процедури закупівлі відкриті торги
###############################################################################

***************
Суть індикатора
***************

Даний індикатор виявляє ситуації, коли договір про закупівлю укладено менше ніж через 10 днів з дати визначення переможця.

************************************
Законодавче обґрунтування індикатора
************************************

Укладення договору про закупівлю менше ніж через 10 днів з дати визначення переможця є порушенням частини 2 статті 32 Закону України "Про публічні закупівлі".

*********************************
Методологія розрахунку індикатора
*********************************

Етап існування процедури
========================
Індикатор розраховується, коли процедура знаходиться на етапі *тендерингу*.

Рівень розрахунку
=================

Індикатор розраховується на рівні *лота*.

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

Індикатор розраховується для замовників типу (``general``) та (``special``).

Очікувана вартість процедур
===========================

Індикатор розраховується для порцедур в яких очікувана вартість ``value:amount`` перевищує наступні встановлені законом пороги:

1) Для замовників типу (``general``) очікувана вартість закупівлі більше 200 000 грн для товарів та послуг, та більше 1 500 000 для робіт. 
2) Для замовників типу  (``special``) очікувана вартість закупівлі більше 1 000 000 грн для товарів та послуг, та більше 5 000 000 для робіт. 

Розподілення на роботи та послуги в CPV 45. На разі закупівлі з CPV 45 вважаються як "роботи" за виключенням коли в назві закупівлі присутні такі буквосполучання як "поточ" та "послуг" - такі закупівлі відносяться до послуг та застосовуються відповідні пороги та інші норми закону.

Стадії процедур
===============

Подія, що вмикає розрахунок індикатора
--------------------------------------

Подія, що вмикає розрахунок індикатора - прехід процедури у статус "визначення переможця" (``data.status = 'active.qualification'``).

Подія, що вимикає розрахунок індикатора
---------------------------------------

Розрахунок індикатора для даного лота вимикається одразу після того, як останнє значення індикатору ``0`` або ``1``.

Статуси процедур
----------------

Виходячи з подій, що вмикають та вимикають розрахунок індикатора, маємо наступні умови розрахунку:

- Індикатор розраховується для наступних статусів процедур:

  - ``active.awarded``
  - ``active.qualification``
  - ``complete``

Додання статусу ``active.qualification`` пов'язане з тим фактом, що процедура переходить у статус ``active.awarded`` тільки тоді, коли переможця визначено по всім лотам.

У випадку, коли переможця визначено по частині лотів, процедура залишається в статусі ``active.qualification``, але вже є можливість розраховувати індикатор для лотів, де переможця визначено.

Частота розрахунку
==================

Індикатор розраховується для даного лоту кожні 30 хв., якщо наявні умови для його розрахунку.

Поля для розрахунку
===================

Для розрахунку індикатора використовуються наступні поля з API модуля тендеринга:

- ``data.awards``
- ``data.awards.lotID``
- ``data.awards.date``
- ``data.contracts``
- ``data.contracts.awardID``
- ``data.contracts.status``
- ``data.contracts.dateSigned``

Визначення часового інтервалу
=============================

Часовий інтервал розраховується наступним чином:
 + від часової мітки ми відкидаємо час, залишаємо лише дату;
 + відкидаємо дані про часовий пояс;
 + відлік ведемо лише по днях, починаючи з наступного від початкової дати та завершуючи днем кінцевої дати.

Формула розрахунку
==================

Якщо у json-документі в блоці ``data.contracts`` відсутній об'єкт, що посилається на цей лот через ланцюг ``data.contracts.awardID``-``data.awards.lotID``, в якому ``data.contracts.status = 'active'``, індикатор приймає значення ``-2``.

Якщо у json-документі в блоці ``data.contracts`` присутній об'єкт, що посилається на цей лот через ланцюг ``data.contracts.awardID``-``data.awards.lotID``, в якому ``data.contracts.status = 'active'``, переходимо на наступний крок.

Індикатор приймає значення ``1`` для лота, якщо виконуються всі нижченаведені умови.

1. На цей лот через ланцюг ``data.contracts.awardID``-``data.awards.lotID`` поислається об'єкт ``data.contracts``, у якого ``data.contracts.status = 'active'``

2. Дата ``data.awards.date`` з об'єкту ``data.awards.``, що посилається на даний лот через ``data.awards.lotID``, та найраніша дата з ``data.contracts.documents.dateModified`` з об'єкту ``data.contracts``, де ``data.contracts.documents.format != 'application/pkcs7-signature'``, що має ``data.contracts.status = 'active'`` та посилається на цей лот через ланцюг ``data.contracts.awardID``-``data.awards.lotID``, відрізняються менше ніж на 10 днів. Для розрахунку беремо лише дати без часу. Не переводимо часові пояси.

В інших випадках індикатор дорівнює ``0``.

Примітка: як було надано пояснення розрахунок днів здійснювати наступного дня з дати оприлюднення повідомлення про намір укласти договір про закупівлю.

Фактори, що впливають на неточність розрахунку
==============================================

1. Індикатор може бути порахований неточно у випадках, коли замовники в окремих сферах господарювання і організації, що не є замовниками, помилково визначають себе в системі як загальні замовники.

2. Індикатор може бути порахований неточно у випадках, коли замовником неправильно визначено тип процедури.

3. Очікувана вартість процедур
===========================

Індикатор розраховується для порцедур в яких очікувана вартість ``value:amount`` перевищує наступні встановлені законом пороги:

1) Для замовників типу (``general``) очікувана вартість закупівлі більше 200 000 грн для товарів та послуг, та більше 1 500 000 для робіт. 
2) Для замовників типу  (``special``) очікувана вартість закупівлі більше 1 000 000 грн для товарів та послуг, та більше 5 000 000 для робіт. 

3) Розподілення на роботи та послуги в CPV 45. На разі закупівлі з CPV 45 вважаються як "роботи" за виключенням коли в назві закупівлі присутні такі буквосполучання як "поточ" та "послуг" - такі закупівлі відносяться до послуг та застосовуються відповідні пороги та інші норми закону.
