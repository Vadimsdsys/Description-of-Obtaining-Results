# Методы получения результатов по лабораториям

## 1. Лаборатории с результатами через FTP

Для повторного получения файла результатов достаточно попросить лабораторию о перевыгрузке. На стороне ЛП дополнительные манипуляции не нужны.

**Список лабораторий:**
- Хеликс
- Инвитро
- ЦМД
- КДЛ
- НЦКМД
- СЗЦДМ
- ЦЛТ АБВ

---

## 2. Лаборатории через веб-сервис

### 2.1. Альфалаб (AlfaLab.dll)

**Лаборатории:**
- Архимед (ЛП)
- ДНКОМ (ЛП)
- ДНКОМ Covid (ЛП)
- Диалаб (ЛП)
- Диалаб Covid (ЛП)
- Диалаб ПМО (ЛП)
- ЕМЛ (ЛП)
- Лабквест (ЛП)
- Лаборатория.ру (ЛП)
- Литех ЛИС (ЛП)
- МобилМед (ЛП)
- Санатест (ЛП)
- Скайлаб (ЛП)
- Хромолаб (ЛП)
- ЭндоМедЛаб (ЛП)
- Эфис (ЛП)

**Процесс:**
Отправляется запрос к очереди на выгрузку результатов с временной меткой, например `Date="13.08.2025 00:10:02"`. Если на указанную дату есть данные, система возвращает результат. Временная метка автоматически обновляется до текущего времени при запросе. Результаты сохраняем только если хотя бы у одной пробы STATE<>1  
"Не найдено ни одной пробы со STATE <> 1, результат не будет загружен"   
```
<?xml version="1.0" encoding="Windows-1251"?>
<Message 
  MessageType="query-next-referral-results" 
  Date="13.08.2025 00:10:02" 
  Sender="" 
  Receiver="LabQuest" 
  Password=""
/>
``` 
После получения результата мы отправляем подтверждение, чтобы удалить его из очереди. Для повторного получения лаборатория должна повторно добавить результат в очередь.

---

### 2.2. InterSystemsLab.dll
**Лаборатории:**
- РЖД КРД (ЛП)
- РЖД МСК (ЛП)
- РЖД НН (ЛП)
- РЖД НН ТЕСТ!
- РЖД СТЕР (ЛП)
- РЖД Уфа (ЛП)
- РЖД Чита (ЛП)
- РЖД Чита ТЕСТ!
- Смартлаб (ЛП)

**Процесс:**
Пример запроса :
```
Request: { "resourceType": "Parameters", "parameter": [   {    "name": "start",    "valueDateTime": "2025-08-12T20:59:42Z"   },  
 {    "name": "end",    "valueDateTime": "2025-08-12T21:00:09Z"   },   {    "name": "requesterOrganization",  
  "valueIdentifier": {      "system": "urn:Organization",      "value": ""     }   } ] }
``` 

  Где `{    "name": "start",    "valueDateTime": "2025-08-12T20:59:42Z"   }` дата берется из `filial_contr_links.LSSTARTDATE`, а
  `{    "name": "end",    "valueDateTime": "2025-08-12T21:00:09Z"   }` является текущей датой на момент запроса.
  После запроса дату обновляем на текущую в `LSSTARTDATE`

  При успешном получении результата отправляем подтверждение о получении.

  Обычно со своей стороны манипуляции не нужны, и достаточно попросить перевыгрузить у лаборатории. Но на крайний случай можно изменить дату в `LSSTARTDATE`, тогда повторно получим все результаты, которые были в это время. У них это не совсем очередь, и результаты оттуда не пропадают после получения.     
При получении файла анализируется ресурс `OBSERVATION`; если его нет, то файл не сохраняем. 
"Сервер не вернул результат (отсутствует тег `OBSERVATION`)"  


---

### 2.3. CityLab_msk.dll
**Лаборатории:**
- Ситилаб ЕКБ (ЛП)
- Ситилаб КЗН (ЛП)
- Ситилаб МСК (ЛП)
- Ситилаб НСБ (ЛП)
- Ситилаб РнД (ЛП)
- Ситилаб СМР (ЛП)
- Ситилаб СПБ (ЛП)
- Ситилаб ТМН (ЛП)

**Процесс:**
По Ситилабу используется несколько методов для получения результатов. `RC_GetInquiriesJournal` - это запрос на получение списка заказов, по которым были изменения на указанную дату в запросе.
Пример:
```
<?xml version="1.0" encoding="Windows-1251"?> 
<content> <e n="login" v="" t="string" /> <e n="password" v="" t="string" /> <e n="Timestamp" v="1755025802" t="datetime" /> </content>
```

Где `n="Timestamp" v="1755025802"` - это дата из `filial_contr_links.LSSTARTDATE`, она обновляется раз в час.

Потом второй метод `RC_GetInquiryAuth`: здесь уже идет сравнение `LastWorkResultDate` из XML с `TREAT.LISTIMESTAMP`. И только если `LastWorkResultDate` > даты в `treat`, то мы его загрузим.

`InternalNum "132278390" LastWorkResultDate 13.08.2025 10:36:32 TREAT.LISTIMESTAMP 13.08.2025 10:28:15 Наряд не закрыт 1`
```
13.08.2025 10:41:14 Запрашиваем детальную информацию RC_GetInquiryAuth :
 <?xml version="1.0" encoding="Windows-1251"?> 
<content> <e n="login" v="" t="string" /> <e n="password" v="" t="string" /> <e n="requestId" v="" t="string" /> </content>
```
Файлы сохраняем только если `sample.state=2` или будет сообщение в логе.   
"Результат не будет сохранен, т.к. sample.state не равен 2"  

