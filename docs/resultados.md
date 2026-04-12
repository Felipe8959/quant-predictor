# Resultados dos Modelos

## Modelo Atual em Produção
**Versão:** xgboost_atr_target  
**Data:** Janeiro 2025 (pós-implementação Lote 1)  
**Threshold Recomendado:** 0.55

### Performance com Threshold 0.55 (Recomendado)

| Métrica | Valor | Interpretação |
|---------|-------|---------------|
| **AUC-ROC** | 0.7730 | ✅ Bom - Capacidade preditiva satisfatória |
| **Acurácia** | 72.18% | ✅ Bom - Acerta mais de 7 em 10 |
| **Precisão (TP)** | 57.04% | ✅ Bom - 57% dos sinais TP estão corretos |
| **Recall (TP)** | 57.02% | ✅ Moderado - Captura 57% das oportunidades |
| **F1-Score** | 0.5703 | ✅ Moderado - Bom balanço precisão/recall |
| **MCC** | 0.3647 | ✅ Positivo - Correlação acima do aleatório |

### Comparativo de Thresholds

| Métrica | Threshold 0.50 | Threshold 0.55 | Diferença |
|---------|----------------|----------------|-----------|
| **Acurácia** | 68.41% | 72.18% | +3.77 p.p. ✅ |
| **Precisão (TP)** | 50.87% | 57.04% | +6.17 p.p. ✅ |
| **Recall (TP)** | 70.51% | 57.02% | -13.49 p.p. ⚠️ |
| **F1-Score** | 0.5910 | 0.5703 | -0.0207 |
| **MCC** | 0.3567 | 0.3647 | +0.0080 ✅ |
| **AUC-ROC** | 0.7730 | 0.7730 | — |

### Por que Threshold 0.55?

**Vantagens:**
* ✅ **Precisão 6.17 p.p. melhor:** 50.87% → 57.04%
* ✅ **Menos falsos positivos:** Redução de 36.9% (3,621 → 2,284)
* ✅ **Acurácia melhor:** 68.41% → 72.18%
* ✅ **MCC ligeiramente superior:** Correlação mais forte

**Trade-offs:**
* ⚠️ **Recall menor:** 70.51% → 57.02% (perde 13% das oportunidades)
* ⚠️ **Menos sinais:** Redução de 27.9% (7,370 → 5,316)
* ⚠️ **Mais conservador:** Pode perder algumas oportunidades válidas

**Impacto Prático:**
* Threshold 0.50: 7,370 sinais → 3,749 TPs corretos (50.87% win rate)
* Threshold 0.55: 5,316 sinais → 3,032 TPs corretos (57.04% win rate)
* **Conclusão:** Menos sinais, mas mais confiáveis

### Dataset

* **Total:** 82,128 registros (após limpeza de NaNs)
* **Train:** 65,702 (80%, mais antigo)
* **Test:** 16,426 (20%, mais recente)
* **Período:** 2021-04-04 a 2024-12-16
* **Features:** ~96 multi-timeframe (após alinhamento)
* **Target:** Adaptativo ATR (2x TP, 1x SL, horizonte 24h)

### Hiperparâmetros

```json
{
  "max_depth": 6,
  "learning_rate": 0.1,
  "n_estimators": 200,
  "subsample": 0.8,
  "colsample_bytree": 0.8,
  "scale_pos_weight": 2.11,
  "early_stopping_rounds": 20,
  "eval_metric": "auc"
}
```

### Top 10 Features Mais Importantes

1. `log_return_4h` (6.0%) - Retorno logarítmico 4h continua sendo o fator mais crítico
2. `roc_10_1h` (5.2%) 🆕 - Rate of Change 10 períodos no timeframe operacional
3. `momentum_1h` (4.8%) - Momentum de curto prazo (velocidade do movimento)
4. `momentum_4h` (3.8%) - Momentum estratégico (tendência de médio prazo)
5. `body_range_ratio_4h` (3.3%) 🆕 - Força do movimento em 4h (candle psychology)
6. `upper_shadow_ratio_4h` (3.0%) 🆕 - Rejeição na máxima 4h (pressão vendedora)
7. `log_return_1h` (2.6%) - Retorno logarítmico operacional
8. `z_score_close_1h` (2.4%) - Normalização do preço 1h (desvio da média)
9. `bb_position_4h` (2.2%) - Posição nas Bollinger Bands 4h
10. `roc_5_1h` (2.2%) 🆕 - Rate of Change 5 períodos (curto prazo)

