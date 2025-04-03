# Procedury
### ***DodajWebinar***
Procedura pozwalająca dodać nowy webinar z deklaracją: nazwy, ceny, daty rozpoczęcia, daty zakończenia, linku do nagrania, identyfikatora wykładowcy, który go prowadzi, identyfikatora tłumacza, który go tłumaczy.
Procedura automatycznie dodaje nowy rekord do tabeli Formy Kształcenia.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajWebinar](
	@Nazwa NVARCHAR(50),
    	@Cena MONEY = NULL,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME,
	@Url NVARCHAR(255),
	@IDWykladowca INT,
	@IDTlumacz INT = NULL
	)
AS
BEGIN
	SET NOCOUNT ON
	DECLARE @IDFormyKsztalcenia INT
	BEGIN TRY
		IF dbo.KolizjaWykladowcow(@IDWykladowca, @DataRozpoczecia, @DataZakonczenia) = 1
			THROW 51000, 'Kolizja wykładowców', 1

		IF dbo.KolizjaTlumaczy(@IDTlumacz, @DataRozpoczecia, @DataZakonczenia) = 1
			THROW 51000, 'Kolizja tłumaczy', 1

		SELECT @IDFormyKsztalcenia = ISNULL(MAX(IDFormyKsztalcenia), 0) + 1 FROM  [Forma Ksztalcenia]
		INSERT INTO [Forma Ksztalcenia] values (@IDFormyKsztalcenia, @Nazwa)
		INSERT INTO Webinar VALUES (@IDFormyKsztalcenia, @Cena, @DataRozpoczecia, @DataZakonczenia, @Url, @IDWykladowca, @IDTlumacz)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu webinaru: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajKurs***
Procedura pozwalająca dodać nowy kurs z deklaracją: nazwy, ceny, daty rozpoczęcia i daty zakończenia.
Procedura automatycznie dodaje nowy rekord do tabeli Formy Kształcenia.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajKurs](
	@Nazwa NVARCHAR(50),
    	@Cena MONEY,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME
	)
AS
BEGIN
	BEGIN TRY
		SET NOCOUNT ON
		DECLARE @IDFormyKsztalcenia INT
		SELECT @IDFormyKsztalcenia = ISNULL(MAX(IDFormyKsztalcenia), 0) + 1 FROM  [Forma Ksztalcenia]
		INSERT INTO [Forma Ksztalcenia] VALUES (@IDFormyKsztalcenia, @Nazwa)
		INSERT INTO Kurs VALUES (@IDFormyKsztalcenia, @Cena, @DataRozpoczecia, @DataZakonczenia)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu kursu: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajStudium***
Procedura pozwalająca dodać nowe studium z deklaracją: nazwy, limitu miejsc, ceny pojedynczego zjazdu, cena pojedynczego zjazdu dla osób spoza studium, wpisowego, daty rozpoczęcia i daty zakończenia.
Procedura automatycznie dodaje nowy rekord do tabeli Formy Kształcenia.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajStudium](
    @Nazwa NCHAR(50),
	@LimitMiejsc INT,
	@CenaZjazdu MONEY,
	@CenaZjazduObcy MONEY,
	@Wpisowe MONEY,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME
	)
AS
BEGIN
	DECLARE @IDFormyKsztalcenia INT
    	BEGIN TRY
		SELECT @IDFormyKsztalcenia = ISNULL(MAX(IDFormyKsztalcenia), 0) + 1 FROM  [Forma Ksztalcenia]
		INSERT INTO [Forma Ksztalcenia] VALUES (@IDFormyKsztalcenia, @Nazwa)
        		INSERT INTO Studium VALUES (@IDFormyKsztalcenia, @LimitMiejsc, @CenaZjazdu, @CenaZjazduObcy, @Wpisowe, @DataRozpoczecia, @DataZakonczenia)
   	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania studium: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
   	END CATCH
END
```

### ***DodajModul***
Procedura pozwalająca dodać nowy moduł do kursu z deklaracją: identyfikatora kursu, nazwy, opisu i identyfikatora wykładowcy, który prowadzi zajęcia w tym module.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajModul](
	@IDKurs INT,
	@Nazwa NVARCHAR(50),
	@Opis NVARCHAR(255),
	@IDWykladowca INT
	)
AS
BEGIN
	SET NOCOUNT ON
	BEGIN TRY
		DECLARE @IDModul INT
		SELECT @IDModul = ISNULL(MAX(IDModul), 0) + 1 FROM Moduly
		INSERT INTO Moduly VALUES (@IDModul, @IDKurs, @Nazwa, @Opis, @IDWykladowca)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu modułu: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajPrzedmiot***
Procedura pozwalająca dodać nowy przedmiot z deklaracją: identyfikatora studium, w ramach którego się odbywa, nazwy, opisu, identyfikatora wykładowcy, który prowadzi zajęcia w tym przedmiocie oraz numeru semestru, w którym się on odbywa.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajPrzedmiot](
    	@IDStudium INT,
	@Nazwa NCHAR(50),
	@Opis TEXT,
	@IDWykladowca INT,
	@IDSemestr INT
	)
AS
BEGIN
	DECLARE @IDPrzedmiot INT
   	BEGIN TRY
		SELECT @IDPrzedmiot = ISNULL(MAX(IDPrzedmiot), 0) + 1 FROM Przedmioty
        		INSERT INTO Przedmioty VALUES (@IDPrzedmiot, @IDStudium, @Nazwa, @Opis, @IDWykladowca, @IDSemestr)
    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania przedmiotu: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
   	END CATCH
END
```

