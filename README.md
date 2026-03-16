# Sentiment-Driven Financial Analysis

> Questo progetto studia se il sentiment giornaliero estratto da fonti online (Reddit e GDELT) contenga informazioni utili per anticipare i rendimenti futuri di tre asset finanziari: l'ETF sull'oro **GLD**, il titolo azionario **NVIDIA (NVDA)** e il titolo **Amazon (AMZN)**.

---

## Obiettivo

L'ipotesi di partenza è che la polarità emotiva dei contenuti pubblicati online — post su Reddit o articoli di stampa indicizzati da GDELT — possa correlare con i movimenti di prezzo successivi degli asset considerati. Si verifica in particolare se la relazione sia **direzionale** (sentiment positivo → rendimenti positivi) oppure **contrarian** (sentiment positivo → rendimenti negativi, per effetto del cosiddetto "euforia da picco").

---

## Asset analizzati

| Notebook | Asset | Fonte sentiment | Finestra temporale |
|---|---|---|---|
| `GLD_analysis.ipynb` | GLD (ETF oro) | Reddit | ~10 anni |
| `NVDA_analysis.ipynb` | NVIDIA (NVDA) | GDELT | 90 giorni (Dic 2025 – Mar 2026) |
| `AMZN_analysis.ipynb` | Amazon (AMZN) | Reddit + GDELT | ~180 giorni (Set 2025 – Mar 2026) |

---

## Struttura del repository

```
.
├── GLD_analysis.ipynb              # Analisi sentiment Reddit su GLD
├── NVDA_analysis.ipynb             # Analisi sentiment GDELT su NVDA
├── AMZN_analysis.ipynb             # Analisi sentiment Reddit + GDELT su AMZN
│
├── reddit_gold_posts_10y.csv       # Post Reddit grezzi - GLD
├── reddit_gold_posts_10y_clean.csv # Post Reddit puliti - GLD
├── daily_gold_sentiment.csv        # Sentiment giornaliero - GLD
├── yfinance_gold_10y.csv           # Prezzi storici GLD
│
├── gdelt_events_90d_nvidia.csv         # Eventi GDELT grezzi - NVDA
├── gdelt_events_90d_nvidia_clean.csv   # Dopo deduplicazione
├── gdelt_events_90d_nvidia_text.csv    # Testi estratti dagli URL
├── gdelt_events_90d_nvidia_daily_sentiment.csv
├── yfinance_nvda_90d.csv
│
├── reddit_amzn_posts_180d.csv
├── reddit_amzn_posts_180d_clean.csv
├── daily_amzn_sentiment_a.csv          # Sentiment Reddit - AMZN
├── gdelt_events_180d_amzn.csv
├── gdelt_events_180d_amzn_clean.csv
├── gdelt_events_180d_amzn_text.csv
├── gdelt_events_180d_amzn_daily_sentiment.csv
└── yfinance_amzn_180d.csv
```

---

## Fonti dei dati

### Reddit (PRAW)
I post vengono estratti tramite le API ufficiali di Reddit usando la libreria **PRAW**. Per ciascun asset vengono interrogati i subreddit più rilevanti:

- **GLD**: `r/Gold`, `r/GoldInvestor`, `r/investing`
- **AMZN**: `r/stocks`, `r/StockMarket`, `r/investing`, `r/wallstreetbets`

Le query combinano ticker, nome dell'asset e alias comuni (es. `"AMZN" OR "Amazon stock"`). Per ogni post vengono raccolti: subreddit di provenienza, timestamp UTC, titolo, corpo del testo, score e numero di commenti.

### GDELT Event Database 2.0
GDELT indicizza quotidianamente centinaia di migliaia di eventi da media globali. Per ogni giorno viene scaricato il file `.export.CSV.zip` dal server pubblico, filtrato per le righe che contengono riferimenti al nome/ticker dell'azienda. Gli URL delle notizie vengono poi visitati per estrarre tramite **BeautifulSoup** il testo effettivo dell'articolo (titolo + primo paragrafo informativo, max 250 parole).

