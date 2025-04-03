# Funkcje
### ***KolizjaDat***
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER FUNCTION [dbo].[KolizjaDat](
	@DataRozpoczecia1 DATETIME,
	@DataZakonczenia1 DATETIME,
	@DataRozpoczecia2 DATETIME,
	@DataZakonczenia2 DATETIME
	)
    RETURNS INT
AS
BEGIN
    IF (@DataRozpoczecia1 < @DataZakonczenia2 AND @DataZakonczenia1 > @DataRozpoczecia2)
        RETURN 1

	IF (@DataRozpoczecia1 > @DataRozpoczecia2 AND @DataZakonczenia1 < @DataZakonczenia2)
		RETURN 1

	RETURN 0
END
```

### ***KolizjaSpotkanModulu***
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER FUNCTION [dbo].[KolizjaSpotkanModulu](
	@IDModul INT,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME,
	@IDSali INT = NULL,
	@IDTlumacz INT = NULL
	)
    RETURNS INT
AS
BEGIN
	DECLARE @IDWykladowca INT

	SELECT @IDWykladowca = IDWykladowca from Moduly WHERE IDModul = @IDModul

	-- Kolizja Modułow
    IF EXISTS (SELECT * FROM [Moduly Spotkania] as ms
					JOIN Spotkanie as s on s.IDSpotkanie = ms.IDSpotkanie
					LEFT JOIN [Spotkanie stacjonarne] as ss on s.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] as sos on s.IDSpotkanie = sos.IDSpotkanie
					WHERE ms.IDModul = @IDModul
					AND (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1 OR dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1))
			RETURN 1

	--Zawiera się w przedziale czasowym kursu
	IF NOT EXISTS (SELECT * FROM Moduly as m
				JOIN Kurs as k ON m.IDKurs = k.IDKurs
				WHERE m.IDModul = @IDModul AND dbo.ZawieraSieWPrzedzialeCzasowym(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1
				)
			RETURN 1

	-- Kolizja Sal
	IF @IDSali IS NOT NULL AND EXISTS (SELECT * FROM [Spotkanie stacjonarne] WHERE IDSali = @IDSali AND dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1)
			RETURN 1

	-- Kolizja Tłumaczy
	IF @IDTlumacz IS NOT NULL AND dbo.KolizjaTlumaczy(@IDTlumacz, @DataRozpoczecia, @DataZakonczenia) = 1
			RETURN 1

	-- Kolizja Wykładowców
	IF dbo.KolizjaWykladowcow(@IDWykladowca, @DataRozpoczecia, @DataZakonczenia) = 1
			RETURN 1

    RETURN 0
END
```

### ***KolizjaSpotkanPrzedmiotu***
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER FUNCTION [dbo].[KolizjaSpotkanPrzedmiotu](
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME,
	@IDSpotkanie INT = NULL,
	@IDPrzedmiot INT = NULL,
	@IDSali INT = NULL,
	@IDTlumacz INT = NULL
	)
    RETURNS INT