Поэтому для повторного получения результатов нужно:  
1. Попросить в лаборатории обновить штамп (чтобы заказ попал в метод `RC_GetInquiriesJournal`).  
2. В таблице `TREAT` удалить дату в `LISTIMESTAMP`, чтобы мы смогли его скачать.  


### LisInfoclinica.dll:
- ДаВинчиЛаб (ЛП)
- РДЛ (ЛП)

Данные лаборатории работают на базе МИС Инфоклиника, и, соответственно, парсинг и получение у них реализованы так же, как в клинике.  
 

### Morfolab.dll:
- Морфолаб (ЛП)

Лаборатория не работает. 

### Litex.dll:
- Литех (ЛП)

В процессе перехода на альфалаб. 

### bion.dll:
- БИОН (ЛП)

**Процесс:**  
Логика почти такая же как у Ситилаба   
Отправляется общий запрос на список заказов   
```https://gost-63087.infoclinica.lan/int5/api/users/events/between/?from_ts=2025-09-03%2015:10:01&until_ts=2025-09-03%2016:16:04&result=updates```  
где ``from_ts=2025-09-03%2015:10:01` — дата из `filial_contr_links.LSSTARTDATE` (обновляется раз в час), а `until_ts=2025-09-03%2016:16:04` — текущее время.  

Если на запрос есть какие то заказы, то уже отправляем индивидуальные для получения самих результатов по заказам.   
```https://gost-63087.infoclinica.lan/int5/api/orders/result/8748000308/```  
Тут идет сравнение даты из xml `"last_change":` с датой в `TREAT.LISTIMESTAMP` и если она больше то сохраняем результат и обновляем  `LISTIMESTAMP`  
Для повторного получения результатов нужно:  
1. Изменить дату в `filial_contr_links.LSSTARTDATE` на нужную  
2. В таблице `TREAT` удалить дату в `LISTIMESTAMP`, чтобы мы смогли его скачать.  


### Gemotest_soap.dll:
- Гемотест SOAP (ЛП)  

**Процесс:**    
Проверяется каждый заказ по которому не закрыт наряд, и отправляется запрос в лабораторию 
```
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:OdoctorControllerwsdl" xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/">
  <soapenv:Header/>
  <soapenv:Body>
    <urn:get_analysis_result soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
      <params xsi:type="urn:request_get_analysis_result">
        <contractor xsi:type="xsd:string">10076</contractor>
        <hash xsi:type="xsd:string">ec4b3d9b079a44d675b9f235991bb016538d0dda</hash>
        <order_num xsi:type="xsd:string"/>
        <ext_num xsi:type="xsd:string">1089049400</ext_num>
      </params>
    </urn:get_analysis_result>
  </soapenv:Body>
</soapenv:Envelope>
``` 

Идет проверка статуса результата; если статус 0, то результат не забираем, иначе мы его сохраняем.  
Потом в таблице `labexchange` есть триггер `GEMOTESTSOAP_NARADCLOSE_BIU`, который закрывает наряды при получении файла результатов и его успешной обработке.  
После этого по заказу больше не будем отправлять запросы на результаты.  

Для повторного получение результата нужно просто открыть наряд в `treat`.  


### GemoHelp.dll:
- Гемохелп (ЛП)

**Процесс:**     
```
https://gost-64105.infoclinica.lan/v1/last-modified-orders?access-token=тут указывается `filial_contr_links.extlabcode`  

В самом теле запроса отправляем дату `{"date": "2025-09-08T18:14:10.059Z"}` из `filial_contr_links.LSSTARTDATE`  
```
Дата обновляется после получения результата.  


В ответе придут внутренние идентификаторы лабы uid, потом по ним идет детальный запрос:  
```
https://gost-64105.infoclinica.lan/v1/order-detail?access-token=PDCLPeYTI0M$AaBix5DJMvAg
```
В теле запроса :  
`{"uuid": "01992ce4-30a9-7287-9e21-8f8181e827e4", "onlyPDFForm": 0}`  
Также результаты сохраняем только со статусом 5 или 6.  

Для повторного получения результатов можно изменить дату в `filial_contr_links.LSSTARTDATE`.  

### innovasystems.dll:
- ДНК-Диагностика (ЛП)

Это некий аналог Альфалаба, у них так же очередь с результатами.   
Изначально отправляем запрос:    
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/"><soapenv:Header/><soapenv:Body><tem:GetFilteredNextResult><tem:filter><HospitalCode>117</HospitalCode></tem:filter></tem:GetFilteredNextResult></soapenv:Body></soapenv:Envelope>
```
`HospitalCode`= `filial_contr_links.extlabcode` 
После отправляем подтверждение 
```
Отправляем подтверждение о получении результата =  #9128802660
=====Перед подтверждением====== > <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/"><soapenv:Header/><soapenv:Body><tem:ConfirmResultObtainment><tem:requestCode>9128802660</tem:requestCode></tem:ConfirmResultObtainment></soapenv:Body></soapenv:Envelope>
=====Подтвердили получение====== > <s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/"><s:Body><ConfirmResultObtainmentResponse xmlns="http://tempuri.org/"/></s:Body></s:Envelope>
```
Для получение результатоа повторно нужно просить лабораторию, что бы переотправили.     

### dcli.dll:
- ДЦЛИ (ЛП)
Тут результаты получаются через репликатор(берем все, что отдают), потом настроено перекидывание файла к ним в папку.  
И уже идет стандартное подключение как на фтп к папке, и получение результата из нее.  


### LisNacph.dll:
- НАКФФ (ЛП)
По НАКФФ в логах к сожалению не пишутся точные запросы, глобальная логика как и в Альфалабе, забираем результаты которые имеются в очереди.   

