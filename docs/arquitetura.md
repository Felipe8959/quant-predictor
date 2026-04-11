# Arquitetura do Projeto Quant-Predictor

## Visão Geral
Sistema de predição de movimentos de criptomoedas usando Machine Learning, com foco em classificação binária (TP vs SL) baseada em targets adaptativos calculados por ATR.

## Estrutura do Projeto

```
quant-predictor/
├── notebooks/                   # Notebooks de análise e desenvolvimento
│   ├── 05_predictor.ipynb       # Modelo rodando em produção
│   └── old/
│       └── modelagem_xgboost.ipynb  # Notebook de treinamento (arquivado)
├── models/                      # Modelos treinados e metadados
│   ├── xgboost_atr_target.pkl   # Modelo em formato Pickle
│   ├── xgboost_atr_target.json  # Modelo em formato JSON XGBoost
│   ├── xgboost_atr_target_metadata.json  # Metadados do modelo
│   └── xgboost_atr_target_feature_importance.csv  # Importâncias das features
├── config/                      # Configurações (JSON)
│   ├── model_config.json        # Hiperparâmetros do modelo
│   └── trading_params.json      # Parâmetros de risk management
├── docs/                        # Documentação
│   ├── arquitetura.md           # Este arquivo
│   ├── features.md              # Documentação das 52 features
│   └── resultados.md            # Performance e histórico de modelos
└── README.md                    # Visão geral do projeto
```

## Pipeline de ML

### 1. Coleta de Dados
* **Fonte:** API de exchange de criptomoedas
* **Storage:** Delta Lake (Databricks)
* **Pares:** 9 criptomoedas (BTC, ETH, BNB, SOL, XRP, ADA, AVAX, LINK, NEAR)
* **Timeframe principal:** 1h
* **Período:** 5 anos de dados históricos

### 2. Feature Engineering
* **Features multi-timeframe:** 1h (operacional), 15m (tático), 4h (estratégico)
* **Total:** 52 features
* **Categorias:**
  - **Price features:** returns, volatility, z-scores, Bollinger Bands
  - **Volume features:** volume changes, ratios, z-scores
  - **Technical indicators:** RSI, ATR, momentum
  - **Correlation features:** beta com BTC/ETH, relative strength, ratios
* **Target adaptativo:** Baseado em ATR
  - **Take Profit (TP):** 2x ATR acima do preço de entrada
  - **Stop Loss (SL):** 1x ATR abaixo do preço de entrada
  - **Horizonte:** 24 horas
  - **Label:** Binário (1 = TP atingido primeiro, 0 = SL atingido primeiro)

### 3. Modelagem
* **Algoritmo:** XGBoost (Gradient Boosting)
* **Objetivo:** Classificação binária (binary:logistic)
* **Balanceamento:** scale_pos_weight (automático baseado em class ratio)
* **Validação:** Split temporal walk-forward (80% treino, 20% teste)
  - Treino: 244,308 registros (períodos mais antigos)
  - Teste: 61,077 registros (períodos mais recentes)
* **Otimização:** Early stopping + AUC-ROC como métrica
* **Hiperparâmetros principais:**
  - max_depth: 6
  - learning_rate: 0.1
  - n_estimators: 200
  - subsample: 0.8
  - colsample_bytree: 0.8

### 4. Produção (05_predictor.ipynb)
* Carregamento do modelo treinado
* Predição em tempo real
* Probabilidades de TP vs SL
* Análise de confiança das predições

## Tecnologias

* **Data:** Delta Lake, PySpark, SQL
* **ML:** XGBoost, scikit-learn, pandas, numpy
* **Visualização:** Matplotlib, Seaborn
* **Compute:** Databricks Serverless (AWS)
* **Formato:** Jupyter Notebooks (.ipynb)

## Fluxo de Trabalho

1. **Dados históricos** são armazenados em Delta Lake
2. **Feature engineering** cria 52 features multi-timeframe + target adaptativo
3. **Treinamento** do modelo XGBoost com validação temporal
4. **Avaliação** usando AUC-ROC, F1-Score, MCC, Precisão/Recall
5. **Salvamento** do modelo + metadados + feature importance
6. **Produção** via notebook 05_predictor.ipynb

## Próximos Passos

### Pipeline de Dados
1. Automatizar coleta de dados em tempo real
2. Criar pipeline de feature engineering modular
3. Implementar data quality checks

### Modelagem
1. Walk-forward validation mais robusta (múltiplos folds)
2. Otimização de hiperparâmetros (Optuna/GridSearch)
3. Ensemble de modelos (XGBoost + LightGBM + CatBoost)
4. Feature engineering avançado (sentiment, order book)

### Produção
1. API de inferência em produção
2. Monitoring e alertas de performance
3. Drift detection automático
4. Retraining pipeline automatizado
5. Integração com bot de trading
