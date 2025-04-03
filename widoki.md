# Widoki

### ***WidokFrekwencjiOsobNaModulach***  

Prezentuje frekwencję kursantów na modułach (procentowo).  

```sql
SELECT wsk.IDKursant, wsk.Imie, wsk.Nazwisko, wsk.Miasto, wsk.Ulica, wsk.KodPocztowy, wsk.Telefon, wsk.Email, wsk.IDFormyKsztalcenia, 
       wm.IDModul, wm.Nazwa AS NazwaModulu, 
       (CASE 
            WHEN wm.[Liczba Spotkan] = 0 THEN 0 
            ELSE (SELECT COUNT(ms.IDSpotkanie)
                  FROM [Moduly Spotkania] AS ms 
                  JOIN [Moduly Obecnosc] AS mo ON ms.IDSpotkanie = mo.IDSpotkanie
                  WHERE ms.IDModul = wm.IDModul AND mo.IDKursant = wsk.IDKursant) * 100 / wm.[Liczba Spotkan] 
        END) AS Frekwencja
FROM dbo.WidokOsobZapisanychNaKurs AS wsk 
INNER JOIN dbo.WidokModulow AS wm ON wsk.IDFormyKsztalcenia = wm.IDKurs
```

### ***WidokFrekwencjiOsobNaPrzedmiotach***  

Prezentuje frekwencję kursantów na przedmiotach (procentowo).  

```sql
SELECT wss.IDKursant, wss.Imie, wss.Nazwisko, wss.Miasto, wss.Ulica, wss.KodPocztowy, wss.Telefon, wss.Email, wss.IDFormyKsztalcenia, 
       ws.IDPrzedmiot, ws.Nazwa AS NazwaModulu, 
       (CASE 
            WHEN ws.[LiczbaSpotkan] = 0 THEN 0 
            ELSE (SELECT COUNT(ps.IDSpotkanie)
                  FROM [Przedmioty Spotkania] AS ps 
                  JOIN [Przedmioty Obecnosc] AS po ON ps.IDSpotkanie = po.IDSpotkanie
                  WHERE ps.IDPrzedmiot = ws.IDPrzedmiot AND po.IDKursant = wss.IDKursant) * 100 / ws.[LiczbaSpotkan] 
        END) AS Frekwencja
FROM dbo.WidokOsobZapisanychNaStudium AS wss 
INNER JOIN dbo.WidokSylabus AS ws ON wss.IDFormyKsztalcenia = ws.IDStudium
```

### ***WidokLiczbyUzytkownikowRoli***
Wyświetla ilu jest użytkowników danej roli.

```sql
SELECT ur.IDRola, ru.Nazwa, COUNT(*) AS 'Liczba użytkowników'
FROM dbo.[Uzytkownicy Role] AS ur INNER JOIN
    dbo.[Role Uzytkownikow] AS ru ON ru.IDRola = urIDRola
GROUP BY ur.IDRola, ru.Nazwa
```

### ***WidokHarmonogramuStudium***
Wyświetla harmonogram studiów - wszystkie dane dotyczące spotkań odbywających się w ramach studiów.
```sql
SELECT s.IDSpotkanie, s.IDTlumacz, u.Imie + ' ' + u.Nazwisko AS Tlumacz, ws.IDPrzedmiot, ws.IDStudium, ws.NazwaStudium, ws.Nazwa, ws.Opis, ws.IDWykladowca, ws.Wykladowca, ISNULL(sos.Url, soa.Url) AS Url, ss.LimitMiejsc, sala.IDSali, 
                  sala.Nazwa AS Sala, ISNULL(ss.DataRozpoczecia, sos.DataRozpoczecia) AS DataRozpoczecia, ISNULL(ss.DataZakonczenia, sos.DataZakonczenia) AS DataZakonczenia
FROM     dbo.[Przedmioty Spotkania] AS ps INNER JOIN
                  dbo.WidokSylabus AS ws ON ps.IDPrzedmiot = ws.IDPrzedmiot INNER JOIN
                  dbo.Spotkanie AS s ON s.IDSpotkanie = ps.IDSpotkanie LEFT OUTER JOIN
                  dbo.Uzytkownicy AS u ON u.IDUzytkownik = s.IDTlumacz LEFT OUTER JOIN
                  dbo.[Spotkanie stacjonarne] AS ss ON s.IDSpotkanie = ss.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie Sala] AS sala ON ss.IDSali = sala.IDSali LEFT OUTER JOIN
                  dbo.[Spotkanie online synchroniczne] AS sos ON s.IDSpotkanie = sos.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie online asynchroniczne] AS soa ON s.IDSpotkanie = soa.IDSpotkanie
```

### ***WidokKursow***
Prezentuje kursy dostępne na platformie (zakończone i aktualne).
```sql
SELECT k.IDKurs, fk.Nazwa, k.Cena, k.DataRozpoczecia, k.DataZakonczenia,
                      (SELECT COUNT(*) AS Expr1
                       FROM      dbo.Moduly
                       WHERE   (IDKurs = k.IDKurs)) AS LiczbaModulow
FROM     dbo.[Forma Ksztalcenia] AS fk INNER JOIN
                  dbo.Kurs AS k ON fk.IDFormyKsztalcenia = k.IDKurs
```

### ***WidokModulow***
Prezentuje moduły, z których składają się kursy.
```sql
SELECT m.IDModul, m.IDKurs, m.Nazwa, m.Opis, m.IDWykladowca, u.Imie + ' ' + u.Nazwisko AS Wykladowca,
                      (SELECT COUNT(IDModul) AS Expr1
                       FROM      dbo.[Moduly Spotkania] AS ms
                       WHERE   (IDModul = m.IDModul)) AS [Liczba Spotkan]
FROM     dbo.Moduly AS m INNER JOIN
                  dbo.Uzytkownicy AS u ON m.IDWykladowca = u.IDUzytkownik
```

