# Databases

## Installazione

- Download [H2](https://www.h2database.com/)
- ```java -jar bin/h2-1.4.200.jar```
- [H2-console](http://localhost:9898/)

> Per default (jdbc:h2:mem:banking) la banca dati non viene persistita!

## Documentazione
- [https://www.h2database.com/](https://www.h2database.com/)
- [https://www.w3schools.com/sql/](https://www.w3schools.com/sql/) 

## Esempi

### SHOP

```
┌────────────┐        ┌────────────┐
│            │        │            │
│            │        │            │
│  CLIENTI   ├────────┤   ORDINI   │
│            │1   0..N│            │
│            │        │            │
└────────────┘        └────────────┘
```

```
SET AUTOCOMMIT OFF;
CREATE SCHEMA IF NOT EXISTS SHOP;
SET SCHEMA SHOP;
```

```
CREATE TABLE CLIENTI (
    CODICE INT PRIMARY KEY, 
    NOME VARCHAR(50) NOT NULL, 
    DATA_DI_NASCITA DATE CHECK (YEAR(CURRENT_DATE) - YEAR(DATA_DI_NASCITA) > 18),
    UNIQUE (NOME, DATA_DI_NASCITA)
);

CREATE TABLE ORDINI (
    ID INT AUTO_INCREMENT PRIMARY KEY,
    CODICE_CLIENTE INT NOT NULL,
    DATA_CREAZIONE DATE NOT NULL DEFAULT CURRENT_DATE
);
```

```
INSERT INTO CLIENTI (CODICE, NOME, DATA_DI_NASCITA) VALUES (1000, 'Andrea', TO_DATE( '03.06.1970', 'DD.MM.YYYY' ));
INSERT INTO CLIENTI (CODICE, NOME, DATA_DI_NASCITA) VALUES (2000, 'Lorena', TO_DATE( '05.10.1971', 'DD.MM.YYYY' ));
INSERT INTO CLIENTI (CODICE, NOME, DATA_DI_NASCITA) VALUES (3000, 'Antonio', TO_DATE( '1.1.1972', 'DD.MM.YYYY' ));

INSERT INTO ORDINI (CODICE_CLIENTE) VALUES (1000);
INSERT INTO ORDINI (CODICE_CLIENTE) VALUES (1000);
INSERT INTO ORDINI (CODICE_CLIENTE) VALUES (2000);

COMMIT;
```

```
SELECT * FROM CLIENTI;
SELECT * FROM ORDINI;
SELECT COUNT(*) FROM CLIENTI;
SELECT * FROM CLIENTI WHERE CODICE = 1000;
SELECT * FROM CLIENTI WHERE NOME LIKE 'An%';
SELECT * FROM CLIENTI WHERE (YEAR(CURRENT_DATE) - YEAR(DATA_DI_NASCITA)) > 50;
SELECT * FROM ORDINI AS O INNER JOIN CLIENTI AS C ON O.CODICE_CLIENTE = C.CODICE;
SELECT * FROM ORDINI AS O INNER JOIN CLIENTI AS C ON O.CODICE_CLIENTE = C.CODICE WHERE C.CODICE = 1000;
```

Ora vediamo come i vincoli definiti permettano di mantenere una certa integrità dei dati inseriti:

```
INSERT INTO CLIENTI (CODICE, NOME, DATA_DI_NASCITA) VALUES (1000, 'XX', TO_DATE( '03.06.1970', 'DD.MM.YYYY' ));

INSERT INTO CLIENTI (CODICE, NOME, DATA_DI_NASCITA) VALUES (5000, 'Andrea', TO_DATE( '03.06.1970', 'DD.MM.YYYY' ));

INSERT INTO ORDINI (CODICE_CLIENTE) VALUES (NULL);

INSERT INTO ORDINI (CODICE_CLIENTE) VALUES (5000);

# https://www.w3schools.com/sql/sql_join.asp
SELECT * FROM ORDINI AS O INNER JOIN CLIENTI AS C ON O.CODICE_CLIENTE = C.CODICE;
SELECT * FROM ORDINI AS O LEFT JOIN CLIENTI AS C ON O.CODICE_CLIENTE = C.CODICE;
SELECT * FROM ORDINI AS O RIGHT JOIN CLIENTI AS C ON O.CODICE_CLIENTE = C.CODICE;

ROLLBACK;
```

Ora impostiamo una FOREIGN KEY

```
ALTER TABLE ORDINI
    ADD CONSTRAINT FK_ORDINI_CLIENTE
    FOREIGN KEY (CODICE_CLIENTE) REFERENCES CLIENTI(CODICE);
```

E vediamo come non sia più possibili inserire un ORDINE per un CLIENTE inesistente:

```
INSERT INTO ORDINI (CODICE_CLIENTE) VALUES (5000);
```

E come non sia possibile cancellare un CLIENTE (padre) che ha degli ORDINI (figli) senza prima aver cancellato gli ORDINI

```
DELETE FROM CLIENTI WHERE CODICE = 1000;

DELETE FROM ORDINI WHERE CODICE_CLIENTE = 1000;
DELETE FROM CLIENTI WHERE CODICE = 1000;

ROLLBACK;
```

Se volessimo cancellare automaticamente gli ORDINI alla cancellazione di un CLIENTE dobbiamo modificare la CONSTRAINT:

```
ALTER TABLE ORDINI DROP CONSTRAINT FK_ORDINI_CLIENTE;

ALTER TABLE ORDINI
    ADD CONSTRAINT FK_ORDINI_CLIENTE_CASCADE
    FOREIGN KEY (CODICE_CLIENTE) REFERENCES CLIENTI(CODICE)
    ON DELETE CASCADE;
```

```
DELETE FROM CLIENTI WHERE CODICE = 1000;
COMMIT;
SELECT * FROM ORDINI AS O INNER JOIN CLIENTI AS C ON O.CODICE_CLIENTE = C.CODICE;
```

```
DROP TABLE SHOP.ORDINI;
DROP TABLE SHOP.CLIENTI;
DROP SCHEMA SHOP;
```

### ALBERI
Alcuni tipi di dati richiedono strutture dati ad [albero](https://en.wikipedia.org/wiki/Tree_(data_structure)). Pensiamo all'assemblaggio di un auto: l'auto è formata da "pezzi". Ogni "pezzo" è a sua volta formato da altri "pezzi".

```
┌────────────┐        ┌────────────┐        ┌────────────┐      
│            │        │            │        │            │
│            │        │            │        │            │
│  PEZZI     ├────────┤  PEZZI_DI  │────────┤PEZZI_DI_DI │
│            │1   0..N│            │1   0..N│            │
│            │        │            │        │            │
└────────────┘        └────────────┘        └────────────┘
```

Questa struttura "recursiva" può essere così modellata
```
         ┌─────────────────┐
         │                 │
    0..N │                 │
┌────────┴────────┐        │
│                 │        │
│                 │        │
│                 │        │
│     PEZZI       ├────────┘
│                 │
│                 │
│                 │
└─────────────────┘
```
```
SET AUTOCOMMIT OFF;
CREATE SCHEMA IF NOT EXISTS ALBERI;
SET SCHEMA ALBERI;
```

```
CREATE TABLE PEZZI (
    CODICE INT PRIMARY KEY, 
    NOME VARCHAR(50) NOT NULL, 
    PEZZO_DI INT REFERENCES PEZZI(CODICE) ON DELETE CASCADE
);
```
```
INSERT INTO PEZZI (CODICE, NOME, PEZZO_DI) VALUES (1, '1', NULL);
INSERT INTO PEZZI (CODICE, NOME, PEZZO_DI) VALUES (11, '1.1', 1);
INSERT INTO PEZZI (CODICE, NOME, PEZZO_DI) VALUES (12, '1.2', 1);
INSERT INTO PEZZI (CODICE, NOME, PEZZO_DI) VALUES (111, '1.1.1', 11);
INSERT INTO PEZZI (CODICE, NOME, PEZZO_DI) VALUES (2, '2', NULL);
COMMIT;
```

```
SELECT * FROM PEZZI P LEFT JOIN PEZZI DI on DI.PEZZO_DI = P.CODICE;
```

```
DELETE FROM PEZZI WHERE CODICE = 1;
SELECT * FROM PEZZI P LEFT JOIN PEZZI DI on DI.PEZZO_DI = P.CODICE;
```

```
DROP TABLE ALBERI.PEZZI;
DROP SCHEMA ALBERI;
```



