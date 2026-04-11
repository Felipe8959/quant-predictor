# Quant-Predictor

Sistema de predição de movimentos de criptomoedas usando Machine Learning, com foco em classificação binária (Take Profit vs Stop Loss) baseada em targets adaptativos calculados por ATR.

## Performance Atual

| Métrica | Valor | Status |
|---------|-------|--------|
| **AUC-ROC** | 0.7331 | ✅ Bom |
| **F1-Score** | 0.5570 | ✅ Moderado |
| **MCC** | 0.3025 | ✅ Positivo |
| **Precisão (TP)** | 50.4% | ⚠️ Moderado |
| **Recall (TP)** | 62.3% | ✅ Bom |

---

## Estrutura do Projeto

```
quant-predictor/
├── notebooks/                      # Notebooks de análise e desenvolvimento
│   ├── 05_predictor.ipynb          # Modelo rodando em produção
│   └── old/
│       └── modelagem_xgboost.ipynb # Notebook de treinamento (arquivado)
│
├── models/                         # Modelos treinados e metadados
│   ├── xgboost_atr_target.pkl      # Modelo em formato Pickle
│   ├── xgboost_atr_target.json     # Modelo em formato JSON XGBoost
│   ├── xgboost_atr_target_metadata.json  # Metadados completos do modelo
│   └── xgboost_atr_target_feature_importance.csv  # Importâncias das features
│
├── scripts/                        # Scripts Python reutilizáveis
│
├── config/                         # Arquivos de configuração
│   ├── model_config.json           # Hiperparâmetros e configurações do modelo
│   └── trading_params.json         # Parâmetros de risk management e trading
│
├── docs/                           # Documentação detalhada
│   ├── arquitetura.md              # Arquitetura e pipeline de ML
│   ├── features.md                 # Documentação das 52 features
│   └── resultados.md               # Resultados e histórico de modelos
│
└── README.md                       
```

---

## Pipeline de Machine Learning

### Coleta de Dados
* **Fonte:** API de exchange de criptomoedas
* **Storage:** Delta Lake (Databricks)
* **Pares:** 9 criptomoedas (BTC, ETH, BNB, SOL, XRP, ADA, AVAX, LINK, NEAR)
* **Timeframe principal:** 1h
* **Período:** 5 anos de dados históricos

### Feature Engineering
* **52 features** divididas em 4 categorias:
  - Price features (returns, volatility, z-scores, Bollinger Bands)
  - Volume features (volume changes, ratios, z-scores)
  - Technical indicators (RSI, ATR, momentum)
  - Correlation features (beta com BTC, relative strength, ratios)
* **Multi-timeframe:** 1h (operacional), 15m (tático), 4h (estratégico)
* **Target adaptativo:** Baseado em ATR (2x TP, 1x SL, horizonte 24h)

### Modelagem
* **Algoritmo:** XGBoost (Gradient Boosting)
* **Balanceamento:** scale_pos_weight (class weights automático)
* **Validação:** Split temporal walk-forward (sem data leakage)
* **Otimização:** Early stopping + AUC-ROC
* **Features mais importantes:** `log_return_4h`, `momentum_1h`, `log_return_1h`

### Avaliação
* **Métricas principais:** AUC-ROC, F1-Score, MCC
* **Test set:** 20% mais recente (61,077 registros)
* **Threshold:** 0.5 (padrão de classificação)
* **Monitoramento:** Feature importance + confusion matrix

---

## Quick Start

### 1. Clonar e acessar o projeto
```python
# No Databricks
%cd /Workspace/Users/fe.computador@gmail.com/quant-predictor
```

### 2. Usar modelo em produção
```python
import pickle
import pandas as pd

# Carregar modelo
model_path = 'models/xgboost_atr_target.pkl'
with open(model_path, 'rb') as f:
    model = pickle.load(f)

# Fazer predição
features = pd.DataFrame([...])  # Suas 52 features
probability = model.predict_proba(features)[:, 1]
prediction = model.predict(features)

print(f"Probabilidade TP: {probability[0]:.2%}")
print(f"Predição: {'TP' if prediction[0] == 1 else 'SL'}")
```

