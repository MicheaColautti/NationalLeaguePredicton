# National League Analytics

Analisi dei dati e previsione dei risultati della National League svizzera di hockey su ghiaccio.

Progetto di Machine Learning presso **SUPSI - Dipartimento tecnologie innovative (DTI)**.

| Informazione | Dettaglio |
| --- | --- |
| Membri del gruppo | Michea Colautti e Claudia Grigorean |
| Classe | I3B |
| Anno accademico | 2026/2027 |
| Termine del progetto | 18 dicembre 2026 |

## Descrizione

Il progetto utilizza i dati delle partite della National League, raccolti dalla Swiss Ice Hockey Federation (SIHF) tramite la libreria Python SportsDataverse. L'obiettivo è studiare il rapporto tra forma recente, fattore campo, riposo e risultati, quindi valutare modelli per stimare il vincitore e i gol delle partite.

Il lavoro comprende la raccolta e la pulizia dei dati, la costruzione degli indicatori pregara, il confronto dei modelli e la valutazione delle previsioni su incontri successivi a quelli utilizzati per l'addestramento.

## Raccolta e preparazione dei dati

Lo script `update_national_league_corretto.py`:

- scarica calendario e risultati delle stagioni selezionate;
- aggiorna i dati già salvati ed elimina le partite duplicate tramite `game_id`;
- distingue gli incontri conclusi da quelli ancora da disputare;
- ricava i gol nei 60 minuti regolamentari e l'esito finale, inclusi eventuali supplementari o rigori;
- recupera le statistiche dettagliate di un numero limitato di partite per esecuzione;
- calcola indicatori basati sulle gare precedenti, come gol segnati e subiti, quota di vittorie, tiri nello specchio e giorni di riposo.

L'intervallo predefinito va dalla stagione **2021/22 alla stagione 2026/27**. Il parametro `season` usa l'anno di fine stagione: ad esempio, `2027` indica il 2026/27.

## Dataset generati

I file vengono salvati nella cartella `national_league_data/`.

| File | Contenuto |
| --- | --- |
| `matches.csv` | Una riga per partita: squadre, data, stato, risultati e gol per periodo. |
| `game_stats.csv` | Statistiche postpartita: una riga per statistica e incontro, con il valore della squadra di casa e di quella ospite. |
| `team_features.csv` | Indicatori pregara delle due squadre e risultati da prevedere, quando disponibili. |
| `ml_dataset.csv` | Copia identica di `team_features.csv`, destinata alle attività di machine learning. |

Con l'opzione `--players` viene prodotto anche `player_stats.csv`, contenente le statistiche dei giocatori recuperate per gli incontri elaborati.

### Convenzioni e dati mancanti

- `home` indica la squadra di casa; `away` quella ospite.
- Una cella vuota rappresenta un dato mancante, non uno zero.
- I gol nelle medie `gf_last5` e `ga_last5` si riferiscono ai 60 minuti regolamentari; le vittorie considerano l'esito finale.
- I tiri derivano dalla voce SIHF `SOG Total` e comprendono gli eventuali supplementari.
- Le medie dei tiri richiedono cinque valori validi su cinque. Le colonne `home_shots_last5_n` e `away_shots_last5_n` indicano quanti valori sono disponibili.
- Nel downloader, le finestre delle ultime cinque gare comprendono casa e trasferta e possono attraversare due stagioni.

I dataset includono anche gli incontri futuri presenti nel calendario SIHF. I relativi risultati restano mancanti; gli indicatori pregara vanno aggiornati man mano che vengono disputate altre partite.

La [guida alle colonne dei CSV e alle sigle SIHF](docs/Guida_dataset_National_League_SIHF.pdf) descrive i campi e le etichette delle statistiche.

## Modelli e valutazione

Il notebook contiene un blocco di previsione che legge `matches.csv` e ricostruisce autonomamente gli indicatori in ordine cronologico. Confronta un modello basato sul rating Elo, regressioni logistiche, modelli XGBoost e combinazioni delle loro probabilità, insieme a un riferimento basato sulla frequenza storica delle vittorie in casa.

Per i gol nei 60 minuti vengono confrontati modelli di Poisson, XGBoost con obiettivo Poisson e una loro combinazione.

La procedura:

1. Utilizza soltanto risultati precedenti alla data da prevedere;
2. Costruisce gli indicatori di ogni giornata prima di aggiornare lo storico con i risultati di quella giornata;
3. Confronta i modelli con una validazione temporale per stagione;
4. Riserva le ultime due stagioni disponibili a un test temporale separato dalla selezione;
5. Riaddestra i modelli scelti sullo storico precedente alla data richiesta e genera le previsioni.

La selezione del modello per il vincitore usa la **log loss**. Sono riportati anche **accuratezza** e **Brier score**. Per i gol vengono utilizzati **devianza di Poisson** e **errore assoluto medio (MAE)**.

