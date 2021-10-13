# API
Опис того, як інтегрувати власну програму або послугу з системою BitFaktura.com.ua

Завдяки API ви можете виставляти рахунки-фактури / рахунки / квитанції з інших систем та керувати цими документами, а також клієнтами та продуктами.

Робочі приклади виклику API BitFaktura можна також знайти у системі BitFaktura (після входу в систему) у меню **Налаштування> API** та на веб -сайті: https://app.bitfaktura.com.ua/api
# Додаткові параметри доступні під час завантаженя списку записів
До викликів можна передавати додаткові параметри - ті самі параметри, які використовуються у програмі, наприклад ` page=` , ` period=`  тощо.

Параметр `page=`  дозволяє переходити по погінованих записах. За замовчуванням він приймає значення 1 і відображає перші N записів, де N - межа кількості повернутих записів. Щоб отримати наступні N записів, слід викликати дію з параметром ` page= 2`  тощо.
Параметр `period=` дозволяє вибрати записи за певний період. Він може приймати такі значення:

last_12_months
this_month
last_30_days
last_month
this_year
last_year
all
more (тут потрібно надати додаткові параметри date_from (наприклад, "2018-12-16") та date_to (наприклад, "2018-12-21")
Параметр `include_positions=` зі значенням `true` дає змогу отримати список записів із їх позиціями

Параметр `income=` зі значенням `no` дозволяє завантажувати рахунки-фактури витрат (витрат)

# Приклади викликів
Завантаження списку рахунків-фактур за поточний місяць
`curl https://twijdomen.bitfaktura.com.ua/invoices.json?period=this_month&api_token=API_TOKEN&page=1`

Завантаження списку рахунків-фактур з їх позиціями
`curl https://twijdomen.bitfaktura.com.ua/invoices.json?include_positions=true&api_token=API_TOKEN&page=1`

Рахунки-фактури даного клієнта
`curl https://twijdomen.bitfaktura.com.ua/invoices.json?client_id=ID_KLIENTA&api_token=API_TOKEN`

Завантаження рахунка-фактури за ідентифікатором
`curl https://twijdomen.bitfaktura.com.ua/invoices/100.json?api_token=API_TOKEN`

Завантаження PDF
`curl https://twijdomen.bitfaktura.com.ua/invoices/100.pdf?api_token=API_TOKEN`

Надсилання рахунку-фактури електронною поштою клієнту (на електронну адресу замовника, надану при створенні рахунку-фактури, поле "buyer_email")
`curl -X POST https://twijdomen.bitfaktura.com.ua/invoices/100/send_by_email.json?api_token=API_TOKEN`

інші параметри PDF:
print_option=original - Оригінил
print_option=copy - Копія
print_option=original_and_copy - Оригінал і копія
print_option=duplicate Дуплікат

Додавання нового рахунка-фактури

``` 
curl https://TWIJ_DOMEN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "kind":"vat",
            "number": null,
            "sell_date": "2013-01-16",
            "issue_date": "2013-01-16",
            "payment_to": "2013-01-23",
            "seller_name": "ФОП Постачальник",
            "seller_tax_no": "5252445767",
            "buyer_name": "ФОП Клієкт",
            "buyer_email": "buyer@testemail.ua",
            "buyer_tax_no": "5252445767",
            "positions":[
                {"name":"Produkt A1", "tax":23, "total_price_gross":10.23, "quantity":1},
                {"name":"Produkt A2", "tax":0, "total_price_gross":50, "quantity":3}
            ]
        }
    }' 
    ```
Додавання нового рахунка-фактури - версія мінімум (лише обов’язкові поля), коли у нас є ідентифікатор продукту (product_id), покупець (client_id) та постачальник (department_id), тоді нам не потрібно надавати повні дані. За бажанням ви також можете вказати ідентифікатор одержувача (recipient_id). Рахунок-фактура з ПДВ буде виставлений з поточною датою та терміном оплати 5 днів.
    ```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "invoice": {
            "payment_to_kind": 5,
            "client_id": 1,
            "positions":[
                {"product_id": 1, "quantity":2}
            ]
        }}'
    ```
    
Додавання нового рахунка-фактури - документа, подібного до рахунка-фактури зі вказаним ідентифікатором (copy_invoice_from).
Додавання ідентичного рахунку-фактури зазначеного типу.
   ```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "copy_invoice_from": ID_DOKUMENTU_ZRODLOWEGO,
            "kind": "RODZAJ_FAKTURY"
        }
    }'
   ```
Додавання авансового рахунку-фактури на основі замовлення -% від повної суми
   ```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "copy_invoice_from": ID_ZAMOWIENIA,
            "kind": "advance",
            "advance_creation_mode": "percent",
            "advance_value": "10",
            "position_name": "Zaliczka na wykonanie zamówienia ZAM-NR"
        }
    }'

   ```
   
   Додавання авансового рахунку-фактури на основі замовлення - зазначеної суми з ПДВ
   
      ```
      curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "copy_invoice_from": ID_ZAMOWIENIA,
            "kind": "advance",
            "advance_creation_mode": "amount",
            "advance_value": "150",
            "position_name": "Zaliczka na wykonanie zamówienia ZAM-NR"
        }
    }'
   ```
   
   Додавання остаточного рахунку-фактури на основі замовлення та авансових рахунків-фактур
  ```
  curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "copy_invoice_from": ID_ZAMOWIENIA,
            "kind": "final",
            "invoice_ids": [ID_ZALICZKI_1, ID_ZALICZKI_2, ...]
        }
    }'
  ```
  Додавання накладної з ПДВ на основі рахунку проформи
  ```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "copy_invoice_from": ID_PROFORMY,
            "kind": "vat"
        }
    }'
  ```
  Додавання нового виправленого рахунка-фактури
   ```
  curl http://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "invoice": {
            "kind": "correction",
            "correction_reason": "Zła ilość",
            "invoice_id": "2432393",
            "from_invoice_id": "2432393",
            "client_id": 1,
            "positions":[
                {"name": "Product A1",
                "quantity":-1,
                "total_price_gross":"-10",
                "tax":"23",
                "kind":"correction",
                "correction_before_attributes": {
                    "name":"Product A1",
                    "quantity":"2",
                    "total_price_gross":"20",
                    "tax":"23",
                    "kind":"correction_before"
                },
                "correction_after_attributes": {
                    "name":"Product A1",
                    "quantity":"1",
                    "total_price_gross":"10",
                    "tax":"23",
                    "kind":"correction_after"
                }
            }]
        }}'
 ```
Оновлення рахунка-фактури
 ```
 curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/111.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "buyer_name": "Nowa nazwa klienta Sp. z o.o."
        }
    }'
 ```
 
 Оновлення позиції рахунка-фактури - щоб відредагувати позицію рахунка-фактури, введіть ідентифікатор позиції.
 ```
 curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/111.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "positions": [{"id":32649087, "name":"test"}]
        }
    }'
 ```
 
 Видалення елемента в рахунку-фактурі - щоб видалити елемент у рахунку-фактурі, введіть ідентифікатор товару з параметром _destroy = 1.
```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/111.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "positions": [{"id":32649087, "_destroy": 1}]
        }
    }'
```
Додавання позиції на рахунку-фактурі. Елемент буде додано останнім.
```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/111.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "positions": [{"name":"Produkt A1", "tax":23, "total_price_gross":10.23, "quantity":1}]
        }
    }'
```

Зміна статусу рахунка-фактури
```
curl "https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/111/change_status.json?api_token=API_TOKEN&status=STATUS" -X POST
```

Завантаження списку визначень періодичних рахунків-фактур
```
curl https://YOUR_DOMAIN.fakturownia.pl/recurrings.json?api_token=API_TOKEN
```
Додавання визначення періодичного рахунку-фактури
```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/recurrings.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "recurring": {
            "name": "Nazwa cyklicznosci",
            "invoice_id": 1,
            "start_date": "2016-01-01",
            "every": "1m",
            "issue_working_day_only": false,
            "send_email": true,
            "buyer_email": "mail1@mail.pl, mail2@mail.pl",
            "end_date": "null"
        }}'
```

Оновлення визначення періодичного рахунку-фактури (зміна дати видачі наступного рахунка-фактури)
```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/recurrings/111.json \
    -X PUT \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "recurring": {
            "next_invoice_date": "2016-02-01"
        }
    }'
```
Видалення рахунку-фактури
```
curl -X DELETE "https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/INVOICE_ID.json?api_token=API_TOKEN"
```

З'єднання наявного рахунку-фактури та квитанції
```
curl https://YOUR_DOMAIN.bitfaktura.test/invoices/ID_FAKTURY.json \
    -X PUT \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "from_invoice_id": ID_PARAGONU,
            "invoice_id": ID_PARAGONU,
            "exclude_from_stock_level": true
        }
    }'
```
Завантаження всіх додатків до рахунків-фактур у ZIP-архіві
```
curl -o attachments.zip https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/INVOICE_ID/attachments_zip.json?api_token=API_TOKEN
```

Додавання нового додатку до рахунку-фактури
1. Завантаження даних, необхідних для надсилання файлу:
```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/INVOICE_ID/get_new_attachment_credentials.json?api_token=API_TOKEN
```

2. Завантаження файлу:
```
curl -F 'AWSAccessKeyId=received_AWSAccessKeyId' \
     -F 'key=received_key' \
     -F 'policy=received_policy' \
     -F 'signature=received_signature' \
     -F 'acl=received_acl' \
     -F 'success_action_status=received_success_action_status' \
     -F 'file=@/file_path/name.ext' \
     received_url
```
Додавання вкладення (надісланого файлу) до рахунку-фактури:
```
curl -X POST https://YOUR_DOMAIN.bitfaktura.com.ua/invoices/INVOICE_ID/add_attachment.json?api_token=API_TOKEN&file_name=name.ext
```

# Посилання для попереднього перегляду рахунку-фактури та завантаження у PDF

Після завантаження даних рахунків-фактур, наприклад:
```
curl https://twojaDomena.bitfaktura.com.ua/invoices/100.json?api_token=API_TOKEN
```

API повертає, серед іншого поле маркера, на основі якого можна отримати посилання для перегляду рахунку-фактури та завантаження PDF-файлу зі згенерованим рахунком -фактурою. Такі посилання дають змогу посилатися на вибраний рахунок-фактуру без необхідності входити в систему - тобто ми можемо, наприклад, надіслати ці посилання клієнту, який отримає доступ до рахунка-фактури та PDF.

Ці рядки мають вигляд:
попередній перегляд: `https://yourdomain.bitfaktura.com.ua/invoice/{{token}}`
PDF: `https://YourDomain.bitfaktura.com.ua/invoice/{{token}}.pdf`

Для лексеми: `HBO3Npx2OzSW79RQL7XV2`
загальнодоступний PDF буде на
`https://YourDomain.bitfaktura.com.ua/invoice/HBO3Npx2OzSW79RQL7XV2.pdf`

# Приклади використання в PHP - придбання курсу
Приклад порталу, який формує рахунок-фактуру для клієнта, надсилає його клієнту і після оплати за нього надсилає квиток клієнту на курс

* Клієнт заповнює дані на Порталі
* Портал викликає API з BitFaktura.com.ua і створює рахунок-фактуру
* Портал надсилає Замовнику рахунок-фактуру у форматі PDF з посиланням на платіж
* Клієнт оплачує фактуру проформа 
* BitFaktura.com.ua отримує інформацію про те, що платіж здійснено, створює рахунок-фактуру з ПДВ і надсилає його Замовнику та викликає API порталу
* Після отримання платіжної інформації (через API) Портал надсилає Клієнту квиток на навчання

# Рахунки-фактури
`GET /invoices/1.json`  завантаження рахунку -фактури
`POST /invoices.json` додавання нового рахунка -фактури
`PUT /invoices/1.json` оновлення рахунків-фактур
`DELETE /invoices/1.json` анулювання рахунку -фактури

Приклад - додавання нового рахунка-фактури (версія мінімум, якщо у є ідентифікатор продукту, ідентифікатор покупця та продавця, тоді нам не потрібно надавати повні дані). Рахунок-фактура з ПДВ буде виставлений з поточною датою та терміном оплати 5 днів. Поле department_id вказує компанію (або відділ), що виставляє рахунок -фактуру (його можна отримати, натиснувши компанію в меню Налаштування> Дані фірмии)
```
curl https://YOUR_DOMAIN.bitfaktura.com.ua/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "invoice": {
            "payment_to_kind": 5,
            "client_id": 1,
            "positions":[
                {"product_id": 1, "quantity":2}
            ]
        }}'
```

Поля на рахунку-фактурі
```
"number" : "13/2012", - numer faktury (jeśli nie będzie podany wygeneruje się automatycznie)
"kind" : "vat", - rodzaj faktury (vat, proforma, bill, receipt, advance, correction, vat_mp, invoice_other, vat_margin, kp, kw, final, estimate)
"income" : "1", - faktura przychodowa (1) lub kosztowa (0)
"issue_date" : "2013-01-16", - data wystawienia
"place" : "Warszawa", - miejsce wystawienia
"sell_date" : "2013-01-16", - data sprzedaży (może być data lub miesiąc postaci 2012-12)
"category_id" : "", - id kategorii
"department_id" : "1", - id działu firmy (w menu Ustawienia > Dane firmy należy kliknąć na firmę/dział i ID działu pojawi się w URL); Jeśli nie będzie tego pola oraz nie będzie pola 'seller_name' wtedy będą wstawione domyślne dane Twojej firmy
"accounting_kind": "", - rodzaj wydatku dla faktur kosztowych - (purchases, expenses, media, salary, incident, fuel0, fuel_expl50, fuel_expl75, fuel_expl100, fixed_assets, fixed_assets50, no_vat_deduction)
"seller_name" : "Radgost Sp. z o.o.", - sprzedawca
"seller_tax_no" : "525-244-57-67", - numer identyfikacji podatkowej sprzedawcy (domyślnie NIP)
"seller_tax_no_kind" : "", - rodzaj numeru identyfikacyjnego sprzedawcy; pole puste (domyślnie) jest interpretowane jako "NIP"; w innym wypadku traktowane jako dowolny wpis własny (np. PESEL, REGON)
"seller_bank_account" : "24 1140 1977 0000 5921 7200 1001", - konto bankowe sprzedawcy
"seller_bank" : "BRE Bank",
"seller_post_code" : "02-548",
"seller_city" : "Warszawa",
"seller_street" : "ul. Olesińska 21",
"seller_country" : "", - kraj sprzedawcy (ISO 3166)
"seller_email" : "platnosci@radgost123.com",
"seller_www" : "",
"seller_fax" : "",
"seller_phone" : "",
"client_id" : "-1" - id kupującego (jeśi -1 to klient zostanie utworzony w systemie)
"buyer_name" : "Nazwa klienta" - nabywca
"buyer_tax_no" : "525-244-57-67", - numer identyfikacji podatkowej nabywcy (domyślnie NIP)
"buyer_tax_no_kind" : "", - rodzaj numeru identyfikacyjnego nabywcy; pole puste (domyślnie) jest interpretowane jako "NIP"; w innym wypadku traktowane jako wpis własny (np. PESEL, REGON)
"disable_tax_no_validation" : "",
"buyer_post_code" : "30-314", - kod pocztowy nabywcy
"buyer_city" : "Warszawa", - miasto nabywcy
"buyer_street" : "Nowa 44", - ulica nabywcy
"buyer_country" : "PL", - kraj nabywcy (ISO 3166)
"buyer_note" : "", - dodatkowy opis nabywcy
"buyer_email" : "", - email nabywcy
"recipient_id" : "", - id odbiorcy (id klienta z systemu)
"recipient_name" : "", - nazwa odbiorcy
"recipient_street" : "", - ulica odbiorcy
"recipient_post_code" : "", - kod pocztowy odbiorcy
"recipient_city" : "", - miasto odbiorcy
"recipient_country" : "", - kraj odbiorcy (ISO 3166)
"recipient_email" : "", - e-mail odbiorcy
"recipient_phone" : "", - numer telefonu odbiorcy
"recipient_note" : "", - dodatkowy opis odbiorcy
"additional_info" : "0" - czy wyświetlać dodatkowe pole na pozycjach faktury
"additional_info_desc" : "PKWiU" - nazwa dodatkowej kolumny na pozycjach faktury
"show_discount" : "0" - czy rabat
"payment_type" : "transfer",
"payment_to_kind" : termin płatności. gdy jest tu "other_date", wtedy można określić konkretną datę w polu "payment_to", jeśli jest tu liczba np. 5 to wtedy mamy 5 dniowy okres płatności
"payment_to" : "2013-01-16",
"status" : "issued",
"paid" : "0,00",
"oid" : "zamowienie10021", - numer zamówienia (np z zewnętrznego systemu zamówień)
"oid_unique" : jeśli to pole będzie ustawione na 'yes' wtedy nie system nie pozwoli stworzyc 2 faktur o takim samym OID (może to być przydatne w synchronizacji ze sklepem internetowym)
"warehouse_id" : "1090",
"seller_person" : "Imie Nazwisko",
"buyer_first_name" : "Imie",
"buyer_last_name" : "Nazwisko",
"paid_date" : "",
"currency" : "PLN",
"lang" : "pl",
"use_moss" : "0" - czy faktura MOSS
"exchange_currency" : "", - przeliczona waluta (przeliczanie sumy i podatku na inną walutę), np. "PLN"
"exchange_kind" : "", - źródło kursu do przeliczenia waluty ("ecb", "nbp", "cbr", "nbu", "nbg", "own")
"exchange_currency_rate" : "", - własny kurs przeliczenia waluty (używany, gdy parametr exchange_kind ustawiony jest na "own")
"invoice_template_id" : "1",
"description" : "- opis faktury",
"description_footer" : "", - opis umieszczony w stopce faktury
"description_long" : "", - opis umieszczony na odwrocie faktury
"invoice_id" : "" - pole z id powiązanego dokumentu, np. id zamówienia przy zaliczce albo id wzorcowej faktury przy fakturze cyklicznej,
"from_invoice_id" : "" - id faktury na podstawie której faktura została wygenerowana (przydatne np. w przypadku generacji Faktura VAT z Faktury Proforma),
"delivery_date" : "" - data wpłynięcia dokumentu (tylko przy wydatkach),
"buyer_company" : "1" - czy klient jest firmą
"additional_invoice_field" : "" - wartość dodatkowego pola na fakturze, Ustawienia > Ustawienia Konta > Konfiguracja > Faktury i dokumenty > Dodatkowe pole na fakturze
"internal_note" : "" - treść notatki prywatnej na fakturze, niewidoczna na wydruku.
"exclude_from_stock_level" : "" - informacja, czy system powinien liczyć tę fakturę do stanów magazynowych (true np., gdy faktura wystawiona na podstawie paragonu)
"gtu_codes" : [] - wartości kodów GTU produktów zawartych na fakturze - UWAGA - podane wartości nadpiszą wartości kodów GTU pobranych z kart produktów podawanych w pozycjach faktury, wartości tych kodów są nadrzędne dla całej faktury
"procedure_designations" : [] - oznaczenia dotyczące procedur
"positions":
   		"product_id" : "1",
   		"name" : "BitFaktura Start",
   		"additional_info" : "", - dodatkowa informacja na pozycji faktury (np. PKWiU)
   		"discount_percent" : "", - zniżka procentowa (uwaga: aby rabat był wyliczany trzeba ustawić pole: 'show_discount' na '1' oraz przed wywołaniem należy sprawdzić czy w Ustawieniach Konta pole: "Jak obliczać rabat" ustawione jest na "procentowo")
   		"discount" : "", - zniżka kwotowa (uwaga: aby rabat był wyliczany trzeba ustawić pole: 'show_discount' na 1 oraz przed wywołaniem należy sprawdzić czy w Ustawieniach Konta pole: "Jak obliczać rabat" ustawione jest na "kwotowo")
   		"quantity" : "1",
   		"quantity_unit" : "szt",
   		"price_net" : "59,00", - jeśli nie jest podana to zostanie wyliczona
   		"tax" : "23", - może być stawka lub "np" - dla nie podlega, "zw" - dla zwolniona
   		"price_gross" : "72,57", - jeśli nie jest podana to zostanie wyliczona
   		"total_price_net" : "59,00", - jeśli nie jest podana to zostanie wyliczona
   		"total_price_gross" : "72,57",
   		"code" : "" - kod produktu,
                "gtu_code" : "" - kod GTU produktu
"calculating_strategy":
{
  "position": "default" lub "keep_gross" - metoda wyliczania kwot na pozycjach faktury
  "sum": "sum" lub "keep_gross" lub "keep_net" - metoda sumowania kwot z pozycji
  "invoice_form_price_kind": "net" lub "gross" - cena jednostkowa na formatce faktury
},
"split_payment" : "1" - 1 lub 0 w zależności, czy faktura podlega pod split payment, czy nie
"accounting_vat_tax_date": "2020-12-18", Data księgowania podatku VAT (jeśli włączono w ustawieniach)
"accounting_income_tax_date": "2020-12-18", Data księgowania podatku dochodowego (jeśli włączono w ustawieniach)
```

Значення полів
Поле: `kind`
```
	"vat" - faktura VAT
	"proforma" - faktura Proforma
	"bill" - rachunek
	"receipt" - paragon
	"advance" - faktura zaliczkowa
	"final" - faktura końcowa
	"correction" - faktura korekta
	"vat_mp" - faktura MP
	"invoice_other" - inna faktura
	"vat_margin" - faktura marża
	"kp" - kasa przyjmie
	"kw" - kasa wyda
	"estimate" - zamówienie
	"vat_mp" - faktura MP
	"vat_rr" - faktura RR
	"correction_note" - nota korygująca
	"accounting_note" - nota księgowa
	"client_order" - własny dokument nieksięgowy
	"dw" - dowód wewnętrzny
	"wnt" - Wewnątrzwspólnotowe Nabycie Towarów
	"wdt" - Wewnątrzwspólnotowa Dostawa Towarów
	"import_service" - import usług
	"import_service_eu" - import usług z UE
	"import_products" - import towarów - procedura uproszczona
	"export_products" - eksport towarów
```

Поле: `lang`
```
	"pl" - рахунок-фактура польською
	"en" - англійска мова
	"en-GB" - англійска (Великобританія)
	"de" - німецька 
	"fr" - французька
	"cz" - чеська
	"ru" - росийська
	"es" -іспанська
	"it" - італійска
	"nl" - голландська
	"hr" - хорвацька
	"ar" - арабська
	"sk" - словацька
	"sl" - словенська мова
	"el" - грецька
	"et" - естоньська
	"cn" - китайска
	"hu" - угорська
	"tr" - турецька
	"fa" - перська

	Ви можете створювати двомовні рахунки-фактури, поєднуючи символи двох мов зі скісною рискою, наприклад:
	"ua/en" - українська і англійська
```

Поле: `income`
```
	"1" - рахунок-фактура (дохід)
	"0" - рахунок-фактура (кошти)
```

Поле: `accounting_kind`
```
  "purchases" - Закупівля товару/послуги
  "expenses" - Витрати бізнесу
  "media" - Медіа та телекомунікаційні послуги
  "salary" - Зарплати
  "incident" - Побічні витрати на закупівлі
  "fixed_assets" - Основні засоби
  "no_vat_deduction" - ПДВ не підлягає вирахуванню
```

Поле: `payment_type`
```
	"transfer" - банківський переказ
	"card" - картка
	"cash" -  готівка
	"barter" - бартер
	"cheque" - чек
	"bill_of_exchange" - вексель
	"cash_on_delivery" - накладений платіж
	"compensation" - компенсації
	"letter_of_credit" - акредитив
	"payu" - PayU
	"paypal" - PayPal
	"off" - "не відображати"
	"будь-який_інший_текст"
  
  Поле: `status`
```
	"issued" - виставлена
	"sent" - надіслана
	"paid" - оплачена
	"partial" - частково оплачена
	"rejected" - відхилена
```

 Поле: `discount_kind` - вид знижки
```
	"percent_unit" - розраховується з ціни за одиницю
	"percent_total" - розраховується з загальної ціни
	"amount" - сума
```
 
# Клієнти
Список клієнтів
`curl "https://YOUR_DOMAIN.fakturownia.pl/clients.json?api_token=API_TOKEN&page=1"`
Пошук клієнтів за іменем, електронною поштою, прізвищем або ідентифікаційним номером податку
```
curl "https://YOUR_DOMAIN.fakturownia.pl/clients.json?api_token=API_TOKEN&name=CLIENT_NAME"
curl "https://YOUR_DOMAIN.fakturownia.pl/clients.json?api_token=API_TOKEN&email=EMAIL_ADDRESS"
curl "https://YOUR_DOMAIN.fakturownia.pl/clients.json?api_token=API_TOKEN&shortcut=SHORT_NAME"
curl "https://YOUR_DOMAIN.fakturownia.pl/clients.json?api_token=API_TOKEN&tax_no=TAX_NO"
```

Завантаження вибраного клієнта за ідентифікатором
```
curl "https://YOUR_DOMAIN.fakturownia.pl/clients/100.json?api_token=API_TOKEN"
```

Завантаження вибраного клієнта за зовнішнім ідентифікатором
```
 curl "https://YOUR_DOMAIN.fakturownia.pl/clients.json?external_id=100&api_token=API_TOKEN"
```

Додавання клієнта
```
curl https://YOUR_DOMAIN.fakturownia.pl/clients.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "client": {
            "name": "Klient1",
            "tax_no": "5252445767",
            "bank" : "bank1",
            "bank_account" : "bank_account1",
            "city" : "city1",
            "country" : "",
            "email" : "email@gmail.com",
            "person" : "person1",
            "post_code" : "post-code1",
            "phone" : "phone1",
            "street" : "street1"
        }}'
