SOAP без SessionID
==================


Обзор API
---------

Предоставляемый API Сервиса отправки SMS сообщений позволяет осуществить:

* получение баланса пользователя текущей сессии;
* получение входящих сообщений пользователя текущей сессии;
* отправка SMS c учетом часового пояса получателя;
* отправка SMS абонентам и возвращение системных идентификаторов SMS;
* получение статуса отправленного SMS и время обновления статуса;
* получение статистики по SMS-рассылкам за заданный промежуток времени

.. warning:: Внимание! Для использования данного вида интеграции необходимо обратиться к своему менеджеру, либо в техническую поддержку support@devinotele.com для настройки доступа.

API Сервиса отправки SMS организовано в соответствии с принципами SOAP. Протокол используется для обмена произвольными сообщениями в формате XML. SOAP может использоваться с любым протоколом прикладного уровня: *SMTP, FTP, HTTP, HTTPS и др.*

WSDL-документ для SOAP доступен по адресу: 

.. code-block:: json

    https://ws.devinotele.com/SmsServicev2.asmx?WSDL
    

Точка подключения: 

.. code-block:: json

    https://ws.devinotele.com/SmsServicev2.asmx
    

.. warning:: Все запросы необходимо выполнять в кодировке UTF-8. Количество запросов 10 запросов/1 сек

Исключительные ситуации.
В случае возникновения исключительной ситуации во время обработки запроса или ошибки
аутентификации, сервис возвращает код ошибки в виде:

.. code-block:: xml

    <soap:Code>
        <soap:Value>soap:Receiver</soap:Value>
    </soap:Code>
    <soap:Reason>
        <soap:Text xml:lang="en">
            Server was unable to process request. ---
            &gt; Invalid user login or password
        </soap:Text>
    </soap:Reason>
    

Получение баланса пользователя
------------------------------

Сервис возвращает значение баланса авторизованного пользователя по Login/Password. Овердрафт при
этом учитывается.
Пример запроса:

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
       <soap12:Body>
            <GetBalance xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
            </GetBalance>
        </soap12:Body>
    </soap12:Envelope>
    

Таблица 1 - Описание параметров GetBalance

+----------------+------------+--------------+--------------------------------------+
|     Параметр   | Тип данных |Обязательность| Описание                             |
+================+============+==============+======================================+
| Login          |  String    | Да           | Логин, полученный при регистрации    |
+----------------+------------+--------------+--------------------------------------+
| Password       |  String    | Да           | Пароль, соответствующий логину       |
+----------------+------------+--------------+--------------------------------------+

Сервис проверяет валидность Login/Password и, в случае успеха, авторизует Пользователя и в ответе присылает баланс пользователя следующего вида. Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <GetBalanceResponse xmlns="http://ws.devinosms.com">
                <GetBalanceResult>decimal</GetBalanceResult>
            </GetBalanceResponse>
        </soap12:Body>
    </soap12:Envelope>
    

Отправка SMS
------------


Отправка SMS с учетом часового пояса получателя
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для того чтобы сообщение получателю было доставлено в срок, задается отложенная отправка SendMessageByTimeZone. Часовой пояс вычисляется на основе номера получателя и, в зависимости от него, сообщение отправляется через заданный временной интервал, чтобы осуществилась доставка по местному времени получателя. Пример запроса:

.. code-block:: xml

    POST /smsservicev2.asmx HTTP/1.1
    Host: ws.devinotele.com
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageByTimeZone xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <sourceAddress>string</sourceAddress>
                <destinationAddress>string</destinationAddress>
                <data>string</data>
                <sendDate>dateTime</sendDate>
                <validity>int</validity>
            </SendMessageByTimeZone>
        </soap12:Body>
    </soap12:Envelope>
    

Таблица 2 - Описание параметров SendMessageByTimeZone

