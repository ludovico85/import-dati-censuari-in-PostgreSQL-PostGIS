# Import-dati-censuari-in-PostgreSQL/PostGIS
Il repository contiene i passaggi per l'importazione dei dati catastali censuari in un database PostgreSQL/PostGIS e per il collegamento dei dati stessi alle geometrie catastali importate utilizzando il plugin per QGIS CXF_in https://github.com/saccon/CXF_in. Nella costruzione del database non sono state inserite esplicitamente delle relazioni tramite chiavi primarie/esterne a causa della non univocità dei valori presenti in alcuni tipi di file. Tuttavia le relazioni sono assicurate attarverso delle operazioni di join.

## Breve descrizione dei dati catastali censuari.
Le informazioni descritte in questa sezione derivano dal documento a cura dell'Agenzia dell'Entrate (DOC. ES-23-IS-05) liberamente consultabile all'indirizzo: https://wwwt.agenziaentrate.gov.it/mt/ServiziComuniIstituzioni/ES-23-IS-05_100909.pdf.

Per maggiori dettagli sui servizi riservati ai comuni di può consultare: https://www.agenziaentrate.gov.it/portale/web/guest/schede/fabbricatiterreni/portale-per-i-comuni/servizi-portale-dei-comuni/estrazione-dati-catastali.

I dati censuari sono costituiti da 4 tipi di file:

- file fabbricati (.FAB);

- file terreni (.TER);

- file soggetti (.SOG);

- file titolarità (.TIT);

- file parametri della richiesta (.PRM).

Ogni tipo di file è costituito da una tabella che può contenere diversi tipi di record. Il collegamento tra i tipi di file è assicurato dalla presenta di chiavi specifiche:

- .FAB/.TER contengono la chiave identificativo immobile;

- .SOG contiene la chiave identificativo soggetto;

- .TIT contiene sia la chiave identificativo immobile che la chiave identificativo soggetto (oltre che la chiave identificativo titolarietà);


## Struttura del database
Per una gestione migliore della varie tabelle e viste che si andranno a creare è opportuno organizzare il database in schemi differenti:
- catasto_terreni = conterrà il dataset relativo al catasto terreni
- catasto_fabbricati = conterrà il dataset relativo al catasto fabbircati
- cxf_in = schema creato in automatico dal plug in cxf_in che conterrà i dati geografici

### Schema catasto_terreni

**Tabelle**

- ter =  tabella derivata dal file .TER.
- sogp = tabella derivata dal file .SOG con TIPO SOGGETTO = 'P' (persona fisica).
- sogg = tabella derivata dal file .SOG con TIPO SOGGETTO = 'G' (persona giuridica).
- tit = tabella derivata dal file .TIT.
- codici_diritto = tabella contenente l'identificativo e la descrizione dei CODICI DIRITTO (pag. 36 del DOC. ES-23-IS-05).
- qualita = tabella contenente l'identificativo e la descrizione dei CODICI QUALITA' (pag. 38 del DOC. ES-23-IS-05).
- partite_speciali_terreni = tabella contenente l'identificativo e la descrizione delle PARTITE SPECIALI DEL CATASTO TERRENI (pag. 45 del DOC. ES-23-IS-05).
- partite_speciali_fabbricati = tabella contenente l'identificativo e la descrizione delle PARTITE SPECIALI DEL CATASTO FABBRICATI (pag. 45 del DOC. ES-23-IS-05).

**Viste**

- ter_1 = vista derivata dalla tabella ter contenente solo il TIPO RECORD = '1'.
- ter_1_clean = vista derivata dalla vista ter_1 ottenuta escludendo i codici PARTITA '0', '4', '5' e escludendo i subalterni.
- titg = vista derivata dalla tabella tit con TIPO SOGGETTO = 'P' (persona fisica).
- titg = vista derivata dalla tabella tit con TIPO SOGGETTO = 'G' (persona giuridica).
- titp_sogp = vista creata tramite join tra titp e sogp utilizzando il campo in comune identificativo_soggetto (persone fisiche).
- titp_sogp_json = vista ottenuta tramite l'aggregazione del campo identificativo_immobile dalla vista titp_sogp.
- titg_sogg = vista creata tramite join tra titg e sogg utilizzando il campo in comune identificativo_soggetto (persone giuridiche).
- titg_sogg_json = vista ottenuta tramite l'aggregazione del campo identificativo_immobile dalla vista titg_sogg.
- titp_sogp_ter_persone_fisiche = vista ottenuta tramite tramite join tra titp_sogp_json e la vista ter_1_clean utilizzando il campo in comune identificativo_immobile
(persone fisiche).
- titp_sogp_ter_persone_giuridiche = vista ottenuta tramite tramite join tra titg_sogg_json e la vista ter_1_clean utilizzando il campo in comune identificativo_immobile (persone giuridiche).
- particelle_partite_speciali_terreni = vista contenente le particelle senza titolairtà ottenuta tramite join tra la vista ter_1 e la tabella partite_speciali_terreni.


