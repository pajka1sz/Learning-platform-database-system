
# Tabele
### ***Forma kształcenia***  
Określa formy kształcenia dostępne w szkole.  
|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDFormyKształcenia**</u>| Unikalne ID formy kształcenia | **Klucz główny** |
| Nazwa | Nazwa formy kształcenia (*Studium/Kurs/Webinar*) |-|
```sql
CREATE TABLE [dbo].[Forma Ksztalcenia] (
    [IDFormyKsztalcenia] [int] NOT NULL,
      NOT NULL,
    CONSTRAINT [PK_Forma Ksztalcenia] PRIMARY KEY CLUSTERED (
        [IDFormyKsztalcenia] ASC
    )
);
```

### ***Webinar***  
Zawiera informacje o każdym webinarze, który się odbył lub ma się odbyć.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDWebinar**</u>|Unikalne ID webinaru|**Klucz główny**|
|<u>IDWykladowca</u>|Unikalne ID wykładowcy|Klucz obcy|
|<u>IDJezyk</u>| Unikalne ID języka, w którym prowadzony jest webinar | Klucz obcy |
|Cena| Koszt webinaru|> 0|
|DataRozpoczecia| Data rozpoczęcia webinaru| < DataZakonczenia <br> > `1/1/2024` |
|DataZakonczenia| Data zakończenia webinaru| > DataRozpoczecia|
|Url|Link do zapisanego nagrania| LIKE `http://%` OR LIKE `https://%` |
```sql
CREATE TABLE [dbo].[Webinar](
	[IDWebinar] [int] NOT NULL,
	[Cena] [money] NULL,
	[DataRozpoczecia] [datetime] NOT NULL,
	[DataZakonczenia] [datetime] NOT NULL,
	[Url] [nchar](255) NOT NULL,
	[IDWykladowca] [int] NOT NULL,
	[IDJezyk] [int] NOT NULL,
	[IDTlumacz] [int] NULL,
	[IDJezyk] [int] NOT NULL,
 CONSTRAINT [PK_Webinar] PRIMARY KEY CLUSTERED 
(
	[IDWebinar] ASC
)
ALTER TABLE [dbo].[Webinar]  WITH CHECK ADD  CONSTRAINT [CK_Webinar_Data] CHECK  (([DataZakonczenia]>[DataRozpoczecia]))
GO
ALTER TABLE [dbo].[Webinar] CHECK CONSTRAINT [CK_Webinar_Data]
GO
ALTER TABLE [dbo].[Webinar]  WITH CHECK ADD  CONSTRAINT [CK_Webinar_DataRozpoczecia_Min] CHECK  (([DataRozpoczecia]>'2024-01-01'))
GO
ALTER TABLE [dbo].[Webinar] CHECK CONSTRAINT [CK_Webinar_DataRozpoczecia_Min]
GO
ALTER TABLE [dbo].[Webinar]  WITH CHECK ADD  CONSTRAINT [CK_Webinar_Url] CHECK  (([Url] like 'https://%' OR [Url] like 'http://%'))
GO
ALTER TABLE [dbo].[Webinar] CHECK CONSTRAINT [CK_Webinar_Url]
GO
```

### ***Kurs***
Zawiera informacje o każdym kursie, który się odbył lub jeszcze nie.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDKurs**</u>|Unikalne ID kursu|**Klucz główny**|
|Cena|Koszt kursu|> 0|
|DataRozpoczecia|Data rozpoczęcia kursu|< DataZakonczenia <br> > `2024-01-01`|
|DataZakonczenia|Data zakończenia kursu|> DataRozpoczecia|
```sql
CREATE TABLE [dbo].[Kurs](
	[IDKurs] [int] NOT NULL,
	[Cena] [money] NOT NULL,
	[DataRozpoczecia] [datetime] NOT NULL,
	[DataZakonczenia] [datetime] NOT NULL,
 CONSTRAINT [PK_Kurs] PRIMARY KEY CLUSTERED 
(
	[IDKurs] ASC
)

ALTER TABLE [dbo].[Kurs]  WITH CHECK ADD  CONSTRAINT [CK_Kurs_Cena] CHECK  (([Cena]>(0)))
GO
ALTER TABLE [dbo].[Kurs] CHECK CONSTRAINT [CK_Kurs_Cena]
GO
ALTER TABLE [dbo].[Kurs]  WITH CHECK ADD  CONSTRAINT [CK_Kurs_Data] CHECK  (([DataZakonczenia]>[DataRozpoczecia]))
GO
ALTER TABLE [dbo].[Kurs] CHECK CONSTRAINT [CK_Kurs_Data]
GO
ALTER TABLE [dbo].[Kurs]  WITH CHECK ADD  CONSTRAINT [CK_Kurs_DataRozpoczecia_Min] CHECK  (([DataRozpoczecia]>'2024-01-01'))
GO
ALTER TABLE [dbo].[Kurs] CHECK CONSTRAINT [CK_Kurs_DataRozpoczecia_Min]
GO
```