### ***DodajPraktyki***
Procedura pozwalająca dodać nowe praktyki z deklaracją: identyfikatora kursanta odbywającego praktyki, identyfikatora firmy, w której są one przeprowadzane, daty rozpoczęcia daty zakończenia oraz identyfikatora studium, w ramach którego się odbywają.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajPraktyki](
	@IDKursant INT,
    	@IDFirmy INT,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME,
	@IDStudium INT
	)
AS
BEGIN
	DECLARE @IDPraktyk INT
    	BEGIN TRY
		IF NOT EXISTS (SELECT * FROM Studium WHERE IDStudium = @IDStudium AND DataRozpoczecia <= @DataRozpoczecia AND DataZakonczenia >= @DataZakonczenia)
			THROW 51000, 'Nie znaleziono danego studium w tym przedziale czasowym', 1

		IF EXISTS (SELECT * FROM Praktyki WHERE IDKursant = @IDKursant AND ((@DataRozpoczecia < DataZakonczenia AND @DataZakonczenia > DataRozpoczecia) OR (@DataRozpoczecia > DataRozpoczecia AND @DataZakonczenia < DataZakonczenia)))
			THROW 51000, 'Kolizja praktyk', 1

		IF (SELECT COUNT(*) FROM Praktyki WHERE ((YEAR(DataRozpoczecia) = YEAR(@DataRozpoczecia) AND MONTH(@DataRozpoczecia) < 10 AND MONTH(DataRozpoczecia) < 10) OR (YEAR(DataRozpoczecia) + 1 = YEAR(@DataRozpoczecia) AND MONTH(@DataRozpoczecia) < 10 AND MONTH(DataRozpoczecia) >= 10))
				OR ((YEAR(DataRozpoczecia) = @DataRozpoczecia AND MONTH(@DataRozpoczecia) >= 10 AND MONTH(DataRozpoczecia) >= 10) OR (YEAR(DataRozpoczecia) - 1 = YEAR(@DataRozpoczecia) AND MONTH(@DataRozpoczecia) >= 10 AND MONTH(DataRozpoczecia) < 10))) = 2
			THROW 51000, 'Kolizja semestrów', 1
		
		SELECT @IDPraktyk = ISNULL(MAX(IDPraktyk), 0) + 1 FROM Praktyki
		INSERT INTO Praktyki VALUES (@IDPraktyk, @IDKursant, @IDFirmy, @DataRozpoczecia, @DataZakonczenia, @IDStudium)
    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd przy tworzeniu praktyk: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajZjazd***
Procedura pozwalająca dodać nowy zjazd z deklaracją: identyfikatora studium, w ramach którego zjazd się odbywa, daty rozpoczęcia oraz daty zakończenia zjazdu.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajZjazd](
    	@IDStudium INT,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME
)
AS
BEGIN
	DECLARE @IDZjazd INT
	BEGIN TRY
		SELECT @IDStudium
		IF NOT EXISTS (SELECT * FROM Studium)
			THROW 51000, 'Nie ma takiego studium', 1

		IF @DataRozpoczecia IS NOT NULL AND @DataZakonczenia IS NOT NULL
			IF dbo.ZawieraSieWPrzedzialeCzasowym(@DataRozpoczecia, @DataZakonczenia,
				(SELECT S.DataRozpoczecia FROM Studium AS S WHERE S.IDStudium = @IDStudium),
				(SELECT S.DataZakonczenia FROM Studium AS S WHERE S.IDStudium = @IDStudium)) = 0
				THROW 51000, 'Czas trwania zjazdu nie zawiera sie w czasie trwania studiow', 1

		SELECT @IDZjazd = ISNULL(MAX(IDZjazd), 0) + 1 FROM Zjazd
        		INSERT INTO Zjazd VALUES (@IDZjazd, @IDStudium, @DataRozpoczecia, @DataZakonczenia)
	END TRY
	BEGIN CATCH
		DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania zjazdu: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajObecnoscDlaModulu***
Procedura pozwalająca dodać obecność danego kursanta na danym spotkaniu w ramach modułu.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajObecnoscDlaModulu](
    	@IDSpotkanie INT,
	@IDKursant INT
	)
AS
BEGIN
    	BEGIN TRY
        		IF NOT EXISTS (SELECT * FROM [Moduly Spotkania] WHERE IDSpotkanie = @IDSpotkanie)
			THROW 51000, 'Nie ma takiego spotkania', 1

         		IF NOT EXISTS (SELECT * FROM [Moduly Spotkania] AS MS
			JOIN Moduly AS M ON MS.IDModul = M.IDModul
			JOIN WidokOsobZapisanychNaKurs AS WZK ON M.IDKurs = wzk.IDFormyKsztalcenia
			WHERE MS.IDSpotkanie = @IDSpotkanie AND WZK.IDKursant = @IDKursant)
			THROW 51000, 'Nie ma takiego kursanta', 1

INSERT INTO [Moduly Obecnosc] (IDSpotkanie, IDKursant) VALUES (@IDSpotkanie, @IDKursant)
    	END TRY
   	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania obecności dla modułu: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajObecnoscDlaPrzedmiotu***
Procedura pozwalająca dodać obecność danego kursanta na danym spotkaniu w ramach przedmiotu.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajObecnoscDlaPrzedmiotu](
	@IDSpotkanie INT,
	@IDKursant INT
	)
