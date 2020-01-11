# Podzapytania

Wynik każdego zapytania `SELECT` można traktować jak kolejną (wirtualną) tabelę. W efekcie możemy wykonywać na takiej tabeli zapytania.

## Podzapytania w `FROM`

Załóżmy, że mamy gotowe zapytanie o listę użytkowników biblioteki wraz z liczbą książek pożyczonych przez każdego z nich:

```SQL
SELECT users.name, users.surname, count(borrowings.ID) as activity
    FROM users
    JOIN borrowings ON users.ID = borrowings.user_id
    GROUP BY users.ID
```

Na wyniku tego zapytania możemy wykonać kolejne zapytanie, by otrzymać listę użytkowników, którzy pożyczyli więcej niż 20 książek:

```SQL
SELECT * FROM
    (SELECT users.name, users.surname, count(borrowings.ID)
        FROM users
        JOIN borrowings ON users.ID = borrowings.user_id
        GROUP BY users.ID)
    WHERE activity > 20
```

## Podzapytania w `WHERE`

Zapytania zwracające w wyniku **pojedynczą** wartość możemy wykorzystać np. w wyrażeniach warunkowych (`WHERE`, `HAVING`). Mając zapytanie o średnią liczbę wypożyczeń dokonanych przez użytkownika:

```SQL
SELECT 1.0 * count(DISTINCT borrowings.ID) / count(DISTINCT borrowings.user_id) FROM borrowings
```

możemy napisać zapytanie zwracające listę użytkowników pożyczających więcej książek niż średnia:

```SQL
SELECT users.name, users.surname, count(borrowings.ID) as books_borrowed FROM users
    JOIN borrowings ON users.ID = borrowings.user_id
    GROUP BY users.ID
    HAVING books_borrowed > (SELECT 1.0 * count(DISTINCT borrowings.ID) / count(DISTINCT borrowings.user_id) FROM borrowings)
```

## Podzapytania w `SELECT`

Analogicznie do poprzedniego rozdziału - jeśli podzapytanie zwraca pojedynczą wartość, możemy je wykorzystać jako jedną z kolumn w głównym zapytaniu.

```SQL
SELECT users.name, users.surname,
        (SELECT count(borrowings.ID) from borrowings WHERE borrowings.user_id = users.ID) as books
    FROM users
```

Co ważne - w podzapytaniu (w tym przypadku z tabeli `borrowings`) możemy odwołać się do tabeli i pól z zapytania zewnętrznego (`users`)

## Wydajność

Nie musimy się martwić o wydajność takich podzapytań - mimo że pozornie liczenie średniej liczby wypożyczeń powinno się wykonywać z osobna dla każdego rekordu użytkownika, baza danych zoptymalizuje plan zapytania. Jeśli chcemy sprawdzić, jak nasze zapytanie przekłada się na operacje wykonywane w bazie, możemy przed treścią zapytania dodać komendę `EXPLAIN QUERY PLAN` - zapytanie nie zostanie wtedy wykonane, otrzymamy jednak listę kroków, które baza danych chce wykonać.

```SQL
EXPLAIN QUERY PLAN
SELECT users.name, users.surname, count(borrowings.ID) as books_borrowed FROM users
    JOIN borrowings ON users.ID = borrowings.user_id
    GROUP BY users.ID
    HAVING books_borrowed > (SELECT 1.0 * count(DISTINCT borrowings.ID) / count(DISTINCT borrowings.user_id) FROM borrowings)
```
