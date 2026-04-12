# Documentação das Features

## Visão Geral
O modelo utiliza **~92 features** divididas em 4 categorias principais e 3 timeframes (1h, 15m, 4h).

**Evolução:**
* **Features Originais:** 52 features (versão inicial)
* **Lote 1:** +40 features (10 novas categorias × timeframes)
* **Total Atual:** ~92 features

---

## 1. Price Features

Features derivadas diretamente dos preços de fechamento (close).

### 1.1 Log Return (Retorno Logarítmico)
**Features:** `log_return_1h`, `log_return_15m`, `log_return_4h`

#### Fórmula
$$r_t = \ln(P_t) - \ln(P_{t-1}) = \ln\left(\frac{P_t}{P_{t-1}}\right)$$

#### Explicação
* Calcula o retorno percentual entre dois períodos usando logaritmos naturais
* Propriedade aditiva: retornos de diferentes períodos podem ser somados
* Aproximadamente igual ao retorno simples para valores pequenos
* Mais adequado para séries temporais financeiras (normalidade, estacionariedade)

#### Interpretação
* $r_t > 0$: Preço subiu em relação ao período anterior
* $r_t < 0$: Preço caiu em relação ao período anterior
* $|r_t|$ grande: Movimento forte (alta volatilidade)

---

### 1.2 Volatilidade
**Features:** `volatility_1h`, `volatility_15m`, `volatility_4h`

#### Fórmula
$$\sigma_t = \sqrt{\frac{1}{n-1} \sum_{i=0}^{n-1} (r_{t-i} - \bar{r})^2}$$

Onde:
* $r_{t-i}$ = log return no período $t-i$
* $\bar{r}$ = média dos retornos na janela
* $n$ = tamanho da janela (tipicamente 20 períodos)

#### Explicação
* Desvio padrão dos retornos logarítmicos em uma janela móvel
* Mede a dispersão dos retornos (risco/incerteza)
* Valores altos indicam movimentos grandes e imprevisíveis

#### Interpretação
* $\sigma$ alto: Alta volatilidade, mercado agitado, maior risco
* $\sigma$ baixo: Baixa volatilidade, mercado calmo, menor risco
* Usado para dimensionar stop loss e take profit

---

### 1.3 Z-Score do Preço
**Features:** `z_score_close_1h`, `z_score_close_15m`, `z_score_close_4h`

#### Fórmula
$$z_t = \frac{P_t - \mu}{\sigma}$$

Onde:
* $P_t$ = preço atual
* $\mu$ = média móvel do preço (janela de 20 períodos)
* $\sigma$ = desvio padrão do preço (janela de 20 períodos)

#### Explicação
* Normaliza o preço em relação à sua distribuição histórica
* Mede quantos desvios padrão o preço está da média
* Remove tendência e escala, tornando ativos comparáveis

#### Interpretação
* $z > 2$: Preço muito acima da média (possível sobrecompra)
* $z < -2$: Preço muito abaixo da média (possível sobrevenda)
* $|z| > 3$: Evento extremo (reversão provável)

---

### 1.4 Média Móvel
**Features:** `ma_close_1h`, `ma_close_15m`, `ma_close_4h`

#### Fórmula (SMA - Simple Moving Average)
$$MA_t = \frac{1}{n} \sum_{i=0}^{n-1} P_{t-i}$$

Onde:
* $n$ = período da média (tipicamente 20)

#### Explicação
* Média aritmética dos últimos $n$ preços
* Suaviza flutuações de curto prazo
* Identifica direção da tendência

#### Interpretação
* $P_t > MA_t$: Preço acima da média (tendência de alta)
* $P_t < MA_t$: Preço abaixo da média (tendência de baixa)
* Cruzamentos: Sinais de compra/venda

---

### 1.5 Bollinger Bands
**Features:** `bb_upper_1h`, `bb_lower_1h`, `bb_position_1h` (e versões 15m, 4h)

#### Fórmulas
$$BB_{upper} = MA_t + k \cdot \sigma_t$$
$$BB_{lower} = MA_t - k \cdot \sigma_t$$
$$BB_{position} = \frac{P_t - BB_{lower}}{BB_{upper} - BB_{lower}}$$

Onde:
* $MA_t$ = média móvel (20 períodos)
* $\sigma_t$ = desvio padrão (20 períodos)
* $k$ = multiplicador (tipicamente 2)

