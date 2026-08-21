# DETECTORES DE REGIMEN — 2026-08-21 16:38 UTC

> **EN RODAJE** (0.1 de 6 meses). Las salidas se registran pero **no informan ninguna decision**. Sirven para acumular historial y medir la tasa de falsos positivos de cada modelo antes de darle voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 685 | no identificado |
| MS-DFM | P(estres) suavizada | 0.000 | 0.70 | no | 713 | ok |
| BVAR-SV | P(sigma_T > q90) | 0.068 | 0.35 | no | 680 | ok |

**Concordancia: 0 de 2 modelos operativos.** Cada estadistico tiene nula distinta (MS-VAR ~0.01, MS-DFM ~0.15, BVAR-SV 0.10 por construccion): no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

no identificado: |Sigma| ratio 92.0 (min 3), duracion 4.8d (min 5): sin regimen distinguible

---

## MS-DFM — factor comun latente

Filtro de Kim conjunto: factor y regimen en una sola verosimilitud.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -6549.6 |
| parametros / obs | 20 / 713 |
| AIC / BIC | 13139.2 / 13230.6 |
| convergencia | si |

**Dinamica del factor**

| Parametro | Valor | Lectura |
|---|---|---|
| phi1 | -0.145 | AR(1) del factor |
| phi2 | -0.019 | AR(2) |
| phi1+phi2 | -0.164 | persistencia; cerca de 1 = factor casi integrado |
| mu calma | -0.022 |  |
| mu estres | 8.220 |  |
| separacion | 8.242 | en sd del factor; si es pequena los regimenes no se distinguen |

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
| d_VIXCLS | 100% | 0.918 | 0.179 | 4.71 |
| r_BTCUSDT | 91% | 0.285 | 0.725 | 0.11 |
| r_NASDAQ100 | 97% | 0.705 | 0.449 | 1.11 |
| d_BAMLH0A0HYM2 | 100% | 0.513 | 0.587 | 0.45 |
| d_NFCI | 100% | 0.090 | 1.043 | 0.01 |
| d_STLFSI4 | 100% | 0.119 | 1.151 | 0.01 |
| d_ANFCI | 100% | 0.075 | 0.930 | 0.01 |

**Cadena de Markov**

| Regimen | p_ii | Duracion | Ergodica |
|---|---|---|---|
| calma | 0.996 | 234.3d | 0.992 |
| estres | 0.500 | 2.0d | 0.008 |

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 795.4 |
| ESS minimo de phi | 14.5 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h) mezcla peor porque phi y sigma_h estan fuertemente correlacionados a posteriori con persistencia alta: **la media de phi es fiable, su intervalo de credibilidad esta subestimado**.


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.434 |
| sd posterior de sigma_T | 0.297 |
| IC 90% de sigma_T | [1.02, 1.98] |
| mediana de la trayectoria | 1.340 |
| cociente sigma_T / mediana | 1.07 |

**Proceso de log-volatilidad por ecuacion**

| Serie | phi (persistencia) | sd | IC 90% | sigma_h | ESS |
|---|---|---|---|---|---|
| r_BTCUSDT | 0.900 | 0.043 | [0.82, 0.96] | 0.066 | 19 |
| r_NASDAQ100 | 0.949 | 0.023 | [0.91, 0.98] | 0.057 | 26 |
| d_DGS10 | 0.956 | 0.029 | [0.90, 0.99] | 0.018 | 15 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## Errores estandar asintoticos

Hessiano numerico en el optimo. **Advertencia:** en modelos Markov-switching el Hessiano suele estar mal condicionado y para p_ii cerca de la frontera la aproximacion normal es mala. No leer como los de una regresion lineal.

**MS-DFM:** cond=2.16e+03 · ok · 15 de 20 parametros con |t| > 1.96 · se mediano=0.0473

---

## Lectura conjunta

Sin concordancia. Nada que evaluar por esta via.

- **MS-VAR**: |Sigma| ratio 92.0 (min 3), duracion 4.8d (min 5): sin regimen distinguible
- **MS-DFM**: loglik=-6549.6, 7 series, muestra comun 2023-11-14..2026-08-20, Kim conjunto
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.43 vs mediana 1.34, persistencia phi=0.90

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.

<!-- prueba de publicacion 2026-08-21T16:56:46Z -->