```

Оновлення клієнта
```
curl https://YOUR_DOMAIN.fakturownia.pl/clients/111.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json'  \
    -d '{"api_token": "API_TOKEN",
        "client": {
            "name": "Klient2",
            "tax_no": "52524457672",
            "bank" : "bank2",
            "bank_account" : "bank_account2",
            "city" : "city2",
            "country" : "PL",
            "email" : "email@gmail.com",
            "person" : "person2",
            "post_code" : "post-code2",
            "phone" : "phone2",
            "street" : "street2"
        }}'
```

Видалення клієнта
```
curl -X DELETE "https://YOUR_DOMAIN.fakturownia.pl/clients/CLIENT_ID.json?api_token=API_TOKEN"
```

 # Продукти
Список продуктів
`curl "https://YOUR_DOMAIN.fakturownia.pl/products.json?api_token=API_TOKEN&page=1"`

Перелік продуктів з наявністю на складі
`curl "https://YOUR_DOMAIN.fakturownia.pl/products.json?api_token=API_TOKEN&warehouse_id=WAREHOUSE_ID&page=1"`

Завантаження вибраного товару за ідентифікатором
`curl "https://YOUR_DOMAIN.fakturownia.pl/products/100.json?api_token=API_TOKEN"`

Завантаження вибраного товару за ідентифікатором з рівнем складів
`curl "https://YOUR_DOMAIN.fakturownia.pl/products/100.json?api_token=API_TOKEN&warehouse_id=WAREHOUSE_ID"`

Додавання товару
```
curl https://YOUR_DOMAIN.fakturownia.pl/products.json \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json'  \
    -d '{"api_token": "API_TOKEN",
        "product": {
            "name": "PoroductAA",
            "code": "A001",
            "price_net": "100",
            "tax": "23"
        }}'