AS
BEGIN
    	BEGIN TRY
        		IF NOT EXISTS (SELECT * FROM [Przedmioty Spotkania] WHERE IDSpotkanie = @IDSpotkanie)
			THROW 51000, 'Nie ma takiego spotkania', 1

		IF NOT EXISTS (SELECT * FROM [Przedmioty Spotkania] AS PS
			JOIN Przedmioty AS P ON PS.IDPrzedmiot = P.IDPrzedmiot
			JOIN WidokOsobZapisanychNaStudium AS WZS ON P.IDStudium = wzs.IDFormyKsztalcenia
			WHERE PS.IDSpotkanie = @IDSpotkanie AND WZS.IDKursant = @IDKursant)
				THROW 51000, 'Nie ma takiego kursanta', 1

		INSERT INTO [Przedmioty Obecnosc] (IDSpotkanie, IDKursant) VALUES (@IDSpotkanie, @IDKursant)
    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania obecności dla przedmiotu: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajObecnoscDlaPraktyk***
Procedura pozwalająca dodać obecność danego kursanta na praktykach.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajObecnoscDlaPraktyk](
	@IDPraktyk INT,
	@Data DATETIME
	)
AS
BEGIN
    	BEGIN TRY
		IF NOT EXISTS (SELECT * FROM Praktyki WHERE IDPraktyk = @IDPraktyk AND @Data NOT BETWEEN DataRozpoczecia AND DataZakonczenia)
			THROW 51000, 'Nie można dodać obecności w tym przedziale czasowym', 1
		IF EXISTS (SELECT * FROM [Praktyki Obecnosc] WHERE DAY(Data) = DAY(@Data) AND MONTH(Data) = MONTH(@Data) AND YEAR(Data) = YEAR(@Data))
			THROW 51000, 'Już dodano obecność w tym dniu', 1
		INSERT INTO [Praktyki Obecnosc] VALUES (@IDPraktyk, @Data)
    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd przy dodawaniu obecności dla praktyk: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajSpotkanieModuluStacjonarnego***
Procedura pozwalająca dodać spotkanie stacjonarne dla modułu deklarując: identyfikator modułu, identyfikator tłumacza, identyfikator sali, limit miejsc, datę rozpoczęcia i datę zakończenia.
Procedura automatycznie dodaje nowe spotkanie do tabeli Spotkanie i tabeli Spotkanie stacjonarne.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajSpotkanieModuluStacjonarnego](
	@IDModul INT,
	@IDTlumacz INT = NULL,
	@IDSali INT,
	@LimitMiejsc INT,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME
	)
AS
BEGIN
	SET NOCOUNT ON
	DECLARE @NastepneIDSpotkanie INT
	BEGIN TRY
		IF (SELECT dbo.KolizjaSpotkanModulu(@IDModul, @DataRozpoczecia, @DataZakonczenia, @IDSali, @IDTlumacz)) = 1
			THROW 51000, 'Kolizcja spotkań', 1

		SELECT @NastepneIDSpotkanie = ISNULL(MAX(IDSpotkanie), 0) + 1 FROM Spotkanie
		INSERT INTO Spotkanie VALUES (@NastepneIDSpotkanie, @IDTlumacz)
		INSERT INTO [Spotkanie stacjonarne] VALUES (@NastepneIDSpotkanie, @IDSali, @LimitMiejsc, @DataRozpoczecia, @DataZakonczenia)

		INSERT INTO [Moduly Spotkania] VALUES (@NastepneIDSpotkanie, @IDModul)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu spotkania stacjonarnego dla modułu: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajSpotkanieModuluSynchronicznego***
Procedura pozwalająca dodać spotkanie synchroniczne dla modułu deklarując: identyfikator modułu, identyfikator tłumacza, datę rozpoczęcia, datę zakończenia i link do nagrania.
Procedura automatycznie dodaje nowe spotkanie do tabeli Spotkanie i tabeli Spotkanie online synchroniczne.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajSpotkanieModuluSynchronicznego](
	@IDModul INT,
	@IDTlumacz INT = NULL,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME,
	@Url NCHAR(255)
	)
AS
BEGIN
	SET NOCOUNT ON
	DECLARE @IDSpotkanie INT
	BEGIN TRY
		IF (SELECT dbo.KolizjaSpotkanModulu(@IDModul, @DataRozpoczecia, @DataZakonczenia, NULL, @IDTlumacz)) = 1
			THROW 51000, 'Kolizja spotkań', 1

		SELECT @IDSpotkanie = ISNULL(MAX(IDSpotkanie), 0) + 1 FROM Spotkanie
		INSERT INTO Spotkanie VALUES (@IDSpotkanie, @IDTlumacz)
		INSERT INTO [Spotkanie online synchroniczne] VALUES (@IDSpotkanie, @DataRozpoczecia, @DataZakonczenia, @Url)

		INSERT INTO [Moduly Spotkania] VALUES (@IDSpotkanie, @IDModul)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu spotkania synchronicznego dla modułu: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajSpotkanieModuluAsynchronicznego***
Procedura pozwalająca dodać spotkanie asynchroniczne dla modułu deklarując: identyfikator modułu, identyfikator tłumacza i link do nagrania.
Procedura automatycznie dodaje nowe spotkanie do tabeli Spotkanie i tabeli Spotkanie online asynchroniczne.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajSpotkanieModuluAsynchronicznego](
	@IDModul INT,
	@IDTlumacz INT = NULL,
	@Url NCHAR(255)
	)
AS
BEGIN
	SET NOCOUNT ON
	DECLARE @IDSpotkanie INT
	BEGIN TRY
		SELECT @IDSpotkanie = ISNULL(MAX(IDSpotkanie), 0) + 1 FROM Spotkanie
		INSERT INTO Spotkanie VALUES (@IDSpotkanie, @IDTlumacz)
		INSERT INTO [Spotkanie online asynchroniczne] VALUES (@IDSpotkanie, @Url)

		INSERT INTO [Moduly Spotkania] VALUES (@IDSpotkanie, @IDModul)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu spotkania asynchronicznego dla modułu: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajSpotkaniePrzedmiotuStacjonarne***