### ***WidokOsobBezZaliczeniaKursu***
Wyświetla kursantów, którzy nie zaliczyli kursów.
```sql
SELECT IDKursant, Imie, Nazwisko, Miasto, Ulica, KodPocztowy, Telefon, Email, IDFormyKsztalcenia,
                      (SELECT COUNT(*) AS Expr1
                       FROM      dbo.WidokFrekwencjiOsobNaModulach
                       WHERE   (IDFormyKsztalcenia = wzk.IDFormyKsztalcenia) AND (IDKursant = wzk.IDKursant) AND (Frekwencja >= 80)) * 100 /
                      (SELECT LiczbaModulow
                       FROM      dbo.WidokKursow
                       WHERE   (IDKurs = wzk.IDFormyKsztalcenia)) AS ZaliczoneModuly
FROM     dbo.WidokOsobZapisanychNaKurs AS wzk
WHERE  ((SELECT COUNT(*) AS Expr1
                   FROM      dbo.WidokFrekwencjiOsobNaModulach
                   WHERE   (IDFormyKsztalcenia = wzk.IDFormyKsztalcenia) AND (IDKursant = wzk.IDKursant) AND (Frekwencja >= 80)) * 100 /
                      (SELECT LiczbaModulow
                       FROM      dbo.WidokKursow
                       WHERE   (IDKurs = wzk.IDFormyKsztalcenia)) < 80)
```

### ***WidokOsobBezZaliczeniaStudium***
Wyświetla kursantów, którzy nie zaliczyli studiów.
```sql
SELECT IDKursant, Imie, Nazwisko, Miasto, Ulica, KodPocztowy, Telefon, Email, IDFormyKsztalcenia
FROM     dbo.WidokOsobZapisanychNaStudium AS wzs
WHERE  (IDKursant IN
                      (SELECT IDKursant
                       FROM      dbo.WidokFrekwencjiOsobNaPrzedmiotach
                       WHERE   (Frekwencja < 80))) OR
                  (IDKursant IN
                      (SELECT IDKursant
                       FROM      dbo.WidokPraktyk AS wfp
                       WHERE   (Frekwencja < 100))) OR
                  (IDKursant IN
                      (SELECT IDKursant
                       FROM      dbo.[Przedmioty Zaliczenia]
                       WHERE   (CzyZaliczyl = 0)))
```

### ***WidokOsobZZaliczeniemKursu***
Wyświetla kursantów, którzy zaliczyli kurs.
```sql
SELECT IDKursant, Imie, Nazwisko, Miasto, Ulica, KodPocztowy, Telefon, Email, IDFormyKsztalcenia,
                      (SELECT COUNT(*) AS Expr1
                       FROM      dbo.WidokFrekwencjiOsobNaModulach
                       WHERE   (IDFormyKsztalcenia = wzk.IDFormyKsztalcenia) AND (IDKursant = wzk.IDKursant) AND (Frekwencja >= 80)) * 100 /
                      (SELECT LiczbaModulow
                       FROM      dbo.WidokKursow
                       WHERE   (IDKurs = wzk.IDFormyKsztalcenia)) AS ZaliczoneModuly
FROM     dbo.WidokOsobZapisanychNaKurs AS wzk
WHERE  ((SELECT COUNT(*) AS Expr1
                   FROM      dbo.WidokFrekwencjiOsobNaModulach
                   WHERE   (IDFormyKsztalcenia = wzk.IDFormyKsztalcenia) AND (IDKursant = wzk.IDKursant) AND (Frekwencja >= 80)) * 100 /
                      (SELECT LiczbaModulow
                       FROM      dbo.WidokKursow
                       WHERE   (IDKurs = wzk.IDFormyKsztalcenia)) >= 80)
```

### ***WidokOsobZZaliczeniemStudium***
Wyświetla kursantów, którzy zaliczyli studia.
```sql
SELECT IDKursant, Imie, Nazwisko, Miasto, Ulica, KodPocztowy, Telefon, Email, IDFormyKsztalcenia
FROM     dbo.WidokOsobZapisanychNaStudium AS wzs
WHERE  (IDKursant IN
                      (SELECT IDKursant
                       FROM      dbo.WidokFrekwencjiOsobNaPrzedmiotach
                       WHERE   (Frekwencja >= 80))) AND (IDKursant IN
                      (SELECT IDKursant
                       FROM      dbo.WidokPraktyk AS wfp
                       WHERE   (Frekwencja = 100))) AND (IDKursant IN
                      (SELECT IDKursant
                       FROM      dbo.[Przedmioty Zaliczenia]
                       WHERE   (CzyZaliczyl = 1)))
```

### ***WidokOsobZapisanychNaKurs***
Wyświetla osoby zapisane na kurs (czyli osoby, które opłaciły kurs).
```sql
SELECT k.IDKursant, u.Imie, u.Nazwisko, m.Nazwa AS Miasto, k.Ulica, k.KodPocztowy, k.Telefon, k.Email, wz.IDFormyKsztalcenia
FROM     dbo.Kursant AS k INNER JOIN
                  dbo.WidokZamowien AS wz ON wz.IDKursant = k.IDKursant INNER JOIN
                  dbo.Kurs AS ks ON wz.IDFormyKsztalcenia = ks.IDKurs INNER JOIN
                  dbo.Uzytkownicy AS u ON k.IDKursant = u.IDUzytkownik INNER JOIN
                  dbo.Miasto AS m ON k.IDMiasto = m.IDMiasto
WHERE  (wz.CzyZaplacil = 1)
```

### ***WidokOsobZapisanychNaStudium***
Wyświetla osoby zapisane na studium (czyli osoby, które opłaciły studia).
```sql
SELECT k.IDKursant, u.Imie, u.Nazwisko, m.Nazwa AS Miasto, k.Ulica, k.KodPocztowy, k.Telefon, k.Email, wz.IDFormyKsztalcenia
FROM     dbo.Kursant AS k INNER JOIN
                  dbo.WidokZamowien AS wz ON wz.IDKursant = k.IDKursant INNER JOIN
                  dbo.Studium AS s ON wz.IDFormyKsztalcenia = s.IDStudium INNER JOIN
                  dbo.Uzytkownicy AS u ON k.IDKursant = u.IDUzytkownik INNER JOIN
                  dbo.Miasto AS m ON k.IDMiasto = m.IDMiasto
WHERE  (wz.CzyZaplacil = 1)
```