```
Оновлення продукту
```
curl https://YOUR_DOMAIN.fakturownia.pl/products/333.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json'  \
    -d '{"api_token": "API_TOKEN",
        "product": {
            "name": "PoroductAA2",
            "code": "A0012",
            "price_gross": "102",
	    "tax": "23"
        }}'
```
#Примітка# : Ціна нетто розраховується на основі валової ціни та вартості податку, її не можна редагувати безпосередньо за допомогою API.

# Прайс-листи
Перелік прайс-листів
`curl "https://YOUR_DOMAIN.fakturownia.pl/price_lists.json?api_token=API_TOKEN"`

Ви можете передати ті самі параметри, які вказані в додатку (на сторінці списку рахунків-фактур)

Додавання прайс-листа
```
curl https://YOUR_DOMAIN.fakturownia.pl/price_lists.json
                -H 'Accept: application/json'
                -H 'Content-Type: application/json'
                -d '{
                "api_token": "API_TOKEN",
                "price_list": {
                    "name": "Nazwa cennika",
		    "description": "Opis",
		    "currency": "PLN",
                    "price_list_positions_attributes": {
		    	"0": {
				"priceable_id": "ID produktu",
				"priceable_name": "Nazwa produktu",
				"priceable_type": "Product",
				"use_percentage": "0",
				"percentage": "",
				"price_net": "111.0",
				"price_gross": "136.53",
				"use_tax": "1",
				"tax": "23"
			}
		    }
                }}'
