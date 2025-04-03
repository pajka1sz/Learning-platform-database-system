# Uprawnienia dostępu

### Wywoływanie procedur:
1. **Kursant**:
    1. Dodaj zamówienie
    2. Dodaj produkt do zamówienia
    3. Wyświetl harmonogram kursanta
    4. Wyświetl podsumowanie kursanta
    ```sql
    CREATE ROLE kursant
    GRANT EXEC ON DodajZamowienie TO kursant
    GRANT EXEC ON DodajProduktDoZamowienia TO kursant
    GRANT EXEC ON WyswietlHarmonogramKursanta TO kursant
    GRANT EXEC ON WyswietlPodsumowanieKursanta TO kursant
    ```
2. **Wykładowca**:
    1. Dodaj spotkanie modułu stacjonarnego
    2. Dodaj spotkanie modułu synchronicznego
    3. Dodaj spotkanie modułu asynchronicznego
    4. Dodaj spotkanie przedmiotu stacjonarnego
    5. Dodaj spotkanie przedmiotu synchronicznego
    6. Dodaj spotkanie przedmiotu asynchronicznego
    7. Aktualizuj spotkanie stacjonarne
    8. Aktualizuj spotkanie synchroniczne
    9. Dodaj obecność dla modułu
    10. Dodaj obecność dla przedmiotu
    11. Dodaj obecność dla praktyk
    12. Dodaj praktyki
    ```sql
    CREATE ROLE wykladowca
    GRANT EXEC ON DodajSpotkanieModuluStacjonarnego TO wykladowca
    GRANT EXEC ON DodajSpotkanieModuluSynchronicznego TO wykladowca
    GRANT EXEC ON DodajSpotkanieModuluAsynchronicznego TO wykladowca
    GRANT EXEC ON DodajSpotkaniePrzedmiotuStacjonarne TO wykladowca
    GRANT EXEC ON DodajSpotkaniePrzedmiotuSynchroniczne TO wykladowca
    GRANT EXEC ON DodajSpotkaniePrzedmiotuAsynchronicznego TO wykladowca
    GRANT EXEC ON AktualizujSpotkanieStacjonarnePrzedmiotu TO wykladowca
    GRANT EXEC ON AktualizujSpotkanieSynchronicznePrzedmiotu TO wykladowca
    GRANT EXEC ON DodajObecnoscDlaModulu TO wykladowca
    GRANT EXEC ON DodajObecnoscDlaPraktyk TO wykladowca
    GRANT EXEC ON DodajObecnoscDlaPrzedmiotu TO wykladowca
    GRANT EXEC ON DodajPraktyki TO wykladowca
    ```
3. **Sekretarz**:
    1. Dodaj personel
    2. Przyznaj rolę
    3. Dodaj kurs
    4. Dodaj webinar
    5. Dodaj studium
    6. Dodaj moduł
    7. Dodaj przedmiot
    8. Dodaj kursanta
    9. Przypisz język do tłumacza
    10. Usuń webinar
    11. Dodaj język
    12. Dodaj kraj
    13. Dodaj miasto
    ```sql
    CREATE ROLE sekretarz
    GRANT EXEC ON DodajPersonel TO sekretarz
    GRANT EXEC ON DodajKurs TO sekretarz
    GRANT EXEC ON DodajWebinar TO sekretarz
    GRANT EXEC ON DodajStudium TO sekretarz
    GRANT EXEC ON DodajModul TO sekretarz
    GRANT EXEC ON DodajPrzedmiot TO sekretarz
    GRANT EXEC ON DodajKursant TO sekretarz
    GRANT EXEC ON PrzyznajRole TO sekretarz
    GRANT EXEC ON PrzypiszJezykDoTlumacza TO sekretarz
    GRANT EXEC ON UsunWebinar TO sekretarz
    GRANT EXEC ON DodajJezyk TO sekretarz
    GRANT EXEC ON DodajKraj TO sekretarz
    GRANT EXEC ON DodajMiasto TO sekretarz
    ```

4. **Dyrektor szkoły**:
    1. Zmień status zamówienia
    2. Dodaj rolę
    ```sql
    CREATE ROLE dyrektor
    GRANT EXEC ON ZmienStatusZamowienia TO dyrektor
    GRANT EXEC ON DodajRole TO dyrektor
    ```

### Wyświetlanie widoków:

1. **Kursant** - widoki:
    1. webinarów
    2. kursów
    3. studiów
    4. modułów
    5. przedmiotów
    6. praktyk
    7. zjazdów
    8. harmonogramu studiów
    9. spotkań modułów
    10. spotkań przedmiotów
    11. spotkań zjazdów
    12. sylabusu
    ```sql
    GRANT SELECT ON WidokWebinarow TO kursant
    GRANT SELECT ON WidokKursow TO kursant
    GRANT SELECT ON WidokStudium TO kursant
    GRANT SELECT ON WidokModulow TO kursant
    GRANT SELECT ON WidokPrzedmiotow TO kursant
    GRANT SELECT ON WidokPraktyk TO kursant
    GRANT SELECT ON WidokZjazdow TO kursant
    GRANT SELECT ON WidokHarmonogramuStudium TO kursant
    GRANT SELECT ON WidokSpotkanModulow TO kursant
    GRANT SELECT ON WidokSpotkanPrzedmiotow TO kursant
    GRANT SELECT ON WidokSpotkanZjazdow TO kursant
    GRANT SELECT ON WidokSylabus TO kursant
    ```
2. **Wykładowca** - widoki:
    1. frekwencji osób na modułach
    2. frekwencji osób na przedmiotach
    3. harmonogramu studiów
    4. osób bez zaliczenia kursu
    5. osób bez zaliczenia studiów
    6. osób z zaliczeniem kursu
    7. osób z zaliczeniem studiów
    8. osób zapisanych na kurs
    9. osób zapisanych na studia
    10. osób z dostępem do webinarów
    11. osób obcych na zjazdach
    12. podsumowania frekwencji
    13. sylabusu
    14. spotkań modułów
    15. spotkań przedmiotów
    16. spotkań zjazdów
    ```sql
    GRANT SELECT ON WidokFrekwencjiOsobNaModulach TO wykladowca
    GRANT SELECT ON WidokFrekwencjiOsobNaPrzedmiotach TO wykladowca
    GRANT SELECT ON WidokHarmonogramuStudium TO wykladowca
    GRANT SELECT ON WidokOsobBezZaliczeniaKursu TO wykladowca
    GRANT SELECT ON WidokOsobZZaliczonymStudium TO wykladowca
    GRANT SELECT ON WidokOsobZZaliczaniemKursu TO wykladowca
    GRANT SELECT ON WidokOsobZapisanychNaKurs TO wykladowca
    GRANT SELECT ON WidokOsobZapisanychNaStudium TO wykladowca
    GRANT SELECT ON WidokOsobZDostepemDoWebinarow TO wykladowca
    GRANT SELECT ON WidokOsobObcychNaZjazdach TO wykladowca
    GRANT SELECT ON WidokPodsumowaniaFrekwencji TO wykladowca
    GRANT SELECT ON WidokSylabus TO wykladowca
    GRANT SELECT ON WidokSpotkanModulow TO wykladowca
    GRANT SELECT ON WidokSpotkanPrzedmiotow TO wykladowca
    GRANT SELECT ON WidokSpotkanZjazdow TO wykladowca
    ```

3. **Raportowiec** - widoki:
    1. dłużników
    2. liczby kursantów dla poszczególnych krajów
    3. liczby kursantów dla poszczególnych miast
    4. liczby spotkań dla danej sali
    5. liczby osób obcych na zjazdach
    6. frekwencji osób na modułach
    7. frekwencji osób na przedmiotach
    8. osób zapisanych na kurs
    9. osób zapisanych na studia
    10. osób z dostępem do webinarów
    11. podsumowania frekwencji
    12. przychodów dla form kształcenia
    13. roli użytkowników
    14. bilokacji zajęć kursantów
    15. podsumowania zamówień
    16. zamówień form kształcenia z opłaconą zaliczką
    17. zamówień form kształcenia bez opłaconej zaliczki
    ```sql
    GRANT SELECT ON WidokDluznikow TO raportowiec
    GRANT SELECT ON WidokLiczbyKursantowDlaKrajow TO raportowiec
    GRANT SELECT ON WidokLiczbyKursantowDlaMiast TO raportowiec
    GRANT SELECT ON WidokLiczbySpotkanDlaSali TO raportowiec
    GRANT SELECT ON WidokOsobObcychNaZjazdach TO raportowiec
    GRANT SELECT ON WidokFrekwencjiOsobNaModulach TO raportowiec
    GRANT SELECT ON WidokFrekwencjiOsobNaPrzedmiotach TO raportowiec
    GRANT SELECT ON WidokOsobZapisanychNaKurs TO raportowiec
    GRANT SELECT ON WidokOsobZapisanychNaStudium TO raportowiec
    GRANT SELECT ON WidokOsobZDostepemDoWebinarow TO raportowiec
    GRANT SELECT ON WidokPodsumowaniaFrekwencji TO raportowiec
    GRANT SELECT ON WidokPrzychodowDlaFormKsztalcenia TO raportowiec
    GRANT SELECT ON WidokRoliUzytkownikow TO raportowiec
    GRANT SELECT ON WidokBilokacjiSpotkanKursanta TO raportowiec
    GRANT SELECT ON WidokPodsumowaniaZamowien TO raportowiec
    GRANT SELECT ON WidokZamowienFormKsztalceniaZOplaconaZaliczka TO raportowiec
    GRANT SELECT ON WidokZamowienFormKsztalceniaBezOplaconejZaliczki TO raportowiec
    ```