#### Explicação
* Bandas que se expandem/contraem com a volatilidade
* Delimitam região "normal" de variação do preço
* `bb_position` normaliza a posição do preço entre as bandas

#### Interpretação
* $BB_{position} > 0.8$: Próximo à banda superior (sobrecompra)
* $BB_{position} < 0.2$: Próximo à banda inferior (sobrevenda)
* Bandas estreitas: Baixa volatilidade (possível breakout)
* Bandas largas: Alta volatilidade (movimento forte)

---

### 1.6 VWAP (Volume Weighted Average Price) 🆕
**Features:** `vwap_1h`, `vwap_15m`, `vwap_4h`

#### Fórmula
$$VWAP_t = \frac{\sum_{i=0}^{n-1} (P_{t-i} \cdot V_{t-i})}{\sum_{i=0}^{n-1} V_{t-i}}$$

Onde:
* $P_{t-i}$ = preço no período $t-i$
* $V_{t-i}$ = volume no período $t-i$
* $n$ = janela de cálculo (20 períodos)

#### Explicação
* Preço médio ponderado pelo volume negociado
* Considera onde a maior parte do volume foi transacionada
* Mais representativo que média simples em mercados com volume variável

#### Interpretação
* Preço > VWAP: Compradores pagaram mais que a média (força compradora)
* Preço < VWAP: Vendedores aceitaram menos que a média (força vendedora)
* Traders institucionais usam VWAP como benchmark de execução

---

### 1.7 VWAP Distance (Distância do VWAP) 🆕
**Features:** `vwap_distance_1h`, `vwap_distance_15m`, `vwap_distance_4h`

#### Fórmula
$$VWAP_{dist,t} = \frac{P_t - VWAP_t}{VWAP_t}$$

#### Explicação
* Distância percentual do preço atual em relação ao VWAP
* Indica se o preço está "caro" ou "barato" em relação ao volume negociado

#### Interpretação
* $VWAP_{dist} > 0.02$ (2%): Preço significativamente acima do VWAP (possível reversão)
* $VWAP_{dist} < -0.02$ (-2%): Preço significativamente abaixo do VWAP (possível recuperação)
* Próximo de 0: Preço alinhado com volume médio

---

### 1.8 Support/Resistance Breaks 🆕
**Features:** `support_break_1h`, `resistance_break_1h` (e versões 15m, 4h)

#### Fórmulas
$$Resistance_{break,t} = \frac{P_t - \max(H_{t-19:t})}{\max(H_{t-19:t})}$$
$$Support_{break,t} = \frac{P_t - \min(L_{t-19:t})}{\min(L_{t-19:t})}$$

Onde:
* $\max(H_{t-19:t})$ = máxima dos últimos 20 períodos (resistência)
* $\min(L_{t-19:t})$ = mínima dos últimos 20 períodos (suporte)

#### Explicação
* Mede distância do preço em relação a níveis técnicos importantes
* Breakouts acima da resistência ou abaixo do suporte são sinais fortes

#### Interpretação
* $Resistance_{break} > 0$: **Breakout de resistência** (sinal de alta)
* $Support_{break} < 0$: **Breakdown de suporte** (sinal de baixa)
* Próximo de 0: Preço está dentro do range estabelecido

---

## 2. Volume Features

Features derivadas do volume de negociação.

### 2.1 Z-Score do Volume
**Features:** `z_score_volume_1h`, `z_score_volume_15m`, `z_score_volume_4h`

#### Fórmula
$$z_{vol,t} = \frac{V_t - \mu_{vol}}{\sigma_{vol}}$$

Onde:
* $V_t$ = volume atual
* $\mu_{vol}$ = média do volume (20 períodos)
* $\sigma_{vol}$ = desvio padrão do volume (20 períodos)

#### Explicação
* Normaliza o volume em relação ao histórico
* Identifica picos anormais de atividade

#### Interpretação
* $z_{vol} > 2$: Volume muito acima da média (forte interesse)
* $z_{vol} < -1$: Volume muito abaixo da média (pouco interesse)
* Volume alto + movimento de preço: Confirmação de tendência

---

### 2.2 Média Móvel do Volume
**Features:** `ma_volume_1h`, `ma_volume_15m`, `ma_volume_4h`

#### Fórmula
$$MA_{vol,t} = \frac{1}{n} \sum_{i=0}^{n-1} V_{t-i}$$

#### Explicação
* Média móvel simples do volume
* Baseline para comparar volume atual