### ***Studium***
Zawiera informacje o każdym kierunku studiów, które się odbywają.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDStudium**</u>| Unikalne ID studium|**Klucz główny**|
|Wpisowe        | Koszt studiów                    | > 0                              |
| LimitMiejsc    | Limit miejsc na kierunku        | > 0                              |
| DataRozpoczecia | Data rozpoczęcia studium       | < DataZakonczenia <br> > `2024-01-01` |
| DataZakonczenia | Data zakończenia studium       | > DataRozpoczecia               |
| CenaZjazdu     | Cena zjazdu dla studenta       | > 0                              |
| CenaZjazduObcy | Cena zjazdu dla osoby niezapisanej na dane studium | > CenaZjazdu  |
```sql
CREATE TABLE [dbo].[Studium](
	[IDStudium] [int] NOT NULL,
	[LimitMiejsc] [int] NOT NULL,
	[Wpisowe] [money] NOT NULL,
	[DataRozpoczecia] [date] NOT NULL,
	[DataZakonczenia] [date] NOT NULL,
	[CenaZjazdu] [date] NOT NULL,
	[CenaZjazduObcy] [date] NOT NULL,
 CONSTRAINT [PK_Studium] PRIMARY KEY CLUSTERED 
(
	[IDStudium] ASC
)

ALTER TABLE [dbo].[Studium]  WITH CHECK ADD  CONSTRAINT [CK_Studium_CenaZjazdu] CHECK  (([CenaZjazdu]>(0)))
GO
ALTER TABLE [dbo].[Studium] CHECK CONSTRAINT [CK_Studium_CenaZjazdu]
GO
ALTER TABLE [dbo].[Studium]  WITH CHECK ADD  CONSTRAINT [CK_Studium_CenaZjazduObcy] CHECK  (([CenaZjazduObcy]>[CenaZjazdu]))
GO
ALTER TABLE [dbo].[Studium] CHECK CONSTRAINT [CK_Studium_CenaZjazduObcy]
GO
ALTER TABLE [dbo].[Studium]  WITH CHECK ADD  CONSTRAINT [CK_Studium_Data] CHECK  (([DataZakonczenia]>[DataRozpoczecia]))
GO
ALTER TABLE [dbo].[Studium] CHECK CONSTRAINT [CK_Studium_Data]
GO
ALTER TABLE [dbo].[Studium]  WITH CHECK ADD  CONSTRAINT [CK_Studium_DataRozpoczecia_Min] CHECK  (([DataRozpoczecia]>'2024-01-01'))
GO
ALTER TABLE [dbo].[Studium] CHECK CONSTRAINT [CK_Studium_DataRozpoczecia_Min]
GO
ALTER TABLE [dbo].[Studium]  WITH CHECK ADD  CONSTRAINT [CK_Studium_LimitMiejsc] CHECK  (([LimitMiejsc]>(0)))
GO
ALTER TABLE [dbo].[Studium] CHECK CONSTRAINT [CK_Studium_LimitMiejsc]
GO
ALTER TABLE [dbo].[Studium]  WITH CHECK ADD  CONSTRAINT [CK_Studium_Wpisowe] CHECK  (([Wpisowe]>(0)))
GO
ALTER TABLE [dbo].[Studium] CHECK CONSTRAINT [CK_Studium_Wpisowe]
GO
```

### ***Uzytkownicy***
Zawiera informacje o każdym użytkowniku systemu.  
|Pole|Opis|Klucze i ograniczenia| 
|-|-|-|
|<u>**IDUzytkownik**</u>|Unikalne ID użytkownika|**Klucz główny**|
|Imie|Imię użytkownika|-|
|Nazwisko|Nazwisko użytkownika|-|
|Zdjecie|Zdjęcie użytkownika|-|

```sql
CREATE TABLE [dbo].[Uzytkownicy](
	[IDUzytkownik] [int] NOT NULL,
	[Imie] [nchar](50) NOT NULL,
	[Nazwisko] [nchar](50) NOT NULL,
	[Zdjecie] [nchar](255) NULL,
 CONSTRAINT [PK_Uzytkownicy] PRIMARY KEY CLUSTERED 
(
	[IDUzytkownik] ASC
)
)
```

### ***Kursant***
Zawiera informacje o każdym kursancie, który korzysta z usług szkoły. 
|Pole|Opis|Klucze i ograniczenia| 
|-|-|-|
|<u>**IDKursant**</u>|Unikalne ID kursanta|**Klucz główny**|
|<u>IDMiasto</u>|Miejsce zamieszkania kursanta|Klucz obcy|
|Ulica|Ulica, przy której mieszka kursant|-|
|KodPocztowy|Kod pocztowy|-|
|Telefon|Numer telefonu kursanta|LIKE `[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]` OR `[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]`
|Email|Email kursanta|UNIQUE <br>LIKE `%@%`|