AS
BEGIN
	DECLARE @IDWykladowca INT

	IF (@IDPrzedmiot IS NULL AND @IDSpotkanie IS NULL) OR (@IDPrzedmiot IS NOT NULL AND @IDSpotkanie IS NOT NULL)
		RETURN 1

	IF @IDSpotkanie IS NOT NULL
		SELECT @IDPrzedmiot = p.IDPrzedmiot, @IDTlumacz = s.IDTlumacz, @IDSali = IDSali, @IDWykladowca = p.IDWykladowca FROM [Przedmioty Spotkania] AS ps
		JOIN Spotkanie AS s on ps.IDSpotkanie = s.IDSpotkanie
		JOIN Przedmioty AS p on ps.IDPrzedmiot = p.IDPrzedmiot
		LEFT JOIN [Spotkanie stacjonarne] AS ss ON s.IDSpotkanie = ss.IDSpotkanie
		WHERE ps.IDSpotkanie = @IDSpotkanie
	ELSE
		SET @IDSpotkanie = -1

	--Zawiera się w przedziale czasowym studium
	IF NOT EXISTS (SELECT * FROM Przedmioty AS p
				JOIN Studium AS s ON p.IDStudium = s.IDStudium
				WHERE p.IDPrzedmiot = @IDPrzedmiot AND dbo.ZawieraSieWPrzedzialeCzasowym(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1
				)
			RETURN 1

	-- Kolizja Przedmiotów
    IF EXISTS (SELECT * FROM [Przedmioty Spotkania] AS ps
				LEFT JOIN [Spotkanie stacjonarne] AS ss ON ps.IDSpotkanie = ss.IDSpotkanie
				LEFT JOIN [Spotkanie online synchroniczne] AS sos ON ps.IDSpotkanie = sos.IDSpotkanie
				WHERE ps.IDPrzedmiot = @IDPrzedmiot AND ps.IDSpotkanie <> @IDSpotkanie
				AND (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1 OR dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1))
			RETURN 1

	-- Kolizja Sal
	IF @IDSali IS NOT NULL AND EXISTS (SELECT * FROM [Spotkanie stacjonarne] WHERE IDSali = @IDSali AND IDSpotkanie <> @IDSpotkanie AND dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1)
			RETURN 1

	IF EXISTS (SELECT * FROM Webinar WHERE IDTlumacz = @IDTlumacz AND dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1)
		OR EXISTS (SELECT * FROM Przedmioty as p
					JOIN [Przedmioty Spotkania] AS ps ON ps.IDPrzedmiot = p.IDPrzedmiot
					JOIN Spotkanie as s on s.IDSpotkanie = ps.IDSpotkanie
					LEFT JOIN [Spotkanie stacjonarne] as ss on ps.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] AS sos ON ps.IDSpotkanie = sos.IDSpotkanie
					WHERE s.IDSpotkanie <> @IDSpotkanie AND s.IDTlumacz = @IDTlumacz AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
		OR EXISTS (SELECT * FROM Moduly AS m
					JOIN [Moduly Spotkania] AS ms ON ms.IDModul = m.IDModul
					JOIN Spotkanie AS s ON s.IDSpotkanie = ms.IDSpotkanie
					LEFT JOIN [Spotkanie stacjonarne] AS ss ON ms.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] AS sos ON ms.IDSpotkanie = sos.IDSpotkanie
					WHERE s.IDSpotkanie <> @IDSpotkanie AND s.IDTlumacz = @IDTlumacz AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
			RETURN 1

	IF EXISTS (SELECT * FROM Webinar WHERE IDWykladowca = @IDWykladowca AND dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1)
		OR EXISTS (SELECT * FROM Przedmioty AS p
					JOIN [Przedmioty Spotkania] AS ps ON ps.IDPrzedmiot = p.IDPrzedmiot
					LEFT JOIN [Spotkanie stacjonarne] AS ss ON ps.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] AS sos ON ps.IDSpotkanie = sos.IDSpotkanie
					WHERE ps.IDSpotkanie <> @IDSpotkanie AND IDWykladowca = @IDWykladowca AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
		OR EXISTS (SELECT * FROM Moduly AS m
					JOIN [Moduly Spotkania] AS ms ON ms.IDModul = m.IDModul
					LEFT JOIN [Spotkanie stacjonarne] AS ss ON ms.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] AS sos ON ms.IDSpotkanie = sos.IDSpotkanie
					WHERE ms.IDSpotkanie <> @IDSpotkanie AND IDWykladowca = @IDWykladowca AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
			RETURN 1

    RETURN 0
END
```

### ***JakieIDRoli***
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER FUNCTION [dbo].[JakieIDRoli]
(
	@Nazwa VARCHAR(50)
)
RETURNS INT
AS
BEGIN
	DECLARE @IDRola INT

	SELECT @IDRola = IDRola FROM [Role Uzytkownikow] WHERE Nazwa = @Nazwa

	RETURN @IDRola
END
```

### ***KolizjaTlumaczy***
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER FUNCTION [dbo].[KolizjaTlumaczy](
	@IDTlumacz INT,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME
	)
RETURNS INT AS
BEGIN
	IF EXISTS (SELECT * FROM Webinar WHERE IDTlumacz = @IDTlumacz AND dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1)
		OR EXISTS (SELECT * FROM Przedmioty as p
					JOIN [Przedmioty Spotkania] as ps on ps.IDPrzedmiot = p.IDPrzedmiot
					JOIN Spotkanie as s on s.IDSpotkanie = ps.IDSpotkanie
					LEFT JOIN [Spotkanie stacjonarne] as ss on ps.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] as sos on ps.IDSpotkanie = sos.IDSpotkanie
					WHERE s.IDTlumacz = @IDTlumacz AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
		OR EXISTS (SELECT * FROM Moduly as m
					JOIN [Moduly Spotkania] as ms on ms.IDModul = m.IDModul
					JOIN Spotkanie as s on s.IDSpotkanie = ms.IDSpotkanie
					LEFT JOIN [Spotkanie stacjonarne] as ss on ms.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] as sos on ms.IDSpotkanie = sos.IDSpotkanie
					WHERE s.IDTlumacz = @IDTlumacz AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
			RETURN 1
	RETURN 0
END
```

### ***KolizjaWykladowcow***
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER FUNCTION [dbo].[KolizjaWykladowcow](
	@IDWykladowca INT,
	@DataRozpoczecia DATETIME,
	@DataZakonczenia DATETIME
	)
RETURNS INT AS
BEGIN
	IF EXISTS (SELECT * FROM Webinar WHERE IDWykladowca = @IDWykladowca AND dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, DataRozpoczecia, DataZakonczenia) = 1)
		OR EXISTS (SELECT * FROM Przedmioty as p
					JOIN [Przedmioty Spotkania] as ps on ps.IDPrzedmiot = p.IDPrzedmiot
					LEFT JOIN [Spotkanie stacjonarne] as ss on ps.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] as sos on ps.IDSpotkanie = sos.IDSpotkanie
					WHERE IDWykladowca = @IDWykladowca AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
		OR EXISTS (SELECT * FROM Moduly as m
					JOIN [Moduly Spotkania] as ms on ms.IDModul = m.IDModul
					LEFT JOIN [Spotkanie stacjonarne] as ss on ms.IDSpotkanie = ss.IDSpotkanie
					LEFT JOIN [Spotkanie online synchroniczne] as sos on ms.IDSpotkanie = sos.IDSpotkanie
					WHERE IDWykladowca = @IDWykladowca AND ((dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, ss.DataRozpoczecia, ss.DataZakonczenia) = 1) OR (dbo.KolizjaDat(@DataRozpoczecia, @DataZakonczenia, sos.DataRozpoczecia, sos.DataZakonczenia) = 1)))
			RETURN 1
	RETURN 0
END
```

### ***ZawieraSieWPrzedzialeCzasowym***
```sql
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER FUNCTION [dbo].[ZawieraSieWPrzedzialeCzasowym](
	@DataRozpoczecia1 DATETIME,
	@DataZakonczenia1 DATETIME,
	@DataRozpoczecia2 DATETIME,
	@DataZakonczenia2 DATETIME
	)
    RETURNS INT
AS
BEGIN
    IF (@DataRozpoczecia1 >= @DataRozpoczecia2 AND @DataZakonczenia1 <= @DataZakonczenia2)
        RETURN 1

	RETURN 0
END
```