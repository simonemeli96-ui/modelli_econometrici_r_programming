# Analisi dei Determinanti Salariali e Gender Pay Gap (R)
## Regressione Lineare sul Salario — Modelli Econometrici Progressivi

## Descrizione del progetto
Analisi econometrica dei fattori che determinano il salario mensile 
dei lavoratori italiani, con focus sul **gender pay gap**. Il progetto 
costruisce progressivamente 6 modelli di regressione OLS con errori 
standard robusti all'eteroschedasticità (HC1), testando il contributo 
incrementale di ogni variabile esplicativa, il fenomeno del **bias da 
variabile omessa** e l'effetto delle **interazioni** tra genere e 
inquadramento professionale.

---

## Domanda di ricerca
Quali fattori determinano il salario mensile? Quanto del gender pay gap 
è spiegato da differenze oggettive di carriera (età, anzianità, 
inquadramento) e quanto rimane come divario residuo non spiegato 
a parità di condizioni?

---

## Dataset
- **File:** ANALISI_WAGE.csv
- **Osservazioni:** 45.879 (11 valori mancanti)
- **Variabile dipendente:** log(wage) — salario mensile trasformato 
  in logaritmo naturale per normalizzare la distribuzione asimmetrica 
  (skewness = 1,72) e interpretare i coefficienti come variazioni 
  percentuali

### Variabili analizzate
| Variabile | Descrizione |
|-----------|-------------|
| wage | Salario mensile in euro — variabile dipendente |
| male | Genere (1 = maschio, 0 = femmina) — 60,8% uomini |
| age | Età anagrafica (range: 20–64 anni, media ≈ 39,7) |
| job_tenure | Anzianità aziendale in anni (mediana: 1 anno) |
| skill_level | Inquadramento: 1=Formazione, 2=Operai, 3=Impiegati, 4=Dirigenti |
| macrogruppo | Settore economico da codici ATECO (7 macrogruppi) |

### Correlazioni chiave con wage
- skill_level: **0,50** — principale determinante
- job_tenure: **0,39**
- age: **0,24**
- age × job_tenure: 0,32 — attenzione alla multicollinearità

---

## Tecnologie e librerie
- **Linguaggio:** R (R Markdown)
- **Librerie:** `dplyr`, `ggplot2`, `lmtest`, `sandwich`, 
  `estimatr` (lm_robust), `modelsummary`, `corrplot`, `knitr`, `broom`

---

## Costruzione progressiva dei modelli

### Modello 1 — Baseline: solo genere
log(wage) = β₀ + β₁·male + ε
| Variabile | β | p-value |
|-----------|---|---------|
| Intercetta | 6,1175 | < 0,001 |
| Male | **+0,0994** | < 0,001 |

**R² = 0,019** — Gli uomini guadagnano circa il **+10,4%** in più 
(100 × (e^0,0994 − 1)). Modello significativo ma con basso potere 
esplicativo.

---

### Modello 2 — Aggiunta età con termine quadratico
log(wage) = β₀ + β₁·male + β₂·age + β₃·age² + ε
| Variabile | β | p-value |
|-----------|---|---------|
| Male | +0,0941 | < 0,001 |
| Age | +0,0379 | < 0,001 |
| Age² | −0,000365 | < 0,001 |

**R² = 0,093** — La relazione età-salario è **non lineare** 
(rendimenti decrescenti). Il coefficiente di male scende da 0,099 
a 0,094: **esempio di bias da variabile omessa** — parte del gap 
era attribuita al genere ma era in realtà spiegata dall'età.

---

### Modello 3 — Aggiunta skill_level (modello principale)
log(wage) = β₀ + β₁·male + β₂·age + β₃·age² + β₄·skill_level + ε
| Variabile | β | p-value |
|-----------|---|---------|
| Male | **+0,1804** | < 0,001 |
| Age | +0,0251 | < 0,001 |
| Age² | −0,000229 | < 0,001 |
| Skill: Operai | +0,0071 | 0,332 (n.s.) |
| Skill: Impiegati | +0,3172 | < 0,001 |
| Skill: Quadri/Dir. | +0,8887 | < 0,001 |

**R² = 0,371** — Salto più grande nell'R². Il gender gap **sale al 
18%**: controllando per skill_level emerge il divario reale a parità 
di inquadramento. Operai non significativo rispetto alla baseline 
Formazione (categorie con salari simili). Dummy trap evitata 
omettendo "Formazione" come baseline.

---

### Modello 4 — Aggiunta job_tenure
log(wage) = β₀ + β₁·male + β₂·age + β₃·age² + β₄·skill_level
+ β₅·job_tenure + ε
| Variabile | β | p-value |
|-----------|---|---------|
| Male | +0,1678 | < 0,001 |
| Job tenure | **+0,0684** | < 0,001 |
| Skill: Operai | +0,0230 | 0,001 ✓ |

**R² = 0,427** — Job_tenure significativa: **+7,1% per anno di 
anzianità**. Operai diventa significativo controlando per anzianità. 
Male stabile a 0,168.

---

### Modello 5 — Test di robustezza con settori (codici ATECO)
log(wage) = β₀ + β₁·male + β₂·age + β₃·age² + β₄·job_tenure
+ β₅·skill_level + β₆·macrogruppo + ε
**R² = 0,434** — L'aggiunta dei 7 macrogruppi ATECO migliora 
marginalmente l'R². I coefficienti chiave restano stabili: 
male = 0,1585, confermando la **robustezza del gender gap** 
indipendentemente dal settore. Agricoltura/Alimentare come 
baseline settoriale.

---

### Modello 6 — Interazione male × skill_level
log(wage) = β₀ + β₁·male + β₂·skill_level
+ β₃·(male×skill_level) + β₄·age + β₅·age²
+ β₆·job_tenure + ε
| Interazione | β | p-value |
|-------------|---|---------|
| Male × Operai | +0,1089 | < 0,001 |
| Male × Impiegati | **+0,1617** | < 0,001 |
| Male × Quadri/Dir. | +0,0527 | 0,006 |

**R² = 0,430** — Il gender gap **non è uniforme** tra categorie: 
è più marcato tra gli impiegati e si riduce tra i dirigenti, 
suggerendo che nelle posizioni apicali il divario uomo-donna 
tende a comprimersi.

---

## Sintesi dei risultati — Evoluzione del gender gap

| Modello | R² | Coeff. male | Gender gap % |
|---------|-----|-------------|--------------|
| 1 — Solo genere | 0,019 | 0,0994 | **+10,4%** |
| 2 — + età (quadratica) | 0,093 | 0,0941 | +9,9% |
| 3 — + skill_level | 0,371 | 0,1804 | **+19,8%** |
| 4 — + job_tenure | 0,427 | 0,1678 | +18,3% |
| 5 — + settori ATECO | 0,434 | 0,1585 | +17,2% |
| 6 — + interazione | 0,430 | variabile per gruppo | — |

**Conclusione chiave:** Il gender pay gap reale, a parità di 
inquadramento e anzianità, è circa il **17-18%** — quasi il doppio 
di quello grezzo (10,4%). L'inquadramento professionale (skill_level) 
è il principale determinante del salario, spiegando da solo il 
maggiore salto nell'R².

---

## 📌 Note
Il dataset ANALISI_WAGE.csv è incluso nel repository.  
Per riprodurre l'analisi aggiornare il percorso del file CSV  
nella prima riga del codice R Markdown.  
Tutti i modelli utilizzano errori standard robusti HC1  
per correggere l'eteroschedasticità rilevata con il test  
di Breusch-Pagan.
