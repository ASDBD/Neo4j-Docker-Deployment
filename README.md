# Laboratorio Neo4j con Docker e Jupyter

Ambiente pronto all'uso per un laboratorio didattico su **Neo4j**, il database a grafo. Con un solo comando si avviano due container:

| Servizio | Immagine | Indirizzo | A cosa serve |
|---|---|---|---|
| `neo4j` | `neo4j:2026.10.1-community` | <http://localhost:7474> (Browser), `neo4j://localhost:7687` (Bolt) | il database |
| `jupyter` | Python 3.13 + JupyterLab, costruita da [`jupyter/Dockerfile`](jupyter/Dockerfile) | <http://localhost:8888> | il notebook del laboratorio |

## Requisiti

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) (o Docker Engine con il plugin Compose)
* circa 3 GB di spazio libero e 2 GB di RAM disponibili per Docker

## Avvio

```bash
docker compose up -d --build
```

Al primo avvio Docker scarica l'immagine di Neo4j e le librerie Python: può richiedere qualche minuto. Il container Jupyter parte solo quando Neo4j è pronto.

Poi si apre <http://localhost:8888> e quindi il notebook **`laboratorio_neo4j.ipynb`**. Le celle vanno eseguite in ordine (oppure con *Run → Run All Cells*).

## Contenuto del laboratorio

Il notebook ([`notebooks/laboratorio_neo4j.ipynb`](notebooks/laboratorio_neo4j.ipynb)) è in italiano e copre:

1. il modello a grafo di proprietà (nodi, etichette, relazioni, proprietà);
2. la connessione con il driver Python ufficiale;
3. le operazioni CRUD in Cypher: `CREATE`, `MATCH`, `SET`, `REMOVE`, `DELETE`, `MERGE`;
4. parametri e transazioni ACID (commit e rollback);
5. vincoli di unicità e indici;
6. l'importazione di dati da CSV con `LOAD CSV` e `CALL { } IN TRANSACTIONS`;
7. interrogazioni: filtri, aggregazioni, `WITH`, `OPTIONAL MATCH`, sottoquery `COUNT { }` / `EXISTS { }`;
8. la navigazione del grafo: raccomandazioni "amici di amici", cammini di lunghezza variabile, `SHORTEST`;
9. la visualizzazione del grafo con `neo4j-viz` e NetworkX;
10. l'uso di Neo4j Browser.

Il dataset di esempio, nella cartella [`import/`](import/), descrive un ateneo **fittizio**: 3 dipartimenti, 6 docenti, 10 corsi, 30 studenti con iscrizioni, voti e amicizie.

## Neo4j Browser

Aprire <http://localhost:7474> e connettersi con:

* **URL**: `neo4j://localhost:7687`
* **utente**: `neo4j`
* **password**: `laboratorio`

## Configurazione

Le impostazioni predefinite si possono cambiare creando un file `.env` a partire da [`.env.example`](.env.example):

```bash
cp .env.example .env
```

* `NEO4J_PASSWORD` – password dell'utente `neo4j` (almeno 8 caratteri). Viene impostata solo alla **prima** creazione del database: per cambiarla dopo, ricrea il volume (vedi sotto).
* `JUPYTER_PORT` – porta locale di JupyterLab, utile se la `8888` è già occupata.

JupyterLab è avviato **senza token** e risponde solo su `127.0.0.1`: è pensato per l'uso sul proprio computer, non per essere esposto in rete.

## Comandi utili

Fermare i container (i dati restano):

```bash
docker compose down
```

Ripartire da zero, cancellando anche il database:

```bash
docker compose down -v
```

Leggere i log di Neo4j:

```bash
docker compose logs -f neo4j
```

Aprire una shell Cypher a riga di comando:

```bash
docker exec -it neo4j-lab cypher-shell -u neo4j -p laboratorio
```

## Struttura del repository

```
.
├── docker-compose.yml        # definizione dei due servizi
├── .env.example              # impostazioni modificabili
├── import/                   # file CSV, montati in /import nel container Neo4j
├── jupyter/
│   ├── Dockerfile            # immagine JupyterLab
│   └── requirements.txt      # versioni fissate delle librerie Python
└── notebooks/
    └── laboratorio_neo4j.ipynb
```

I notebook nella cartella `notebooks/` sono montati nel container: le modifiche fatte da JupyterLab vengono salvate direttamente nel repository.

## Problemi comuni

* **`port is already allocated`**: un altro programma usa la porta. Per Jupyter si imposta `JUPYTER_PORT` nel file `.env`; per Neo4j si modificano le porte `7474`/`7687` in `docker-compose.yml`.
* **`Unauthorized` / autenticazione fallita**: la password nel file `.env` non corrisponde a quella con cui è stato creato il database. Occorre eseguire `docker compose down -v` e riavviare.
* **Il grafo interattivo non compare**: JupyterLab non esegue il JavaScript dei notebook "non fidati" aperti con output già salvati. Basta rieseguire la cella.