### Yahoo Finance (yfinance)
I prezzi OHLCV giornalieri vengono scaricati tramite **yfinance** per la finestra temporale corrispondente ai dati di sentiment disponibili.

---

## Pipeline di elaborazione

```
Raccolta dati grezzi
        │
        ▼
Pulizia e filtraggio
(lunghezza testo ≥ 50 char, score ≥ 1,
 blacklist spam: Telegram, Discord, segnali trading)
        │
        ▼
Classificazione FinBERT
(ProsusAI/finbert → score polarità giornaliero in [-1, +1])
        │
        ▼
Merge con prezzi yfinance
(sentiment al giorno t accoppiato con rendimenti da t+1,
 per eliminare ogni look-ahead bias)
        │
        ▼
Suite di test statistici
```

---

## Modello di sentiment: FinBERT

Il modello usato per classificare i testi è **[ProsusAI/finbert](https://huggingface.co/ProsusAI/finbert)**, un BERT fine-tuned su corpus finanziari. Per ogni testo produce tre probabilità (positive, negative, neutral); lo score di polarità è calcolato come:

```
score = P(positive) - P(negative)   ∈ [-1, +1]
```

Il sentiment giornaliero è la media degli score di tutti i testi associati a quella data.

---

## Test statistici applicati

| Test | Scopo |
|---|---|
| ADF Stationarity | Verifica che le serie siano stazionarie prima delle analisi |
| Correlazioni Pearson e Spearman | Misura lineare e monotona tra sentiment e rendimenti futuri |
| Regressione OLS | Quantifica il coefficiente β e il potere esplicativo R² |
| Analisi Lead-Lag (±10/15 giorni) | Individua se è il sentiment a precedere i rendimenti o viceversa |
| Granger Causality | Test formale di causalità temporale tra le due serie |
| Event-Based (regimi Bull/Bear) | Confronta i rendimenti medi dopo giorni di sentiment estremo |
| T-Test HighBull vs HighBear | Verifica se i due regimi producono rendimenti statisticamente diversi |
| Backtest strategie | Simula portafogli basati sul segnale sentiment vs Buy & Hold |
| Rolling Correlation | Controlla la stabilità temporale della correlazione |

Gli orizzonti temporali analizzati variano da 1 a 40–60 giorni lavorativi per ciascun asset.

---

## Librerie utilizzate

| Libreria | Utilizzo |
|---|---|
| `praw` | Accesso alle API Reddit |
| `requests` + `BeautifulSoup` | Download e parsing HTML degli articoli GDELT |
| `transformers` (Hugging Face) | Caricamento e inferenza con FinBERT |
| `yfinance` | Download prezzi storici |
| `pandas` / `numpy` | Manipolazione e calcolo dei dati |
| `scipy.stats` | Correlazioni, regressioni, t-test |
| `statsmodels` | Granger causality, ADF test |
| `matplotlib` | Visualizzazioni |
| `tqdm` | Barre di avanzamento durante lo scraping e l'inferenza |

---

## Come riprodurre l'analisi

1. Installare le dipendenze:
   ```bash
   pip install praw transformers yfinance pandas numpy scipy statsmodels matplotlib beautifulsoup4 tqdm
   ```

2. Configurare le credenziali Reddit nel codice (variabili `client_id`, `client_secret`, `user_agent`).

3. Eseguire i notebook nell'ordine:
   1. `GLD_analysis.ipynb`
   2. `NVDA_analysis.ipynb`
   3. `AMZN_analysis.ipynb`

   Ogni notebook è autonomo: dalla raccolta dati fino ai test statistici finali.

> **Nota**: il download dei file GDELT e lo scraping degli URL richiedono tempo (rate limiting a 0.5s/richiesta). Per i modelli FinBERT è consigliata una GPU o almeno 8 GB di RAM.