+------------------+------------+--------------+---------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                  |
+==================+============+==============+===========================================================================+
| Login            |  String    | Да           | Логин, полученный при регистрации                                         |
+------------------+------------+--------------+---------------------------------------------------------------------------+
| Password         |  String    | Да           | Пароль, соответствующий логину                                            |
+------------------+------------+--------------+---------------------------------------------------------------------------+
|DestinationAddress|  String    |  Да          | Номер получателя сообщения в международном формате: код страны +          |
|                  |            |              | код сети + номер телефона.                                                |
|                  |            |              | Пример:                                                                   |
|                  |            |              | 79031234567, +79031234567, 89031234567                                    |
+------------------+------------+--------------+---------------------------------------------------------------------------+
| Data             |  String    | Да           | Текст сообщения, сообщение не должно быть длиннее 2000 символов           |
+------------------+------------+--------------+---------------------------------------------------------------------------+
| SourceAddress    | String     | Да           | Адрес отправителя сообщения. До 11 латинских символов или до 15 цифровых. |
|                  |            |              | Как получить адресотправителя см. в начале документа.                     |
+------------------+------------+--------------+---------------------------------------------------------------------------+
| SendDate         | DateTime   | Да           | Дата и время отправки (пример 2010-0601T19:14:00).                        |
|                  |            |              | Сообщение будет отправлено только при наступлении полученных даты         |
|                  |            |              | и времени с учетомтекущего часового пояса получателя.                     |
+------------------+------------+--------------+---------------------------------------------------------------------------+
| Validity         | Int        | Нет          |  Время жизни сообщения (мин), по умолчанию 2880 мин.                      |
+------------------+------------+--------------+---------------------------------------------------------------------------+

Перед отправкой SMS Сервис проверяет запрос на:

* наличие обязательных параметров;
* валидность Login/Password;
* достаточно ли Баланса Пользователя на отправку SMS (достаточность определяется на основании тарифа Пользователя на отправку SMS для мобильного оператора указанного в запросе номера);
* валидность указанного в запросе номера;
* валидность адреса отправителя;
* длину сообщения.

Если все проверки пройдены успешно, то Сервис отправит сообщение в SMS-центр и вернет идентификатор отправленного сообщения с параметрами как в примере ответа. Размер 1 сообщения составляет: 70 русских символов или 160 символов латиницей. Сервис может возвратить более 1 идентификатора, если текст сообщения выходит за пределы 1 sms. Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageByTimeZoneResponse xmlns="http://ws.devinosms.com">
                <SendMessageByTimeZoneResult>
                    <string>string</string>
                    <string>string</string>
                </SendMessageByTimeZoneResult>
            </SendMessageByTimeZoneResponse>
       </soap12:Body>
    </soap12:Envelope>
    

Отправка SMS адресатам и возвращение системных идентификаторов сообщений
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Данный метод поддерживает массовую отправку сообщений (до 1000 сообщений) в одном запросе.**

Пример запроса:

.. code-block:: xml

    POST /smsservicev2.asmx HTTP/1.1
    Host: ws.devinotele.com
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessage xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <message>
                    <Data>string</Data>
                    <DelayUntilUtc>dateTime</DelayUntilUtc>
                    <DestinationAddresses>
                        <string>string</string>
                        <string>string</string>
                    </DestinationAddresses>
                    <SourceAddress>string</SourceAddress>
                    <ReceiptRequested>boolean</ReceiptRequested>
                    <Validity>int</Validity>
                </message>
            </SendMessage>
         </soap12:Body>
    </soap12:Envelope>
    

Таблица 3 - Описание параметров SendMessage

+------------------+------------+--------------+-------------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                      |
+==================+============+==============+===============================================================================+
| Login            |  String    | Да           | Логин, полученный при регистрации                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Password         |  String    | Да           | Пароль, соответствующий логину                                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Data             |  String    |  Да          | Текст сообщения, сообщение не должно быть длиннее 2000 символов               |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| DelayUntilUtc    |  DateTime  |  Нет         | Время отправки. Если не заполнено, то отправляется немедленно.                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
|DestinationAddress|  String [] | Да           | Номер получателя сообщения в международном формате:                           |
|                  |            |              | код страны + код сети + номер телефона.                                       |  
|                  |            |              | Пример: 79031234567, +79031234567, 89031234567                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| SourceAddress    | String     | Да           | Адрес отправителя сообщения. До 11 латинских имволов или до 15 цифровых.      |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| ReceiptRequested | Boolean    | Нет          | Запрос о доставке                                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Validity         | Int        | Нет          |  Время жизни сообщения (мин), по умолчанию 2880 мин.                          |
+------------------+------------+--------------+-------------------------------------------------------------------------------+

Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageResponse xmlns="http://ws.devinosms.com">
                <SendMessageResult>
                    <string>string</string>
                    <string>string</string>
                </SendMessageResult>
            </SendMessageResponse>
        </soap12:Body>
    </soap12:Envelope>
    
    
