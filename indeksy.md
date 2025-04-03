# Indeksy
### ***Forma Kształcenia***
```sql
CREATE NONCLUSTERED INDEX FormaKsztalcenia_Nazwa ON [Forma Ksztalcenia] (Nazwa)
```

### ***Kurs***
```sql
CREATE NONCLUSTERED INDEX Kurs_DataRozpoczecia ON Kurs (DataRozpoczecia)
CREATE NONCLUSTERED INDEX Kurs_DataZakonczenia ON Kurs (DataZakonczenia)
CREATE NONCLUSTERED INDEX Kurs_Cena ON Kurs (Cena)
```

### ***Studium***
```sql
CREATE NONCLUSTERED INDEX Studium_DataRozpoczecia ON Studium (DataRozpoczecia)
CREATE NONCLUSTERED INDEX Studium_DataZakonczenia ON Studium (DataZakonczenia)
CREATE NONCLUSTERED INDEX Studium_LimitMiejsc ON Studium (LimitMiejsc)
```

### ***Webinar***
```sql
CREATE NONCLUSTERED INDEX Webinar_DataRozpoczecia ON Webinar (DataRozpoczecia)
CREATE NONCLUSTERED INDEX Webinar_DataZakonczenia ON Webinar (DataZakonczenia)
CREATE NONCLUSTERED INDEX Webinar_Cena ON Webinar (Cena)
CREATE NONCLUSTERED INDEX Webinar_IDWykladowca ON Webinar (IDWykladowca)
CREATE NONCLUSTERED INDEX Webinar_IDTlumacz ON Webinar (IDTlumacz)
CREATE NONCLUSTERED INDEX Webinar_IDJezyk ON Webinar (IDJezyk)
```

### ***Kursant***
```sql
CREATE NONCLUSTERED INDEX Kursant_IDMiasto ON Kursant (IDMiasto)
CREATE NONCLUSTERED INDEX Kursant_Ulica ON Kursant (Ulica)
CREATE NONCLUSTERED INDEX Kursant_KodPocztowy ON Kursant (KodPocztowy)
```

### ***Moduły***
```sql
CREATE NONCLUSTERED INDEX Moduly_IDKurs ON Moduly (IDKurs)
CREATE NONCLUSTERED INDEX Moduly_Nazwa ON Moduly (Nazwa)
CREATE NONCLUSTERED INDEX Moduly_IDWykladowca ON Moduly (IDWykladowca)
```

### ***Moduły Spotkania***
```sql
CREATE NONCLUSTERED INDEX ModulySpotkania_IDModul ON [Moduly Spotkania] (IDModul)
```

### ***Przedmioty***
```sql
CREATE NONCLUSTERED INDEX Przedmioty_IDStudium ON Przedmioty (IDStudium)
CREATE NONCLUSTERED INDEX Przedmioty_IDSemestr ON Przedmioty (IDSemestr)
CREATE NONCLUSTERED INDEX Przedmioty_IDWykladowca ON Przedmioty (IDWykladowca)
CREATE NONCLUSTERED INDEX Przedmioty_Nazwa ON Przedmioty (Nazwa)
```

### ***Przedmioty Spotkania***
```sql
CREATE NONCLUSTERED INDEX PrzedmiotySpotkania_IDPrzedmiot ON [Przedmioty Spotkania] (IDPrzedmiot)
```

### ***Przedmioty Obecność***
```sql
CREATE NONCLUSTERED INDEX PrzedmiotyObecnosc_Data ON [Przedmioty Obecnosc] (Data)
```

### ***Przedmioty Zaliczenia***
```sql
CREATE NONCLUSTERED INDEX PrzedmiotyZaliczenia_CzyZaliczyl ON [Przedmioty Zaliczenia] (CzyZaliczyl)
```

### ***Zjazd***
```sql
CREATE NONCLUSTERED INDEX Zjazd_IDStudium ON Zjazd (IDStudium)
CREATE NONCLUSTERED INDEX Zjazd_DataRozpoczecia ON Zjazd (DataRozpoczecia)
CREATE NONCLUSTERED INDEX Zjazd_DataZakonczenia ON Zjazd (DataZakonczenia)
```

### ***Praktyki***
```sql
CREATE NONCLUSTERED INDEX Praktyki_IDKursant ON Praktyki (IDKursant)
CREATE NONCLUSTERED INDEX Praktyki_IDFirmy ON Praktyki (IDFirmy)
CREATE NONCLUSTERED INDEX Praktyki_IDStudium ON Praktyki (IDStudium)
CREATE NONCLUSTERED INDEX Praktyki_DataRozpoczecia ON Praktyki (DataRozpoczecia)
CREATE NONCLUSTERED INDEX Praktyki_DataZakonczenia ON Praktyki (DataZakonczenia)
```

### ***Semestr***
```sql
CREATE NONCLUSTERED INDEX Semestr_Nazwa ON Semestr (Nazwa)
```

### ***Spotkanie***
```sql
CREATE NONCLUSTERED INDEX Spotkanie_IDTlumacz ON Spotkanie (IDTlumacz)
CREATE NONCLUSTERED INDEX Spotkanie_IDJezyk ON Spotkanie (IDJezyk)
```

### ***Spotkanie Online Synchroniczne***
```sql
CREATE NONCLUSTERED INDEX SpotkanieOS_DataRozpoczecia ON [Spotkanie online synchroniczne] (DataRozpoczecia)
CREATE NONCLUSTERED INDEX SpotkanieOS_DataZakonczenia ON [Spotkanie online synchroniczne] (DataZakonczenia)
```

### ***Spotkanie Stacjonarne***
```sql
CREATE NONCLUSTERED INDEX SpotkanieS_DataRozpoczecia ON [Spotkanie stacjonarne] (DataRozpoczecia)
CREATE NONCLUSTERED INDEX SpotkanieS_DataZakonczenia ON [Spotkanie stacjonarne] (DataZakonczenia)
CREATE NONCLUSTERED INDEX SpotkanieS_IDSali ON [Spotkanie stacjonarne] (IDSali)
```

### ***Zamówienia***
```sql
CREATE NONCLUSTERED INDEX Zamowienia_IDKursant ON Zamowienia (IDKursant)
CREATE NONCLUSTERED INDEX Zamowienia_IDStatus ON Zamowienia (IDStatus)
CREATE NONCLUSTERED INDEX Zamowienia_DataZamowienia ON Zamowienia (DataZamowienia)
```

### ***Zamówienia Informacje***
```sql
CREATE NONCLUSTERED INDEX ZamowieniaInformacje_DataWplaty ON [Zamowienia Informacje] (DataWplaty)
CREATE NONCLUSTERED INDEX ZamowieniaInformacje_CzyZaliczka ON [Zamowienia Informacje] (CzyZaliczka)
```

### ***Zamówienia Informacje Zjazdy***
```sql
CREATE NONCLUSTERED INDEX ZamowieniaInformacjeZjazdy_DataWplaty ON [Zamowienia Informacje Zjazdy] (DataWplaty)[Zamowienia Informacje] (CzyZaliczka)
```

### ***Miasto***
```sql
CREATE NONCLUSTERED INDEX Miatso_IDKraj ON Miasto (IDKraj)
CREATE NONCLUSTERED INDEX Miasto_Nazwa ON Miasto (Nazwa)
```