#### Interpretação
* $V_t > MA_{vol,t}$: Volume acima da média (interesse crescente)
* Breakouts com volume alto: Mais confiáveis

---

### 2.3 Volume Change (Mudança Percentual)
**Features:** `vol_change_1h`, `vol_change_15m`, `vol_change_4h`

#### Fórmula
$$\Delta V_t = \frac{V_t - V_{t-1}}{V_{t-1}}$$

#### Explicação
* Variação percentual do volume em relação ao período anterior
* Detecta acelerações/desacelerações na atividade

#### Interpretação
* $\Delta V_t > 1$: Volume dobrou (forte aumento de interesse)
* $\Delta V_t < -0.5$: Volume caiu pela metade (diminuição de interesse)

---

### 2.4 OBV (On-Balance Volume) 🆕
**Features:** `obv_1h`, `obv_15m`, `obv_4h`

#### Fórmula
$$OBV_t = OBV_{t-1} + \begin{cases}
+V_t & \text{se } P_t > P_{t-1} \\
-V_t & \text{se } P_t < P_{t-1} \\
0 & \text{se } P_t = P_{t-1}
\end{cases}$$

(Implementação: soma móvel de 20 períodos do delta)

#### Explicação
* Volume cumulativo direcional
* Volume "comprador" vs volume "vendedor"
* Mede pressão de compra/venda acumulada

#### Interpretação
* OBV crescente: Acumulação (pressão compradora)
* OBV decrescente: Distribuição (pressão vendedora)
* Divergência OBV vs preço: Sinal de reversão antecipado

---

## 3. Technical Indicators

Indicadores técnicos clássicos.

### 3.1 RSI (Relative Strength Index)
**Features:** `rsi_1h`, `rsi_15m`, `rsi_4h`

#### Fórmula
$$RSI = 100 - \frac{100}{1 + RS}$$

Onde:
$$RS = \frac{\text{Média de ganhos (14 períodos)}}{\text{Média de perdas (14 períodos)}}$$

$$\text{Ganho}_t = \max(P_t - P_{t-1}, 0)$$
$$\text{Perda}_t = \max(P_{t-1} - P_t, 0)$$

#### Explicação
* Oscilador de momentum (0-100)
* Compara magnitude de ganhos recentes vs perdas recentes
* Identifica condições de sobrecompra/sobrevenda

#### Interpretação
* $RSI > 70$: Sobrecompra (possível reversão de baixa)
* $RSI < 30$: Sobrevenda (possível reversão de alta)
* $RSI \approx 50$: Equilíbrio entre compra e venda
* Divergências: RSI vs preço indicam possíveis reversões

---

### 3.2 ATR (Average True Range)
**Features:** `atr_1h`, `atr_15m`, `atr_4h`

#### Fórmula
$$TR_t = \max(H_t - L_t, |H_t - C_{t-1}|, |L_t - C_{t-1}|)$$
$$ATR_t = \frac{1}{n} \sum_{i=0}^{n-1} TR_{t-i}$$

Onde:
* $H_t$ = máxima do período
* $L_t$ = mínima do período
* $C_{t-1}$ = fechamento anterior
* $n$ = período (tipicamente 14)

#### Explicação
* Mede a volatilidade real do ativo (range médio)
* Considera gaps entre períodos
* Base para stop loss e take profit adaptativos

#### Interpretação
* $ATR$ alto: Alta volatilidade, maior risco, stops mais largos
* $ATR$ baixo: Baixa volatilidade, menor risco, stops mais apertados
* **Usado no projeto:** TP = 2×ATR, SL = 1×ATR

---

### 3.3 Momentum
**Features:** `momentum_1h`, `momentum_15m`, `momentum_4h`

#### Fórmula
$$M_t = \frac{P_t - P_{t-n}}{P_{t-n}}$$

Onde:
* $n$ = lookback period (tipicamente 3-5 períodos)

#### Explicação
* Velocidade da mudança de preço
* Retorno percentual ao longo de $n$ períodos

#### Interpretação
* $M_t > 0$: Momentum positivo (alta)
* $M_t < 0$: Momentum negativo (baixa)
* $|M_t|$ grande: Movimento forte e rápido

---

### 3.4 ROC (Rate of Change) 🆕
**Features:** `roc_5_1h`, `roc_10_1h`, `roc_20_1h` (e versões 15m, 4h)