```sql
CREATE TABLE [dbo].[Kursant](
	[IDKursant] [int] NOT NULL,
	[IDMiasto] [int] NOT NULL,
	[Ulica] [nchar](100) NOT NULL,
	[KodPocztowy] [nchar](20) NOT NULL,
	[Telefon] [nchar](20) NOT NULL,
	[Email] [nchar](50) NOT NULL,
 CONSTRAINT [PK_Kursant] PRIMARY KEY CLUSTERED 
)
 CONSTRAINT [Unique_Kursant_Email] UNIQUE NONCLUSTERED 
(
	[Email] ASC
)

ALTER TABLE [dbo].[Kursant]  WITH CHECK ADD  CONSTRAINT [CK_Kursant_Email] CHECK  (([Email] like '%@%'))
GO
ALTER TABLE [dbo].[Kursant] CHECK CONSTRAINT [CK_Kursant_Email]
GO
ALTER TABLE [dbo].[Kursant]  WITH CHECK ADD  CONSTRAINT [CK_Kursant_Telefon] CHECK  (([Telefon] like '%[^0-9]%'))
GO
ALTER TABLE [dbo].[Kursant] CHECK CONSTRAINT [CK_Kursant_Telefon]
GO
```

### ***Uzytkownicy Role***
Zawiera informacje o tym, jakie dany użytkownik posiada role w systemie.
<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucz</b></td>
  </tr>
  <tr>
    <td><b><u>IDUzytkownik</u></b></td>
    <td>Unikalne ID użytkownika</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDRola</u></b></td>
    <td>ID roli użytkownika<td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Uzytkownicy Role](
	[IDUzytkownik] [int] NOT NULL,
	[IDRola] [int] NOT NULL,
 CONSTRAINT [PK_Uzytkownicy Role] PRIMARY KEY CLUSTERED 
(
	[IDUzytkownik] ASC,
	[IDRola] ASC
)
)
```

### ***Role Uzytkownikow***
Zawiera informacje o każdym kierunku kursancie, który korzysta z usług szkoły.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDRola**</u>|Unikalne ID roli użytkownika|**Klucz główny**|
|Nazwa|ID roli użytkownika|`UNIQUE`|

```sql
CREATE TABLE [dbo].[Role Uzytkownikow](
	[IDRola] [int] NOT NULL,
	[Nazwa] [nchar](50) NOT NULL,
 CONSTRAINT [PK_Role Uzytkownikow] PRIMARY KEY CLUSTERED 
(
	[IDRola] ASC
)
 CONSTRAINT [Unique_Role_Uzytkownikow_Nazwa] UNIQUE NONCLUSTERED 
(
	[Nazwa] ASC
)
)
```

### ***Moduly***
Zawiera informacje dotyczące modułów kursów.  
|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDModul**</u>|Unikalne ID modułu|**Klucz główny**|
|<u>IDKurs</u>|Unikalne ID kursu, którego częścią jest dany moduł|Klucz obcy|
|<u>IDRolaWykladowca</u>|Unikalne ID roli użytkownika (wykładowcy)|Klucz obcy|
|<u>IDWykladowca</u>|Unikalne ID prowadzącego moduł|Klucz obcy|
|Nazwa|Nazwa kursu|-|
|Opis|Opis modułu|-|

```sql
CREATE TABLE [dbo].[Moduly](
	[IDModul] [int] NOT NULL,
	[IDKurs] [int] NOT NULL,
	[Nazwa] [nvarchar](50) NOT NULL,
	[Opis] [nvarchar](255) NOT NULL,
	[IDWykladowca] [int] NOT NULL,
	[IDRolaWykladowca] [int] NOT NULL,
 CONSTRAINT [PK_Moduły_1] PRIMARY KEY CLUSTERED 
(
	[IDModul] ASC
)
)
```

### ***Moduly Spotkania***
Zawiera identyfikatory modułów kursów i identyfikatory spotkań, a których składają się te moduły.  
<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucz</b></td>
  </tr>
  <tr>
    <td><b><u>IDSpotkanie</u></b></td>
    <td>Unikalne ID spotkania</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDModul</u></b></td>
    <td>Unikalne ID modułu, którego częścią jest dane spotkanie<td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Moduly Spotkania](
	[IDSpotkanie] [int] NOT NULL,
	[IDModul] [int] NOT NULL,
 CONSTRAINT [PK_Moduły Spotkania] PRIMARY KEY CLUSTERED 
)
```

### ***Moduly Obecnosc***
Zawiera identyfikatory spotkań, ich daty i identyfikatory kursantów, którzy byli obecni na tych spotkaniach.  
<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucze i ograniczenia</b></td>
  </tr>
  <tr>
    <td><b><u>IDSpotkanie</u></b></td>
    <td>Unikalne ID spotkania</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDKursant</u></b></td>
    <td>Unikalne ID kursanta, który uczestniczy w danym spotkaniu<td>
  </tr>
  <tr>
    <td>Data</td>
    <td>Termin spotkania</td>
    <td><code>DEFAULT: getDate()</code></td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Moduly Obecnosc](
	[IDSpotkanie] [int] NOT NULL,
	[IDKursant] [int] NOT NULL,
	[Data] [datetime] NOT NULL,
CONSTRAINT [PK_Moduly Obecnosc] PRIMARY KEY CLUSTERED 
(
	[IDSpotkanie] ASC,
	[IDKursant] ASC
)
)

