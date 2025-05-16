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
Pobranie listy paragonów

```shell
curl -H 'Authorization: Bearer __API_TOKEN__' https://PREFIX.fakturownia.pl/invoices.json
```

```shell
curl -H 'Authorization: Bearer __API_TOKEN__' 'https://PREFIX.fakturownia.pl/invoices.json?kind=receipt'
```

Podbranie szczegółowych danych jednego paragonu

```shell
curl -H 'Authorization: Bearer __API_TOKEN__' https://PREFIX.fakturownia.pl/invoices/376751870.json
```

Dodanie nowego paragonu

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
curl -H 'Authorization: Bearer __API_TOKEN__' 'https://PREFIX.fakturownia.pl/invoices/fiscal_print?fiskator_name=_PRINTER_ID_&id=_ID_&mode=e-receipt'
```

natomiast `_PRINTER_ID_` musimy pobrać ze strony: https://PREFIX.fakturownia.pl/printers.json

a mode może być:
- `e-receipt` - e-paragon
- `print` - wydruk paragonu papierowego

Aby wydrukować paragon lub e-paragon trzeba też wcześniej podłączyć do systeu Drukarkę Fiskalną. Tu jest opis jak to zrobić: https://pomoc.fakturownia.pl/45289196-Paragony-pl-2-5-1-modul-do-wydrukow-fiskalnych-instalacja-i-czynnosci-poinstalacyjne

Gdy drukarka będzie skonfigurowana wtedy po utworzeniu e-paragonu zostanie dla niego utworzony specjalny adres typu:


https://PREFIX.paragony.pl/iB000002Ef4cSVbaNyYupWFpwe33DDW [TODO: opisać jak to przez API pobrać]


i ten url może zostać przekazany klientowi (jest to poprawny E-Paragon)


## TODO do opisania / poprawienia
- jak automatycznie wysłać e-mailem klientowi link do e-paragony
- do ustalenia parametr jak od razu wydrukować e-paragon (lub ew za pomocą kolejnego requesta)
- do ustalenia pola w KSEF buyer_company i currency
- ja przez api pobrać URL e-paragonu
- zmienić wydruk paragonu na POST (z GET)