#### Fórmula
$$ROC_{n,t} = \frac{P_t - P_{t-n}}{P_{t-n}}$$

Onde:
* $n$ = período (5, 10 ou 20)

#### Explicação
* Similar ao momentum, mas em múltiplas janelas temporais
* Captura velocidade de mudança em diferentes escalas
* ROC_5 = curto prazo, ROC_10 = médio, ROC_20 = longo

#### Interpretação
* $ROC > 0$: Preço subindo nos últimos $n$ períodos
* $ROC < 0$: Preço caindo nos últimos $n$ períodos
* Divergência entre ROC_5, ROC_10, ROC_20: Mudança de regime

---

### 3.5 ADX (Average Directional Index) 🆕
**Features:** `adx_1h`, `adx_15m`, `adx_4h`

#### Fórmula (Simplificada)
$$+DM = \max(H_t - H_{t-1}, 0)$$
$$-DM = \max(L_{t-1} - L_t, 0)$$
$$+DI = \frac{\text{média}(+DM, 14)}{ATR} \times 100$$
$$-DI = \frac{\text{média}(-DM, 14)}{ATR} \times 100$$
$$DX = \frac{|+DI - -DI|}{+DI + -DI} \times 100$$
$$ADX = \text{média}(DX, 14)$$

#### Explicação
* Mede **força da tendência** (não direção)
* Valores altos indicam tendência forte (alta ou baixa)
* Independente de direção (diferente de RSI/Momentum)

#### Interpretação
* $ADX > 25$: Tendência forte (tradeable)
* $ADX < 20$: Mercado sem tendência (range-bound)
* $ADX$ crescente: Tendência fortalecendo
* $ADX$ decrescente: Tendência enfraquecendo

---

### 3.6 Volatility Ratio 🆕
**Features:** `vol_ratio_1h`, `vol_ratio_15m`, `vol_ratio_4h`

#### Fórmula
$$VolRatio_t = \frac{\sigma_{short}}{\sigma_{long}}$$

Onde:
* $\sigma_{short}$ = volatilidade de 10 períodos
* $\sigma_{long}$ = volatilidade de 50 períodos

#### Explicação
* Compara volatilidade recente com histórica
* Detecta mudanças no regime de volatilidade

#### Interpretação
* $VolRatio > 1$: Volatilidade aumentando (regime agitado)
* $VolRatio < 1$: Volatilidade diminuindo (regime calmo)
* $VolRatio \gg 1$: Spike de volatilidade (risco extremo)

---

## 4. Price Action & Candle Psychology 🆕

Features derivadas da anatomia dos candles (OHLC).

### 4.1 Body-to-Range Ratio
**Features:** `body_range_ratio_1h`, `body_range_ratio_15m`, `body_range_ratio_4h`

#### Fórmula
$$BodyRatio_t = \frac{|Close_t - Open_t|}{High_t - Low_t}$$

#### Explicação
* Proporção do corpo do candle em relação ao range total
* Mede força direcional do movimento
* Candles com corpo grande = movimento decisivo

#### Interpretação
* $BodyRatio > 0.7$: Candle forte, movimento decisivo
* $BodyRatio < 0.3$: Candle fraco, indecisão (doji-like)
* Alta após consolidação: Possível breakout

---

### 4.2 Shadow Ratios (Proporção das Sombras)
**Features:** `upper_shadow_ratio_1h`, `lower_shadow_ratio_1h` (e versões 15m, 4h)

#### Fórmulas
$$UpperShadow_t = \frac{High_t - \max(Open_t, Close_t)}{High_t - Low_t}$$
$$LowerShadow_t = \frac{\min(Open_t, Close_t) - Low_t}{High_t - Low_t}$$

#### Explicação
* Proporção das sombras superior e inferior
* Captura rejeições de preço (pressão compradora/vendedora)

#### Interpretação
* **Upper Shadow alto:** Rejeição na máxima (pressão vendedora)
* **Lower Shadow alto:** Rejeição na mínima (pressão compradora)
* Upper + Lower baixos: Candle com corpo dominante (força)
* Upper + Lower altos: Alta indecisão (possível reversão)

---

## 5. Temporal Features 🆕

Features baseadas em padrões temporais.

### 5.1 Hour of Day (Componentes Cíclicas)
**Features:** `hour_sin_1h`, `hour_cos_1h`, `hour_sin_15m`, `hour_cos_15m`