Il blocco di previsione richiede almeno cinque stagioni di risultati precedenti alla data scelta. Ricostruisce forma nelle ultime 5, 10 e 20 gare, rendimento stagionale, rating Elo e riposo. Le sue finestre di forma vengono azzerate al cambio di stagione. Attualmente non utilizza i tiri, perché la copertura dello storico dettagliato è incompleta.

## Installazione con Anaconda e VS Code

Eseguire i comandi seguenti nel terminale. Se l'ambiente `sports_sihf` esiste già, saltare il comando di creazione.

```bash
conda create -n sports_sihf python=3.12 pip -y
conda activate sports_sihf

python -m pip install --upgrade pip setuptools wheel
python -m pip install --upgrade "sportsdataverse==0.2.0b1"
python -m pip install pandas numpy scipy pyarrow matplotlib seaborn scikit-learn xgboost requests tqdm ipykernel

python -m ipykernel install --user --name sports_sihf --display-name "Python (sports_sihf)"

python -c "from sportsdataverse.hockey.sihf import sihf_results, sihf_game_team_stats, sihf_game_player_stats; print('Import SIHF riuscito!')"
```

In VS Code installare le estensioni **Python** e **Jupyter** di Microsoft. Per i notebook selezionare il kernel **Python (sports_sihf)**; per gli script selezionare lo stesso ambiente tramite **Python: Select Interpreter**.

## Utilizzo

### Aggiornare i dataset

Dalla cartella del progetto, con l'ambiente attivo:

```bash
python update_national_league_corretto.py --start-season 2022 --end-season 2027 --max-details 100
```

| Parametro | Significato |
| --- | --- |
| `--start-season` | Prima stagione, indicata con l'anno di fine. Valore predefinito: `2022`. |
| `--end-season` | Ultima stagione, indicata con l'anno di fine. Valore predefinito: `2027`. |
| `--max-details` | Numero massimo di nuovi incontri per cui richiedere le statistiche dettagliate. Valore predefinito: `100`. |
| `--players` | Recupera anche le statistiche dei giocatori per gli incontri elaborati. |
| `--out` | Cartella di destinazione. Valore predefinito: `national_league_data`. |

Le richieste sono distanziate per limitare il carico sul servizio. Rieseguendo lo script si amplia lo storico dei dettagli, partendo dagli incontri conclusi più recenti ancora da recuperare. L'aggiornamento avviene a ogni esecuzione, senza una pianificazione automatica in background.

### Generare le previsioni

Aprire `DownlaodAndUpdate.ipynb` in VS Code e utilizzare il blocco autonomo di previsione dopo aver aggiornato i CSV.

Nel blocco configurare:

```python
DATA_PREVISIONE = "2026-10-30"  # Esempio: sostituire con la giornata da analizzare.
DATA_DIR = Path("national_league_data")
OUTPUT_DIR = Path("national_league_predictions")
```

La data attualmente presente nel notebook è il **30 ottobre 2026**: non cambia automaticamente con l'orologio del computer. La giornata scelta deve essere presente nel calendario salvato in `matches.csv`. Impostare `DATA_DIR` sulla propria cartella dei dati, evitando di dipendere da percorsi personali presenti nel codice.

Le esportazioni comprendono:

- `previsioni_YYYY-MM-DD.csv`: vincitore previsto, probabilità delle due squadre, gol attesi nei 60 minuti e punteggio indicativo;
- `confronto_modelli.csv`: confronto dei modelli per il vincitore in validazione;
- `confronto_modelli_gol.csv`: confronto dei modelli per i gol in validazione;
- `verifica_test_temporale.csv`: prestazioni del modello scelto nelle stagioni di test;
- `report_YYYY-MM-DD.json`: configurazione, metriche, indicatori utilizzati e versioni delle librerie.

## Limiti e interpretazione

La disponibilità dei dati dipende dalla fonte e dal numero di statistiche dettagliate già recuperate. Infortuni, formazioni e portieri titolari non sono presenti negli input del modello attuale.

La probabilità di vittoria riguarda l'esito finale, inclusi supplementari e rigori. Il punteggio indicativo deriva da un'approssimazione con distribuzioni di Poisson: la probabilità del vincitore non coincide con quella del punteggio esatto.

I risultati effettivi della partita e le sue statistiche postpartita devono essere esclusi dagli input usati per prevedere quella stessa partita. Le prestazioni vanno valutate su dati successivi a quelli di addestramento e confrontate con il modello di riferimento.

## Riferimenti

- [Swiss Ice Hockey Federation (SIHF)](https://www.sihf.ch/)
- [SportsDataverse Python](https://github.com/sportsdataverse/sportsdataverse-py)
- [scikit-learn](https://scikit-learn.org/)
- [XGBoost](https://xgboost.readthedocs.io/)