Procedura pozwalająca dodać spotkanie stacjonarne dla przedmiotu deklarując: identyfikator przedmiotu, identyfikator tłumacza, identyfikator sali, limit miejsc, datę rozpoczęcia i datę zakończenia.
Procedura automatycznie dodaje nowe spotkanie do tabeli Spotkanie i tabeli Spotkanie stacjonarne.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajSpotkaniePrzedmiotuStacjonarne](
	@IDPrzedmiot INT,
    	@IDTlumacz INT = NULL,
	@IDSali INT,
	@LimitMiejsc INT,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME
	)
AS
BEGIN
	DECLARE @IDSpotkanie INT
    	BEGIN TRY
		IF (SELECT dbo.KolizjaSpotkanPrzedmiotu(@DataRozpoczecia, @DataZakonczenia, NULL, @IDPrzedmiot, @IDSali, @IDTlumacz)) = 1
			THROW 51000, 'Kolizcja przedmiotów', 1

		SELECT @IDSpotkanie = ISNULL(MAX(IDSpotkanie), 0) + 1 FROM Spotkanie
		INSERT INTO Spotkanie VALUES (@IDSpotkanie, @IDTlumacz)
		INSERT INTO [Spotkanie stacjonarne] VALUES (@IDSpotkanie, @IDSali, @LimitMiejsc, @DataRozpoczecia, @DataZakonczenia)

		INSERT INTO [Przedmioty Spotkania] VALUES (@IDSpotkanie, @IDPrzedmiot)
   	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania spotkania dla przedmiotu: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajSpotkaniePrzedmiotuSynchroniczne***
Procedura pozwalająca dodać spotkanie synchroniczne dla przedmiotu deklarując: identyfikator przedmiotu, identyfikator tłumacza, datę rozpoczęcia, datę zakończenia i link do nagrania.
Procedura automatycznie dodaje nowe spotkanie do tabeli Spotkanie i tabeli Spotkanie online synchroniczne.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajSpotkaniePrzedmiotuSynchroniczne](
	@IDPrzedmiot INT,
    	@IDTlumacz INT = NULL,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME,
	@Url NCHAR(255)
	)
AS
BEGIN
	DECLARE @IDSpotkanie INT
    	BEGIN TRY
		IF (SELECT dbo.KolizjaSpotkanPrzedmiotu(@DataRozpoczecia, @DataZakonczenia, NULL, @IDPrzedmiot, NULL, @IDTlumacz)) = 1
			THROW 51000, 'Kolizcja przedmiotów', 1

		SELECT @IDSpotkanie = ISNULL(MAX(IDSpotkanie), 0) + 1 FROM Spotkanie
		INSERT INTO Spotkanie VALUES (@IDSpotkanie, @IDTlumacz)
		INSERT INTO [Spotkanie online synchroniczne] VALUES (@IDSpotkanie, @DataRozpoczecia, @DataZakonczenia, @Url)

		INSERT INTO [Przedmioty Spotkania] VALUES (@IDSpotkanie, @IDPrzedmiot)
    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd przy tworzeniu spotkania synchronicznego dla przedmiotu: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajSpotkanieModuluAsynchronicznego***
Procedura pozwalająca dodać spotkanie asynchroniczne dla przedmiotu deklarując: identyfikator przedmiotu, identyfikator tłumacza i link do nagrania.
Procedura automatycznie dodaje nowe spotkanie do tabeli Spotkanie i tabeli Spotkanie online asynchroniczne.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajSpotkaniePrzedmiotuAsynchroniczne](
	@IDPrzedmiot INT,
	@IDTlumacz INT = NULL,
	@Url NCHAR(255)
	)
AS
BEGIN
	DECLARE @IDSpotkanie INT
	BEGIN TRY
		SELECT @IDSpotkanie = ISNULL(MAX(IDSpotkanie), 0) + 1 FROM Spotkanie
		INSERT INTO Spotkanie VALUES (@IDSpotkanie, @IDTlumacz)
		INSERT INTO [Spotkanie online asynchroniczne] VALUES (@IDSpotkanie, @Url)

		INSERT INTO [Przedmioty Spotkania] VALUES (@IDSpotkanie, @IDPrzedmiot)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu spotkania asynchronicznego dla przedmiotu: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```
### ***PrzypiszSpotkanieDoZjazdu***
Procedura pozwalająca przypisać utworzone już spotkanie do zjazdu deklarując: identyfikator spotkania i identyfikator zjazdu.

```sql
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE dbo.[PrzypiszSpotkanieDoZjazdu] (
	@IDSpotkanie INT,
	@IDZjazd INT
)
AS
BEGIN
	DECLARE @DataRozpoczeciaSpotkania DATETIME, @DataZakonczeniaSpotkania DATETIME,
		@DataRozpoczeciaZjazdu DATETIME, @DataZakonczeniaZjazdu DATETIME
	BEGIN TRY
		SELECT @IDSpotkanie
		IF NOT EXISTS (SELECT * FROM Spotkanie)
			THROW 51000, 'Nie ma takiego spotkania', 1

		SELECT @IDZjazd
		IF NOT EXISTS (SELECT * FROM Zjazd)
			THROW 51000, 'Nie ma takiego zjazdu', 1

		SELECT @DataRozpoczeciaSpotkania = (SELECT ISNULL(SS.DataRozpoczecia, SOS.DataRozpoczecia)
			FROM Spotkanie AS S
			INNER JOIN [Spotkanie stacjonarne] AS SS ON S.IDSpotkanie = SS.IDSpotkanie
			INNER JOIN [Spotkanie online synchroniczne] AS SOS ON S.IDSpotkanie = SOS.IDSpotkanie
			WHERE S.IDSpotkanie = @IDSpotkanie)
		SELECT @DataZakonczeniaSpotkania = (SELECT ISNULL(SS.DataZakonczenia, SOS.DataZakonczenia)
			FROM Spotkanie AS S
			INNER JOIN [Spotkanie stacjonarne] AS SS ON S.IDSpotkanie = SS.IDSpotkanie
			INNER JOIN [Spotkanie online synchroniczne] AS SOS ON S.IDSpotkanie = SOS.IDSpotkanie
			WHERE S.IDSpotkanie = @IDSpotkanie)
		SELECT @DataRozpoczeciaZjazdu = (SELECT ZJ.DataRozpoczecia FROM Zjazd AS ZJ WHERE IDZjazd = @IDZjazd)
		SELECT @DataZakonczeniaZjazdu = (SELECT ZJ.DataRozpoczecia FROM Zjazd AS ZJ WHERE IDZjazd = @IDZjazd)

		IF @DataRozpoczeciaSpotkania IS NOT NULL AND @DataZakonczeniaSpotkania IS NOT NULL
			AND @DataRozpoczeciaZjazdu IS NOT NULL AND @DataZakonczeniaZjazdu IS NOT NULL
			IF dbo.ZawieraSieWPrzedzialeCzasowym(@DataRozpoczeciaSpotkania, @DataZakonczeniaSpotkania,
				@DataRozpoczeciaZjazdu, @DataZakonczeniaZjazdu) = 0
				THROW 51000, 'Czas trwania spotkania nie zawiera sie w czasie trwania zjazdu', 1

		INSERT INTO [Zjazdy Spotkania] VALUES (@IDZjazd, @IDSpotkanie)
	END TRY
	BEGIN CATCH
		DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania spotkania do zjazdu: ' + ERROR_MESSAGE();
        THROW 52000, @error, 1;
    END CATCH