ALTER TABLE [dbo].[Moduly Obecnosc] ADD  CONSTRAINT [DF_Moduly Obecnosc_Data]  DEFAULT (getdate()) FOR [Data]
GO
```

### ***Spotkanie***
Zawiera identyfikator spotkania i identyfikator tłumacza, który je tłumaczy.
|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDSpotkanie**</u>|Unikalne ID spotkania|**Klucz główny**|
|<u>IDTlumacz</u>|Unikalne ID tłumacza, który tłumaczy dane spotkanie|Klucz obcy|

```sql
CREATE TABLE [dbo].[Spotkanie](
	[IDSpotkanie] [int] NOT NULL,
	[IDTlumacz] [int] NULL,
 CONSTRAINT [PK_Spotkanie_1] PRIMARY KEY CLUSTERED 
)
```

### ***Spotkanie stacjonarne***
Zawiera informacje na temat spotkania stacjonarnego. 
|Pole|Opis|Klucze i ograniczenia|
|-|-|-| 
|<u>**IDSpotkanie**</u>|Unikalne ID spotkania|**Klucz główny**|
|Sala|Sala, w której odbywa się spotkanie|
|LimitMiejsc|Limit miejsc|> 0|
|DataRozpoczecia|Data rozpoczęcia spotkania|< DataZakonczenia <br>> `2024-01-01`|
|DataZakonczenia|Data zakończenia spotkania|> DataRozpoczecia|

```sql
CREATE TABLE [dbo].[Spotkanie](
	[IDSpotkanie] [int] NOT NULL,
	[IDTlumacz] [int] NULL,
 CONSTRAINT [PK_Spotkanie_1] PRIMARY KEY CLUSTERED 
)

ALTER TABLE [dbo].[Spotkanie stacjonarne]  WITH CHECK ADD  CONSTRAINT [[CK_Spotkanie stacjonarne]_Data] CHECK  (([DataZakonczenia]>[DataRozpoczecia]))
GO
ALTER TABLE [dbo].[Spotkanie stacjonarne] CHECK CONSTRAINT [[CK_Spotkanie stacjonarne]_Data]
GO
ALTER TABLE [dbo].[Spotkanie stacjonarne]  WITH CHECK ADD  CONSTRAINT [[CK_Spotkanie stacjonarne]_LimitMiejsc] CHECK  (([LimitMiejsc]>(0)))
GO
ALTER TABLE [dbo].[Spotkanie stacjonarne] CHECK CONSTRAINT [[CK_Spotkanie stacjonarne]_LimitMiejsc]
GO
```

### ***Spotkanie online synchroniczne***
Zawiera informacje na temat spotkania synchronicznego.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDSpotkanie**</u>|Unikalne ID spotkania|**Klucz główny**|
|DataRozpoczecia|Data rozpoczęcia spotkania|< DataZakonczenia <br>> `2024-01-01’`
|DataZakonczenia|Data zakończenia spotkania
|> DataRozpoczecia|
|Url|Link do nagrania spotkania|LIKE `http://%` OR LIKE `https://%`

```sql
CREATE TABLE [dbo].[Spotkanie online synchroniczne](
	[IDSpotkanie] [int] NOT NULL,
	[DataRozpoczecia] [datetime] NOT NULL,
	[DataZakonczenia] [datetime] NOT NULL,
	[Url] [nchar](255) NOT NULL,
 CONSTRAINT [PK_Spotkanie online synchroniczne] PRIMARY KEY CLUSTERED 
)

ALTER TABLE [dbo].[Spotkanie online synchroniczne]  WITH CHECK ADD  CONSTRAINT [[CK_Spotkanie online synchroniczne]_Data] CHECK  (([DataZakonczenia]>[DataRozpoczecia]))
GO
ALTER TABLE [dbo].[Spotkanie online synchroniczne] CHECK CONSTRAINT [[CK_Spotkanie online synchroniczne]_Data]
GO
ALTER TABLE [dbo].[Spotkanie online synchroniczne]  WITH CHECK ADD  CONSTRAINT [[CK_Spotkanie online synchroniczne]_Url] CHECK  (([Url] LIKE 'https://%' OR [Url] LIKE 'http://%'))
GO
ALTER TABLE [dbo].[Spotkanie online synchroniczne] CHECK CONSTRAINT [[CK_Spotkanie online synchroniczne]_Url]
GO
```

### ***Spotkanie online asynchroniczne***
Zawiera informacje na temat spotkania asynchronicznego.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDSpotkanie**</u>|Unikalne ID spotkania|**Klucz główny**|
|Url|Link do nagrania spotkania|LIKE `http://%` OR LIKE `https://%`|