Отправка SMS адресатам и возвращение системных идентификаторов сообщений с учетом часового пояса получателя
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для того чтобы сообщение получателю было доставлено в срок, задается отложенная отправка SendMessageByTimeZoneToAddresses. Часовой пояс вычисляется на основе номера получателя и, в зависимости от него, сообщение отправляется через заданный временной интервал, чтобы осуществилась доставка по местному времени получателя. Пример запроса:

Пример запроса:

.. code-block:: xml

    POST / HTTP/1.1
    Host: localhost
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageByTimeZoneToAddresses xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <sourceAddress>string</sourceAddress>
                <DestinationAddresses>
                        <string>string</string>
                        <string>string</string>
                </DestinationAddresses>
                <data>string</data>
                <Validity>int</Validity>
            </SendMessageByTimeZoneToAddresses>
         </soap12:Body>
    </soap12:Envelope>
    

Таблица 4 - Описание параметров SendMessageByTimeZoneToAddresses

+------------------+------------+--------------+-------------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                      |
+==================+============+==============+===============================================================================+
| Login            |  String    | Да           | Логин, полученный при регистрации                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Password         |  String    | Да           | Пароль, соответствующий логину                                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Data             |  String    |  Да          | Текст сообщения, сообщение не должно быть длиннее 2000 символов               |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| SendDate         |  DateTime  |  Да          | Дата и время отправки (пример 2010-0601T19:14:00). Сообщение будет отправлено |
|                  |            |              | только при наступлении полученных даты и времени с учетом текущего часового   |
|                  |            |              | пояса получателя.                                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
|DestinationAddress|  String [] | Да           | Номер получателя сообщения в международном формате:                           |
|                  |            |              | код страны + код сети + номер телефона.                                       |  
|                  |            |              | Пример: 79031234567, +79031234567, 89031234567                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| SourceAddress    | String     | Да           | Адрес отправителя сообщения. До 11 латинских имволов или до 15 цифровых.      |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Validity         | Int        | Нет          |  Время жизни сообщения (мин), по умолчанию 2880 мин.                          |
+------------------+------------+--------------+-------------------------------------------------------------------------------+

Перед отправкой SMS Сервис проверяет запрос на:
        * наличие обязательных параметров;
        * валидность Login/Password;
        * достаточно ли Баланса Пользователя на отправку SMS (достаточность определяется на основании тарифа Пользователя на отправку  SMS для мобильного оператора указанного в запросе номера);
        * валидность указанных в запросе номеров;
        * валидность адреса отправителя;
        * длину сообщения.
          
Если все проверки пройдены успешно, то Сервис отправит сообщение в SMS-центр и вернет идентификаторы отправленных сообщений с параметрами как в примере ответа. Размер 1 сообщения составляет: 70 русских символов или 160 символов латиницей. Сервис может возвратить более 1 идентификатора, если текст сообщения выходит за пределы 1 sms. 
Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageByTimeZoneToAddressesResponse xmlns="http://ws.devinosms.com">
                <SendMessageByTimeZoneToAddressesResult>>
                    <string>string</string>
                    <string>string</string>
                <SendMessageByTimeZoneToAddressesResult>
            </SendMessageByTimeZoneToAddressesResponse>
        </soap12:Body>
    </soap12:Envelope>

Получение статуса отправленного SMS
-----------------------------------

Сервис возвращает статус отправленного sms в соответствии со значениями параметров по Login/Password и messageID. Пример запроса:

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
     <soap12:Body>
         <GetMessageState xmlns="http://ws.devinosms.com">
             <login>string</login>
             <password>string</password>
             <messageID>string</messageID>
         </GetMessageState>
     </soap12:Body>
    </soap12:Envelope>
    
Таблица 5 - Описание параметров GetMessageState