## Schema catasto_fabbricati

**Tabelle**

**Viste**


 ## Schema cxf_in

**Tabelle**

- Acque = layer creato dal plugin cxf_in.
- Confine = layer creato dal plugin cxf_in.
- Fabbricati = layer creato dal plugin cxf_in.
- Fiduciali = layer creato dal plugin cxf_in.
- Linee = layer creato dal plugin cxf_in.
- Particelle = layer creato dal plugin cxf_in.
- Selezione = layer creato dal plugin cxf_in.
- Simboli = layer creato dal plugin cxf_in.
- Strade = layer creato dal plugin cxf_in.
- Testi = layer creato dal plugin cxf_in.

**Viste materializzate**

- particellare_persone_fisiche_mv: vista ottenuta tramite join tra titp_sogp_ter_persone_fisiche e il layer particelle utilizzando il campo in comune cm_fg_plla che identifica in modo univoco le particelle (persone fisiche). **Contiene le geometrie.**
- particellare_persone_giuridiche_mv: vista ottenuta tramite join tra titg_sogg_ter_persone_fisiche e il layer particelle utilizzando il campo in comune cm_fg_plla che identifica in modo univoco le particelle (persone giuridiche). **Contiene le geometrie.**
- particellare_partite_speciali_mv: vista ottenuta tramite join tra particelle_partite_speciali_terreni e il layer particelle utilizzando il campo in comune cm_fg_plla che identifica in modo univoco le particelle. **Contiene le geometrie.**


## Impostazioni iniziali del database
Il pimo step riguarda la creazione degli schemi catasto_terreni e catasto_fabbricati:

```sql
CREATE SCHEMA catasto_terreni;
CREATE SCHEMA catasto_fabbricati;
```
Lo schema cxf_in viene costruito in automatico dal plugin cxf_in. In caso si voglia importare i cxf in altro modo:

```sql
CREATE SCHEMA cxf_in;
```

Bisogna installare l'estensione postgis:

```sql
CREATE EXTENSION postgis;
```

Per verificare quale sia lo schema corrente, nel quale si sta lavorando, bisogna interrogare il search_path:

```sql
SHOW search_path;
```

Di default lo schema è il public. Per abilitare lo schema nel quale si vuole lavorare:

```sql
SET search_path TO schema; -- sostituire con il nome dello schema
```


## Elaborazione dei dati nello schema catasto_terreni
### Importazione dei dati censuari in PostgreSQL/PostGIS
#### Importazione della tabella .TER.
Il file è costituito da 4 differenti tipi record. La particella è identificata attraverso il campo IDENTIFICATIVO IMMOBILE. La presenza di diversi tipi di record può creare delle righe duplicate per ogni particella.

- TIPO RECORD 1: contiene le caratteristiche della particella. E' il record di interesse che verrà utilizzato per ricostruire il dato spaziale;

- TIPO RECORD 2: deduzioni della particella;

- TIPO RECORD 3: riserva della particella;

- TIPO RECORD 4: porzioni della particella.

##### 1) Creazione della tabella .ter contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi).

