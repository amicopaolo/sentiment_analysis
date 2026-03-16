# Risultati e Conclusioni

> Questo documento riassume i risultati dei test statistici applicati su tre asset (GLD, NVDA, AMZN) e le conclusioni che se ne traggono sull'utilità predittiva del sentiment online per i rendimenti finanziari.

---

## Metodologia comune

Per tutti e tre gli asset il sentiment giornaliero viene sempre accoppiato con i rendimenti **futuri** (da t+1 in poi), garantendo l'assenza di look-ahead bias. I rendimenti forward sono calcolati su orizzonti multipli: da 1 a 60 giorni lavorativi. I test statistici principali sono nove: ADF stationarity, correlazioni Pearson/Spearman, regressione OLS, analisi lead-lag, Granger causality, event-based, t-test Bull vs Bear, backtest strategie, rolling correlation.

---

## GLD — ETF sull'oro (fonte: Reddit)

**Dataset:** 669 post puliti, 248 giorni di serie temporale, 82 giorni con sentiment ≠ 0. Finestra: ~10 anni di prezzi GLD.

### Correlazioni e regressione
Nel breve termine (1–20 giorni) nessuna correlazione raggiunge la significatività statistica: tutti i p-value superano 0.12 e i coefficienti oscillano attorno allo zero senza direzione coerente (massimo in valore assoluto: Pearson r = −0.116 a 20 giorni, p = 0.120). Nel lungo termine (30–60 giorni) la correlazione è sistematicamente negativa — a 40 giorni r = −0.154 (p = 0.051) — ma non supera mai la soglia convenzionale del 5%. La regressione OLS conferma: β massimo di −0.030 a 40 giorni con R² = 2.4%, sostanzialmente privo di potere esplicativo.

### Lead-Lag e Granger
L'unico segnale significativo dell'intera analisi è il **lag = −3** (r = −0.314, p = 0.004), che indica una relazione inversa: i rendimenti di 3 giorni prima correlano negativamente con il sentiment di oggi. In altre parole, **è il mercato a guidare Reddit**, non il contrario: dopo un calo di GLD, la community diventa più negativa circa 3 giorni dopo. I test di Granger confermano questa direzione unilaterale.

### Event-Based e Backtest
I regimi HighBull e HighBear non producono rendimenti medi statisticamente diversi da zero su nessun orizzonte (tutti i p > 0.21). La strategia "long solo nei giorni di sentiment bullish" genera un rendimento cumulato di **−7.29%** (Sharpe −1.430), mentre il Buy & Hold sullo stesso periodo rende **+19.95%** (Sharpe 1.080). La rolling correlation oscilla tra −0.657 e +0.916 con appena 2 finestre su 162 (1.2%) statisticamente significative.

### Conclusione GLD
Il sentiment Reddit non contiene informazione predittiva utile sui rendimenti di GLD. Il segnale contrarian a lungo termine è appena al limite della significatività e non si consolida su campioni aggiornati. GLD è un asset strutturalmente poco sensibile al sentiment retail: è guidato da dinamiche macro (dollaro, tassi reali, domanda istituzionale) che i post su Reddit non catturano.

---

## NVDA — NVIDIA Corporation (fonte: GDELT)

**Dataset:** 3.047 testi estratti da GDELT, 90 giorni di copertura completa (Dic 2025 – Mar 2026), 60 trading days. Entrambe le serie sono stazionarie (ADF p ≈ 0.000).

### Correlazioni e regressione
Nel brevissimo termine (1–3 giorni) il segnale è assente: a 1 giorno r = −0.016, p = 0.90. A partire da **5 giorni** emerge un segnale contrarian forte e coerente: r = −0.442 (p < 0.001), confermato da Spearman r = −0.465 (p = 0.0003). Il picco è a **40 giorni**: r = −0.557, p = 0.009, **R² = 31%**. La regressione OLS stima che un aumento di 1 punto nel sentiment sia associato a un rendimento inferiore di circa 24 punti percentuali su 40 giorni.

