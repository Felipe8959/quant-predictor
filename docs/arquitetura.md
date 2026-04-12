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
│   ├── features.md              # Documentação das ~92 features
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
* **Total:** ~92 features (52 originais + 40 do Lote 1)
* **Categorias:**
  - **Price features:** returns, volatility, z-scores, Bollinger Bands, VWAP, support/resistance
  - **Volume features:** volume changes, ratios, z-scores, OBV
  - **Technical indicators:** RSI, ATR, momentum, ROC, ADX, volatility ratio
  - **Price action:** body/shadow ratios (candle psychology)
  - **Temporal features:** hour of day (sin/cos encoding)
  - **Correlation features:** beta com BTC/ETH, relative strength, ratios, market breadth
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
  - Treino: 65,702 registros (períodos mais antigos)
  - Teste: 16,426 registros (períodos mais recentes)
* **Otimização:** Early stopping + AUC-ROC como métrica
* **Hiperparâmetros principais:**
  - max_depth: 6
  - learning_rate: 0.1
  - n_estimators: 200
  - subsample: 0.8
  - colsample_bytree: 0.8
  - scale_pos_weight: 2.11

### 4. Produção (05_predictor.ipynb)
* **Carregamento:** Modelo treinado em formato pickle
* **Predição:** Probabilidades de TP vs SL em tempo real
* **Threshold recomendado:** 0.55 (otimizado para qualidade dos sinais)
* **Performance com threshold 0.55:**
  - Precisão: 57.04%
  - Recall: 57.02%
  - Acurácia: 72.18%
  - AUC-ROC: 0.773
* **Classificação de confiança:**
  - ≥ 0.65: Alta confiança de TP
  - 0.55-0.65: Confiança moderada de TP
  - 0.45-0.55: Zona cinza (evitar)
  - < 0.45: Sinal de SL

## Tecnologias

* **Data:** Delta Lake, PySpark, SQL
* **ML:** XGBoost, scikit-learn, pandas, numpy
* **Visualização:** Matplotlib, Seaborn
* **Compute:** Databricks Serverless (AWS)
* **Formato:** Jupyter Notebooks (.ipynb)

## Fluxo de Trabalho

1. **Dados históricos** são armazenados em Delta Lake
2. **Feature engineering** cria ~92 features multi-timeframe + target adaptativo
3. **Treinamento** do modelo XGBoost com validação temporal
4. **Avaliação** usando AUC-ROC, F1-Score, MCC, Precisão/Recall
5. **Threshold optimization** validou 0.55 como superior ao padrão 0.50
6. **Salvamento** do modelo + metadados + feature importance
7. **Produção** via notebook 05_predictor.ipynb com threshold 0.55

## Decisões de Design

### Por que Threshold 0.55?
Após análise comparativa, threshold 0.55 foi escolhido como recomendado porque:
* ✅ Precisão 6.17 p.p. melhor que 0.50 (50.87% → 57.04%)
* ✅ Reduz falsos positivos em 36.9%
* ✅ Acurácia geral 3.77 p.p. melhor (68.41% → 72.18%)
* ✅ Win rate mais realista e confiável
* ⚠️ Trade-off aceitável: Recall reduz 13.49 p.p., mas ganha-se qualidade

### Threshold Alternativo
* **0.50:** Para perfil agressivo (mais sinais, menor precisão)
* **0.60+:** Para perfil conservador (menos sinais, maior precisão)

## Próximos Passos

### Pipeline de Dados
1. Automatizar coleta de dados em tempo real
2. Criar pipeline de feature engineering modular
3. Implementar data quality checks

### Modelagem
1. ✅ Threshold optimization (completo - validado 0.55)
2. Walk-forward validation mais robusta (múltiplos folds)
3. Otimização de hiperparâmetros (Optuna/GridSearch)
4. Ensemble de modelos (XGBoost + LightGBM + CatBoost)
5. Feature engineering avançado (sentiment, order book, on-chain metrics)

### Produção
1. API de inferência em produção
2. Monitoring e alertas de performance
3. Drift detection automático
4. Retraining pipeline automatizado
5. Integração com bot de trading
6. Calibração de probabilidades (Platt scaling, isotonic regression)