### ***WidokOsobZDostepemDoWebinarow***
Wyświetla osoby, które zapisały się na webinar - bezpłatny, bądź płatny.
```sql
SELECT ww.*, k.IDKursant, u.Imie, u.Nazwisko, m.Nazwa AS Miasto, k.Ulica, k.KodPocztowy, k.Telefon, k.Email
FROM     Webinar AS w JOIN
                  WidokWebinarow AS ww ON w.IDWebinar = ww.IDWebinar CROSS JOIN
                  Kursant AS k JOIN
                  Uzytkownicy AS u ON k.IDKursant = u.IDUzytkownik JOIN
                  Miasto AS m ON k.IDMiasto = m.IDMiasto
WHERE  w.Cena IS NULL AND GETDATE() < DATEADD(DAY, 30, w.DataZakonczenia)
UNION
SELECT ww.*, k.IDKursant, u.Imie, u.Nazwisko, m.Nazwa AS Miasto, k.Ulica, k.KodPocztowy, k.Telefon, k.Email
FROM     Webinar AS w JOIN
                  WidokWebinarow AS ww ON w.IDWebinar = ww.IDWebinar CROSS JOIN
                  Kursant AS k JOIN
                  Uzytkownicy AS u ON k.IDKursant = u.IDUzytkownik JOIN
                  Miasto AS m ON k.IDMiasto = m.IDMiasto
WHERE  w.Cena IS NOT NULL AND w.IDWebinar IN
                      (SELECT wz.IDFormyKsztalcenia
                       FROM      WidokZamowien AS wz
                       WHERE   wz.CzyZaplacil = 1 AND wz.IDFormyKsztalcenia = w.IDWebinar AND wz.IDKursant = k.IDKursant AND (GETDATE() < DATEADD(DAY, 30, w.DataZakonczenia) OR
                                         GETDATE() < DATEADD(DAY, 30, wz.DataOstatniejWplaty)))
```

### ***WidokPraktyk***
Wyświetla praktyki, w których biorą udział kursanci.
```sql
SELECT p.IDPraktyk, p.IDKursant, u.Imie, u.Nazwisko, m.Nazwa AS Miasto, k.Ulica, k.KodPocztowy, k.Telefon, k.Email, f.IDFirma, f.Nazwa, p.DataRozpoczecia, p.DataZakonczenia,
                      (SELECT COUNT(*) AS Expr1
                       FROM      dbo.[Praktyki Obecnosc] AS po
                       WHERE   (IDPraktyk = p.IDPraktyk)) * 100 / 14 AS Frekwencja
FROM     dbo.Praktyki AS p INNER JOIN
                  dbo.[Forma Ksztalcenia] AS fk ON p.IDStudium = fk.IDFormyKsztalcenia INNER JOIN
                  dbo.Firma AS f ON p.IDFirmy = f.IDFirma INNER JOIN
                  dbo.Uzytkownicy AS u ON u.IDUzytkownik = p.IDKursant INNER JOIN
                  dbo.Kursant AS k ON k.IDKursant = p.IDKursant INNER JOIN
                  dbo.Miasto AS m ON m.IDMiasto = k.IDMiasto
```

### ***WidokLiczbyPraktykantowDlaFirm***
Wyświetla liczbę praktykantów dla każdej firmy.
```sql
SELECT p.IDFirmy, f.Nazwa, COUNT(*) AS 'Liczba praktykantów'
FROM     dbo.Firma AS f INNER JOIN
                  dbo.Praktyki AS p ON f.IDFirma = p.IDFirmy
GROUP BY p.IDFirmy, f.Nazwa
```

### ***WidokLiczbyObecnosciKursantaNaPraktykach***
```sql
SELECT p.IDFirmy, f.Nazwa, p.IDKursant, COUNT(*) AS 'Obecności'
FROM     dbo.Praktyki AS p INNER JOIN
                  dbo.Firma AS f ON p.IDFirmy = f.IDFirma INNER JOIN
                  dbo.[Praktyki Obecnosc] AS po ON po.IDPraktyk = p.IDPraktyk
GROUP BY p.IDFirmy, f.Nazwa, p.IDKursant
```

### ***WidokOsobObcychNaZjazdach***
Wyświetla osoby, które zapisały się na zjazdy bez uczestniczenia w całym studium.
```sql
SELECT K.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, ZJ.IDZjazd, ZJ.DataRozpoczecia, ZJ.DataZakonczenia
FROM Kursant AS K
INNER JOIN Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik
LEFT JOIN Zamowienia AS Z ON K.IDKursant = Z.IDKursant
LEFT JOIN [Zamowienia Informacje Zjazdy] AS ZIZ ON Z.IDZamowienia = ZIZ.IDZamowienia
INNER JOIN Zjazd AS ZJ ON ZIZ.IDZjazd = ZJ.IDZjazd
INNER JOIN Studium AS S ON ZJ.IDStudium = S.IDStudium
INNER JOIN [Forma Ksztalcenia] AS FK ON S.IDStudium = FK.IDFormyKsztalcenia
WHERE FK.IDFormyKsztalcenia NOT IN
	(SELECT ZI1.IDFormyKsztalcenia FROM Kursant AS K1
	INNER JOIN Zamowienia AS Z1 ON K1.IDKursant = Z1.IDKursant
	INNER JOIN [Zamowienia Informacje] AS ZI1 ON Z1.IDZamowienia = ZI1.IDZamowienia
WHERE K1.IDKursant = K.IDKursant)
```

### ***WidokPrzychodowDlaFormKsztalcenia***
Widok wyświetla przychody dla form kształcenia z rozróżnieniem na typy form kształcenia.
```sql
SELECT IDZamowienia, IDFormyKsztalcenia, Zaplacono, DoZaplaty, DataWplaty
FROM     dbo.[Zamowienia Informacje]
UNION
SELECT IDZamowienia, IDZjazd, Zaplacono, DoZaplaty, DataWplaty
FROM     dbo.[Zamowienia Informacje Zjazdy])
    SELECT T .IDFormyKsztalcenia, FK.Nazwa, 'Webinar' AS 'Typ formy kształcenia', SUM(ISNULL(Zaplacono, 0)) AS Przychod
    FROM     tab AS T INNER JOIN
                      [Forma Ksztalcenia] AS FK ON T .IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                      Webinar AS W ON FK.IDFormyKsztalcenia = W.IDWebinar
    GROUP BY T .IDFormyKsztalcenia, FK.Nazwa
UNION
SELECT T .IDFormyKsztalcenia, FK.Nazwa, 'Kurs' AS 'Typ formy kształcenia', SUM(ISNULL(Zaplacono, 0)) AS Przychod
FROM     tab AS T INNER JOIN
                  [Forma Ksztalcenia] AS FK ON T .IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Kurs AS K ON FK.IDFormyKsztalcenia = K.IDKurs
GROUP BY T .IDFormyKsztalcenia, FK.Nazwa
UNION
SELECT T .IDFormyKsztalcenia, FK.Nazwa, 'Studia' AS 'Typ formy kształcenia', SUM(ISNULL(Zaplacono, 0)) AS Przychod
FROM     tab AS T INNER JOIN
                  [Forma Ksztalcenia] AS FK ON T .IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Studium AS S ON FK.IDFormyKsztalcenia = S.IDStudium
GROUP BY T .IDFormyKsztalcenia, FK.Nazwa
```