END
```

### ***AktualizujSpotkanieStacjonarnePrzedmiotu***
Procedura pozwalająca zaktualizować spotkanie stacjonarne przedmiotu (zmienić jego datę i przypisaną salę zajęciową) deklarując: identyfikator spotkania, nowy identyfikator sali, nową datę rozpoczęcia i nową datę zakończenia.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[AktualizujSpotkanieStacjonarnePrzedmiotu](
	@IDSpotkanie INT,
	@IDSali INT = NULL,
	@DataRozpoczecia DATETIME = NULL,
	@DataZakonczenia DATETIME = NULL
	)
AS
BEGIN
	DECLARE @AktualneIDSali INT, @AktualnaDataRozpoczecia DATETIME, @AktualnaDataZakonczenia DATETIME
    	BEGIN TRY
		SELECT @AktualneIDSali = IDSali, @AktualnaDataRozpoczecia = DataRozpoczecia, @AktualnaDataZakonczenia = DataZakonczenia
		FROM [Spotkanie stacjonarne] AS ss
		WHERE ss.IDSpotkanie = @IDSpotkanie

		IF @IDSali IS NOT NULL
				IF (SELECT dbo.KolizjaSpotkanPrzedmiotu(@AktualnaDataRozpoczecia, @AktualnaDataZakonczenia, @IDSpotkanie, NULL, @IDSali, NULL)) = 0
					BEGIN
						UPDATE [Spotkanie stacjonarne] SET IDSali = @IDSali WHERE IDSpotkanie = @IDSpotkanie
						SET @AktualneIDSali = @IDSali
					END
				ELSE
					THROW 51000, 'Błędne ID sali', 1

		IF @DataRozpoczecia IS NOT NULL
			IF (SELECT dbo.KolizjaSpotkanPrzedmiotu(@DataRozpoczecia, @AktualnaDataZakonczenia, @IDSpotkanie, NULL, @AktualneIDSali, NULL)) = 0
				BEGIN
					UPDATE [Spotkanie stacjonarne] SET DataRozpoczecia = @DataRozpoczecia WHERE IDSpotkanie = @IDSpotkanie
					SET @AktualnaDataRozpoczecia = @DataRozpoczecia
				END
			ELSE
				THROW 51000, 'Błędna data rozpoczęcia', 1

		IF @DataZakonczenia IS NOT NULL 
			IF (SELECT dbo.KolizjaSpotkanPrzedmiotu(@AktualnaDataRozpoczecia, @DataZakonczenia, @IDSpotkanie, NULL, @AktualneIDSali, NULL)) = 0
				UPDATE [Spotkanie stacjonarne] SET DataZakonczenia = @DataZakonczenia WHERE IDSpotkanie = @IDSpotkanie
			ELSE
				THROW 51000, 'Błędna data zakończenia', 1

    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas aktualizacji spotkania: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
   	END CATCH
END
```

### ***AktualizujSpotkanieSynchronicznePrzedmiotu***
Procedura pozwalająca zaktualizować spotkanie synchroniczne przedmiotu (zmienić jego datę) deklarując: identyfikator spotkania, nową datę rozpoczęcia i nową datę zakończenia.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[AktualizujSpotkanieSynchronicznePrzedmiotu](
	@IDSpotkanie INT,
	@DataRozpoczecia DATETIME = NULL,
	@DataZakonczenia DATETIME = NULL
	)