```sql
CREATE TABLE [dbo].[Spotkanie online asynchroniczne](
	[IDSpotkanie] [int] NOT NULL,
	[Url] [nchar](255) NOT NULL,
 CONSTRAINT [PK_Spotkanie online asynchroniczne] PRIMARY KEY CLUSTERED 
)
ALTER TABLE [dbo].[Spotkanie online asynchroniczne]  WITH CHECK ADD  CONSTRAINT [[CK_Spotkanie online asynchroniczne]_Url] CHECK  (([Url] LIKE 'https://%' OR [Url] LIKE 'http://%'))
GO
ALTER TABLE [dbo].[Spotkanie online asynchroniczne] CHECK CONSTRAINT [[CK_Spotkanie online asynchroniczne]_Url]
GO
```

### ***Spotkanie Sala***
Zawiera identyfikator sali i jej nazwę.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDSali**</u>|Unikalne ID sali|**Klucz główny**|
|Nazwa|Unikalna nazwa sali|Klucz obcy|

```sql
CREATE TABLE [dbo].[Spotkanie Sala](
	[IDSali] [int] NOT NULL,
	[Nazwa] [nchar](10) NOT NULL,
 CONSTRAINT [PK_Spotkanie Sala] PRIMARY KEY CLUSTERED 
(
	[IDSali] ASC
)
 CONSTRAINT [Unique_Spotkanie_Sala_Nazwa] UNIQUE NONCLUSTERED 
(
	[Nazwa] ASC
)
)
```

### ***Przedmioty***
Zawiera informacje na temat przedmiotu będącego częścią danego studium.  
|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDPrzedmiot**</u>|Unikalne ID przedmiotu|**Klucz główny**|
|<u>IDStudium</u>|ID kierunku, na którym jest dany przedmiot|Klucz obcy|
|<u>IDWykladowca</u>|ID prowadzącego przedmiot|Klucz obcy|
|<u>IDSemestr</u>|ID semestru, na którym jest dany przedmiot|Klucz obcy|
|Nazwa|Nazwa przedmiotu|-|
|Opis|Opis przedmiotu|-|

```sql
CREATE TABLE [dbo].[Przedmioty](
	[IDPrzedmiot] [int] NOT NULL,
	[IDStudium] [int] NOT NULL,
	[Nazwa] [nchar](50) NOT NULL,
	[Opis] [text] NOT NULL,
	[IDWykladowca] [int] NOT NULL,
	[IDSemestr] [int] NOT NULL,
 CONSTRAINT [PK_Przedmioty] PRIMARY KEY CLUSTERED 
)
```

### ***Przedmioty Spotkania***
Zawiera informacje na temat spotkań odbywających się w ramach danego przedmiotu.  
|<u>**IDSpotkanie**</u>|Unikalne ID spotkania|**Klucz główny**|
|<u>IDPrzedmiot</u>|Unikalne ID przedmiotu|Klucz obcy|

```sql
CREATE TABLE [dbo].[Przedmioty Spotkania](
	[IDSpotkanie] [int] NOT NULL,
	[IDPrzedmiot] [int] NOT NULL,
 CONSTRAINT [PK_Spotkanie] PRIMARY KEY CLUSTERED 
(
	[IDSpotkanie] ASC
)
)
```