### ***WidokDluznikow***
Widok prezentuje osoby, które nie uiściły opłat za daną formę kształcenia w wymaganym terminie.
```sql
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, NULL AS 'IDZjazd', ZI.Zaplacono, ZI.DoZaplaty
FROM     dbo.[Zamowienia Informacje] AS ZI INNER JOIN
                  dbo.Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia LEFT JOIN
                  dbo.Kurs AS K ON FK.IDFormyKsztalcenia = K.IDKurs
WHERE  ZI.Zaplacono < ZI.DoZaplaty AND K.DataRozpoczecia < DATEADD(DAY, 3, GETDATE())
UNION
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, NULL AS 'IDZjazd', ZI.Zaplacono, ZI.DoZaplaty
FROM     dbo.[Zamowienia Informacje] AS ZI INNER JOIN
                  dbo.Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia LEFT JOIN
                  dbo.Webinar AS W ON FK.IDFormyKsztalcenia = W.IDWebinar
WHERE  ZI.Zaplacono < ZI.DoZaplaty AND W.DataRozpoczecia < GETDATE()
UNION
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, NULL AS 'IDZjazd', ZI.Zaplacono, ZI.DoZaplaty
FROM [Zamowienia Informacje] AS ZI INNER JOIN
                  dbo.Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia LEFT JOIN
                  dbo.Studium AS S ON FK.IDFormyKsztalcenia = S.IDStudium
WHERE  ZI.Zaplacono < ZI.DoZaplaty AND S.DataRozpoczecia < DATEADD(DAY, 3, GETDATE())
UNION
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, ZJ.IDZjazd AS 'IDZjazd', ZIZ.Zaplacono, ZIZ.DoZaplaty
FROM [Zamowienia Informacje Zjazdy] AS ZIZ INNER JOIN
                  dbo.Zamowienia AS Z ON ZIZ.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.Zjazd AS ZJ ON ZIZ.IDZjazd = ZJ.IDZjazd LEFT JOIN
                  dbo.Studium AS S ON ZJ.IDStudium = S.IDStudium INNER JOIN
				  dbo.[Forma Ksztalcenia] AS FK ON S.IDStudium = FK.IDFormyKsztalcenia
WHERE  ZIZ.Zaplacono < ZIZ.DoZaplaty AND ZJ.DataRozpoczecia < DATEADD(DAY, 3, GETDATE())
```

### ***WidokSpotkanModulow***
Prezentuje spotkania dla każdego modułu, z którego składają się kursy.
```sql
SELECT s.IDSpotkanie, wm.IDModul, wm.IDKurs, fk.Nazwa AS NazwaKursu, s.IDTlumacz, u.Imie + ' ' + u.Nazwisko AS Tlumacz, wm.Nazwa, wm.Opis, wm.IDWykladowca, wm.Wykladowca, wm.[Liczba Spotkan], ISNULL(sos.Url, soa.Url) AS Url, 
                  ss.LimitMiejsc, sala.IDSali, sala.Nazwa AS Sala, ISNULL(ss.DataRozpoczecia, sos.DataRozpoczecia) AS DataRozpoczecia, ISNULL(ss.DataZakonczenia, sos.DataZakonczenia) AS DataZakonczenia
FROM     dbo.[Moduly Spotkania] AS ms INNER JOIN
                  dbo.Spotkanie AS s ON ms.IDSpotkanie = s.IDSpotkanie INNER JOIN
                  dbo.WidokModulow AS wm ON ms.IDModul = wm.IDModul INNER JOIN
                  dbo.[Forma Ksztalcenia] AS fk ON fk.IDFormyKsztalcenia = wm.IDKurs LEFT OUTER JOIN
                  dbo.Uzytkownicy AS u ON u.IDUzytkownik = s.IDTlumacz LEFT OUTER JOIN
                  dbo.[Spotkanie stacjonarne] AS ss ON s.IDSpotkanie = ss.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie Sala] AS sala ON ss.IDSali = sala.IDSali LEFT OUTER JOIN
                  dbo.[Spotkanie online synchroniczne] AS sos ON s.IDSpotkanie = sos.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie online asynchroniczne] AS soa ON s.IDSpotkanie = soa.IDSpotkanie
```

### ***WidokSylabus***
Prezentuje syllabus kursu - wyświetla przedmioty, ich opisy i prowadzących je wykładowców.
```sql
SELECT p.IDPrzedmiot, s.IDStudium, fk.Nazwa AS NazwaStudium, p.Nazwa, p.Opis, p.IDWykladowca, u.Imie + ' ' + u.Nazwisko AS Wykladowca, sem.IDSemestr, sem.Nazwa AS Semestr,
                      (SELECT COUNT(IDPrzedmiot) AS Expr1
                       FROM      dbo.[Przedmioty Spotkania]
                       WHERE   (IDPrzedmiot = p.IDPrzedmiot)) AS LiczbaSpotkan
FROM     dbo.Przedmioty AS p INNER JOIN
                  dbo.Studium AS s ON p.IDStudium = s.IDStudium INNER JOIN
                  dbo.[Forma Ksztalcenia] AS fk ON s.IDStudium = fk.IDFormyKsztalcenia INNER JOIN
                  dbo.Uzytkownicy AS u ON p.IDWykladowca = u.IDUzytkownik INNER JOIN
                  dbo.Semestr AS sem ON p.IDSemestr = sem.IDSemestr
```

