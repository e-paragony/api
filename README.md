# API do wystawiania e-paragonów w systemie Paragony.pl oraz Fakturownia.pl

Opis jak zintegrować własną aplikację lub serwis z e-paragonami za pomocą [Paragony.pl](https://paragony.pl) lub [Fakturownia.pl](https://fakturownia.pl)

Dzięki API można z innych systemów wystawiać e-paragony lub paragony.

Działające przykłady wywołania API Fakturowni znajdują się też w w systemie Fakturownia (po zalogowaniu) w menu <b>Ustawienia > API</b> oraz na stronie: https://paragony.pl/api oraz https://fakturownia.pl/api

<a name="token"></a>

## API token

`API_TOKEN` token trzeba pobrać z ustawień aplikacj: `Ustawienia -> Ustawienia konta -> Integracja -> Kod autoryzacyjny API`

<a name="examples"></a>
## Przykłady

<a name="f1"></a>
### Pobranie listy paragonów

```shell
curl https://PREFIX.fakturownia.pl/invoices.json -H 'Authorization: Bearer __API_TOKEN__'
```

```shell
curl 'https://PREFIX.fakturownia.pl/invoices.json?kind=receipt' -H 'Authorization: Bearer __API_TOKEN__'
```

### Podbranie szczegółowych danych jednego paragonu

```shell
curl https://PREFIX.fakturownia.pl/invoices/376751870.json -H 'Authorization: Bearer __API_TOKEN__'
```

tu w polu `e_receipt_view_url` mamy link do E-Paragonu (jeśli został utworzony)

### Dodanie nowego paragonu

```shell
curl https://PREFIX.fakturownia.pl/invoices.json \
  -H 'Authorization: Bearer __API_TOKEN__' \
  -H 'Content-Type: application/json' \
  -d '{
    "invoice": {
      "kind":"receipt",
      "number": null,
      "sell_date": "2025-05-16",
      "issue_date": "2025-05-16",
      "payment_to": "2025-05-23",
      "seller_name": "Seller1 SA",
      "seller_tax_no": "6272616681",
      "buyer_name": "Client1 SA",
      "buyer_tax_no": "6272616682",
      "positions":[
        {"name":"Produkt A1", "tax":23, "total_price_gross":10.23, "quantity":1},
        {"name":"Produkt A2", "tax":0, "total_price_gross":50, "quantity":2}
      ]
    }
  }'
```

po stworzeniu paragonu zostanie wypisany `_ID_` i aby wydrukować e-paragon wywołujemu:

```shell
curl 'https://PREFIX.fakturownia.pl/invoices/fiscal_print?fiskator_name=_PRINTER_ID_&id=_ID_&mode=e-receipt' -H 'Authorization: Bearer __API_TOKEN__'
```

natomiast `_PRINTER_ID_` musimy pobrać ze strony: https://PREFIX.fakturownia.pl/printers.json

mode może przyjmować wartości:
- `e-receipt` - e-paragon
- `print` - wydruk paragonu papierowego

Aby wydrukować paragon lub e-paragon trzeba też wcześniej podłączyć do systeu Drukarkę Fiskalną. Tu jest opis jak to zrobić: https://pomoc.fakturownia.pl/45289196-Paragony-pl-2-5-1-modul-do-wydrukow-fiskalnych-instalacja-i-czynnosci-poinstalacyjne

Gdy drukarka będzie skonfigurowana wtedy po utworzeniu e-paragonu zostanie dla niego utworzony specjalny adres typu:


https://PREFIX.paragony.pl/iB000002Ef4cSVbaNyYupWFpwe33DDW (ten url jest dostępny w podglądzie paragonu w polu `e_receipt_view_url`)


i ten url może zostać przekazany klientowi (jest to poprawny E-Paragon)


## FAQ

### jak od razu wydrukować e-paragon

Klient może to włączyć na swoim koncie w Fakturowni - każdy Paragon wystawiony przez API może być automatycznie zlecany do fiskalizacji:
https://pomoc.fakturownia.pl/179236547-Automatyczna-fiskalizacja-paragonow-po-utworzeniu-przez-API

###  jak automatycznie wysłać e-mailem klientowi link do e-paragonu

Klient może to włączyć na swoim koncie w Fakturowni - jak tylko `e_receipt_view_url` zostanie uzupełnione, Fakturownia wyśle e-paragon mailem
https://pomoc.fakturownia.pl/179433234-Automatyczne-wysylanie-e-paragonow-e-mailem


### linki do e-paragonu

Link do podglądu e-paragonu e_receipt_view_url zwracany jest przy pobieraniu dokumentu

```
curl https://PREFIX.fakturownia.pl/invoices/376751870.json -H 'Authorization: Bearer __API_TOKEN__'
```

jeśli do danego dokumentu powstał już e-paragon. Natomiast np. w przypadku przerwy w dostawie prądu / internetu na drukarce fiskalnej, e-paragon może powstać z opóźnieniem od czasu wystawienia paragonu. Dlatego sugerowanym sposobem uzyskania linku do e-paragonu są webhooki.
Webhook `invoice:update`, po uzupełnieniu linku do e-paragonu przy danym Paragonie w Fakturowni, zwróci `e_receipt_view_url`. Webhooki można skonfigurować w `Ustawienia -> Ustawienia konta -> Integracja`, lub można zrobić to przez API:

- https://pomoc.fakturownia.pl/1239014-Integracja-danych-z-Fakturowni-za-pomoca-webhookow
- https://github.com/fakturownia/api?tab=readme-ov-file#webhooks

# TODO
- zmienić wydruk paragonu na POST (z GET)
- w druku fiskalnym powinna być możliwość niepodawania drukarki (i wtedy jest domyślna)
