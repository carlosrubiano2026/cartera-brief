# DETECTORES DE REGIMEN — 2026-08-25 05:16 UTC

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (0.1 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (0.1 meses, EN RODAJE)
- **cDCC**: 2026-08-23 (0.1 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (0.1 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 686 | no identificado · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.173 | 0.35 | no | 682 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.506 | 0.90 | no | 686 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.930 | 0.90 | SI | 7 | ok · rodaje |

**Concordancia: 1 de 3 modelos operativos.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y DOS DE ELLOS NO SON PROBABILIDADES -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

MSH-VAR(1) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. Sigma conmuta como matriz.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -1585.7 |
| parametros | 29 |
| AIC / BIC | 3229.3 / 3360.7 |
| convergencia | **NO** — lectura poco fiable |
| iteraciones | 566 |

**Cadena de Markov**

| Regimen | p_ii | Duracion esperada | Prob. ergodica | |Sigma| |
|---|---|---|---|---|
| calma | 0.924 | 13.2d | 0.819 | 6.344e-03 |
| estres | 0.658 | 2.9d | 0.181 | 7.184e-01 |

Ratio |Sigma| estres/calma: **113.2x** (minimo exigido 3). Prob. suavizada del ultimo dia: 0.054 — inestable, por eso el estadistico reportado es la fraccion de 20 dias.

**Chequeo cruzado de arranques** (criterio: dispersion relativa de duracion; <10% identificado, 10-20% marginal, >20% no identificado — ver models/msvar.py): duracion 2.86d vs 3.09d (**8.0% relativo → identificado**). p11 (rango 0.650-0.676, solo de referencia, ver docstring del modulo), fun 2.3114-2.3131.

**Desviaciones tipicas por regimen y correlaciones en estres**

| Serie | sd calma | sd estres | ratio |
|---|---|---|---|
| r_BTCUSDT | 2.311 | 4.133 | 1.79 |
| r_NASDAQ100 | 0.989 | 2.218 | 2.24 |
| d_BAMLH0A0HYM2 | 0.042 | 0.140 | 3.36 |

| Par | corr calma | corr estres |
|---|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.291 | 0.426 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.232 | -0.337 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.472 | -0.680 |

La correlacion cruzada en estres es lo que un modelo univariante no puede ver, y es lo que salta en un episodio real.

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 843.2 |
| ESS minimo de phi | 15.0 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h) mezcla peor porque phi y sigma_h estan fuertemente correlacionados a posteriori con persistencia alta: **la media de phi es fiable, su intervalo de credibilidad esta subestimado**.

**ESS(phi)=15.0 sigue por debajo de 400 (Vehtari et al. 2021), con DOS intentos de correccion probados y revertidos** (auditoria 2026-08-22 y 2026-08-23, ver docstring de models/bvarsv.py): la correccion del paso de Gibbs de B, y el interweaving ASIS (Kastner-Fruhwirth-Schnatter 2014). Ninguno mejoro el ESS sobre datos reales sin degradar otra cosa. Limitacion ABIERTA: el intervalo de credibilidad de phi/sigma_h no es fiable; el estadistico operativo (P(sigma_T>q90)) SI lo es, porque su ESS esta comodamente sobre el umbral.


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.575 |
| sd posterior de sigma_T | 0.411 |
| IC 90% de sigma_T | [1.03, 2.33] |
| mediana de la trayectoria | 1.307 |
| cociente sigma_T / mediana | 1.21 |

**Proceso de log-volatilidad por ecuacion**

| Serie | phi (persistencia) | sd | IC 90% | sigma_h | ESS |
|---|---|---|---|---|---|
| r_BTCUSDT | 0.813 | 0.095 | [0.63, 0.95] | 0.139 | 15 |
| r_NASDAQ100 | 0.955 | 0.020 | [0.92, 0.98] | 0.045 | 40 |
| d_DGS10 | 0.961 | 0.026 | [0.91, 0.99] | 0.016 | 22 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.506 |
| rho_avg (hoy) | -0.153 |
| rho_avg (mediana historica) | -0.153 |
| persistencia_dcc (a+b) | 0.991 |
| convergio | si |

**persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.**

**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.313 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.247 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.523 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1004 | 4.38 | cola pesada | 0.965 | 2.63 |
| ETH | 1004 | 3.74 | cola pesada | 0.802 | 2.66 |
| BNSOL | 1004 | 6.72 | cola pesada | 0.972 | 2.54 |
| BNB | 1004 | 4.23 | cola pesada | 0.860 | 2.64 |
| PAXG | 1004 | 3.69 | cola pesada | 0.356 | 2.66 |
| SPYB | 8447 | 6.44 | cola pesada | 0.714 | 2.55 |
| SMHB | 6592 | 9.45 | cola pesada | 0.429 | 2.48 |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Errores estandar asintoticos

No calculados en esta corrida. Se estiman los viernes o con `--stderr`: el Hessiano del MS-VAR son ~1700 evaluaciones de la verosimilitud y no cabe en la corrida diaria.

---

## Indices de referencia (lectura directa, no votan)

Construidos por bancos centrales o academicos sobre decenas o cientos de series primarias. No entran en ningun modelo de este brief ni cuentan rodaje: se leen tal cual.

**Semanales** (cambio vs. ~7 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 7d | N obs |
|---|---|---|---|---|---|
| NFCI | 2026-08-14 | -0.559 | 30% | -0.004 | 2902 |
| ANFCI | 2026-08-14 | -0.587 | 25% | 0.004 | 2902 |
| STLFSI4 | 2026-08-14 | -0.829 | 6% | -0.058 | 1703 |
| CISS (BCE) | 2026-08-21 | 0.017 | 19% | 0.007 | 7219 |

**Mensuales** (cambio vs. ~30 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 30d | N obs |
|---|---|---|---|---|---|
| EPU (Baker-Bloom-Davis) | 2026-07-01 | 183.768 | 91% | -29.527 | 499 |
| GPR (Caldara-Iacoviello) | 2026-07-01 | 152.668 | 92% | -27.054 | 499 |
| JLN 1M (Jurado-Ludvigson-Ng) | 2026-06-01 | 0.675 | 75% | 0.014 | 792 |
| JLN 3M | 2026-06-01 | 0.824 | 77% | 0.017 | 792 |
| JLN 12M | 2026-06-01 | 0.931 | 74% | 0.009 | 792 |

---

## Lectura conjunta

Sin concordancia. Nada que evaluar por esta via.

- **MS-VAR**: |Sigma| ratio 113.2 (min 3), duracion 2.9d (min 5): sin regimen distinguible
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.58 vs mediana 1.31, persistencia phi=0.81
- **cDCC**: pctl_corr=0.506 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.31, -0.25, -0.52], persistencia_dcc=0.991 — persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.965; proxies: SPYB<-SPY, SMHB<-SMH

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