### ***WidokPrzedmiotow***
```sql
SELECT P.IDPrzedmiot, P.Nazwa, P.Opis, S.IDStudium, FK.Nazwa AS Studia, SE.Nazwa AS Semestr, S.DataRozpoczecia AS [Data rozpoczecia studiow], S.DataZakonczenia AS [Data zakonczenia studiow], U.IDUzytkownik AS IDWykladowcy, 
                  U.Imie + ' ' + U.Nazwisko AS Wykladowca
FROM     dbo.Przedmioty AS P INNER JOIN
                  dbo.Studium AS S ON P.IDStudium = S.IDStudium LEFT OUTER JOIN
                  dbo.Semestr AS SE ON P.IDSemestr = SE.IDSemestr INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON S.IDStudium = FK.IDFormyKsztalcenia INNER JOIN
                  dbo.[Uzytkownicy Role] AS UR ON P.IDRolaWykladowca = UR.IDRola AND P.IDWykladowca = UR.IDUzytkownik INNER JOIN
                  dbo.Uzytkownicy AS U ON UR.IDUzytkownik = U.IDUzytkownik
```

### ***WidokSpotkanPrzedmiotow***
```sql
SELECT SP.IDSpotkanie, ISNULL(SS.DataRozpoczecia, SOS.DataRozpoczecia) AS [Data rozpoczecia], ISNULL(SS.DataZakonczenia, SOS.DataZakonczenia) AS [Data zakonczenia], SPSA.Nazwa AS Sala, SS.LimitMiejsc, P.Nazwa AS Przedmiot, 
                  U.IDUzytkownik AS IDWykladowcy, U.Imie + ' ' + U.Nazwisko AS Wykladowca, SE.Nazwa AS Semestr, ST.IDStudium, FK.Nazwa AS Studia
FROM     dbo.Przedmioty AS P INNER JOIN
                  dbo.Studium AS ST ON P.IDStudium = ST.IDStudium LEFT OUTER JOIN
                  dbo.Semestr AS SE ON P.IDSemestr = SE.IDSemestr INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON ST.IDStudium = FK.IDFormyKsztalcenia INNER JOIN
                  dbo.[Przedmioty Spotkania] AS PS ON P.IDPrzedmiot = PS.IDPrzedmiot INNER JOIN
                  dbo.Spotkanie AS SP ON PS.IDSpotkanie = SP.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie stacjonarne] AS SS ON SP.IDSpotkanie = SS.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie online synchroniczne] AS SOS ON SP.IDSpotkanie = SOS.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie online asynchroniczne] AS SOA ON SP.IDSpotkanie = SOA.IDSpotkanie INNER JOIN
                  dbo.[Spotkanie Sala] AS SPSA ON SS.IDSali = SPSA.IDSali INNER JOIN
                  dbo.[Uzytkownicy Role] AS UR ON P.IDRolaWykladowca = UR.IDRola AND P.IDWykladowca = UR.IDUzytkownik INNER JOIN
                  dbo.Uzytkownicy AS U ON UR.IDUzytkownik = U.IDUzytkownik
```

### ***WidokSpotkanDlaKursantow***
```sql
SELECT K.IDKursant, U.Imie, U.Nazwisko, WSM.IDSpotkanie, WSM.DataRozpoczecia, WSM.DataZakonczenia
FROM Kursant AS K
LEFT JOIN WidokOsobZapisanychNaKurs AS WOZNK ON K.IDKursant = WOZNK.IDKursant
INNER JOIN Kurs AS KU ON WOZNK.IDFormyKsztalcenia = KU.IDKurs
INNER JOIN WidokSpotkanModulow AS WSM ON KU.IDKurs = WSM.IDKurs
INNER JOIN Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik
UNION
SELECT K.IDKursant, U.Imie, U.Nazwisko, WSP.IDSpotkanie, WSP.[Data rozpoczecia], WSP.[Data zakonczenia]
FROM Kursant AS K
LEFT JOIN WidokOsobZapisanychNaStudium AS WOZNS ON K.IDKursant = WOZNS.IDKursant
INNER JOIN Studium AS S ON WOZNS.IDFormyKsztalcenia = S.IDStudium
INNER JOIN WidokSpotkanPrzedmiotow AS WSP ON S.IDStudium = WSP.IDStudium
INNER JOIN Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik
UNION
SELECT K.IDKursant, U.Imie, U.Nazwisko, WSZ.IDSpotkanie, WSZ.[Data rozpoczecia], WSZ.[Data rozpoczecia]
FROM Kursant AS K
LEFT JOIN WidokOsobObcychNaZjazdach AS WOONZ ON K.IDKursant = WOONZ.IDKursant
INNER JOIN Zjazd AS ZJ ON WOONZ.IDZjazd = ZJ.IDZjazd
INNER JOIN WidokSpotkanZjazdow AS WSZ ON ZJ.IDZjazd = WSZ.IDZjazd
INNER JOIN Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik
UNION
SELECT K.IDKursant, U.Imie, U.Nazwisko, W.IDWebinar, W.DataRozpoczecia, W.DataZakonczenia
FROM Kursant AS K
INNER JOIN WidokOsobZDostepemDoWebinarow AS WOZDDW ON K.IDKursant = WOZDDW.IDKursant
INNER JOIN Webinar AS W ON WOZDDW.IDWebinar = W.IDWebinar
INNER JOIN Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik
```

### ***WidokBilokacjiSpotkanKursanta***
```sql
SELECT WSDK.IDKursant, WSDK.IDSpotkanie, WSDK.DataRozpoczecia, WSDK.DataZakonczenia, WSDK1.IDSpotkanie AS Expr1, WSDK1.DataRozpoczecia AS Expr2, WSDK1.DataZakonczenia AS Expr3
FROM     dbo.WidokSpotkanDlaKursantow AS WSDK INNER JOIN
                  dbo.WidokSpotkanDlaKursantow AS WSDK1 ON WSDK.IDKursant = WSDK1.IDKursant AND WSDK.IDSpotkanie < WSDK1.IDSpotkanie
WHERE  (dbo.KolizjaDat(WSDK.DataRozpoczecia, WSDK.DataZakonczenia, WSDK1.DataRozpoczecia, WSDK1.DataZakonczenia) = 1)
```

### ***WidokWebinarow***
Wyświetla webinary dostępne na platformie.
```sql
SELECT w.IDWebinar, fk.Nazwa, w.Cena, w.DataRozpoczecia, w.DataZakonczenia, w.Url, u1.Imie + ' ' + u1.Nazwisko AS Wykladowca, u2.Imie + ' ' + u2.Nazwisko AS Tlumacz
FROM     dbo.[Forma Ksztalcenia] AS fk INNER JOIN
                  dbo.Webinar AS w ON fk.IDFormyKsztalcenia = w.IDWebinar INNER JOIN
                  dbo.Uzytkownicy AS u1 ON w.IDWykladowca = u1.IDUzytkownik LEFT OUTER JOIN
                  dbo.Uzytkownicy AS u2 ON w.IDTlumacz = u2.IDUzytkownik
```