```

Оновлення прайсу
```
curl https://YOUR_DOMAIN.fakturownia.pl/price_lists/100.json
		-X PUT
                -H 'Accept: application/json'
                -H 'Content-Type: application/json'
                -d '{
                "api_token": "API_TOKEN",
                "price_list": {
                    "name": "Nazwa cennika",
		    "description": "Opis",
		    "currency": "PLN",
                }}'
```

Видалення прайс-листа
`curl -X DELETE "https://YOUR_DOMAIN.fakturownia.pl/price_lists/100.json?api_token=API_TOKEN"`

Складські документи
Всі складські документи
`curl "https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents.json?api_token=API_TOKEN"`

Ви можете передати ті самі параметри, які вказані в додатку (на сторінці списку рахунків-фактур)

Завантажте вибраний документ за ідентифікатором
`curl "https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents/555.json?api_token=API_TOKEN"`

Додавання складського документа (переміщення)
```
curl https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents.json
                -H 'Accept: application/json'
                -H 'Content-Type: application/json'
                -d '{
                "api_token": "API_TOKEN",
                "warehouse_document": {
                    "kind":"mm",
                    "number": null,
                    "warehouse_id": "1",
                    "issue_date": "2017-10-23",
                    "department_name": "Department1 SA",
                    "client_name": "Client1 SA",
                    "warehouse_actions":[
                        {"product_name":"Produkt A1", "purchase_tax":23, "purchase_price_net":10.23, "quantity":1, "warehouse2_id":13}
                    ]
                }}'