AS
BEGIN
	DECLARE @AktualnaDataRozpoczecia DATETIME, @AktualnaDataZakonczenia DATETIME
    	BEGIN TRY
		SELECT @AktualnaDataRozpoczecia = DataRozpoczecia, @AktualnaDataZakonczenia = DataZakonczenia
		FROM [Spotkanie online synchroniczne]
		WHERE IDSpotkanie = @IDSpotkanie

		IF @DataRozpoczecia IS NOT NULL
			IF (SELECT dbo.KolizjaSpotkanPrzedmiotu(@DataRozpoczecia, @AktualnaDataZakonczenia, @IDSpotkanie, NULL, NULL, NULL)) = 0
				BEGIN
					UPDATE [Spotkanie online synchroniczne] SET DataRozpoczecia = @DataRozpoczecia WHERE IDSpotkanie = @IDSpotkanie
					SET @AktualnaDataRozpoczecia = @DataRozpoczecia
				END
			ELSE
				THROW 51000, 'Błędna data rozpoczęcia', 1

		IF @DataZakonczenia IS NOT NULL 
			IF (SELECT dbo.KolizjaSpotkanPrzedmiotu(@AktualnaDataRozpoczecia, @DataZakonczenia, @IDSpotkanie, NULL, NULL, NULL)) = 0
				UPDATE [Spotkanie online synchroniczne] SET DataZakonczenia = @DataZakonczenia WHERE IDSpotkanie = @IDSpotkanie
			ELSE
				THROW 51000, 'Błędna data zakończenia', 1

	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas aktualizacji spotkania: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***DodajJezyk***
Procedura pozwalająca dodać język do tabeli deklarując jego nazwę.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajJezyk] (@Nazwa VARCHAR(50))
AS
BEGIN
	BEGIN TRY
		SET NOCOUNT ON
		DECLARE @IDJezyk INT
		SELECT @IDJezyk = ISNULL(MAX(IDJezyk), 0) + 1 FROM Jezyk
		INSERT INTO Jezyk VALUES (@IDJezyk, @Nazwa)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy dodawaniu języka: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajKraj***
Procedura pozwalająca dodać kraj do tabeli deklarując jego nazwę.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajKraj] (@Nazwa VARCHAR(50))
AS
BEGIN
	BEGIN TRY
		SET NOCOUNT ON
		DECLARE @IDKraj INT
		SELECT @IDKraj = ISNULL(MAX(IDKraj), 0) + 1 FROM Kraj
		INSERT INTO Kraj (IDKraj, Nazwa) VALUES (@IDKraj, @Nazwa)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy dodawaniu kraju: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajMiasto***
Procedura pozwalająca dodać miasto do tabeli deklarując jego nazwę i kraj, w którym leży.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajMiasto] (@Nazwa VARCHAR(100), @IDKraj INT)
AS
BEGIN
	BEGIN TRY
		SET NOCOUNT ON

		IF (SELECT COUNT(*) FROM Kraj WHERE IDKraj = @IDKraj) = 0
			THROW 51000, 'Nie ma takiego kraju', 1

		DECLARE @IDMiasto INT
		SELECT @IDMiasto = ISNULL(MAX(IDMiasto), 0) + 1 FROM Miasto
		INSERT INTO Miasto VALUES (@IDMiasto, @Nazwa, @IDKraj)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy dodawaniu miasta: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajKursant***
Procedura pozwalająca dodać kursanta do tabeli deklarując: imię, nazwisko, zdjęcie, identyfikator miasta, ulicę, kod pocztowy, email i telefon.
Procedura automatycznie dodaje nowego użytkownika do tabeli Uzytkownicy.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajKursant] (
@Imie VARCHAR(50),
@Nazwisko VARCHAR(50),
@Zdjecie VARCHAR(255) = NULL,
@IDMiasto INT,
@Ulica VARCHAR(100),
@KodPocztowy VARCHAR(20),
@Email VARCHAR(50),
@Telefon VARCHAR(20))
AS
BEGIN
	BEGIN TRY
		SET NOCOUNT ON

		IF (SELECT COUNT(*) FROM Miasto WHERE IDMiasto = @IDMiasto) = 0
			THROW 51000, 'Nie ma takiego miasta', 1

		DECLARE @IDUzytkownik INT, @IDRola INT

		IF @IDRola IS NULL
			THROW 51000, 'Nie ma takiej roli', 1

		IF EXISTS (SELECT * FROM Kursant WHERE Email = @Email)
			THROW 51000, 'Ten email już istnieje', 1

		SELECT @IDUzytkownik = ISNULL(MAX(IDUzytkownik), 0) + 1 FROM Uzytkownicy
		INSERT INTO Uzytkownicy VALUES (@IDUzytkownik, @Imie, @Nazwisko, @Zdjecie)
		INSERT INTO Kursant VALUES (@IDUzytkownik, @IDMiasto, @Ulica, @KodPocztowy, @Telefon, @Email)
		INSERT INTO [Uzytkownicy Role] VALUES (@IDUzytkownik, @IDRola)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy dodawaniu kursanta: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajPersonel***
Procedura pozwalająca dodać członka personelu (wykładowca, raportowiec, tłumacz, dyrektor) do tabeli podając: imię, nazwisko, zdjęcie i typ użytkownika.
Procedura automatycznie dodaje nowy rekord do tabeli użytkowników odpowiedniego typu.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajPersonel] (
@Imie VARCHAR(50),
@Nazwisko VARCHAR(50),
@Zdjecie VARCHAR(255) = NULL,
@Typ VARCHAR(20))
AS
BEGIN
	BEGIN TRY
		DECLARE @IDUzytkownik INT, @IDRola INT

		SELECT @IDRola = dbo.JakieIDRoli(@Rola)

		IF @IDRola IS NULL
			THROW 51000, 'Nie ma takiej roli', 1

		SET NOCOUNT ON
		
		SELECT @IDUzytkownik = ISNULL(MAX(IDUzytkownik), 0) + 1 FROM Uzytkownicy
		INSERT INTO Uzytkownicy values (@IDUzytkownik, @Imie, @Nazwisko, @Zdjecie)
		INSERT INTO [Uzytkownicy Role] values (@IDUzytkownik, @IDRola)

	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy dodawaniu personelu: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH		