```sql
SET search_path TO catasto_terreni; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public

CREATE TABLE ter(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER NOT NULL,
  tipo_immobile TEXT,
  progressivo INTEGER,
  tipo_record INTEGER,
  foglio TEXT,
  numero TEXT,
  denominatore TEXT,
  subalterno TEXT,
  edificialita TEXT,
  qualita TEXT,
  classe TEXT,
  ettari INTEGER,
  are INTEGER,
  centiare INTEGER,
  flag_reddito TEXT,
  flag_porzione TEXT,
  flag_deduzioni TEXT,
  reddito_dominicale_lire INTEGER,
  reddito_agrario_lire INTEGER,
  reddito_dominicale_euro REAL,
  reddito_agrario_euro REAL,
  data_efficacia_valore_atto TEXT,
  data_registrazione_atti_valore_atto TEXT,
  tipo_nota_valore_atto TEXT,
  numero_nota_valore_atto TEXT,
  progressivo_nota_valore_atto TEXT,
  anno_nota_valore_atto TEXT,
  data_efficacia_registrazione_atto TEXT,
  data_registrazione_registrazione_atto TEXT,
  tipo_nota_registrazione_atto TEXT,
  numero_nota_registrazione_atto TEXT,
  progressivo_nota_registrazione_atto TEXT,
  anno_nota_registrazione_atto TEXT,
  partita TEXT,
  annotazione TEXT,
  identificativo_mutazione_iniziale TEXT,
  identificativo_mutazione_finale TEXT,
  codice_causale_atto_generante TEXT,
  descrizione_atto_generante TEXT,
  codice_causale_atto_conclusivo TEXT,
  descrizione_atto_conclusivo TEXT
);
```
##### 2) Importazione dei dati.
Convertire il file .TER in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/TER.csv). Inoltre è necessario sostituire il separatore decimale virgola (',') con il punto ('.'). Utilizzare un qualsiasi editor di testo (Notepad++, ATOM, ecc.).

#### Importazione della tabella .TIT.
Il file contiene un unico tipo di record.
##### 1) Creazione della tabella titolarità contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi).

```sql
CREATE TABLE tit
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione TEXT,
	identificativo_soggetto INTEGER NOT NULL,
	tipo_soggetto TEXT,
	identificativo_immobile INTEGER NOT NULL,
	tipo_immobile TEXT,
	codice_diritto TEXT,
	titolo_non_codificato TEXT,
	quota_numeratore_possesso INTEGER,
	quota_denominatore_possesso INTEGER,
	regime TEXT,
	soggetto_riferimento TEXT,
	data_validita_atto_generato TEXT,
	tipo_nota_generato TEXT,
	numero_nota_generato TEXT,
	progressivo_nota_generato TEXT,
	anno_nota_generato TEXT,
	data_registrazione_atti_generato TEXT,
	partita TEXT,
	data_validita_atto_concluso TEXT,
	tipo_nota_concluso TEXT,
	numero_nota_concluso TEXT,
	progessivo_nota_concluso TEXT,
	anno_nota_concluso TEXT,
	data_registrazione_atti_concluso TEXT,
	identificativo_mutazione_iniziale TEXT,
	identificativo_mutazione_finale TEXT,
	identificativo_titolarita INTEGER NOT NULL,
    codice_causale_atto_generante TEXT,
    descrizione_atto_generante TEXT,
    codice_causale_atto_conclusivo TEXT,
    descrizione_atto_conclusivo TEXT
);
```

##### 2) Importazione dei dati.
Convertire il file .TIT in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/TIT.csv). Inoltre è necessario sostituire il separatore decimale virgola (',') con il punto ('.'). Utilizzare un qualsiasi editor di testo (Notepad++, ATOM, ecc.).

#### Importazione della tabella .SOG.
Il file è costituito da 2 differenti tipi record. Il soggetto è identificato attraverso il campo IDENTIFICATIVO SOGGETTO.

- TIPO RECORD P: intestato a persona fisica;

- TIPO RECORD G: intestato a persona giuridica;

Per una corretta gestione del file è necessario suddividere il file .SOG in due tabella, una per ogni record. Tale operazione si può effettuare in excel, calc, ecc.

##### 1) Creazione della tabella per i soggetti fisici sogP, contenente tutti i campi (Per non crare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi);

```sql
CREATE TABLE sogp
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione	TEXT,
	identificativo_soggetto	INTEGER NOT NULL,
	tipo_soggetto TEXT,
	cognome TEXT,
	nome TEXT,
	sesso TEXT,
	data_nascita TEXT,
	codice_amministratvio_comune_nascita TEXT,
	codice_fiscale	TEXT,
	indicazioni_supplementari TEXT
);
```

##### 2) Importazione dei dati.
Convertire il file .SOG in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/SOGP.csv). Inoltre è necessario sostituire il separatore decimale virgola (',') con il punto ('.'). Utilizzare un qualsiasi editor di testo (Notepad++, ATOM, ecc.).

https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

##### 3) Creazione della tabella per i soggetti giuridici sogG, contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi);

```sql
CREATE TABLE sogg
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione TEXT,
	identificativo_soggetto  integer NOT NULL,
	tipo_soggetto TEXT,
	denominazione TEXT,
	codice_amministrativo_sede TEXT,
	codice_fiscale TEXT
);
```
##### 4) Importazione dei dati.
Convertire il file .SOG in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibile che i CSV abbiano l'header definito come da [esempio.csv](csv/SOGG.csv)