### Lead-Lag e Granger
La struttura è **bidirezionale**. A lag negativi (−1, −2) la correlazione è positiva: i rialzi di NVDA migliorano il sentiment online nei giorni successivi. A lag +3 la correlazione è negativa (r = −0.337, p = 0.010): il sentiment alto oggi anticipa un calo 3 giorni dopo. Il test di Granger conferma la direzione dominante Ret → Sent (p = 0.006 a lag 1) ma sfiora la significatività anche nella direzione Sent → Ret a lag 3 (p = 0.061), inedito rispetto a GLD.

### Event-Based e T-Test
Dopo i giorni di **sentiment molto negativo (HighBear)**, NVDA rimbalza in media del **+2.62% a 5 giorni** (p = 0.021) e **+2.95% a 15 giorni** (p = 0.001). Il confronto diretto HighBull vs HighBear produce una differenza di **−4.58 punti percentuali a 5 giorni** (p = 0.006) e **−3.18 pp a 15 giorni** (p = 0.011). Questi sono i risultati più robusti dell'intero progetto: sopravvivono alla correzione per test multipli di Bonferroni.

### Backtest e Rolling Correlation
Le strategie a breve termine falliscono perché operano sull'orizzonte sbagliato: la strategia "long sui giorni bullish" perde −1.66% contro un Buy & Hold di −4.17% (periodo ribassista). La rolling correlation (finestra 20 giorni, vs rendimento a 5 giorni) ha media **−0.577** con il **75% delle finestre significative** (p < 0.05) — valore incomparabile con il 1.2% di GLD — indicando che la relazione contrarian è strutturalmente stabile nel campione.

### Conclusione NVDA
NVDA è l'asset più reattivo al sentiment del progetto. La relazione contrarian esiste, è statisticamente robusta e stabile nel tempo, con R² fino al 31% sugli orizzonti medi. Il segnale operativo ottimale è: **long dopo giorni di sentiment negativo estremo, con holding period di 5–15 giorni**. Prima di un utilizzo operativo resta necessaria una validazione out-of-sample su un periodo esteso.

---

## AMZN — Amazon.com (fonti: Reddit + GDELT)

**Dataset Reddit:** 279 post grezzi → 203 puliti, 117 giorni di serie.
**Dataset GDELT:** 19.477 eventi → 5.886 URL unici → 5.147 testi estratti, finestra Set 2025 – Mar 2026.

### Reddit: segnale assente
Le correlazioni tra sentiment Reddit e rendimenti AMZN sono piccole e instabili su tutti gli orizzonti: coefficienti di Pearson tra −0.06 e +0.09 con p-value sempre > 0.43, R² sistematicamente sotto l'1%. Il t-test Bull vs Bear non raggiunge mai la significatività (tutti p > 0.38). Il test di Granger mostra solo la direzione Ret → Sent come formalmente significativa (p = 0.044 a lag 1), non il contrario. Il backtest contrarian Reddit chiude a **−8.11%** (Sharpe −0.565) contro un Buy & Hold di −5.25%.

### GDELT: segnale contrarian parziale
GDELT mostra un comportamento più informativo su orizzonti medi e lunghi. La regressione OLS identifica β = −0.13 a 10 giorni (p = 0.018, R² ≈ 7.4%), β = −0.17 a 40 giorni (p = 0.013, R² ≈ 13.5%). Il segno negativo è coerente con un effetto contrarian: un tone mediatico positivo su Amazon si associa a rendimenti inferiori nelle settimane successive. L'analisi lead-lag evidenzia tre lag significativi: −3 (r = −0.24, p = 0.033), −1 (r = 0.21, p = 0.050) e +6 (r = −0.24, p = 0.033). Il backtest contrarian GDELT ottiene **+6.42%** (Sharpe 0.808) contro −5.25% del Buy & Hold nello stesso periodo.

### Rolling Correlation
La rolling correlation a 20 giorni è debolmente negativa per entrambe le fonti (media ≈ −0.03) con alta variabilità. Reddit mostra il 47% di finestre negative e il 15% con r < −0.3; GDELT ha il 59% di finestre negative ma con deviazione standard molto più contenuta (0.121 vs 0.268), indicando un segnale più stabile anche se di modesta entità.