+------------------+------------+--------------+-------------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                      |
+==================+============+==============+===============================================================================+
| Login            |  String    |  Да          | Логин, полученный при регистрации                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Password         |  String    |  Да          | Пароль, соответствующий логину                                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| messageId        |  String    |  Да          | Идентификатор сообщения (сегментасообщения). Для одного запроса будет выполнен|
|                  |            |              | возврат статуса только одного сообщения (сегмента сообщения).                 |
+------------------+------------+--------------+-------------------------------------------------------------------------------+

Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <GetMessageStateResponse xmlns="http://ws.devinosms.com">
                <GetMessageStateResult>
                    <State>int</State>
                    <CreationDateUtc>dateTime</CreationDateUtc>
                    <SubmittedDateUtc>dateTime</SubmittedDateUtc>
                    <ReportedDateUtc>dateTime</ReportedDateUtc>
                    <StateDescription>string</StateDescription>
                    <Price>decimal</Price>
                </GetMessageStateResult>
            </GetMessageStateResponse>
       </soap12:Body>
    </soap12:Envelope>

Таблица 6 - Описание возвращаемых параметров 

+--------------------+------------+---------------------------------------------------------------------------+
|      Название      | Тип        |    Описание                                                               |
+====================+============+===========================================================================+
| State              |  int       |  Статус. Типы статусов сообщений приведены в примечании.                  |
+--------------------+------------+---------------------------------------------------------------------------+
| CreationDateUtc    |  dateTime  |  Дата и время создания (пример 2010-0601T19:14:00) в UTC.                 |
+--------------------+------------+---------------------------------------------------------------------------+
| SubmittedDateUtc   |  dateTime  | Время получения в Devino (в UTC).                                         |
+--------------------+------------+---------------------------------------------------------------------------+
| ReportedDateUtc    |  dateTime  | Время получения отчета (в UTC).                                           |
+--------------------+------------+---------------------------------------------------------------------------+
| StateDescription   |  string    | Описание статуса (напримерDescription("Недопустимый адрес получателя")).  |
+--------------------+------------+---------------------------------------------------------------------------+
| Price              |  decimal   | Цена                                                                      |
+--------------------+------------+---------------------------------------------------------------------------+


Получение статистики по SMS-рассылкам за заданный промежуток времени
--------------------------------------------------------------------

Сервис возвращает статистику по SMS-рассылкам за период, в соответствии со значениями параметров, передаваемых сервису в POST-запросе следующего формата. Пример запроса:

.. code-block:: xml

    POST /smsservicev2.asmx HTTP/1.1
    Host: ws.devinotele.com
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <GetStatistics xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <startDateTime>dateTime</startDateTime>
                <endDateTime>dateTime</endDateTime>
            </GetStatistics>
        </soap12:Body>
    </soap12:Envelope>
    

Таблица 7 - Описание параметров GetStatistics

+------------------+------------+--------------+-------------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                      |
+==================+============+==============+===============================================================================+
| Login            |  String    |  Да          | Логин, полученный при регистрации                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Password         |  String    |  Да          | Пароль, соответствующий логину                                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| startDateTime    |  DateTime  |  Да          | Дата и время начала периода, закоторый необходимо получитьстатистику,         |
|                  |            |              | например 2012-01-18Т00:00:00. Время в UTC.                                    |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| endDateTime      |  DateTime  |  Да          | Дата и время конца периода, закоторый необходимо получить статистику,         |
|                  |            |              | например 2012-01-18Т23:59:00. Время в UTC.                                    |
+------------------+------------+--------------+-------------------------------------------------------------------------------+

После получения запроса сервис проверит валидность присланного по Login/Password и даты начала/окончания формирования статистики (включая ограничение на то, что охватываемый диапазон должен не превышать 3 месяцев). Если все проверки пройдены успешно, то сервис вернет статистику по sms со следующими параметрами:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <GetStatisticsResponse xmlns="http://ws.devinosms.com">
                <GetStatisticsResult>
                    <Sent>int</Sent>
                    <Delivered>int</Delivered>
                    <Errors>int</Errors>
                    <InProcess>int</InProcess>
                    <Expired>int</Expired>
                    <Rejected>int</Rejected>
                </GetStatisticsResult>
            </GetStatisticsResponse>
        </soap12:Body>
    </soap12:Envelope>
    
Таблица 8 - Описание возвращаемых параметров 

+------------+-------+---------------------------------------------+
| Название   | Тип   |    Описание                                 |
+============+=======+=============================================+
| Sent       |  int  |  Количество отправленных сообщений          |
+------------+-------+---------------------------------------------+
| Delivered  |  int  | Количество доставленных сообщений.          |
+------------+-------+---------------------------------------------+
| Errors     |  int  | Количество ошибок                           |
+------------+-------+---------------------------------------------+
| InProcess  |  int  | Количество сообщений «в процессе отправки»  |
+------------+-------+---------------------------------------------+
| Expired    |  int  | Количество просроченных сообщений.          |
+------------+-------+---------------------------------------------+
| Rejected   |  int  | Количество отклоненных сообщений            |
+------------+-------+---------------------------------------------+