### ***WidokZamowien***
Wyświetla złożone zamówienia, ich status i dane dotyczące kursanta oraz ostatniej wpłaty.
```sql
SELECT z.IDZamowienia, z.IDKursant, u.Imie + ' ' + u.Nazwisko AS Kursant, zs.Nazwa AS Status, fk.IDFormyKsztalcenia, fk.Nazwa AS NazwaFormyKsztalcenia, z.DataZamowienia, zj.DataWplaty AS DataOstatniejWplaty, 
                  CASE WHEN Zaplacono - DoZaplaty = 0 THEN 1 ELSE 0 END AS CzyZaplacil
FROM     dbo.Zamowienia AS z INNER JOIN
                  dbo.[Zamowienia Informacje] AS zj ON z.IDZamowienia = zj.IDZamowienia INNER JOIN
                  dbo.[Forma Ksztalcenia] AS fk ON zj.IDFormyKsztalcenia = fk.IDFormyKsztalcenia INNER JOIN
                  dbo.[Zamowienie Status] AS zs ON z.IDStatus = zs.IDStatus INNER JOIN
                  dbo.Uzytkownicy AS u ON z.IDKursant = u.IDUzytkownik
WHERE  (z.IDStatus = 2)
```

### ***WidokLiczbyKursantowDlaKrajow***
Wyświetla liczbę kursantów z poszczególnych krajów.
```sql
SELECT ss.IDSali, ss.Nazwa, COUNT(*) AS 'Liczba spotkań'
FROM     dbo.[Spotkanie Sala] AS ss INNER JOIN
                  dbo.[Spotkanie stacjonarne] AS st ON st.IDSali = ss.IDSali
GROUP BY ss.IDSali, ss.Nazwa
```

### ***WidokLiczbyKursantowDlaMiast***
Wyświetla liczbę kursantów z poszczególnych miast.
```sql
SELECT k.IDMiasto, m.Nazwa, COUNT(*) AS 'Liczba kursantów'
FROM     dbo.Kursant AS k INNER JOIN
                  dbo.Miasto AS m ON m.IDMiasto = k.IDMiasto
GROUP BY k.IDMiasto, m.Nazwa
```

### ***WidokLiczbySpotkanDlaSali***
Wyświetla liczbę spotkań, które odbywają się w danej sali.
```sql
SELECT c.IDKraj, c.Nazwa, COUNT(*) AS 'Liczba kursantów'
FROM     dbo.Kursant AS k INNER JOIN
                  dbo.Miasto AS m ON m.IDMiasto = k.IDMiasto INNER JOIN
                  dbo.Kraj AS c ON c.IDKraj = m.IDKraj
GROUP BY c.IDKraj, c.Nazwa
```

### ***WidokTlumaczyIJezykow***
Wyświetla tłumaczy i języki, którymi potrafią się posługiwać.
```sql
SELECT j.Nazwa, tj.IDTlumacz, u.Imie, u.Nazwisko, u.Zdjecie
FROM     dbo.[Tlumacz Jezyki] AS tj INNER JOIN
                  dbo.Jezyk AS j ON j.IDJezyk = tj.IDJezyk INNER JOIN
                  dbo.[Uzytkownicy Role] AS ur ON ur.IDUzytkownik = tj.IDTlumacz AND ur.IDRola = tj.IDRolaTlumacz INNER JOIN
                  dbo.Uzytkownicy AS u ON u.IDUzytkownik = ur.IDUzytkownik
```

### ***WidokZjazdow***
Wyświetla zjazdy.
```sql
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, NULL AS 'IDZjazd', ZI.Zaplacono, ZI.DoZaplaty
FROM     dbo.[Zamowienia Informacje] AS ZI INNER JOIN
                  dbo.Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia LEFT JOIN
                  dbo.Kurs AS K ON FK.IDFormyKsztalcenia = K.IDKurs
WHERE  ZI.Zaplacono < ZI.DoZaplaty AND K.DataRozpoczecia < DATEADD(DAY, 3, GETDATE())
UNION
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, NULL AS 'IDZjazd', ZI.Zaplacono, ZI.DoZaplaty
FROM     dbo.[Zamowienia Informacje] AS ZI INNER JOIN
                  dbo.Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia LEFT JOIN
                  dbo.Webinar AS W ON FK.IDFormyKsztalcenia = W.IDWebinar
WHERE  ZI.Zaplacono < ZI.DoZaplaty AND W.DataRozpoczecia < GETDATE()
UNION
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, NULL AS 'IDZjazd', ZI.Zaplacono, ZI.DoZaplaty
FROM     [Zamowienia Informacje] AS ZI INNER JOIN
                  dbo.Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia LEFT JOIN
                  dbo.Studium AS S ON FK.IDFormyKsztalcenia = S.IDStudium
WHERE  ZI.Zaplacono < ZI.DoZaplaty AND S.DataRozpoczecia < DATEADD(DAY, 3, GETDATE())
UNION
SELECT Z.IDKursant, U.Imie, U.Nazwisko, FK.IDFormyKsztalcenia, FK.Nazwa, ZJ.IDZjazd AS 'IDZjazd', ZIZ.Zaplacono, ZIZ.DoZaplaty
FROM     [Zamowienia Informacje Zjazdy] AS ZIZ INNER JOIN
                  dbo.Zamowienia AS Z ON ZIZ.IDZamowienia = Z.IDZamowienia INNER JOIN
                  dbo.Kursant AS KU ON Z.IDKursant = KU.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON KU.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.Zjazd AS ZJ ON ZIZ.IDZjazd = ZJ.IDZjazd LEFT JOIN
                  dbo.Studium AS S ON ZJ.IDStudium = S.IDStudium INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON S.IDStudium = FK.IDFormyKsztalcenia
WHERE  ZIZ.Zaplacono < ZIZ.DoZaplaty AND ZJ.DataRozpoczecia < DATEADD(DAY, 3, GETDATE())
```