### ***Przedmioty Obecnosc***
Zawiera informacje na temat obecności kursantów na spotkaniach odbywających się w ramach danego przedmiotu.
<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucze i ograniczenia</b></td>
  </tr>
  <tr>
    <td><b><u>IDSpotkanie</u></b></td>
    <td>Unikalne ID spotkania</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDKursant</u></b></td>
    <td>Unikalne ID kursanta, który uczestniczy w danym spotkaniu<td>
  </tr>
  <tr>
    <td>Data</td>
    <td>Termin spotkania</td>
    <td><code>DEFAULT: getDate()</code></td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Przedmioty Obecnosc](
	[IDSpotkanie] [int] NOT NULL,
	[IDKursant] [int] NOT NULL,
	[Data] [datetime] NOT NULL,
CONSTRAINT [PK_Przedmioty Obecnosc] PRIMARY KEY CLUSTERED 
(
	[IDSpotkanie] ASC,
	[IDKursant] ASC
)

ALTER TABLE [dbo].[Przedmioty Obecnosc] ADD  CONSTRAINT [DF_Przedmioty Obecnosc_Data]  DEFAULT (getdate()) FOR [Data]
```

### ***Przedmioty Zaliczenia***
Zawiera informacje na temat studentów, którzy zaliczyli (CzyZaliczyl = 1), bądź nie zaliczyli (CzyZaliczyl = 0) danego przedmiotu.  
<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucz</b></td>
  </tr>
  <tr>
    <td><b><u>IDPrzedmiot</u></b></td>
    <td>Unikalne ID przedmiotu</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDKursant</u></b></td>
    <td>Unikalne ID kursanta zapisanego na przedmiot<td>
  </tr>
  <tr>
    <td>CzyZaliczyl</td>
    <td>Wiadomość czy kursant zaliczył przedmiot (0 jeśli nie, 1 jeśli tak)</td>
    <td>-</td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Przedmioty Zaliczenia](
	[IDPrzedmiot] [int] NOT NULL,
	[IDKursant] [int] NOT NULL,
	[CzyZaliczyl] [bit] NOT NULL
CONSTRAINT [PK_Przedmioty Zaliczenia] PRIMARY KEY CLUSTERED 
(
	[IDPrzedmiot] ASC,
	[IDKursant] ASC
)
)
```

### ***Semestr***
Zawiera identyfikator semestru i jego nazwę.  
|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDSemestr**</u>|Unikalne ID semestru|**Klucz główny**|
|Nazwa|Nazwa semestru|-|

```sql
CREATE TABLE [dbo].[Semestr](
	[IDSemestr] [int] NOT NULL,
	[Nazwa] [nchar](10) NOT NULL,
 CONSTRAINT [PK_Semestr] PRIMARY KEY CLUSTERED 
(
	[IDSemestr] ASC
)
)
```

### ***Zjazd***
Zawiera informacje na temat zjazdów.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDZjazd**</u>|Unikalne ID zjazdu|**Klucz główny**|
|<u>IDStudium</u>|Unikalne ID studium, którego częścią jest zjazd|Klucz obcy|
|DataRozpoczecia|Data rozpoczęcia spotkania|> `2024-01-01` <br>< DataZakonczenia|
|DataZakonczenia|Data zakończenia spotkania|> DataRozpoczecia|

```sql
CREATE TABLE [dbo].[Zjazd](
	[IDZjazd] [int] NOT NULL,
	[IDStudium] [int] NOT NULL,
	[DataRozpoczecia] [datetime] NOT NULL,
	[DataZakonczenia] [datetime] NOT NULL,
 CONSTRAINT [PK_Zjazd] PRIMARY KEY CLUSTERED 
(
	[IDZjazd] ASC
)
)

ALTER TABLE [dbo].[Zjazd]  WITH CHECK ADD  CONSTRAINT [CK_Zjazd_Data] CHECK  (([DataZakonczenia]>[DataRozpoczecia]))
GO
ALTER TABLE [dbo].[Zjazd] CHECK CONSTRAINT [CK_Zjazd_Data]
GO
ALTER TABLE [dbo].[Zjazd]  WITH CHECK ADD  CONSTRAINT [CK_Zjazd_DataRozpoczecia_Min] CHECK  (([DataRozpoczecia]>'2024-01-01'))
GO
ALTER TABLE [dbo].[Zjazd] CHECK CONSTRAINT [CK_Zjazd_DataRozpoczecia_Min]
GO
```

### ***Zjazdy Spotkania***
Zawiera informacje na temat spotkań odbywających się w ramach danego zjazdu.  
<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucz</b></td>
  </tr>
  <tr>
    <td><b><u>IDZjazd</u></b></td>
    <td>Unikalne ID zjazdu</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDSpotkania</u></b></td>
    <td>Unikalne ID spotkania na zjeździe<td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Zjazdy Spotkania](
	[IDZjazd] [int] NOT NULL,
	[IDSpotkanie] [int] NOT NULL
CONSTRAINT [PK_Zjazdy Spotkania] PRIMARY KEY CLUSTERED 
(
	[IDZjazd] ASC,
	[IDSpotkanie] ASC
)
)
```

### ***Praktyki***
Zawiera informacje na temat praktyk.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDPraktyk**</u>|Unikalne ID praktyk|**Klucz główny**|
|<u>IDKursant</u>|Unikalne ID kursanta biorącego udział w praktykach|Klucz obcy|
|<u>IDFirmy</u>|Unikalne ID firmy, która wykupiła praktyki|Klucz obcy|
|<u>IDStudium</u>|Unikalne ID studium, którego częścią są praktyki|Klucz obcy|
|<u>IDSemestr</u>|Unikalne ID semestru, na którym są praktyki|Klucz obcy|
|DataRozpoczecia|Data rozpoczęcia praktyk|> `2024-01-01` <br>< DataZakonczenia|
|DataZakonczenia|Data zakończenia praktyk|> DataRozpoczecia|

```sql
CREATE TABLE [dbo].[Praktyki](
	[IDPraktyk] [int] NOT NULL,
	[IDKursant] [int] NOT NULL,
	[IDFirmy] [int] NOT NULL,
	[DataRozpoczecia] [datetime] NOT NULL,
	[DataZakonczenia] [datetime] NOT NULL,
	[IDStudium] [int] NOT NULL,
	[IDSemestr] [int] NOT NULL,
 CONSTRAINT [PK_Praktyki] PRIMARY KEY CLUSTERED 
)