Получение входящих сообщений
----------------------------

Система позволяет заводить входящие номера и на них получать sms. Входящий номер заводится через личный кабинет. Сервис возвращает входящие сообщения пользователя в интервале maxDate minDate(который передан в этом запросе). Пример запроса:

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <GetIncomingMessages xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <maxDateUTC>dateTime</maxDateUTC>
                <minDateUTC>dateTime</minDateUTC>
            </GetIncomingMessages>
        </soap12:Body>
    </soap12:Envelope>
    

Таблица 9 - Описание параметров GetIncomingMessages

+------------------+------------+--------------+-------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                              |
+==================+============+==============+=======================================================+
| Login            |  String    |  Да          | Логин, полученный при регистрации                     |
+------------------+------------+--------------+-------------------------------------------------------+
| Password         |  String    |  Да          | Пароль, соответствующий логину                        |
+------------------+------------+--------------+-------------------------------------------------------+
| maxDateUTC       |  DateTime  |  Да          | Значение интервала _по. Пример: 2014-11-01T11:30      |
+------------------+------------+--------------+-------------------------------------------------------+
| minDateUTC       |  DateTime  |  Да          | Значение интервала с_. Пример: 2014-11-01T11:30       |
|                  |            |              | например 2012-01-18Т23:59:00. Время в UTC.            |
+------------------+------------+--------------+-------------------------------------------------------+

Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
     <soap12:Body>
        <GetIncomingMessagesResponse xmlns="http://ws.devinosms.com">
            <GetIncomingMessagesResult>
                <IncomingMessage>
                     <Data>string</Data>
                     <SourceAddress>string</SourceAddress>
                     <DestinationAddress>string</DestinationAddress>
                     <CreatedDateUtc>dateTime</CreatedDateUtc>
                </IncomingMessage>
                <IncomingMessage>
                     <Data>string</Data>
                     <SourceAddress>string</SourceAddress>
                     <DestinationAddress>string</DestinationAddress>
                     <CreatedDateUtc>dateTime</CreatedDateUtc>
                </IncomingMessage>
            </GetIncomingMessagesResult>
         </GetIncomingMessagesResponse>
     </soap12:Body>
    </soap12:Envelope>
    

Таблица 10 - Описание параметров GetIncomingMessages

+-------------------+---------+-----------------------------------+
| Название          | Тип     |  Описание                         |
+===================+=========+===================================+
| Data              | String  |  Текст сообщения                  |
+-------------------+---------+-----------------------------------+
|SourceAddress      | String  | Адрес отправителя                 |
+-------------------+---------+-----------------------------------+
| DestinationAddress| String  | Адрес получателя                  |
+-------------------+---------+-----------------------------------+
| CreatedDateUtc    | DateTime| Дата создания                     |
+-------------------+---------+-----------------------------------+


Отправка Viber-сообщений
------------------------


Отправка Viber адресатам и возвращение системных идентификаторов сообщений
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Данный метод поддерживает массовую отправку сообщений (до 1000 сообщений) в одном запросе.

Пример запроса:

.. code-block:: xml

    POST /ViberServiceV2.asmx HTTP/1.1
    Host: ws.devinotele.com
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessage xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <message>
                    <Data>string</Data>
                    <DestinationAddresses>
                        <string>string</string>
                        <string>string</string>
                    </DestinationAddresses>
                    <SourceAddress>string</SourceAddress>
                    <Validity>int</Validity>
                    <Optional>string</Optional>
                </message>
            </SendMessage>
         </soap12:Body>
    </soap12:Envelope>
    

Таблица 11 - Описание параметров SendMessage