### Conclusione AMZN
Su AMZN, Reddit si comporta essenzialmente come rumore, mentre GDELT fornisce un segnale contrarian debole ma presente su orizzonti da 10 a 40 giorni. Il confronto diretto tra le due fonti conferma che **la stampa globale (GDELT) è più informativa dei social media (Reddit)** per questo tipo di asset, probabilmente perché Amazon è un titolo ad alta copertura istituzionale, più sensibile a notizie strutturate che a opinioni retail.

---

## Confronto generale

| Asset | Fonte | Segnale | Orizzonte ottimale | R² max | Backtest vs B&H |
|---|---|---|---|---|---|
| GLD | Reddit | Assente | — | 2.4% (40d) | −7.29% vs +19.95% |
| NVDA | GDELT | Contrarian forte | 5–15 giorni | 31.0% (40d) | — |
| AMZN | Reddit | Assente | — | < 1% | −8.11% vs −5.25% |
| AMZN | GDELT | Contrarian debole | 10–40 giorni | 13.5% (40d) | +6.42% vs −5.25% |

Tre osservazioni trasversali emergono dal confronto:

1. **GDELT supera Reddit come fonte predittiva**: il contenuto di articoli di stampa strutturati porta più informazione dei post social per tutti gli asset testati.
2. **La relazione è contrarian, non direzionale**: in nessun caso il sentiment alto anticipa rendimenti positivi; dove un segnale esiste, è sempre inverso.
3. **L'orizzonte conta**: le strategie giornaliere falliscono sistematicamente. Il segnale, dove presente, opera su orizzonti da 5 a 40 giorni lavorativi.

---

## Limiti

- **Campioni ridotti**: GLD ha solo 82 giorni con sentiment ≠ 0; NVDA copre appena 90 giorni. La soglia minima consigliata per test statistici robusti è 200–300 osservazioni.
- **Test multipli**: con 9+ test per asset e più orizzonti, il rischio di falsi positivi è elevato. Applicando la correzione di Bonferroni (soglia p < 0.003), solo il lead-lag di GLD a lag −3 e i risultati event-based di NVDA a 5–15 giorni sopravvivono.
- **Assenza di out-of-sample**: tutti i risultati sono in-sample. La validità predittiva reale è da verificare su dati futuri.
- **Fonte singola e metrica singola**: l'aggregazione in un unico score giornaliero medio comprime molta informazione (volume dei post, sentiment delle news più citate, sentiment ponderato per engagement).
- **GLD è strutturalmente poco sensibile al sentiment retail**: un asset guidato da variabili macro (dollaro, tassi reali) difficilmente risponde a post su Reddit.

---

## Raccomandazioni future

- Estendere la raccolta a 2–3 anni di sentiment giornaliero continuo (500+ osservazioni)
- Aggiungere fonti alternative: Twitter/X, Google Trends, news via API finanziarie
- Integrare variabili di controllo (momentum, VIX, volumi) nelle regressioni
- Applicare modelli non lineari (Random Forest, Gradient Boosting) per catturare relazioni non monotone
- Implementare una validazione out-of-sample con walk-forward testing
- Ponderare il sentiment per engagement (upvotes, numero di commenti) anziché usare la media semplice

---

## Strumento per replicare i test

Tutti i test statistici utilizzati in questa analisi sono disponibili nella libreria Python open-source **[sentimentlab](https://pypi.org/project/sentimentlab/)**, sviluppata come parte di questo progetto. Permette di caricare un file di sentiment giornaliero e un file di prezzi yfinance, eseguire l'intera suite di 9 test statistici e simulare strategie di backtest, il tutto con poche righe di codice.

```bash
pip install sentimentlab
```

```python
import sentimentlab as sl

sent   = sl.load_sentiment_csv("gdelt_daily_sentiment.csv")
prices = sl.load_finance_csv("yfinance_nvda_90d.csv")
df     = sl.merge_sentiment_finance(sent, prices)

print(sl.test_pearson_spearman(df))
print(sl.test_ols_regression(df))
print(sl.test_granger_causality(df))
```
