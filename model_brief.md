# DETECTORES DE REGIMEN — 2026-08-22 14:08 UTC

Ultima reespecificacion: **2026-08-22** — el historial de `regime_history.csv` cuenta desde ahi; lo de antes queda en `regime_history.pre-audit.csv`.

> **EN RODAJE** (0.0 de 6 meses desde la ultima reespecificacion). Las salidas se registran pero **no informan ninguna decision**. Sirven para acumular historial y medir la tasa de falsos positivos de cada modelo antes de darle voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 685 | no identificado |
| MS-DFM | P(estres) suavizada | — | 0.70 | no | 713 | mecanismo de outliers |
| BVAR-SV | P(sigma_T > q90) | 0.246 | 0.35 | no | 681 | ok |

**Concordancia: 0 de 1 modelos operativos.** Cada estadistico tiene nula distinta (MS-VAR ~0.01, MS-DFM ~0.15, BVAR-SV 0.10 por construccion): no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

no identificado: |Sigma| ratio 92.0 (min 3), duracion 4.8d (min 5): sin regimen distinguible

---

## MS-DFM — factor comun latente

Filtro de Kim conjunto: factor y regimen en una sola verosimilitud.

Este es un modelo de **comovimiento**. BTCUSDT se captura en vivo a la hora en que corre la tarea diaria (sin hora fija), mientras que VIX/HY OAS/Nasdaq llevan la fecha de su propio cierre de mercado: la misma fecha del panel puede mezclar instantes reales distintos entre columnas (ver docstring de collect_binance.py). No es look-ahead, pero un desfase sistematico entre columnas sesga el comovimiento medido a la baja.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -3472.3 |
| parametros / obs | 14 / 713 |
| AIC / BIC | 6972.5 / 7036.5 |
| convergencia | si |

**Cumulador:** sin cumulador — todas las series puntuales.


**Dinamica del factor**

| Parametro | Valor | Lectura |
|---|---|---|
| phi1 | -0.154 | AR(1) del factor |
| phi2 | -0.021 | AR(2) |
| phi1+phi2 | -0.175 | persistencia; cerca de 1 = factor casi integrado |
| mu calma | -0.024 |  |
| mu estres | 8.252 |  |
| separacion | 8.276 | en sd del factor; si es pequena los regimenes no se distinguen |

**Series descartadas por cobertura insuficiente (<5%)**

| Serie | cobertura |
|---|---|
| dram_contract_qoq_pct | 0.3% |
| btc_etf_flow_wk_musd | 0.0% |
| hyperscaler_capex_2026_bnusd | 0.1% |

Una serie con muy pocas observaciones no informa el factor: el optimizador la colapsa contra la frontera y deja parametros no identificados.


**Cargas y senal-ruido por serie**

Λ mide cuanto pesa cada serie en el factor; la ratio senal-ruido (Λ²/varianza idiosincratica) dice cuanta de su variacion es comun. Una serie con ratio baja no esta informando el factor.

| Serie | cobertura | carga Λ | var idio | senal/ruido |
|---|---|---|---|---|
| d_VIXCLS | 100% | 0.928 | 0.149 | 5.76 |
| r_BTCUSDT | 91% | 0.281 | 0.728 | 0.11 |
| r_NASDAQ100 | 97% | 0.698 | 0.461 | 1.06 |
| d_BAMLH0A0HYM2 | 100% | 0.503 | 0.602 | 0.42 |

**Cadena de Markov**

| Regimen | p_ii | Duracion | Ergodica |
|---|---|---|---|
| calma | 0.996 | 234.1d | 0.993 |
| estres | 0.370 | 1.6d | 0.007 |

**p_ii(estres)=0.370 < 0.5: menos persistente que una moneda. Esto no es un regimen, es un mecanismo de outliers** — una componente de mixtura que absorbe dias atipicos de alta desviacion, sin la inercia que definiria un estado economico. Leer `p_stress` de este modelo como "probabilidad de un dia raro", no como "probabilidad de estar en un regimen de estres".

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 690.7 |
| ESS minimo de phi | 10.8 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h) mezcla peor porque phi y sigma_h estan fuertemente correlacionados a posteriori con persistencia alta: **la media de phi es fiable, su intervalo de credibilidad esta subestimado**.


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.697 |
| sd posterior de sigma_T | 0.427 |
| IC 90% de sigma_T | [1.13, 2.42] |
| mediana de la trayectoria | 1.300 |
| cociente sigma_T / mediana | 1.30 |

**Proceso de log-volatilidad por ecuacion**

| Serie | phi (persistencia) | sd | IC 90% | sigma_h | ESS |
|---|---|---|---|---|---|
| r_BTCUSDT | 0.811 | 0.104 | [0.61, 0.93] | 0.149 | 11 |
| r_NASDAQ100 | 0.947 | 0.024 | [0.90, 0.98] | 0.059 | 48 |
| d_DGS10 | 0.958 | 0.027 | [0.91, 0.99] | 0.018 | 21 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## Errores estandar asintoticos

No calculados en esta corrida. Se estiman los viernes o con `--stderr`: el Hessiano del MS-VAR son ~1700 evaluaciones de la verosimilitud y no cabe en la corrida diaria.

---

## Lectura conjunta

Sin concordancia. Nada que evaluar por esta via.

- **MS-VAR**: |Sigma| ratio 92.0 (min 3), duracion 4.8d (min 5): sin regimen distinguible
- **MS-DFM**: loglik=-3472.3, 4 series, muestra comun 2023-11-14..2026-08-20, Kim conjunto — p_ii(estres)=0.370 < 0.5: no es un regimen distinguible, ver tabla de Markov
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.70 vs mediana 1.30, persistencia phi=0.81

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