#### Creazione Tabelle aggiuntive
Le tabelle aggiuntive sono 3 e sono le qualità colturali, i codici di diritto e le partite speciali
##### Tabella delle qualità colturali
```sql
CREATE TABLE qualita
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	id_qualita TEXT,
	descrizione TEXT
);
```

Per inserire i valori utilizzare la funzione di PgAdmin per l'importazione dei CSV e utilizzare il file [qualita.csv](csv/qualita.csv)

##### Tabella dei codici di diritto
```sql
CREATE TABLE codici_diritto
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_diritto TEXT,
	descrizione TEXT
);
```
Per inserire i valori utilizzare la funzione di PgAdmin per l'importazione dei CSV e utilizzare il file [diritto.csv](csv/diritto.csv)

##### Tabella delle partite speciali
```sql
CREATE TABLE partite_speciali_terreni
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_partita TEXT,
	descrizione TEXT
);

INSERT INTO partite_speciali_terreni (codice_partita, descrizione)
VALUES
('1', 'aree di enti urbani e promiscui'),
('2', 'accessori comuni ad enti rurali e ad enti rurali e urbani'),
('3', 'aree di fabbricati rurali, o urbani da accertare, divisi in subalterni'),
('4', 'acque esenti da estimo'),
('5', 'strade pubbliche'),
('0', 'particelle soppresse')
```
### Elaborazioni dei file
#### Elaborazione della tabella .TER.
Le elaborazioni consistono nell'assegnazione della descrizione della qualità colturale contenuta nella tabella qualità e nella pulizia delle particelle partite speciali.
##### 1) Aggiornamento della descrizione della qualità colturale
```sql
ALTER TABLE ter
ADD COLUMN descrizione_qualita TEXT;

UPDATE ter as t
SET descrizione_qualita = q.descrizione
FROM qualita as q
WHERE t.qualita = q.id_qualita

```
##### 2) Creazione della vista contenente solo il TIPO RECORD 1.
Il tipo di record contiene la seguente codifica:
tipo di record | descrizione
:------ | :------
1   | contenente le informazioni descrittive della particella ed i suoi identificativi
2   | contenente le eventuali deduzioni
3   | contenente le riserve della particella
4   | contenente le porzioni della particella

```sql
CREATE OR REPLACE VIEW ter_1 AS
  SELECT * FROM ter
  WHERE tipo_record = '1';
```
##### 2) Pulizia della vista ter_1.
La Vista risultante dalla selezione del tipo_record = '1' può essere ulteriormente "pulita" eliminando quelle particelle che non hanno titolarità come le particelle soppresse, acque, strade. Le informazioni possono essere ricavate dal campo partita:

partita | descrizione
:------ | :------
1   | aree di enti urbani e promiscui
2   | accessori comuni ad enti rurali e ad enti rurali e urbani
3   | aree di fabbircati rurali, o urbani da accertare, divisi in subalterni
4   | acque esenti da estimo
5   | strade pubbliche
0   | particelle soppresse

Altre particelle che possono essere "pulite" sono quelle che contengono i subalterni.

```sql
CREATE OR REPLACE VIEW ter_1_clean AS
	SELECT * FROM ter_1
	WHERE ter_1.partita IS DISTINCT FROM '0' AND ter_1.partita IS DISTINCT FROM '4' AND ter_1.partita IS DISTINCT FROM '5' AND subalterno IS NULL;
```

N.B. se nella richiesta dei dati le partite speciali non sono state scaricate è possibile saltare il passaggio (bisogna tuttavia eliminare i subalterni). Nel file .PRM è possibile verificare la Tipologia di esportazione che può essere 1) Terreni completa ptaspec no oppure 2) Terreni completa ptaspec si. In entrambi casi il passaggio della pulizia della vista ter_1 non modifica il risultato. Si consiglia comunque di eseguire il passaggio.

#### Elaborazione della tabella .TIT.
Le elaborazioni consistono nell'assegnazione della descrizione dei codici di diritto contenuti nella tabella codici_diritto e nella creazione di due distinte tabelle, per le persone fisiche e per le persone giuridiche.
##### 1) Aggiornamento della descrizione della qualità colturale
```sql
ALTER TABLE tit
ADD COLUMN descrizione_diritto TEXT;

UPDATE tit as t
SET descrizione_diritto = d.descrizione
FROM codici_diritto as d
WHERE
t.codice_diritto = d.codice_diritto
```
##### 2) Creazione della vista titolarità per i soggetti fisici.