END
```

### ***DodajProduktDoZamowienia***
Procedura pozwalająca dodać produkt do zamówienia deklarując: identyfikator zamówienia, identyfikator formy kształcenia, informację czy zaliczka została wpłacona i informację czy wpisowe zostało wpłacone.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER procedure [dbo].[DodajProduktDoZamowienia] (@IDZamowienia INT = NULL, @IDKursant INT,
	@IDFormyKsztalcenia INT, @Zaliczka BIT = 0)
AS
BEGIN
	DECLARE @IDZamowienia INT
    BEGIN TRY
		IF @Zaliczka = 1 AND NOT EXISTS (SELECT * FROM [Forma Ksztalcenia] as fk
						LEFT JOIN Kurs as k ON fk.IDFormyKsztalcenia = k.IDKurs
						WHERE fk.IDFormyKsztalcenia = @IDFormyKsztalcenia)
			THROW 5100, 'Zaliczka może być naliczona tylko dla kursu', 1

		IF NOT EXISTS (SELECT * FROM [Forma Ksztalcenia] as fk
						LEFT JOIN Webinar as w ON fk.IDFormyKsztalcenia = w.IDWebinar
						LEFT JOIN Studium as s ON fk.IDFormyKsztalcenia = s.IDStudium
						LEFT JOIN Kurs as k ON fk.IDFormyKsztalcenia = k.IDKurs
						WHERE fk.IDFormyKsztalcenia = @IDFormyKsztalcenia)
			THROW 51000, 'Nie ma takiej formy kształcenia', 1

		IF NOT EXISTS (SELECT * FROM [Forma Ksztalcenia] as fk
						LEFT JOIN Webinar as w ON fk.IDFormyKsztalcenia = w.IDWebinar
						LEFT JOIN Kurs as k ON fk.IDFormyKsztalcenia = k.IDKurs
						WHERE fk.IDFormyKsztalcenia = @IDFormyKsztalcenia AND w.Cena IS NOT NULL OR k.Cena IS NOT NULL)
			THROW 51000, 'Ta forma kształcenia nie ma ceny', 1

		SELECT @IDZamowienia = MAX(IDZamowienia) FROM Zamowienia WHERE IDKursant = @IDKursant AND IDStatus = 1

		IF @IDZamowienia IS NULL
			Begin
				SELECT @IDZamowienia = ISNULL(MAX(IDZamowienia), 0) + 1 FROM Zamowienia
				INSERT INTO Zamowienia (IDZamowienia, IDKursant, IDStatus) VALUES (@IDZamowienia, @IDKursant, 1)
			END

		INSERT INTO [Zamowienia Informacje] (IDZamowienia, IDFormyKsztalcenia, CzyZaliczka) VALUES (@IDZamowienia, @IDFormyKsztalcenia, @Zaliczka)
    END TRY
    BEGIN CATCH
        DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania produktu do zamówienia: ' + ERROR_MESSAGE();
        THROW 52000, @error, 1;
    END CATCH
END
```

### ***DodajZamowienie***
Procedura pozwalająca dodać nowe zamówienie do tabeli zamówień podając identyfikator kursanta zamawiającego.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[DodajZamowienie](
	@IDKursant INT,
	@IDStatus INT = 1
	)
AS
BEGIN
	DECLARE @IDZamowienia INT
    	BEGIN TRY
		IF @IDStatus NOT IN (1, 3)
			THROW 51000, 'Nie można utworzyć opłaconego zamówienia', 1

		SELECT @IDZamowienia = ISNULL(MAX(IDZamowienia), 0) + 1 FROM Zamowienia

		INSERT INTO Zamowienia (IDZamowienia, IDKursant, IDStatus) VALUES (@IDZamowienia, @IDKursant, @IDStatus)
    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas tworzenia zamówienia: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***PrzypiszJezykDoTlumacza***
Procedura pozwalająca przypisać język do tłumacza podając: identyfikator tłumacza i identyfikator języka.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[PrzypiszJezykDoTlumacza] (
@IDTlumacz INT,
@IDJezyk INT)
AS
BEGIN
	BEGIN TRY
		SET NOCOUNT ON
		INSERT INTO [Tlumacz Jezyki] VALUES (@IDTlumacz, @IDJezyk)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy przypisywaniu języka: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***DodajRole***
Procedura pozwala dodać nową rolę do bazy.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE procedure [dbo].[DodajRole] (@Nazwa VARCHAR(50))
as
begin
	SET NOCOUNT ON
	DECLARE @IDRola INT
	BEGIN TRY
		IF (SELECT COUNT(*) FROM [Role Uzytkownikow] WHERE Nazwa LIKE @Nazwa) = 1
			THROW 51000, 'Ta rola już istnieje', 1

		SELECT @IDRola = ISNULL(MAX(IDRola), 0) + 1 FROM  [Role Uzytkownikow]
		INSERT INTO [Role Uzytkownikow] values (@IDRola, @Nazwa)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy tworzeniu roli: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
end;
```

### ***UsunWebinar***
Procedura pozwalająca usunąć webinar z tabeli podając jego identyfikator.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[UsunWebinar](
	@IDWebinar INT
	)
AS
BEGIN
	SET NOCOUNT ON
	BEGIN TRY
		IF NOT EXISTS (SELECT IDWebinar FROM Webinar WHERE IDWebinar = @IDWebinar)
			THROW 51000, 'Nie ma takiego webinaru', 1
		ELSE
			DELETE FROM Webinar WHERE IDWebinar = @IDWebinar
		IF NOT EXISTS (SELECT IDFormyKsztalcenia FROM [Forma Ksztalcenia] WHERE IDFormyKsztalcenia = @IDWebinar)
			THROW 51000, 'Nie ma takiej formy kształcenia', 1
		ELSE
			DELETE FROM [Forma Ksztalcenia] WHERE IDFormyKsztalcenia = @IDWebinar
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy usuwaniu webinaru: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
END
```

