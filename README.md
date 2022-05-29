# **shipping_datamart**

## Overview

Витрина доставки интернет заказов.

## Структура таблицы

>Атрибут|Тип|Ограничение|Описание
>---|---|---|---
>shippingid|bigint|primary key|Идентификатор доставки
>vendorid|bigint|-|Идентификатор вендора
>transfer_type|text|-|Тип доставки
>full_day_at_shipping|integer|-|Количество полных дней, в течение которых длилась доставка
>is_delay|smallint|CHECK (is_delay IS NOT NULL AND is_delay IN (1, 0))|Статус, показывающий просрочена ли доставка
>is_shipping_finish|smallint|CHECK (is_shipping_finish IS NOT NULL AND is_shipping_finish IN (1, 0))|Статус, показывающий, что доставка завершена
>delay_day_at_shipping|integer|-|Количество дней, на которые была просрочена доставка.
>payment_amount|double precision|-|Сумма платежа пользователя
>vat|double precision|-|Итоговый налог на доставку
>profit|double precision|-|Итоговый доход компании с доставки

## Параметры хранения

>Наименование|Описание
>---|---
>База данных|PostgreSQL
>Название базы|de
>Название схемы|public
>Глубина хранения|с 2021 года
>Срок хранения|Постоянно
>Регламент|Обновление по запросу

## Источники

>Наименование|Описание
>---|---
>public.shipping_country_rates|Справочник стоимости доставки в страны
>public.shipping_agreement|Справочник тарифов доставки вендора по договору
>public.shipping_transfer|Справочник о типах доставки
>public.shipping_info|Таблица с доставками
>public.shipping_status|Таблица статусов доставок

## Описание источников:

- ## **public.shipping_country_rates**

>Атрибут|Тип|Ограниечение|Описание
>---|---|---|---
>id|serial|primary key|Идентификатор страны
>shipping_country|text|-|Страна доставки
>shipping_country_base_rate|double precision|-|Налог на доставку в страну

 - ## **public.shipping_agreement**

>Атрибут|Тип|Ограниечение|Описание
>---|---|---|---
>agreementid|integer|primary key|Идентификатор договора
>agreement_number|text|-|Номер договора в бухгалтерии
>agreement_rate|double precision|-|Ставка налога за стоимость доставки товара для вендора
>agreement_commission|double precision|-|Комиссия, то есть доля в платеже являющаяся доходом компании от сделки

 - ## **public.shipping_transfer**

>Атрибут|Тип|Ограниечение|Описание
>---|---|---|---
>id|serial|primary key|Идентификатор договора
>transfer_type|text|-|Тип доставки. 1p означает, что компания берёт ответственность за доставку на себя, 3p — что за отправку ответственен вендор
>transfer_model|text|-|Cпособ, которым заказ доставляется до точки
>shipping_transfer_rate|double precision|-|Процент стоимости доставки для вендора в зависимости от типа и модели доставки, который взимается интернет-магазином для покрытия расходов

 - ## **public.shipping_info**

>Атрибут|Тип|Ограниечение|Описание
>---|---|---|---
>shippingid|bigint|primary key|Идентификатор доставки
>vendorid|bigint|-|Идентификатор вендора
>payment_amount|double precision|-|Сумма платежа пользователя
>shipping_plan_datetime|timestamp|-|Плановая дата доставки
>transfer_type_id|bigint|foreign key (transfer_type_id) references public.shipping_transfer(id) ON UPDATE cascade|Идентификатор типа доставки
>shipping_country_id|bigint|foreign key (shipping_country_id) references public.>shipping_country_rates(id) ON UPDATE cascade|Идентификатор страны доставки
>agreementid|integer|foreign key (agreementid) references public.shipping_agreement(agreementid) ON UPDATE cascade|Идентификатор договора

 - ## **public.shipping_status**


> Атрибут|Тип|Ограниечение|Описание
> ---|---|---|---
> shippingid|bigint|primary key|Идентификатор доставки
> status|text|CHECK (status in ('in_progress', 'finished'))|Cтатус доставки.Может принимать значения in_progress — доставка в процессе, либо > finished — доставка завершена.
> state|text|CHECK (state in ('booked','fulfillment', 'queued', 'transition', 'pending', 'recieved', 'returned'))|Промежуточные точки заказа. booked - заказано. fulfillment — заказ доставлен на склад отправки. queued (пер. «в очереди») — заказ в очереди на запуск доставки. transition (пер. «передача») — запущена доставка заказа. pending (пер. «в ожидании») — заказ доставлен в пункт выдачи и ожидает получения. received (пер. «получено») — покупатель забрал заказ. returned (пер. «возвращено») — покупатель возвратил заказ после того, как его забрал
> shipping_start_fact_datetime|timestamp|-|Время, когда *state* заказа перешёл в состояние *booked*
> shipping_end_fact_datetime|timestamp|-|Время, когда *state* заказа перешёл в состояние *received*