#### Fórmulas
$$HourSin_t = \sin\left(\frac{2\pi \cdot hour(t)}{24}\right)$$
$$HourCos_t = \cos\left(\frac{2\pi \cdot hour(t)}{24}\right)$$

#### Explicação
* Codificação cíclica da hora do dia (0-23)
* Sin/Cos preservam natureza circular do tempo (23h está perto de 0h)
* Captura padrões intraday (horários de NY, Ásia, Europa)

#### Interpretação
* Captura padrões como:
  * Maior liquidez durante horário de NY (13h-20h UTC)
  * Menor liquidez durante madrugada asiática
  * Movimentos típicos de abertura/fechamento de mercados
* Não disponível para 4h (janela muito larga)

---

## 6. Correlation Features

Features de correlação com ativos benchmark (BTC, ETH).

### 6.1 Beta com BTC
**Feature:** `beta_btc`

#### Fórmula
$$\beta = \frac{\text{Cov}(R_{asset}, R_{BTC})}{\text{Var}(R_{BTC})}$$

Onde:
* $R_{asset}$ = retornos do ativo
* $R_{BTC}$ = retornos do Bitcoin
* Janela de cálculo: 30 períodos

#### Explicação
* Mede sensibilidade do ativo em relação ao Bitcoin
* Beta do modelo CAPM (Capital Asset Pricing Model)
* Indica quanto o ativo move quando o BTC move 1%

#### Interpretação
* $\beta = 1$: Move igual ao BTC (correlação perfeita)
* $\beta > 1$: Move mais que o BTC (mais volátil)
* $\beta < 1$: Move menos que o BTC (mais estável)
* $\beta < 0$: Move na direção oposta (correlação negativa)

---

### 6.2 Relative Strength (Força Relativa)
**Features:** `relative_strength_btc`, `relative_strength_eth`

#### Fórmula
$$RS_t = \frac{P_{asset,t} / P_{asset,t-n}}{P_{benchmark,t} / P_{benchmark,t-n}}$$

Onde:
* $n$ = período de lookback (tipicamente 20)

#### Explicação
* Compara performance relativa do ativo vs benchmark
* Ratio dos retornos percentuais
* Identifica ativos que estão superando (outperforming) ou ficando para trás (underperforming)

#### Interpretação
* $RS > 1$: Ativo mais forte que benchmark (outperforming)
* $RS < 1$: Ativo mais fraco que benchmark (underperforming)
* $RS$ crescente: Força relativa aumentando
* $RS$ decrescente: Força relativa diminuindo

---

### 6.3 Volatility Ratio
**Feature:** `volatility_ratio_btc`

#### Fórmula
$$VR_t = \frac{\sigma_{asset,t}}{\sigma_{BTC,t}}$$

#### Explicação
* Compara a volatilidade do ativo com a do Bitcoin
* Identifica ativos mais ou menos voláteis que o benchmark

#### Interpretação
* $VR > 1$: Ativo mais volátil que BTC (maior risco)
* $VR < 1$: Ativo menos volátil que BTC (menor risco)

---

### 6.4 Volume Ratio
**Feature:** `volume_ratio_btc`

#### Fórmula
$$VR_{vol,t} = \frac{V_{asset,t}}{V_{BTC,t}}$$

#### Explicação
* Compara o volume do ativo com o do Bitcoin
* Identifica ativos com maior ou menor liquidez relativa

#### Interpretação
* $VR_{vol}$ alto: Alto volume relativo (boa liquidez)
* $VR_{vol}$ baixo: Baixo volume relativo (possível iliquidez)

---

### 6.5 BTC/ETH Raw Features
**Features:** `btc_close`, `btc_log_return`, `btc_volume`, `btc_volatility`, `eth_close`, `eth_log_return`, `eth_volume`, `eth_volatility`

#### Explicação
* Features brutas do Bitcoin e Ethereum
* Permitem ao modelo capturar dinâmicas diretas dos benchmarks
* Complementam as features de correlação

---

### 6.6 Market Breadth 🆕
**Feature:** `market_breadth`

#### Fórmula
$$MarketBreadth_t = \frac{\text{Número de pares com } Close_t > MA_{close,20}}{\text{Total de pares}}$$

#### Explicação
* Percentual de criptomoedas em tendência de alta
* Feature agregada (mesma para todos os pares no mesmo timestamp)
* Mede "saúde" geral do mercado