**Insight Principal:** Timeframe 4h continua extremamente importante (log_return_4h #1), mas o **Lote 1 trouxe mudanças significativas**: ROC (Rate of Change) agora domina, com `roc_10_1h` em 2º lugar e 5 versões de ROC no top 30. Price action (candle psychology) é altamente preditiva: `body_range_ratio_4h` (#5) e `upper_shadow_ratio_4h` (#6) capturam a força e rejeições dos movimentos.

**Insights Adicionais (Lote 1):**
* **16 das top 30 features são do Lote 1** (53% do ranking) - novas features capturam aspectos críticos não cobertos pelas originais
* **ROC supera momentum tradicional:** Múltiplas janelas (5, 10, 20) capturam melhor a velocidade de mudança de preço
* **Candle Psychology funciona:** Proporções de corpo/sombras revelam psicologia do mercado e pressão compradora/vendedora
* **Padrões temporais relevantes:** `hour_sin_1h` (#15) e `hour_cos_1h` (#12) - horários de liquidez (NY, Ásia, Europa) impactam
* **Support/Resistance técnico preditivo:** `support_break_1h` (#16), `resistance_break_1h` (#18), `vwap_distance_1h` (#19)

---

## Análise de Performance

### Pontos Fortes
* **AUC-ROC 0.773:** 55% melhor que baseline aleatório (0.5)
* **Precisão 57.0% com threshold 0.55:** Win rate superior ao acaso
* **Acurácia 72.2%:** Modelo acerta mais de 7 em 10 predições
* **Features 4h funcionais:** Timeframe estratégico contribui significativamente
* **Multi-timeframe efetivo:** Modelo captura dinâmicas de curto (15m), operacional (1h) e estratégico (4h)
* **Lote 1 altamente efetivo:** 53% do top 30 são novas features - ROC, price action, padrões temporais e suporte/resistência agregam valor preditivo
* **Threshold otimizado:** 0.55 oferece melhor balanço entre qualidade e quantidade

### Pontos de Atenção
* **Precisão 57% não é perfeita:** 43% dos sinais TP ainda são falsos positivos
* **Recall 57% com threshold 0.55:** Perde ~43% das oportunidades reais
* **Trade-off threshold:** Escolha entre precisão (0.55+) e recall (0.50)

### Confusion Matrix - Threshold 0.55

```
                 Predito SL    Predito TP
Real SL (0)       8,825         2,284
Real TP (1)       2,285         3,032
```

**Métricas Derivadas:**
* **True Negatives (TN):** 8,825 - SL corretamente identificados
* **False Positives (FP):** 2,284 - Sinais errados de TP (43% dos sinais)
* **False Negatives (FN):** 2,285 - TPs perdidos (43% das oportunidades)
* **True Positives (TP):** 3,032 - TPs corretamente identificados (57% dos sinais)

### Uso Recomendado

#### Threshold por Perfil de Risco

| Perfil | Threshold | Precisão | Recall | Quando Usar |
|--------|-----------|----------|--------|-------------|
| **Balanceado** ✅ | 0.55 | 57.04% | 57.02% | Recomendado para maioria |
| Agressivo | 0.50 | 50.87% | 70.51% | Capital amplo, custos baixos |
| Conservador | 0.60 | ~65%* | ~45%* | Capital limitado, custos altos |
| Extremamente Conservador | 0.70 | ~75%* | ~30%* | Apenas sinais de altíssima confiança |

*Estimado baseado na curva precision-recall

#### Interpretação das Probabilidades

* **≥ 0.70:** Sinal muito forte de TP - alta confiança
* **0.55 - 0.70:** Sinal de TP - confiança moderada/boa
* **0.45 - 0.55:** Zona cinza - evitar (indecisão)
* **< 0.45:** Sinal de SL - evitar entrada

---

## Roadmap de Melhorias

### Prioridade ALTA
- [x] **Lote 1 de features implementado** (~92 features totais)
- [x] **Threshold optimization** - Validado 0.55 como superior
- [ ] **Otimização de hiperparâmetros** (Optuna/GridSearch)
- [ ] **Ensemble de modelos** (XGBoost + LightGBM + CatBoost)
- [ ] **Walk-forward validation** mais robusta (múltiplos folds temporais)

### Prioridade MÉDIA (Lote 2)
- [ ] **Feature engineering:** Sentiment (funding rate, Open Interest, long/short ratio)
- [ ] **Feature engineering:** Order book (spread, depth, imbalance)
- [ ] **Feature engineering:** On-chain metrics (active addresses, exchange flows, MVRV)
- [ ] **Análise por regime de mercado** (bull/bear/sideways)
- [ ] **Calibração de probabilidades** (Platt scaling, isotonic regression)

### Prioridade BAIXA
- [ ] **SHAP values** (explainability avançada)
- [ ] **Monitoring automático** (drift detection)
- [ ] **Retraining pipeline** automatizado
- [ ] **A/B testing** de diferentes versões do modelo

---

## Comparativo Histórico

### Evolução do Modelo

| Versão | Features | Threshold | Precisão | Recall | F1-Score | AUC-ROC |
|--------|----------|-----------|----------|--------|----------|---------|
| v1.0 (baseline) | 52 | 0.50 | 50.4% | 62.3% | 0.557 | 0.733 |
| v2.0 (Lote 1) | ~92 | 0.50 | 50.87% | 70.51% | 0.591 | 0.773 |
| **v2.1 (Lote 1 + Threshold)** ✅ | ~92 | **0.55** | **57.04%** | **57.02%** | **0.570** | **0.773** |

**Melhorias:**
* AUC-ROC: 0.733 → 0.773 (+5.5%)
* Precisão: 50.4% → 57.04% (+13.2%)
* Acurácia: ~67% → 72.18% (+7.7%)

---

## Recomendações de Uso

### Para Traders

1. **Use threshold 0.55 como padrão**
   * Melhor balanço entre precisão e recall
   * Win rate de 57% é superior ao acaso
   * Reduz falsos positivos significativamente

2. **Ajuste threshold conforme perfil**
   * Conservador: 0.60-0.70 (mais precisão, menos sinais)
   * Agressivo: 0.45-0.50 (mais sinais, menos precisão)

3. **Sempre use stop loss/take profit**
   * TP = 2x ATR
   * SL = 1x ATR
   * Não opere sem proteção

4. **Monitore performance**
   * Track win rate real vs esperado
   * Ajuste threshold se necessário
   * Considere retraining se performance degradar

### Para Desenvolvedores

1. **Próximas otimizações**
   * Hyperparameter tuning (Optuna)
   * Ensemble methods (stacking, voting)
   * Calibração de probabilidades

2. **Feature engineering avançado**
   * Sentiment analysis (funding rate, social media)
   * Order book features (depth, imbalance)
   * On-chain metrics (network activity)

3. **Production readiness**
   * API de inferência
   * Monitoring e alertas
   * Automated retraining pipeline

---

**Última atualização:** Janeiro 2025 (pós-Lote 1 + Threshold Optimization)