```sql
CREATE OR REPLACE VIEW titp AS
	SELECT * FROM tit
	WHERE tipo_soggetto = 'P';
```
##### 3) Creazione della vista titolarità per i soggetti giuridici.
```sql
CREATE OR REPLACE VIEW titg AS
	SELECT * FROM tit
	WHERE tipo_soggetto = 'G';
```
### Creazione delle relazioni tra i tipi di file: soggetti persone fisicihe (sogp) e titolarità persone fisiche (titp).
Ogni immobile (particella o fabbricato) può appartenere a più titolari. Per gestire questa relazione (uno a molti) è possibile utilizzare le funzioni di aggregazione. In questo specifico caso è la scelta è ricaduta sulla creazione di un json che contiene i diversi titolari appartenenti ad un dato immobile. Il vantaggio di utilizzare il json è che questo è interrogabile. La creazione della relazione viene fatta in due step.

#### 1) Creazione della vista. La relazione del tipo uno a molti viene esplicitata tramite il join. Il risultato duplicherà le righe relative all'immobile che appartiene a più soggetti.

```sql
CREATE OR REPLACE VIEW  titp_sogp AS SELECT
	row_number() OVER ()::integer AS gid,
	tit.identificativo_immobile,
	tit.tipo_immobile,
	tit.identificativo_soggetto,
	tit.tipo_soggetto,
	tit.descrizione_diritto as diritto,
	concat(tit.quota_numeratore_possesso, '/', tit.quota_denominatore_possesso) AS quota,
	sogp.cognome,
	sogp.nome,
	sogp.codice_fiscale,
	sogp.data_nascita
	FROM titp tit
	JOIN sogP ON tit.identificativo_soggetto = sogp.identificativo_soggetto
```

#### 2) Creazione della vista aggregata. Viene creata la colonna soggetto che contiene in un'unica riga tutti i titolari dell'immobile.

```sql
CREATE OR REPLACE VIEW titp_sogp_json AS SELECT
	row_number() OVER ()::integer AS gid,
	identificativo_immobile,
	tipo_immobile,
	json_agg
	(
		json_build_object
			(
				'identificativo_soggetto', identificativo_soggetto,
            			'cognome', cognome,
            			'nome', nome,
				'codice_fiscale', codice_fiscale,
				'data_nascita', data_nascita,
				'tipo_soggetto', tipo_soggetto,
				'quota', quota,
				'diritto', diritto
			)
	) as soggetto
FROM titp_sogp
GROUP by identificativo_immobile, tipo_immobile, tipo_soggetto;
```

### Creazione delle relazioni tra i tipi di file: soggetti giuridici (sogg) e titolarità soggetti giuridici (titg).
Ogni immobile (particella o fabbricato) può appartenere a più titolari. Per gestire questa relazione (uno a molti) è possibile utilizzare le funzioni di aggregazione. In questo specifico caso è la scelta è ricaduta sulla creazione di un json che contiene i diversi titolari appartenenti ad un dato immobile. Il vantaggio di utilizzare il json è che questo è interrogabile. La creazione della relazione viene fatta in due step.

#### 1) Creazione della vista. La relazione del tipo uno a molti viene esplicitata tramite il join. Il risultato duplicherà le righe relative all'immobile che appartiene a più soggetti.

```sql
CREATE OR REPLACE VIEW  titg_sogg AS SELECT
	row_number() OVER ()::integer AS gid,
	tit.identificativo_immobile,
	tit.tipo_immobile,
	tit.identificativo_soggetto,
	tit.tipo_soggetto,
	tit.descrizione_diritto as diritto,
	concat(tit.quota_numeratore_possesso, '/', tit.quota_denominatore_possesso) AS quota,
	sogg.denominazione,
	sogg.codice_amministrativo_sede,
	sogg.codice_fiscale
	FROM titg tit
	JOIN sogg ON tit.identificativo_soggetto = sogg.identificativo_soggetto
```

#### 2) Creazione della vista aggregata. Viene creata la colonna soggetto che contiene in un'unica riga tutti i titolari dell'immobile.

```sql
CREATE OR REPLACE VIEW titg_sogg_json AS SELECT
	row_number() OVER ()::integer AS gid,
	identificativo_immobile,
	tipo_immobile,
	json_agg
	(
		json_build_object
			(
				'denominazione', denominazione,
            			'codice_amministrativo_sede', codice_amministrativo_sede,
				'codice_fiscale', codice_fiscale,
				'tipo_soggetto', tipo_soggetto,
				'quota', quota,
				'diritto', diritto
			)
	) as soggetto
FROM titg_sogg
GROUP by identificativo_immobile, tipo_immobile, tipo_soggetto;
```