ALTER TABLE [dbo].[Praktyki]  WITH CHECK ADD  CONSTRAINT [CK_Praktyki_Data] CHECK  (([DataZakonczenia]>[DataRozpoczecia]))
GO
ALTER TABLE [dbo].[Praktyki] CHECK CONSTRAINT [CK_Praktyki_Data]
GO
ALTER TABLE [dbo].[Praktyki]  WITH CHECK ADD  CONSTRAINT [CK_Praktyki_DataRozpoczecia_Min] CHECK  (([DataRozpoczecia]>'2024-01-01'))
GO
ALTER TABLE [dbo].[Praktyki] CHECK CONSTRAINT [CK_Praktyki_DataRozpoczecia_Min]
GO
```

### ***Praktyki Obecnosc***
Zawiera informacje na obecności na praktykach.  
<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucze i ograniczenia</b></td>
  </tr>
  <tr>
    <td><b><u>IDPraktyk</u></b></td>
    <td>Unikalne ID praktyk</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td rowspan="2"><b><u>Data</u></b></td>
    <td rowspan="2">Data praktyk<td>
  </tr>
  <tr>
    <td>> <code>1/1/2001</code></td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Praktyki Obecnosc](
	[IDPraktyk] [int] NOT NULL,
	[Data] [datetime] NOT NULL
CONSTRAINT [PK_Praktyki Obecnosc] PRIMARY KEY CLUSTERED 
(
	[IDPraktyk] ASC,
	[Data] ASC
)
)
ALTER TABLE [dbo].[Praktyki Obecnosc]  WITH CHECK ADD  CONSTRAINT [CK_Praktyki Obecnosc] CHECK  (([Data]>'1/1/2001'))
GO
ALTER TABLE [dbo].[Praktyki Obecnosc] CHECK CONSTRAINT [CK_Praktyki Obecnosc]
GO
```

### ***Firma***
Zawiera informacje na temat firm prowadzących praktyki.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDFirma**</u>|Unikalne ID firmy|**Klucz główny**|
|Nazwa|Nazwa firmy|`UNIQUE`|

```sql
CREATE TABLE [dbo].[Firma](
	[IDFirma] [int] NOT NULL,
	[Nazwa] [nchar](50) NOT NULL,
 CONSTRAINT [PK_Firma] PRIMARY KEY CLUSTERED 
(
	[IDFirma] ASC
)
CONSTRAINT [Unique_Firma_Nazwa] UNIQUE NONCLUSTERED 
(
	[Nazwa] ASC
)
)
```

### ***Zamowienia***
Zawiera identyfikatory zamówień, ich status, identyfikator zamawiającego kursanta i datę zamówienia.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDZamowienia**</u>|Unikalne ID zamówienia|**Klucz główny**|
|<u>IDKursant</u>|Unikalne ID kursanta zamawiającego|Klucz obcy|
|<u>IDStatus</u>|Unikalne ID statusu zamówienia|Klucz obcy|
|DataZamowienia|Data zamówienia|`DEFAULT: getDate()`|

```sql
CREATE TABLE [dbo].[Zamowienia](
	[IDZamowienia] [int] NOT NULL,
	[DataZamowienia] [datetime] NULL,
	[IDKursant] [int] NOT NULL,
	[IDStatus] [int] NOT NULL,
 CONSTRAINT [PK_Zamowienia] PRIMARY KEY CLUSTERED 
)

ALTER TABLE [dbo].[Zamowienia] ADD  CONSTRAINT [DF_Zamowienia_DataZamowienia_1]  DEFAULT (getdate()) FOR [DataZamowienia]
GO
```

### ***Zamowienie Status***
Zawiera informacje na temat statusu danych zamówień.  
|Pole|Opis|Klucze i ograniczenia|
|-|-|-|
|<u>**IDStatus**</u>|Unikalne ID statusu|**Klucz główny**|
|Nazwa|Nazwa statusu|`UNIQUE`|

```sql
CREATE TABLE [dbo].[Zamowienie Status](
	[IDStatus] [int] NOT NULL,
	[Nazwa] [nchar](50) NOT NULL,
 CONSTRAINT [PK_Zamowienie Status] PRIMARY KEY CLUSTERED
CONSTRAINT [Unique_ZamowienieStatus_Nazwa] UNIQUE NONCLUSTERED 
(
	[Nazwa] ASC
)
)

ALTER TABLE [dbo].[Zamowienie Status] ADD  CONSTRAINT [DF_Zamowienie Status_Nazwa]  DEFAULT (N'Niezłożone') FOR [Nazwa]
```

### ***Zamowienia Informacje***
Zawiera informacje na temat zamówień spotkań.  

<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucze i ograniczenia</b></td>
  </tr>
  <tr>
    <td><b><u>IDZamowienia</u></b></td>
    <td>ID zamówienia</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDFormyKsztalcenia</u></b></td>
    <td>ID formy kształcenia<td>
  </tr>
  <tr>
    <td>DoZaplaty</td>
    <td>Ile zostało do zapłacenia</td>
    <td>>= Zaplacono</td>
  </tr>
  <tr>
    <td>Zaplacono</td>
    <td>Ile już zapłacono</td>
    <td>>= 0 <br><code>DEFAULT: 0</code></td>
  </tr>
  <tr>
    <td>DataWplaty</td>
    <td>Data wpłaty</td>
    <td><code>DEFAULT: getDate()</code></td>
  </tr>
  <tr>
    <td>CzyZaliczka</td>
    <td>Czy zaliczka została już opłacona (1 - Tak, 0 - Nie)</td>
    <td><code>DEFAULT: 0</code></td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Zamowienia Informacje](
	[IDZamowienia] [int] NOT NULL,
	[IDFormyKsztalcenia] [int] NOT NULL,
	[Zaplacono] [money] NOT NULL,
	[DoZaplaty] [money] NOT NULL,
	[DataWplaty] [datetime] NOT NULL
)

