# DETECTORES DE REGIMEN — 2026-08-23 05:19 UTC

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (0.0 meses, EN RODAJE)
- **MS-DFM**: 2026-08-22 (0.0 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (0.0 meses, EN RODAJE)
- **cDCC**: 2026-08-23 (0.0 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (0.0 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, MS-DFM, BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 685 | no identificado · rodaje |
| MS-DFM | P(estres) suavizada | — | 0.70 | no | 713 | mecanismo de outliers · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.246 | 0.35 | no | 681 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.559 | 0.90 | no | 685 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.365 | 0.90 | no | 7 | ok · rodaje |

**Concordancia: 0 de 3 modelos operativos.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, MS-DFM ~0.15, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y DOS DE ELLOS NO SON PROBABILIDADES -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

no identificado: p11 no coincide entre arranques (0.626 vs 0.684) con verosimilitud casi igual (fun 2.3132 vs 2.3143): cresta plana, regimen no identificado por los datos

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
| separacion | 8.275 | en sd del factor; si es pequena los regimenes no se distinguen |

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

**ESS(phi)=10.8 sigue por debajo de 400 (Vehtari et al. 2021), con DOS intentos de correccion probados y revertidos** (auditoria 2026-08-22 y 2026-08-23, ver docstring de models/bvarsv.py): la correccion del paso de Gibbs de B, y el interweaving ASIS (Kastner-Fruhwirth-Schnatter 2014). Ninguno mejoro el ESS sobre datos reales sin degradar otra cosa. Limitacion ABIERTA: el intervalo de credibilidad de phi/sigma_h no es fiable; el estadistico operativo (P(sigma_T>q90)) SI lo es, porque su ESS esta comodamente sobre el umbral.


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

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.559 |
| rho_avg (hoy) | -0.151 |
| rho_avg (mediana historica) | -0.153 |
| persistencia_dcc (a+b) | 0.991 |
| convergio | si |

**persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.**

**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.326 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.256 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.522 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1002 | 4.37 | cola pesada | 0.318 | 2.63 |
| ETH | 1002 | 3.74 | cola pesada | 0.249 | 2.66 |
| BNSOL | 1002 | 6.68 | cola pesada | 0.321 | 2.54 |
| BNB | 1002 | 4.21 | cola pesada | 0.208 | 2.64 |
| PAXG | 1002 | 3.69 | cola pesada | 0.489 | 2.66 |
| SPYB | 8447 | 6.44 | cola pesada | 0.714 | 2.55 |
| SMHB | 6592 | 9.45 | cola pesada | 0.429 | 2.48 |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Errores estandar asintoticos

No calculados en esta corrida. Se estiman los viernes o con `--stderr`: el Hessiano del MS-VAR son ~1700 evaluaciones de la verosimilitud y no cabe en la corrida diaria.

---

## Lectura conjunta

Sin concordancia. Nada que evaluar por esta via.

- **MS-VAR**: p11 no coincide entre arranques (0.626 vs 0.684) con verosimilitud casi igual (fun 2.3132 vs 2.3143): cresta plana, regimen no identificado por los datos
- **MS-DFM**: loglik=-3472.3, 4 series, muestra comun 2023-11-14..2026-08-20, Kim conjunto — p_ii(estres)=0.370 < 0.5: no es un regimen distinguible, ver tabla de Markov
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.70 vs mediana 1.30, persistencia phi=0.81
- **cDCC**: pctl_corr=0.559 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.33, -0.26, -0.52], persistencia_dcc=0.991 — persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.318; proxies: SPYB<-SPY, SMHB<-SMH

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