### ***WidokSpotkanZjazdow***
```sql
SELECT SP.IDSpotkanie, ISNULL(SS.DataRozpoczecia, SOS.DataRozpoczecia) AS [Data rozpoczecia], ISNULL(SS.DataZakonczenia, SOS.DataZakonczenia) AS [Data Zakonczenia], SS.IDSali, SS.LimitMiejsc AS [Limit miejsc], SOA.Url, Z.IDZjazd, 
                  Z.DataRozpoczecia AS [Rozpoczecie zjazdu], Z.DataZakonczenia AS [Zakonczenie zjazdu], Z.IDStudium, FK.Nazwa AS Studium, P.IDPrzedmiot, P.Nazwa AS Przedmiot, P.Opis AS [Opis przedmiotu], U.IDUzytkownik AS IDWykladowcy, 
                  U.Imie + ' ' + U.Nazwisko AS Wykladowca
FROM     dbo.Zjazd AS Z INNER JOIN
                  dbo.Studium AS S ON Z.IDStudium = S.IDStudium INNER JOIN
                  dbo.[Forma Ksztalcenia] AS FK ON S.IDStudium = FK.IDFormyKsztalcenia LEFT OUTER JOIN
                  dbo.[Zjazdy Spotkania] AS ZS ON Z.IDZjazd = ZS.IDZjazd LEFT OUTER JOIN
                  dbo.Spotkanie AS SP ON ZS.IDSpotkanie = SP.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie stacjonarne] AS SS ON SP.IDSpotkanie = SS.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie online synchroniczne] AS SOS ON SP.IDSpotkanie = SOS.IDSpotkanie LEFT OUTER JOIN
                  dbo.[Spotkanie online asynchroniczne] AS SOA ON SP.IDSpotkanie = SOA.IDSpotkanie INNER JOIN
                  dbo.[Przedmioty Spotkania] AS PS ON SP.IDSpotkanie = PS.IDSpotkanie INNER JOIN
                  dbo.Przedmioty AS P ON PS.IDPrzedmiot = P.IDPrzedmiot INNER JOIN
                  dbo.[Uzytkownicy Role] AS UR ON P.IDRolaWykladowca = UR.IDRola AND P.IDWykladowca = UR.IDUzytkownik INNER JOIN
                  dbo.Uzytkownicy AS U ON UR.IDUzytkownik = U.IDUzytkownik
```

### ***WidokPodsumowaniaFrekwencji***
```sql
SELECT wfm.IDFormyKsztalcenia, fk.Nazwa, avg(wfm.Frekwencja) AS Frekwencja
FROM     WidokFrekwencjiOsobNaModulach AS wfm JOIN
                  [Forma Ksztalcenia] AS fk ON fk.IDFormyKsztalcenia = wfm.IDFormyKsztalcenia JOIN
                  Kurs AS k ON wfm.IDFormyKsztalcenia = k.IDKurs
WHERE  k.DataZakonczenia < GETDATE()
GROUP BY wfm.IDFormyKsztalcenia, fk.Nazwa
UNION
SELECT wfp.IDFormyKsztalcenia, fk.Nazwa, avg(wfp.Frekwencja) AS Frekwencja
FROM     WidokFrekwencjiOsobNaPrzedmiotach AS wfp JOIN
                  [Forma Ksztalcenia] AS fk ON fk.IDFormyKsztalcenia = wfp.IDFormyKsztalcenia JOIN
                  Studium AS s ON wfp.IDFormyKsztalcenia = s.IDStudium
WHERE  s.DataZakonczenia < GETDATE()
GROUP BY wfp.IDFormyKsztalcenia, fk.Nazwa
```

### ***WidokRoliUzytkownikow***
```sql
SELECT U.IDUzytkownik, U.Imie, U.Nazwisko, RU.IDRola, RU.Nazwa
FROM     dbo.Uzytkownicy AS U INNER JOIN
                  dbo.[Uzytkownicy Role] AS UR ON U.IDUzytkownik = UR.IDUzytkownik INNER JOIN
                  dbo.[Role Uzytkownikow] AS RU ON UR.IDRola = RU.IDRola
```

### ***WidokFormKsztalceniaZOplaconaZaliczka***
```sql
SELECT ZI.IDZamowienia, ZI.IDFormyKsztalcenia, 'Webinar' AS 'Typ formy ksztalcenia', FK.Nazwa AS 'Nazwa', ZI.Zaplacono, ZI.DoZaplaty, ZI.DataWplaty, U.Imie, U.Nazwisko, K.Telefon, K.Email, KR.Nazwa AS 'Kraj', M.Nazwa AS 'Miasto', K.Ulica, 
                  K.KodPocztowy
FROM     [Zamowienia Informacje] AS ZI INNER JOIN
                  Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  Kursant AS K ON Z.IDKursant = K.IDKursant INNER JOIN
                  Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik INNER JOIN
                  Miasto AS M ON K.IDMiasto = M.IDMiasto INNER JOIN
                  Kraj AS KR ON M.IDKraj = KR.IDKraj INNER JOIN
                  [Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Webinar AS W ON FK.IDFormyKsztalcenia = W.IDWebinar
WHERE  CzyZaliczka = 1
UNION
SELECT ZI.IDZamowienia, ZI.IDFormyKsztalcenia, 'Kurs' AS 'Typ formy ksztalcenia', FK.Nazwa AS 'Nazwa', ZI.Zaplacono, ZI.DoZaplaty, ZI.DataWplaty, U.Imie, U.Nazwisko, K.Telefon, K.Email, KR.Nazwa AS 'Kraj', M.Nazwa AS 'Miasto', K.Ulica, 
                  K.KodPocztowy
FROM     [Zamowienia Informacje] AS ZI INNER JOIN
                  Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  Kursant AS K ON Z.IDKursant = K.IDKursant INNER JOIN
                  Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik INNER JOIN
                  Miasto AS M ON K.IDMiasto = M.IDMiasto INNER JOIN
                  Kraj AS KR ON M.IDKraj = KR.IDKraj INNER JOIN
                  [Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Kurs ON FK.IDFormyKsztalcenia = Kurs.IDKurs
WHERE  CzyZaliczka = 1
UNION
SELECT ZI.IDZamowienia, ZI.IDFormyKsztalcenia, 'Studium' AS 'Typ formy ksztalcenia', FK.Nazwa AS 'Nazwa', ZI.Zaplacono, ZI.DoZaplaty, ZI.DataWplaty, U.Imie, U.Nazwisko, K.Telefon, K.Email, KR.Nazwa AS 'Kraj', M.Nazwa AS 'Miasto', K.Ulica, 
                  K.KodPocztowy
FROM     [Zamowienia Informacje] AS ZI INNER JOIN
                  Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  Kursant AS K ON Z.IDKursant = K.IDKursant INNER JOIN
                  Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik INNER JOIN
                  Miasto AS M ON K.IDMiasto = M.IDMiasto INNER JOIN
                  Kraj AS KR ON M.IDKraj = KR.IDKraj INNER JOIN
                  [Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Studium AS S ON FK.IDFormyKsztalcenia = S.IDStudium
WHERE  CzyZaliczka = 1
```