#### Interpretação
* $MarketBreadth > 0.7$ (70%): Mercado em alta generalizada (bull market)
* $MarketBreadth < 0.3$ (30%): Mercado em baixa generalizada (bear market)
* $MarketBreadth \approx 0.5$: Mercado misto (seletividade)
* Divergência: Market breadth caindo enquanto BTC sobe = alerta de correção

---

## Feature Importance (Top 30)

Com base no último treinamento com todas as features implementadas:

1. **log_return_4h** (6.0%) - Retorno logarítmico 4h continua sendo o fator mais crítico
2. **roc_10_1h** (5.2%) 🆕 - Rate of Change 10 períodos no timeframe operacional
3. **momentum_1h** (4.8%) - Momentum de curto prazo (velocidade do movimento)
4. **momentum_4h** (3.8%) - Momentum estratégico (tendência de médio prazo)
5. **body_range_ratio_4h** (3.3%) 🆕 - Força do movimento em 4h (candle psychology)
6. **upper_shadow_ratio_4h** (3.0%) 🆕 - Rejeição na máxima 4h (pressão vendedora)
7. **log_return_1h** (2.6%) - Retorno logarítmico operacional
8. **z_score_close_1h** (2.4%) - Normalização do preço 1h (desvio da média)
9. **bb_position_4h** (2.2%) - Posição nas Bollinger Bands 4h
10. **roc_5_1h** (2.2%) 🆕 - Rate of Change 5 períodos (curto prazo)
11. **lower_shadow_ratio_4h** (2.2%) 🆕 - Rejeição na mínima 4h (pressão compradora)
12. **hour_cos_1h** (1.8%) 🆕 - Padrão temporal (horário do dia - componente cosseno)
13. **roc_20_1h** (1.7%) 🆕 - Rate of Change 20 períodos (médio prazo)
14. **roc_5_4h** (1.5%) 🆕 - Rate of Change 5 períodos em 4h
15. **hour_sin_1h** (1.5%) 🆕 - Padrão temporal (horário do dia - componente seno)
16. **support_break_1h** (1.4%) 🆕 - Distância do suporte em 1h
17. **bb_position_1h** (1.4%) - Posição nas Bollinger Bands 1h
18. **resistance_break_1h** (1.3%) 🆕 - Distância da resistência em 1h
19. **vwap_distance_1h** (1.3%) 🆕 - Distância do VWAP em 1h
20. **btc_close** (1.3%) - Preço do Bitcoin (contexto macro)
21. **resistance_break_15m** (1.2%) 🆕 - Breakout de resistência no micro timeframe
22. **bb_position_15m** (1.2%) - Posição nas Bollinger Bands 15m
23. **lower_shadow_ratio_1h** (1.2%) 🆕 - Rejeição na mínima 1h
24. **eth_close** (1.2%) - Preço do Ethereum (contexto macro)
25. **z_score_close_4h** (1.2%) - Normalização do preço 4h
26. **resistance_break_4h** (1.1%) 🆕 - Breakout de resistência em 4h
27. **support_break_4h** (1.1%) 🆕 - Breakdown de suporte em 4h
28. **support_break_15m** (1.1%) 🆕 - Breakdown de suporte no micro timeframe
29. **rsi_1h** (1.0%) - Relative Strength Index operacional
30. **body_range_ratio_1h** (1.0%) 🆕 - Força do movimento em 1h

---

## Insights Atualizados

### Impacto do Lote 1 (Novas Features)

**Dominância das Novas Features:**
* **16 das top 30 features são do Lote 1** (53% do top 30)
* Novas features capturam aspectos não cobertos pelas originais
* ROC (Rate of Change) é extremamente relevante - 5 versões no top 30

### Timeframe Analysis