### Creazione delle relazioni tra i tipi di file: soggetti_titolarità persone fisiche (titp_sogp_json) e immobili (ter_1_clean).

```sql
CREATE OR REPLACE VIEW titp_sogp_ter_persone_fisiche AS
SELECT row_number() OVER ()::integer AS gid,
ter.identificativo_immobile AS identificativo_immobile,
ter.foglio,
ter.numero,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per il join con le geometrie del catasto
	WHEN length(ter.foglio) = 1 THEN concat(ter.codice_amministrativo, '_000', ter.foglio, '_', ter.numero)
    	ELSE
		(
			CASE
		 	WHEN length(ter.foglio) = 2 THEN concat(ter.codice_amministrativo, '_00', ter.foglio, '_', ter.numero)
		 	ELSE
				(
					CASE
			 		WHEN length(ter.foglio) = 3 THEN concat(ter.codice_amministrativo, '_0', ter.foglio, '_', ter.numero)
			 		ELSE
			 			(
							CASE
							WHEN length(ter.foglio) = 4 THEN concat(ter.codice_amministrativo, '_', ter.foglio, '_', ter.numero)
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
ter.descrizione_qualita AS qualita,
ter.classe,
ter.ettari,
ter.are,
ter.centiare,
j.soggetto
FROM ter_1_clean ter
JOIN titp_sogp_json j ON ter.identificativo_immobile = j.identificativo_immobile;
```

### Creazione delle relazioni tra i tipi di file: soggetti_titolarità persone giuridiche (titg_sogg_json) e immobili (ter_1_clean).

```sql
CREATE OR REPLACE VIEW titg_sogg_ter_persone_giuridiche AS
SELECT row_number() OVER ()::integer AS gid,
ter.identificativo_immobile AS identificativo_immobile,
ter.foglio,
ter.numero,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per il join con le geometrie del catasto
	WHEN length(ter.foglio) = 1 THEN concat(ter.codice_amministrativo, '_000', ter.foglio, '_', ter.numero)
    	ELSE
		(
			CASE
		 	WHEN length(ter.foglio) = 2 THEN concat(ter.codice_amministrativo, '_00', ter.foglio, '_', ter.numero)
		 	ELSE
				(
					CASE
			 		WHEN length(ter.foglio) = 3 THEN concat(ter.codice_amministrativo, '_0', ter.foglio, '_', ter.numero)
			 		ELSE
			 			(
							CASE
							WHEN length(ter.foglio) = 4 THEN concat(ter.codice_amministrativo, '_', ter.foglio, '_', ter.numero)
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
ter.descrizione_qualita AS qualita,
ter.classe,
ter.ettari,
ter.are,
ter.centiare,
j.soggetto
FROM ter_1_clean ter
JOIN titg_sogg_json j ON ter.identificativo_immobile = j.identificativo_immobile;
```


## Elaborazione dei dati nello schema catasto_fabbricati

### Importazione dei singoli file in PostgreSQL/PostGIS - .FAB (in costruzione).
### Importazione dei singoli file in PostgreSQL/PostGIS - .SOG (in costruzione).
### Importazione dei singoli file in PostgreSQL/PostGIS - .TIT (in costruzione).
### Creazione delle tabella aggiuntive per la codifica dei codici (in costruzione).

#### Tabella delle partite speciali.

```sql
CREATE TABLE partite_speciali_fabbricati
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_partita TEXT,
	descrizione TEXT
);

INSERT INTO partite_speciali_fabbricati (codice_partita, descrizione)
VALUES
('0', 'beni comuni censibili'),
('A', 'beni coumni non censibili'),
('R', 'fabbircati rurali'),
('C', 'unità immobiliari soppresse');
```

### Creazione delle relazioni tra i tipi di file: soggetti persone fisicihe (sogp) e titolarità persone fisiche (titp) (in costruzione).
### Creazione delle relazioni tra i tipi di file: soggetti giuridici (sogg) e titolarità soggetti giuridici (titg) (in costruzione).
### Creazione delle relazioni tra i tipi di file: soggetti_titolarità persone fisiche (titp_sogp_json) e FABBRICATI (in costruzione).
### Creazione delle relazioni tra i tipi di file: soggetti_titolarità persone giuridiche (titg_sogg_json) e FABBRICATI (in costruzione).