ALTER TABLE [dbo].[Zamowienia Informacje] ADD  CONSTRAINT [DF_Zamowienia Informacje_Zaplacono]  DEFAULT ((0)) FOR [Zaplacono]
GO
ALTER TABLE [dbo].[Zamowienia Informacje] ADD  CONSTRAINT [DF_Zamowienia Informacje_CzyZaliczka]  DEFAULT ((0)) FOR [CzyZaliczka]
GO
ALTER TABLE [dbo].[Zamowienia Informacje] ADD  CONSTRAINT [DF_Zamowienia Informacje_DataWplaty]  DEFAULT (getdate()) FOR [DataWplaty]
GO
ALTER TABLE [dbo].[Zamowienia Informacje]  WITH CHECK ADD  CONSTRAINT [[CK_Zamowienia Informacje]_Zaplacono] CHECK  (([Zaplacono]>=(0)))
GO
ALTER TABLE [dbo].[Zamowienia Informacje] CHECK CONSTRAINT [[CK_Zamowienia Informacje]_Zaplacono]
GO
ALTER TABLE [dbo].[Zamowienia Informacje]  WITH CHECK ADD  CONSTRAINT [[CK_Zamowienia Informacje]_Dlug] CHECK  (([DoZaplaty]>=[Zaplacono]))
GO
ALTER TABLE [dbo].[Zamowienia Informacje] CHECK CONSTRAINT [[CK_Zamowienia Informacje]_Dlug]
GO
```

### ***Miasto***
Zawiera informacje na temat miast, w których mieszkają kursanci.  
|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDMiasto**</u>|Unikalne ID miasta|**Klucz główny**|
|<u>IDKraj</u>|Unikalne ID kraju|Klucz obcy|
|Nazwa|Nazwa miasta|-|

```sql
CREATE TABLE [dbo].[Miasto](
	[IDMiasto] [int] NOT NULL,
	[Nazwa] [nchar](100) NOT NULL,
	[IDKraj] [int] NOT NULL,
 CONSTRAINT [PK_Miasto] PRIMARY KEY CLUSTERED 
(
	[IDMiasto] ASC
)
)
```

### ***Kraj***
Zawiera informacje na temat krajów, w których mieszkają kursanci.

|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDKraj**</u>|Unikalne ID kraju|**Klucz główny**|
|Nazwa|Nazwa kraju|`UNIQUE`|

```sql
CREATE TABLE [dbo].[Kraj](
	[IDKraj] [int] NOT NULL,
	[Nazwa] [nchar](50) NOT NULL,
 CONSTRAINT [PK_Kraj] PRIMARY KEY CLUSTERED 
(
	[IDKraj] ASC
)
CONSTRAINT [Unique_Kraj_Nazwa] UNIQUE NONCLUSTERED 
(
	[Nazwa] ASC
)
)
```

### ***Jezyk***
Zawiera informacje na temat języków, w których prowadzone są spotkania.

|Pole|Opis|Klucz|
|-|-|-|
|<u>**IDJezyk**</u>|Unikalne ID języka|**Klucz główny**|
|Nazwa|Nazwa języka|`UNIQUE`|

```sql
CREATE TABLE [dbo].[Jezyk](
	[IDJezyk] [int] NOT NULL,
	[Nazwa] [nchar](50) NOT NULL,
 CONSTRAINT [PK_Jezyk] PRIMARY KEY CLUSTERED 
)
(
	[IDJezyk] ASC
)
CONSTRAINT [Unique_Jezyk_Nazwa] UNIQUE NONCLUSTERED 
(
	[Nazwa] ASC
)
```

### ***Tlumacz Jezyki***
Zawiera identyfikatory tłumaczy i języków.

<table>
  <tr>
    <td><b>Pole</b></td>
    <td><b>Opis</b></td>
    <td><b>Klucz</b></td>
  </tr>
  <tr>
    <td><b><u>IDJezyk</u></b></td>
    <td>Unikalne ID języka</td>
    <td rowspan="2"><b>Klucz główny złożony</b></td>
  </tr>
  <tr>
    <td><b><u>IDTlumacz</u></b></td>
    <td>Unikalne ID tłumacza<td>
  </tr>
</table>

```sql
CREATE TABLE [dbo].[Tlumacz Jezyki](
	[IDTlumacz] [int] NOT NULL,
	[IDJezyk] [int] NOT NULL
CONSTRAINT [PK_Tlumacz Jezyki] PRIMARY KEY CLUSTERED 
(
	[IDTlumacz] ASC,
	[IDJezyk] ASC
)
)
```