+------------------+------------+--------------+-------------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                      |
+==================+============+==============+===============================================================================+
| Login            |  String    | Да           | Логин, полученный при регистрации                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Password         |  String    | Да           | Пароль, соответствующий логину                                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Data             |  String    |  Да          | Текст сообщения, сообщение не должно быть длиннее 1000 символов               |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
|DestinationAddress|  String [] | Да           | Номер получателя сообщения в международном формате:                           |
|                  |            |              | код страны + код сети + номер телефона.                                       |  
|                  |            |              | Пример: 79031234567, +79031234567, 89031234567                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| SourceAddress    | String     | Да           | Адрес отправителя сообщения. До 11 латинских имволов или до 15 цифровых.      |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Validity         | Int        | Да           | Время жизни сообщения (мин, от 1 до 1440)                                     |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Optional         | String     | Нет          |  Дополнительный параметр                                                      |
+------------------+------------+--------------+-------------------------------------------------------------------------------+

Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageResponse xmlns="http://ws.devinosms.com">
                <SendMessageResult>
                    <string>string</string>
                    <string>string</string>
                </SendMessageResult>
            </SendMessageResponse>
        </soap12:Body>
    </soap12:Envelope>
    
    
Отправка Viber адресатам и возвращение системных идентификаторов сообщений с переотправкой по sms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Данный метод поддерживает массовую отправку сообщений (до 1000 сообщений) в одном запросе.

Пример запроса:

.. code-block:: xml

    POST /ViberServiceV2.asmx HTTP/1.1
    Host: ws.devinotele.com
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageWithResend xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <message>
                    <Data>string</Data>
                    <DestinationAddresses>
                        <string>string</string>
                        <string>string</string>
                    </DestinationAddresses>
                    <SourceAddress>string</SourceAddress>
                    <Validity>int</Validity>
                    <Optional>string</Optional>
                </message>
            </SendMessageWithResend>
         </soap12:Body>
    </soap12:Envelope>
    

Таблица 12 - Описание параметров SendMessageWithResend

+------------------+------------+--------------+-------------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                      |
+==================+============+==============+===============================================================================+
| Login            |  String    | Да           | Логин, полученный при регистрации                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Password         |  String    | Да           | Пароль, соответствующий логину                                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Data             |  String    |  Да          | Текст сообщения, сообщение не должно быть длиннее 1000 символов               |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
|DestinationAddress|  String [] | Да           | Номер получателя сообщения в международном формате:                           |
|                  |            |              | код страны + код сети + номер телефона.                                       |  
|                  |            |              | Пример: 79031234567, +79031234567, 89031234567                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| SourceAddress    | String     | Да           | Адрес отправителя сообщения. До 11 латинских имволов или до 15 цифровых.      |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Validity         | Int        | Да           | Время жизни сообщения (мин, от 1 до 1440)                                     |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Optional         | String     | Нет          |  Дополнительный параметр                                                      |
+------------------+------------+--------------+-------------------------------------------------------------------------------+

Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <SendMessageWithResendResponse xmlns="http://ws.devinosms.com">
                <SendMessageResult>
                    <string>string</string>
                    <string>string</string>
                </SendMessageResult>
            </SendMessageWithResendResponse>
        </soap12:Body>
    </soap12:Envelope>
    
Получение статуса отправленного Viber-сообщения
-----------------------------------------------
Сервис возвращает статус отправленного viber-сообщения в соответствии со значениями параметров по Login/Password и messageID.

Пример запроса:

.. code-block:: xml

    POST /ViberServiceV2.asmx HTTP/1.1
    Host: 127.0.0.1
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <GetMessageState xmlns="http://ws.devinosms.com">
                <login>string</login>
                <password>string</password>
                <messageID>long</messageID>
            </GetMessageState>
         </soap12:Body>
    </soap12:Envelope>
    

Таблица 13 - Описание параметров GetMessageState

+------------------+------------+--------------+-------------------------------------------------------------------------------+
|     Параметр     | Тип данных |Обязательность| Описание                                                                      |
+==================+============+==============+===============================================================================+
| Login            |  String    | Да           | Логин, полученный при регистрации                                             |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| Password         |  String    | Да           | Пароль, соответствующий логину                                                |
+------------------+------------+--------------+-------------------------------------------------------------------------------+
| messageId        |  String    |  Да          | Идентификатор сообщения. Для одного запроса будет выполнен возврат статуса    |
|                  |            |              |  только одного сообщения.                                                     |
+------------------+------------+--------------+-------------------------------------------------------------------------------+


Пример ответа:

.. code-block:: xml

    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: length
    <?xml version="1.0" encoding="utf-8"?>
    <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
        <soap12:Body>
            <GetMessageStateResponse xmlns="http://ws.devinosms.com">
                <GetMessageStateResult>
                    <State>Enqueued or Sent or Delivered or Read or Undelivered or Failed or Unknown or Expired</State>
                    <ResentSms>
                      <ViberSmsMessageStateInfo>
                        <Id>long</Id>
                      </ViberSmsMessageStateInfo>
                      <ViberSmsMessageStateInfo>
                        <Id>long</Id>
                      </ViberSmsMessageStateInfo>  
                    </ResentSms>  
                </GetMessageStateResult>
            </GetMessageStateResponse>
        </soap12:Body>
    </soap12:Envelope>

Таблица 14 - Описание возвращаемых параметров 

+------------------+---------------------------+---------------------------------------------------------+
| Название         | Тип                       |    Описание                                             |
+==================+===========================+=========================================================+
| State            |  int                      |  Статус. Типы статусов viber-сообщений приведены в      |
|                  |                           |  примечании.                                            |
+------------------+---------------------------+---------------------------------------------------------+
| CreationDateUtc  |  dateTime                 | Дата и время создания (пример 2010-0601T19:14:00) в UTC.|
+------------------+---------------------------+---------------------------------------------------------+
| SubmittedDateUtc |  dateTime                 | Время получения в Devino (в UTC).                       |
+------------------+---------------------------+---------------------------------------------------------+
| ReportedDateUtc  |  dateTime                 | Время получения отчета (в UTC).                         |
+------------------+---------------------------+---------------------------------------------------------+
| StateDescription |  string                   | Описание статуса (например Description                  |               
|                  |                           | (“Недопустимый адрес получателя”)).                     |
+------------------+---------------------------+---------------------------------------------------------+
| Price            |  decimal                  | Цена                                                    |
+------------------+---------------------------+---------------------------------------------------------+
| ResentSms        | ViberSmsMessageStateInfo[]| Коллекция статусов sms-сообщений, которые были          |         
|                  |                           | отправлены в результате отправки текущего               |
|                  |                           | viber-сообщения.                                        |
+------------------+---------------------------+---------------------------------------------------------+

Приложение. Коды ошибок и статусы сообщений
-------------------------------------------

+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
|   БД Devino | Наименование      |Описание                                       | Подробное описание                           |  
+=============+===================+===============================================+==============================================+
| -200        | Ошибка            | Errors=-200                                   | Статус для фильтра "Ошибка" вдетализации     |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| -100        | Протарифицировано | Tarificated = -100                            | Статус для фильтра "Протирифицировано" в     |
|             |                   |                                               | детализации                                  |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| -3          | Ошибка            | ErrorSendingDateTimeInterpretation= -3        | Ошибка интерпретации даты и времени отправки |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| -1          | Отправлено        | Sent = -1                                     | Сообщение отправлено                         |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| -2          | Отправляется      | LocalQueued = -2                              | Сообщение отправляется                       |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| -40         | Ожидание          | Queued = -40                                  | Сообщение в статусе «ожидание»               |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| -30         | Остановлено       | Sending_To_Gateway = -30                      | Отправлено в шлюз                            |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| -20         | Отправлено/       |                                               |                                              |
|             | получателю        | Sending_To_Recipient = -20                    | Сообщение отправлено получателю              |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 0           | Доставлено        | Delivered_To_Recipient = 0                    | Сообщение доставлено                         |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 0x0000000B  | Ошибка            | Error_Invalid_Destination_Address =0x0000000B | Неверно введён адрес получателя              |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 0x0000000A  | Ошибка            | Error_Invalid_Source_Address =0x0000000A      | Неверно введён адрес отправителя             |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 41          | Ошибка            | Error_Incompatible_Destination = 41           | Недопустимый адрес получателя                |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 42          | Ошибка            | Error_Rejected = 42                           | Отклонено                                    |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 46          | Ошибка            | Error_Expired = 46                            | Просрочен                                    |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 47          | Ошибка            | Deleted = 47                                  | Просрочено                                   |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 48          | Ошибка            | Devino_Rejected = 48                          | Ошибка                                       |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 0x000000FF  | Неизвестный       | Unknown = 0x000000FF                          | Внутренняя ошибка                            |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
| 0x00000008  | Ошибка            | System_Error = 0x00000008                     | Внутренняя ошибка                            |
+-------------+-------------------+-----------------------------------------------+----------------------------------------------+