## Elaborazione dei dati nello schema cxf_in
Il plugin CXF_in importa i dati nel databse PostgreSQL/PostGIS nello schema cxf_in. Se si vuole, si possono copiare le tabelle in un altro schema (in questo modo consente di avere anche una copia di backup dei dati importati).
**Nota bene: può succedere che l'importazione nel DB attraverso il plugin CXF_in generi degli errori a causa di geometrie non valide, interrompendo il processo. In tal caso occorre importare dapprima le geometrie nel db spatialite (in modo da consentire la georeferenzazione), caricarle in QGIS, ripararle (tramite lo strumento ripara geometrie) e importarle in PostgreSQL/PostGIS (utilizzando il tool Esporta in PostgreSQL).**

![Error](/img/Error_PostGIS.JPG)

#### Copia delle tabelle in altro schema (OPZIONALE)

```sql
CREATE TABLE schema.acque AS (SELECT * FROM cxf_in.acque); -- sostituire il nome dello schema con quello desiderato (es. public).
CREATE TABLE schema.confine AS (SELECT * FROM cxf_in.confine);
CREATE TABLE schema.fabbricati AS (SELECT * FROM cxf_in.fabbricati);
CREATE TABLE schema.fiduciali AS (SELECT * FROM cxf_in.fiduciali);
CREATE TABLE schema.linee AS (SELECT * FROM cxf_in.linee);
CREATE TABLE schema.particelle AS (SELECT * FROM cxf_in.particelle);
CREATE TABLE schema.selezione AS (SELECT * FROM cxf_in.selezione);
CREATE TABLE schema.simboli AS (SELECT * FROM cxf_in.simboli);
CREATE TABLE schema.strade AS (SELECT * FROM cxf_in.strade);
CREATE TABLE schema.testi AS (SELECT * FROM cxf_in.testi);
```

### Creazione delle relazioni tra le geometrie particellari e i dati censuari (persone fisiche)

#### 1) Creazione dell'identificativo univoco di particella da utilizzare nel join con la vista tit_sog_ter_persone_fisiche
#### N.B. Se le geometrie sono state importate attraverso il tool Esporta in PostgreSQL di QGIS, è necessario specificare, negli script seguenti, la tabella Particelle tra doppi apici ("Particelle").

```sql
SET search_path TO cxf_in; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public;

ALTER TABLE Particelle
ADD COLUMN com_fg_plla TEXT;

UPDATE Particelle
SET com_fg_plla = CONCAT(codice_comune, '_',fg,'_', mappale);
```

#### 2) Join delle informazioni delle particelle e della titolarità relative ai soggetti fisici

```sql
CREATE OR REPLACE VIEW particellare_persone_fisiche AS -- Per questioni di performance sostituire con una Materialized View
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titp_sogp_ter_persone_fisiche j ON p.com_fg_plla = j.com_fg_plla
```

In alternativa con una materialized view (Consigliato)

```sql
CREATE MATERIALIZED VIEW particellare_persone_fisiche_MV AS
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titp_sogp_ter_persone_fisiche j ON p.com_fg_plla = j.com_fg_plla
WITH DATA
```

### Creazione delle relazioni tra le geometrie particellari e i dati censuari (persone giuridiche).

#### 1) Join delle informazioni delle particelle e della titolarità relative ai soggetti giuridici

```sql
CREATE OR REPLACE VIEW particellare_persone_giuridiche AS -- Per questioni di performance sostituire con una Materialized View
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titg_sogg_ter_persone_giuridiche j ON p.com_fg_plla = j.com_fg_plla
```

In alternativa con una materialized view (Consigliato)

```sql
CREATE MATERIALIZED VIEW particellare_persone_giuridiche_MV AS
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS foglio,
	p.mappale as particella,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.identificativo_immobile as identificativo_immobile,
	j.qualita,
	j.classe,
	j.ettari,
	j.are,
	j.centiare,
	j.soggetto,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.titg_sogg_ter_persone_giuridiche j ON p.com_fg_plla = j.com_fg_plla
WITH DATA
```


### Particelle senza titolarità
La tabella partite_speciali_terreni contiene le particelle che non hanno titolarità. Può tornare utile creare un layer geometrico distinto per tale categoria di particelle.

#### 1) Selezione e creazione della vista con la partite speciali terreni.

