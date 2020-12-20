# Import-dati-censuari-in-PostgreSQL/PostGIS

<h4>Breve descrizione dei dati catastali censuari</h4>
Le informazioni descritte in questa sezione derivano dal documento a cura dell'Agenzia dell'Entrate (DOC. ES-23-IS-05) liberamente consultabile all'indirizzo https://wwwt.agenziaentrate.gov.it/mt/ServiziComuniIstituzioni/ES-23-IS-05_100909.pdf<br>
Per maggiori dettagli si può consultare https://www.agenziaentrate.gov.it/portale/web/guest/schede/fabbricatiterreni/portale-per-i-comuni/servizi-portale-dei-comuni/estrazione-dati-catastali<br>
I dati censuari sono costituiti da 4 tipi di file:<br>
- file fabbricati (.FAB);<br>
- file terreni (.TER);<br>
- file soggetti (.SOG);<br>
- file titolarità (.TIT)<br>
Ogni tipo di file è costituito da una tabella che può contenere diversi tipi di record. Il collegamento tra i tipi di file è assicurato dalla presenta di chiavi specifiche:<br>
- .FAB/.TER contengono la chiave identificativo immobile
- .SOG contiene la chiave identificativo soggetto
- .TIT contiene sia la chiave identificativo immobile che la chiave identificativo soggetto
<h5> Descrizione dei singoli tipi di file</h5>