### ***WyswietlHarmonogramKursanta***
Procedura pozwalająca wyświetlić harmonogram danego kursanta podając identyfikator tego kursanta podając jego identyfikator.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[WyswietlHarmonogramKursanta](@IDKursant INT)
AS
BEGIN
    	BEGIN TRY
        		IF NOT EXISTS(
                		SELECT IDKursant
                		FROM WidokOsobZapisanychNaKurs
                		WHERE IDKursant = @IDKursant
			UNION
			SELECT IDKursant
			FROM WidokOsobZapisanychNaStudium
			WHERE IDKursant = @IDKursant
            )
               	THROW 51000, 'Nie jesteś zapisany do kursu bądź studium', 1

       		SELECT IDKursant, IDFormyKsztalcenia, NazwaStudium AS NazwaFormyKsztalcenia, IDSpotkanie, IDTlumacz, Tlumacz, Nazwa, CAST(Opis AS NVARCHAR(MAX)), IDWykladowca, Wykladowca, Url, LimitMiejsc, Sala, DataRozpoczecia, DataZakonczenia FROM WidokOsobZapisanychNaStudium AS wzs
		JOIN WidokHarmonogramuStudium AS whs ON wzs.IDFormyKsztalcenia = whs.IDStudium
		WHERE IDKursant = @IDKursant
		UNION
		SELECT IDKursant, IDFormyKsztalcenia, NazwaKursu AS NazwaFormyKsztalcenia, IDSpotkanie, IDTlumacz, Tlumacz, Nazwa, Opis, IDWykladowca, Wykladowca, Url, LimitMiejsc, Sala, DataRozpoczecia, DataZakonczenia FROM WidokOsobZapisanychNaKurs AS wzk
		JOIN WidokSpotkanModulow AS wsm ON wzk.IDFormyKsztalcenia = wsm.IDKurs
		WHERE IDKursant = @IDKursant
    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Wystąpił błąd podczas wyświetlania harmonogramu: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***ZmienStatusZamowienia***
Procedura, która pozwala zmienić status zamówienia z nieopłaconego na opłacony.

```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER procedure [dbo].[ZmienStatusZamowienia] (@IDZamowienia INT, @Status VARCHAR(50))
as
BEGIN
	DECLARE @IDStatus INT
    BEGIN TRY
		SELECT @IDStatus = IDStatus FROM [Zamowienie Status] WHERE Nazwa = @Status

		IF @IDStatus IS NULL
			THROW 51000, 'Nie ma takiego statusu', 1

		UPDATE Zamowienia SET IDStatus = @IDStatus WHERE IDZamowienia = @IDZamowienia
    END TRY
    BEGIN CATCH
        DECLARE @error NVARCHAR(1000) = 'Błąd podczas dodawania produktu do zamówienia: ' + ERROR_MESSAGE();
        THROW 52000, @error, 1;
    END CATCH
END
```

### ***WyswietlPodsumowanieKursanta***
Procedura pozwalająca wyświetlić widok frekwencji danego studenta na danym kursie bądź studium.
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER procedure [dbo].[WyswietlPodsumowanieKursanta] (@IDKursant INT, @FormaKsztalcenia VARCHAR(50) = NULL)
as
BEGIN
    	BEGIN TRY
		IF @FormaKsztalcenia IS NOT NULL AND @FormaKsztalcenia NOT IN ('Kurs', 'Studium')
			THROW 51000, 'Nie ma takiej formy kształcenia', 1

        		IF NOT EXISTS(
                	SELECT IDKursant
                	FROM WidokOsobZapisanychNaKurs
                	WHERE IDKursant = @IDKursant
		union
		SELECT IDKursant
			FROM WidokOsobZapisanychNaStudium
			WHERE IDKursant = @IDKursant
            )
                		THROW 51000, 'Nie jesteś zapisany do kursu bądź studium', 1

		IF @FormaKsztalcenia IS NULL
			select * from WidokFrekwencjiOsobNaModulach where IDKursant = @IDKursant
			union
			select * from WidokFrekwencjiOsobNaPrzedmiotach where IDKursant = @IDKursant

		IF @FormaKsztalcenia = 'Kurs'
			select * from WidokFrekwencjiOsobNaModulach where IDKursant = @IDKursant

		IF @FormaKsztalcenia = 'Studium'
			select * from WidokFrekwencjiOsobNaPrzedmiotach where IDKursant = @IDKursant

    	END TRY
    	BEGIN CATCH
        		DECLARE @error NVARCHAR(1000) = 'Błąd podczas wyświetlania podsumowania: ' + ERROR_MESSAGE();
        		THROW 52000, @error, 1;
    	END CATCH
END
```

### ***PrzyznajRole***
Procedura, która pozwala przyznać użytkownikowi nową rolę.
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER procedure [dbo].[PrzyznajRole] (@IDRola INT, @IDUzytkownik INT)
as
begin
	SET NOCOUNT ON
	BEGIN TRY
		INSERT INTO [Uzytkownicy Role] values (@IDUzytkownik, @IDRola)
	END TRY
	BEGIN CATCH
		DECLARE @error VARCHAR(1000) = 'Błąd przy przyznawaniu roli: ' + ERROR_MESSAGE();
		THROW 52000, @error, 1
	END CATCH
end
```