4. **Sekretarz** - widoki:
    1. harmonogramu studiów
    2. roli użytkowników
    3. tłumaczy i języków, które tłumaczą
    4. webinarów
    5. kursów
    6. studiów
    7. modułów
    8. przedmiotów
    9. praktyk
    10. zjazdów
    ```sql
    GRANT SELECT ON WidokHarmonogramuStudium TO sekretarz
    GRANT SELECT ON WidokRoliUzytkownikow TO sekretarz
    GRANT SELECT ON WidokTlumaczyIJezykow TO sekretarz
    GRANT SELECT ON WidokWebinarow TO sekretarz
    GRANT SELECT ON WidokKursow TO sekretarz
    GRANT SELECT ON WidokStudium TO sekretarz
    GRANT SELECT ON WidokModulow TO sekretarz
    GRANT SELECT ON WidokPrzedmiotow TO sekretarz
    GRANT SELECT ON WidokPraktyk TO sekretarz
    GRANT SELECT ON WidokZjazdow TO sekretarz
    ```
5. **Dyrektor szkoły** - widoki:
    1. podsumowania frekwencji
    2. liczby kursantów dla poszczególnych krajów
    3. liczby kursantów dla poszczególnych miast
    4. liczby spotkań dla danej sali
    5. przychodów dla form kształcenia
    6. roli użytkowników
    ```sql
    GRANT SELECT ON WidokPodsumowaniaFrekwencji TO dyrektor
    GRANT SELECT ON WidokLiczbyKursantowDlaKrajow TO dyrektor
    GRANT SELECT ON WidokLiczbyKursantowDlaMiast TO dyrektor
    GRANT SELECT ON WidokLiczbySpotkanDlaSali TO dyrektor
    GRANT SELECT ON WidokPrzychodowDlaFormKsztalcenia TO dyrektor
    GRANT SELECT ON WidokRoliUzytkownikow TO dyrektor
    ```
### Wywoływanie funkcji
1. **Wykładowca**:
    1. KolizjaSpotkanModulu
    2. KolizjaSpotkanPrzedmiotu
    3. KolizjaDat
    4. ZawieraSieWPrzedzialeCzasowym
    ```sql
    GRANT EXEC ON KolizjaSpotkanModulu TO wykladowca
    GRANT EXEC ON KolizjaSpotkanPrzedmiotu TO wykladowca
    GRANT EXEC ON KolizjaDat TO wykladowca
    GRANT EXEC ON ZawieraSieWPrzedzialeCzasowym TO wykladowca
    ```

2. **Sekretarz**:
    1. KolizjaSpotkanModulu
    2. KolizjaSpotkanPrzedmiotu
    3. KolizjaDat
    4. ZawieraSieWPrzedzialeCzasowym
    5. KolizjaTlumaczy
    6. KolizjaWykladowcow
    7. JakieIDRoli
    ```sql
    GRANT EXEC ON KolizjaSpotkanModulu TO sekretarz
    GRANT EXEC ON KolizjaSpotkanPrzedmiotu TO sekretarz
    GRANT EXEC ON KolizjaDat TO sekretarz
    GRANT EXEC ON ZawieraSieWPrzedzialeCzasowym TO sekretarz
    GRANT EXEC ON KolizjaTlumaczy TO sekretarz
    GRANT EXEC ON KolizjaWykladowcow TO sekretarz
    GRANT EXEC ON JakieIDRoli TO sekretarz
    ```