### ***WidokFormKsztalceniaBezOplaconejZaliczki***
```sql
SELECT ZI.IDZamowienia, ZI.IDFormyKsztalcenia, 'Webinar' AS 'Typ formy ksztalcenia', FK.Nazwa AS 'Nazwa', ZI.Zaplacono, ZI.DoZaplaty, ZI.DataWplaty, U.Imie, U.Nazwisko, K.Telefon, K.Email, KR.Nazwa AS 'Kraj', M.Nazwa AS 'Miasto', K.Ulica, 
                  K.KodPocztowy
FROM     [Zamowienia Informacje] AS ZI INNER JOIN
                  Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  Kursant AS K ON Z.IDKursant = K.IDKursant INNER JOIN
                  Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik INNER JOIN
                  Miasto AS M ON K.IDMiasto = M.IDMiasto INNER JOIN
                  Kraj AS KR ON M.IDKraj = KR.IDKraj INNER JOIN
                  [Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Webinar AS W ON FK.IDFormyKsztalcenia = W.IDWebinar
WHERE  CzyZaliczka = 0
UNION
SELECT ZI.IDZamowienia, ZI.IDFormyKsztalcenia, 'Kurs' AS 'Typ formy ksztalcenia', FK.Nazwa AS 'Nazwa', ZI.Zaplacono, ZI.DoZaplaty, ZI.DataWplaty, U.Imie, U.Nazwisko, K.Telefon, K.Email, KR.Nazwa AS 'Kraj', M.Nazwa AS 'Miasto', K.Ulica, 
                  K.KodPocztowy
FROM     [Zamowienia Informacje] AS ZI INNER JOIN
                  Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  Kursant AS K ON Z.IDKursant = K.IDKursant INNER JOIN
                  Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik INNER JOIN
                  Miasto AS M ON K.IDMiasto = M.IDMiasto INNER JOIN
                  Kraj AS KR ON M.IDKraj = KR.IDKraj INNER JOIN
                  [Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Kurs ON FK.IDFormyKsztalcenia = Kurs.IDKurs
WHERE  CzyZaliczka = 0
UNION
SELECT ZI.IDZamowienia, ZI.IDFormyKsztalcenia, 'Studium' AS 'Typ formy ksztalcenia', FK.Nazwa AS 'Nazwa', ZI.Zaplacono, ZI.DoZaplaty, ZI.DataWplaty, U.Imie, U.Nazwisko, K.Telefon, K.Email, KR.Nazwa AS 'Kraj', M.Nazwa AS 'Miasto', K.Ulica, 
                  K.KodPocztowy
FROM     [Zamowienia Informacje] AS ZI INNER JOIN
                  Zamowienia AS Z ON ZI.IDZamowienia = Z.IDZamowienia INNER JOIN
                  Kursant AS K ON Z.IDKursant = K.IDKursant INNER JOIN
                  Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik INNER JOIN
                  Miasto AS M ON K.IDMiasto = M.IDMiasto INNER JOIN
                  Kraj AS KR ON M.IDKraj = KR.IDKraj INNER JOIN
                  [Forma Ksztalcenia] AS FK ON ZI.IDFormyKsztalcenia = FK.IDFormyKsztalcenia INNER JOIN
                  Studium AS S ON FK.IDFormyKsztalcenia = S.IDStudium
WHERE  CzyZaliczka = 0
```

### ***WidokPodsumowaniaZamowien***
```sql
SELECT Z.IDZamowienia, ISNULL(SUM(ZI.DoZaplaty), 0) + ISNULL(SUM(ZIZ.DoZaplaty), 0) AS 'Calkowity koszt', ISNULL(SUM(ZI.Zaplacono), 0) + ISNULL(SUM(ZIZ.Zaplacono), 0) AS 'Zaplacono', ISNULL(SUM(ZI.DoZaplaty), 0) 
                  + ISNULL(SUM(ZIZ.DoZaplaty), 0) - ISNULL(SUM(ZI.Zaplacono), 0) - ISNULL(SUM(ZIZ.Zaplacono), 0) AS 'Do zaplaty', Z.DataZamowienia, ZS.IDStatus, ZS.Nazwa AS 'Status zamowienia', K.IDKursant, U.Imie, U.Nazwisko, K.Telefon, K.Email, 
                  K.KodPocztowy, K.Ulica, KR.Nazwa AS 'Kraj', M.Nazwa AS 'Miasto'
FROM     dbo.Zamowienia AS Z INNER JOIN
                  dbo.Kursant AS K ON Z.IDKursant = K.IDKursant INNER JOIN
                  dbo.Uzytkownicy AS U ON K.IDKursant = U.IDUzytkownik INNER JOIN
                  dbo.[Zamowienie Status] AS ZS ON Z.IDStatus = ZS.IDStatus LEFT OUTER JOIN
                  dbo.[Zamowienia Informacje] AS ZI ON Z.IDZamowienia = ZI.IDZamowienia LEFT OUTER JOIN
                  dbo.[Zamowienia Informacje Zjazdy] AS ZIZ ON Z.IDZamowienia = ZIZ.IDZamowienia INNER JOIN
                  dbo.Miasto AS M ON K.IDMiasto = M.IDMiasto INNER JOIN
                  dbo.Kraj AS KR ON M.IDKraj = KR.IDKraj
WHERE  (ZI.CzyZaliczka = 1) AND (Z.IDZamowienia IS NOT NULL) AND (ZS.Nazwa = 'Oplacone')
GROUP BY Z.IDZamowienia, Z.DataZamowienia, ZS.IDStatus, ZS.Nazwa, K.IDKursant, U.Imie, U.Nazwisko, K.Telefon, K.Email, K.KodPocztowy, K.Ulica, KR.Nazwa, M.Nazwa
```