```sql
SET search_path TO catasto_terreni; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public;

CREATE OR REPLACE VIEW particelle_partite_speciali_terreni AS
SELECT ter_1.*,
	CASE -- nuova colonna che permette di assegnare un codice univoco per foglio e particella. Servirà per il join con le geometrie del catasto
	WHEN length(ter_1.foglio) = 1 THEN concat(ter_1.codice_amministrativo, '_000', ter_1.foglio, '_', ter_1.numero)
    	ELSE
		(
			CASE
		 	WHEN length(ter_1.foglio) = 2 THEN concat(ter_1.codice_amministrativo, '_00', ter_1.foglio, '_', ter_1.numero)
		 	ELSE
				(
					CASE
			 		WHEN length(ter_1.foglio) = 3 THEN concat(ter_1.codice_amministrativo, '_0', ter_1.foglio, '_', ter_1.numero)
			 		ELSE
			 			(
							CASE
							WHEN length(ter_1.foglio) = 4 THEN concat(ter_1.codice_amministrativo, '_', ter_1.foglio, '_', ter_1.numero)
                					END
						)
					END
				)
			END
		)
	END AS com_fg_plla,
p.descrizione as descrizione_partita
FROM ter_1
JOIN partite_speciali_terreni p ON ter_1.partita = p.codice_partita
WHERE ter_1.partita IN ('1', '2', '3', '4', '5', '0')
```

#### 2) Creazione delle geometrie

```sql
SET search_path TO cxf_in; -- IMPORTANTE: ABILITARE LO SCHEMA ESATTO, ALTRIMENTI LE TABELLE VERRANNO CREATE NELLO SCHEMA DI DEFAULT public;

CREATE MATERIALIZED VIEW particellare_partite_speciali_mv AS
SELECT row_number() OVER ()::integer AS gid,
	p.codice_comune AS codice_comune,
	p.fg AS fg,
	p.mappale as plla,
	CONCAT(p.codice_comune,'_', p.fg,'_', p.mappale) as fg_plla,
	j.*,
	p.geom as geom
FROM
	Particelle p
	JOIN catasto_terreni.particelle_partite_speciali_terreni j ON p.com_fg_plla = j.com_fg_plla
WITH DATA
```

### Estrazione delle particelle appartenenti ad un determinato soggetto.
Si vogliono estrarre, per esempio, le particelle di un dato comune. Bisogna interrogare il campo soggetto (che è un json array):

```sql
SELECT * FROM particellare_persone_giuridiche_mv WHERE (soggetto::jsonb @> '[{"denominazione": "VALORE"}]'); -- sotituire a VALORE il valore desiderato (es. COMUNE DI XXXX)
```

Per creare il particellare con il solo soggetto:

```sql
CREATE MATERIALIZED VIEW particellare_SOGGETTO_mv AS
SELECT * FROM particellare_persone_giuridiche_mv WHERE (soggetto::jsonb @> '[{"denominazione": "VALORE"}]') -- sotituire a VALORE il valore desiderato (es. COMUNE DI XXXX)
WITH DATA
```


## Verifica delle particelle

Nel file titolarità sono riportati i codici univoci degli immobili e dei titolari. Per conoscere quante particelle sono riportare nel file titolarità:

```sql
SELECT DISTINCT identificativo_immobile FROM tit WHERE tipo_immobile = 'T'
```

Per conoscere quante sono le particelle la cui titolerità riguarda soggetti fisici:

```sql
SELECT DISTINCT identificativo_immobile FROM tit WHERE tipo_immobile = 'T' AND tipo_soggetto = 'P'
```

Per conoscere quante sono le particelle la cui titolerità riguarda soggetti giuridici:

```sql
SELECT DISTINCT identificativo_immobile FROM tit WHERE tipo_immobile = 'T' AND tipo_soggetto = 'G'
```

La somma del numero delle particelle soggetti fisici e del numero delle particelle soggetti giuridici non è sempre uguale al numero totale degli immobili poiché alcune particelle potrebbero essere in comune tra i due gruppi.

## EXTRA
### Estrazione del geojson utilizzando ogr2ogr

```
ogr2ogr -f GeoJSON out.json "PG:host=myhost dbname=mydb user=ubuntu password=mypassword" \ -sql "select * from table"
```

### Verifica del Sistema di coordinate

```sql
SELECT ST_SRID(geom) FROM particellare_proprieta_comunale LIMIT 1;
```

### Riproiezione

```sql
CREATE MATERIALIZED VIEW particellare_proprieta_comunale_4326 AS
SELECT gid, codice_comune, foglio, particella, fg_plla, qualita, classe, ettari, are, centiare, soggetto,
	ST_Transform(geom,4326)
FROM
	particellare_proprieta_comunale
WITH DATA
```