* **4h mantém supremacia estratégica:** 
  * log_return_4h (#1), body_range_ratio_4h (#5), upper_shadow_ratio_4h (#6)
  * Timeframe captura tendências de médio prazo com menos ruído
  
* **1h domina em quantidade:** 
  * Múltiplas features 1h no top 30 (roc_10, momentum, log_return, z_score)
  * Timeframe operacional ideal para horizonte de 24h
  
* **15m contribui taticamente:** 
  * resistance_break_15m, support_break_15m, bb_position_15m
  * Captura microestrutura de curto prazo para timing preciso

### Feature Categories (por importância)

1. **Momentum/ROC (NOVOS):** roc_10_1h (#2), momentum_1h (#3), momentum_4h (#4), roc_5_1h (#10)
   * Velocidade de mudança é o fator mais preditivo após log_return

2. **Price Action/Candles (NOVOS):** body_range_ratio_4h (#5), upper_shadow_ratio_4h (#6), lower_shadow_ratio_4h (#11)
   * Anatomia dos candles captura psicologia do mercado

3. **Support/Resistance (NOVOS):** support_break_1h (#16), resistance_break_1h (#18)
   * Breakouts são sinais fortes de continuação/reversão

4. **Temporal Patterns (NOVOS):** hour_cos_1h (#12), hour_sin_1h (#15)
   * Padrões intraday são relevantes (liquidez, horário de mercados)

5. **VWAP (NOVO):** vwap_distance_1h (#19)
   * Distância do VWAP é feature útil (institucional benchmark)

6. **Price Features Clássicas:** log_return_4h (#1), z_score_close_1h (#8), bb_position_4h (#9)
   * Continuam relevantes, mas complementadas pelas novas

7. **Volume Features:** Menor importância relativa
   * OBV não aparece no top 30 (possível necessidade de ajuste)

8. **Correlação com BTC/ETH:** btc_close (#20), eth_close (#24)
   * Contexto macro continua relevante

### Key Findings

* **ROC supera Momentum tradicional:** Múltiplas janelas (5, 10, 20) capturam melhor a velocidade de mudança
* **Candle Psychology é preditiva:** Proporções de corpo/sombras no top 10
* **Padrões temporais importam:** Hour_sin/cos no top 15 (liquidez intraday)
* **Multi-timeframe efetivo:** Modelo usa bem 1h, 15m e 4h
* **Breakouts técnicos funcionam:** Support/resistance breaks no top 20

---

## Próximas Features (Roadmap - Lote 2)

### 1. Sentiment & Market Microstructure
* **Funding rate:** Taxa de financiamento de contratos perpétuos (sentimento bull/bear)
* **Open interest:** Volume total de contratos abertos (interesse no mercado)
* **Long/short ratio:** Proporção de posições compradas vs vendidas

### 2. Order Book Features
* **Bid-ask spread:** Diferença entre melhor oferta de compra e venda (liquidez)
* **Order book imbalance:** Desequilíbrio entre ordens de compra/venda
* **Market depth:** Profundidade do livro de ofertas (resistência a movimentos)

### 3. On-Chain Metrics (para criptomoedas)
* **Transaction volume:** Volume de transações on-chain
* **Active addresses:** Número de endereços ativos (adoção)
* **Exchange inflow/outflow:** Fluxo para/de exchanges (pressão de venda/compra)
* **MVRV ratio:** Market Value to Realized Value (sobrecompra/sobrevenda on-chain)

### 4. Cross-Asset Correlations
* **S&P 500:** Correlação com mercado acionário tradicional (risk-on/risk-off)
* **DXY (US Dollar Index):** Correlação com dólar (risk sentiment)
* **Gold:** Correlação com safe-haven assets
* **VIX:** Correlação com volatilidade implícita de ações (fear index)

### 5. Advanced Technical Indicators
* **Ichimoku Cloud:** Sistema completo de análise técnica japonesa
* **Fibonacci Retracements:** Níveis de retração (suporte/resistência dinâmicos)
* **Elliott Wave patterns:** Padrões de ondas (psicologia de massa)
* **Volume Profile:** Distribuição de volume por nível de preço

### 6. Machine Learning Features
* **Lag features:** Valores defasados de features importantes (memória temporal)
* **Rolling statistics:** Min/Max/Percentis móveis (regime detection)
* **Feature interactions:** Produtos/razões entre features (não-linearidades)
* **Cluster labels:** Regime de mercado identificado por clustering

---

## Referências

* **RSI:** Wilder, J. W. (1978). New Concepts in Technical Trading Systems
* **Bollinger Bands:** Bollinger, J. (2001). Bollinger on Bollinger Bands
* **ATR:** Wilder, J. W. (1978). New Concepts in Technical Trading Systems
* **Beta/CAPM:** Sharpe, W. F. (1964). Capital Asset Prices
* **ADX:** Wilder, J. W. (1978). New Concepts in Technical Trading Systems
* **OBV:** Granville, J. (1963). Granville's New Key to Stock Market Profits
* **VWAP:** Berkowitz, S. A. et al. (1988). The Total Cost of Transactions on the NYSE

---

**Última atualização:** Janeiro 2025 (pós-implementação Lote 1)
