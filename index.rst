Документация по API
===================

В данном разделе представлена документация по различными протоколам для взаимодействия с платформой Devino Telecom.
Документация содержит описание API cервисов для отправки и приема SMS-сообщений, отправки Email и выполнения запросов IMSI
Каждое руководство включает в себя описание схемы работы с API, краткий обзор всех параметров, а также подробные описания по каждому из параметров, снабженные реальными примерами. 

Документация предназначена для разработчиков, которые хотят добавить возможность взаимодействия с сервисом отправки SMS и Email сообщений на страницы своих сайтов или в свои приложения.

Для начала использования API: 

1. Зарегистрируйтесь в Личном кабинете; 
2. Заключите договор; 
3. Ознакомьтесь с интересующей документацией. 


.. _sms-docs:

.. toctree::
   :maxdepth: 2
   :caption: SMS Рассылки

   httpapi
   httpapiv2
   smpp
   smtp
   modyl1c
   xml
   soap
   soapv2
   ftps
   1cbitrix
   
.. _incoming-docs:

.. toctree::
   :maxdepth: 2
   :caption: Входящие сообщения
   
   httppriem

.. _imsi-docs:

.. toctree::
   :maxdepth: 2
   :caption: IMSI
   
   imsi


.. _phoneverification-docs:
.. toctree::
   :maxdepth: 2
   :caption: Сервис двухфакторной аутентификации
   
   phoneverification
   
   
.. _viber-docs:
.. toctree::
   :maxdepth: 2
   :caption: Viber
   
   viber-resender

.. _addressbookapi-docs:
.. toctree::
   :maxdepth: 2
   :caption: Адресная книга
   
   addressbookapi

.. _email-docs:   
.. toctree::
   :maxdepth: 2
   :caption: E-Mail Рассылки
   
   emailsmtp
   emailhttp