### 3. Verificar metadados do modelo
```python
import json

# Carregar metadados
with open('models/xgboost_atr_target_metadata.json', 'r') as f:
    metadata = json.load(f)

print(f"Data do treino: {metadata['training_date']}")
print(f"AUC-ROC: {metadata['performance']['auc_roc']:.4f}")
print(f"Features: {metadata['n_features']}")
```

---

## Resultados e Insights

### Principais Conquistas
* **AUC-ROC 0.733:** 47% melhor que baseline aleatório (0.5)
* **MCC 0.303:** Correlação positiva entre predições e realidade
* **Recall 62.3%:** Boa capacidade de identificar oportunidades de TP
* **Multi-timeframe efetivo:** Features de 1h, 15m e 4h contribuem
* **Modelo balanceado:** Trade-off adequado entre precisão e recall

### Feature Importance (Top 5)

1. **log_return_4h** (13.8%) - Retorno logarítmico 4h é o fator mais crítico
2. **momentum_1h** (9.8%) - Momentum de curto prazo (velocidade do movimento)
3. **log_return_1h** (5.4%) - Retorno logarítmico operacional
4. **z_score_close_1h** (4.2%) - Normalização do preço 1h (desvio da média)
5. **momentum_4h** (3.9%) - Momentum estratégico (tendência de médio prazo)

### Insights Principais

#### 1. Timeframe 4h é Extremamente Importante
* Feature mais relevante: `log_return_4h` (13.8%)
* Captura tendências de médio prazo
* Reduz ruído e falsos sinais

#### 2. Momentum é Crítico
* `momentum_1h` e `momentum_4h` no top 5
* Velocidade de mudança do preço é preditiva
* Multi-timeframe captura diferentes horizontes

#### 3. Bollinger Bands Consistente
* `bb_position_1h`, `bb_position_15m`, `bb_position_4h` no top 10
* Posição relativa nas bandas é importante
* Funciona em todos os timeframes

#### 4. Correlação com BTC é Relevante
* `beta_btc` aparece no ranking de importância
* Mercado de criptomoedas segue Bitcoin
* Força relativa também contribui

### Próximos Passos (Roadmap)
- [ ] **Otimização de hiperparâmetros** (Optuna/GridSearch)
- [ ] **Ensemble de modelos** (XGBoost + LightGBM + CatBoost)
- [ ] **Walk-forward validation** mais robusta (múltiplos folds)
- [ ] **Feature engineering:** Sentiment (funding rate, OI, long/short ratio)
- [ ] **Feature engineering:** Order book (spread, depth, imbalance)
- [ ] **Threshold optimization** por curva Precision-Recall
- [ ] **Integração:** API de inferência em produção
- [ ] **Monitoring:** Drift detection + retraining automático

---

## Tecnologias

* **Data:** Delta Lake, PySpark, SQL
* **ML:** XGBoost, scikit-learn, pandas, numpy
* **Visualização:** Matplotlib, Seaborn
* **Compute:** Databricks Serverless (AWS)
* **Formato:** Jupyter Notebooks (.ipynb)

---

## Documentação Completa

* **[Arquitetura](docs/arquitetura.md)** - Estrutura do projeto e pipeline de ML
* **[Features](docs/features.md)** - Documentação das 52 features
* **[Resultados](docs/resultados.md)** - Performance e histórico de modelos
* **[Configurações](config/)** - Parâmetros de modelo e trading

---

## Uso do Modelo

### Interpretação das Predições

#### Probabilidades
* **> 0.6:** Alta confiança de TP - sinal forte de entrada
* **0.5 - 0.6:** Confiança moderada de TP - sinal válido mas cautela
* **0.4 - 0.5:** Confiança moderada de SL - evitar entrada
* **< 0.4:** Alta confiança de SL - sinal forte contra entrada

#### Threshold Customizado
```python
# Threshold padrão (balanceado)
threshold = 0.5

# Threshold conservador (mais precisão, menos sinais)
threshold = 0.6  

# Threshold agressivo (mais sinais, menos precisão)
threshold = 0.4

# Aplicar threshold
prediction = (probability >= threshold).astype(int)
```

### Limitações e Considerações

⚠️ **Importante:**
* Modelo treinou com horizonte de 24h - não use para day trading
* Precisão de 50.4% significa ~50% de falsos positivos
* Sempre use stop loss e take profit conforme ATR
* Backtest em produção antes de usar dinheiro real
* Modelo pode degradar com mudanças de regime de mercado