```
Додавання складського документа (прийняття)
```
curl https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents.json
				-H 'Accept: application/json'
				-H 'Content-Type: application/json'
				-d '{
				"api_token": "API_TOKEN",
				"warehouse_document": {
					"kind":"pz",
					"number": null,
					"warehouse_id": "1",
					"issue_date": "2017-10-23",
					"department_name": "Department1 SA",
					"client_name": "Client1 SA",
					"warehouse_actions":[
						{"product_name":"Produkt A1", "purchase_tax":23, "purchase_price_net":10.23, "quantity":1},
						{"product_name":"Produkt A2", "purchase_tax":0, "purchase_price_net":50, "quantity":2}
					]
				}}'

```
Додавання складського документа (видання)
```
curl https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents.json
				-H 'Accept: application/json'
				-H 'Content-Type: application/json'
				-d '{
				"api_token": "API_TOKEN",
				"warehouse_document": {
					"kind":"wz",
					"number": null,
					"warehouse_id": "1",
					"issue_date": "2017-10-23",
					"department_name": "Department1 SA",
					"client_name": "Client1 SA",
					"warehouse_actions":[
						{"product_id":"333", "tax":23, "price_net":10.23, "quantity":1},
						{"product_id":"333", "tax":0, "price_net":50, "quantity":2}
					]
				}}'
