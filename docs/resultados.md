# Resultados dos Modelos

## Modelo Atual em Produção
**Versão:** xgboost_atr_target  
**Data:** 04/04/2026 às 21:34

### Performance

| Métrica | Valor | Interpretação |
|---------|-------|---------------|
| **AUC-ROC** | 0.7331 | ✅ Bom - Capacidade preditiva satisfatória |
| **Precision-Recall AUC** | 0.5954 | ⚠️ Razoável - Classe minoritária desafiadora |
| **F1-Score** | 0.5570 | ✅ Moderado - Bom balanço precisão/recall |
| **MCC** | 0.3025 | ✅ Positivo - Correlação acima do aleatório |
| **Acurácia** | 66.97% | ℹ️ Referência (cuidado com desbalanceamento) |

### Métricas Derivadas

* **Precisão (TP):** 50.4% - Quando prevê TP, 50% estão corretos
* **Recall (TP):** 62.3% - Dos TPs reais, identificou 62%
* **Win Rate (Test):** 50.0% - Balanceado após split temporal

### Dataset

* **Total:** 305,385 registros
* **Train:** 244,308 (80%, mais antigo)
* **Test:** 61,077 (20%, mais recente)
* **Período:** 5 anos de dados históricos
* **Features:** 52 multi-timeframe
* **Target:** Adaptativo ATR (2x TP, 1x SL, horizonte 24h)

### Hiperparâmetros

```json
{
  "max_depth": 6,
  "learning_rate": 0.1,
  "n_estimators": 200,
  "subsample": 0.8,
  "colsample_bytree": 0.8,
  "scale_pos_weight": 2.07,
  "early_stopping_rounds": 20,
  "eval_metric": "auc"
}
```

### Top 10 Features Mais Importantes

1. `log_return_4h` (13.8%) - Retorno logarítmico 4h é o fator mais crítico
2. `momentum_1h` (9.8%) - Momentum de curto prazo
3. `log_return_1h` (5.4%) - Retorno logarítmico 1h
4. `z_score_close_1h` (4.2%) - Normalização do preço 1h
5. `momentum_4h` (3.9%) - Momentum estratégico
6. `bb_position_4h` (3.6%) - Posição nas Bollinger Bands 4h
7. `volatility_1h` (2.5%) - Volatilidade operacional
8. `bb_position_1h` (2.5%) - Posição nas Bollinger Bands 1h
9. `bb_position_15m` (2.5%) - Posição nas Bollinger Bands 15m
10. `z_score_close_15m` (2.3%) - Normalização do preço 15m

**Insight Principal:** Timeframe 4h é extremamente importante (feature mais relevante), seguido pelo 1h. O modelo captura bem a dinâmica multi-timeframe com retornos logarítmicos e momentum sendo os principais preditores.

**Insight Adicional:** Features de posição nas Bollinger Bands são consistentemente importantes em todos os timeframes (1h, 15m, 4h), indicando que a posição relativa do preço nas bandas de volatilidade é um forte indicador.

---

## Análise de Performance

### Pontos Fortes
* **AUC-ROC 0.733:** 42% melhor que baseline aleatório (0.5)
* **Recall 62.3%:** Boa capacidade de identificar oportunidades de TP
* **Features 4h funcionais:** Timeframe estratégico contribui significativamente
* **Multi-timeframe efetivo:** Modelo captura dinâmicas de curto (15m), operacional (1h) e estratégico (4h)

### Pontos de Atenção
* **Precisão 50.4%:** Moderada - metade dos sinais de TP podem ser falsos positivos
* **Balanceamento:** Model trade-off entre precisão e recall para maximizar AUC

### Uso Recomendado
* **Threshold:** 0.5 (padrão) para balanço entre precisão/recall
* **Threshold > 0.6:** Aumenta precisão, reduz recall (menos sinais, mais confiáveis)
* **Threshold < 0.4:** Aumenta recall, reduz precisão (mais sinais, menos confiáveis)

---

## Roadmap de Melhorias

### Prioridade ALTA
- [ ] **Otimização de hiperparâmetros** (Optuna/GridSearch)
- [ ] **Ensemble de modelos** (XGBoost + LightGBM + CatBoost)
- [ ] **Walk-forward validation** mais robusta (múltiplos folds temporais)

### Prioridade MÉDIA
- [ ] **Feature engineering:** Sentiment (funding rate, Open Interest)
- [ ] **Feature engineering:** Order book (spread, depth, imbalance)
- [ ] **Análise por regime de mercado** (bull/bear/sideways)
- [ ] **Threshold optimization** por curva Precision-Recall

### Prioridade BAIXA
- [ ] **SHAP values** (explainability avançada)
- [ ] **Monitoring automático** (drift detection)
- [ ] **Retraining pipeline** automatizado
- [ ] **A/B testing** de diferentes versões do modelo

---

**Última atualização:** 04/04/2026