```
Додавання інвентарного документа (прийняття) для існуючого клієнта, відділу та товару
```curl https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents.json
				-H 'Accept: application/json'
				-H 'Content-Type: application/json'
				-d '{
				"api_token": "API_TOKEN",
				"warehouse_document": {
					"kind":"pz",
					"number": null,
					"warehouse_id": "1",
					"issue_date": "2017-10-23",
					"department_id": "222",
					"client_id": "111",
					"warehouse_actions":[
						{"product_id":"333", "purchase_tax":23, "price_net":10.23, "quantity":1},
						{"product_id":"333", "purchase_tax":0, "price_net":50, "quantity":2}
					]
				}}'
```
Оновлення документів
```
curl https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents/555.json
			 	-X PUT
				-H 'Accept: application/json'
				-H 'Content-Type: application/json'
				-d '{"api_token": "API_TOKEN",
					"warehouse_document": {
						"client_name": "New client name SA"
				    }}'
```
Вилучення документа
`curl -X DELETE "https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents/100.json?api_token=API_TOKEN"`

Поєднання наявних рахунків-фактур та складських документів
```curl https://YOUR_DOMAIN.fakturownia.pl/warehouse_documents/555.json
				-X PUT
				-H 'Accept: application/json'
				-H 'Content-Type: application/json'
				-d '{"api_token": "API_TOKEN",
					"warehouse_document": {
						"invoice_ids": [
							100,
							111
						]
					}}'
```
