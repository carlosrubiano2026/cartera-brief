# DETECTORES DE REGIMEN — 2026-08-26 15:13 UTC

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (0.1 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (0.1 meses, EN RODAJE)
- **cDCC**: 2026-08-23 (0.1 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (0.1 meses, EN RODAJE)
- **Growth-at-Risk** (mensual, no vota): 2026-08-26 (0.0 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 688 | no identificado · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.120 | 0.35 | no | 683 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.468 | 0.90 | no | 688 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.839 | 0.90 | no | 7 | ok · rodaje |
| Growth-at-Risk | P(crecimiento PIB<0, h=4) — NO prob. de regimen | 0.020 | — | no (mensual, no vota) | 218 | ok · rodaje |

**Concordancia: 0 de 3 modelos operativos.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y VARIOS DE ELLOS NO SON PROBABILIDADES DE REGIMEN COMPARABLES ENTRE SI -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|, y P(crecimiento<0) de Growth-at-Risk es la probabilidad de un evento macro a un anio vista -no de un regimen de mercado hoy, y ni siquiera vota-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

MSH-VAR(1) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. Sigma conmuta como matriz.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -1589.0 |
| parametros | 29 |
| AIC / BIC | 3236.0 / 3367.4 |
| convergencia | si |
| iteraciones | 134 |

**Cadena de Markov**

| Regimen | p_ii | Duracion esperada | Prob. ergodica | |Sigma| |
|---|---|---|---|---|
| calma | 0.926 | 13.4d | 0.825 | 6.399e-03 |
| estres | 0.649 | 2.9d | 0.175 | 7.351e-01 |

Ratio |Sigma| estres/calma: **114.9x** (minimo exigido 3). Prob. suavizada del ultimo dia: 0.028 — inestable, por eso el estadistico reportado es la fraccion de 20 dias.

**Chequeo cruzado de arranques** (criterio: dispersion relativa de duracion; <10% identificado, 10-20% marginal, >20% no identificado — ver models/msvar.py): duracion 2.85d vs 2.90d (**1.7% relativo → identificado**). p11 (rango 0.649-0.655, solo de referencia, ver docstring del modulo), fun 2.3096-2.3113.

**Desviaciones tipicas por regimen y correlaciones en estres**

| Serie | sd calma | sd estres | ratio |
|---|---|---|---|
| r_BTCUSDT | 2.317 | 4.139 | 1.79 |
| r_NASDAQ100 | 0.989 | 2.231 | 2.25 |
| d_BAMLH0A0HYM2 | 0.042 | 0.141 | 3.38 |

| Par | corr calma | corr estres |
|---|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.291 | 0.428 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.231 | -0.339 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.472 | -0.682 |

La correlacion cruzada en estres es lo que un modelo univariante no puede ver, y es lo que salta en un episodio real.

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 877.0 |
| ESS minimo de phi | 12.3 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h) mezcla peor porque phi y sigma_h estan fuertemente correlacionados a posteriori con persistencia alta: **la media de phi es fiable, su intervalo de credibilidad esta subestimado**.

**ESS(phi)=12.3 sigue por debajo de 400 (Vehtari et al. 2021), con DOS intentos de correccion probados y revertidos** (auditoria 2026-08-22 y 2026-08-23, ver docstring de models/bvarsv.py): la correccion del paso de Gibbs de B, y el interweaving ASIS (Kastner-Fruhwirth-Schnatter 2014). Ninguno mejoro el ESS sobre datos reales sin degradar otra cosa. Limitacion ABIERTA: el intervalo de credibilidad de phi/sigma_h no es fiable; el estadistico operativo (P(sigma_T>q90)) SI lo es, porque su ESS esta comodamente sobre el umbral.


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.424 |
| sd posterior de sigma_T | 0.445 |
| IC 90% de sigma_T | [0.84, 2.22] |
| mediana de la trayectoria | 1.300 |
| cociente sigma_T / mediana | 1.09 |

**Proceso de log-volatilidad por ecuacion**

| Serie | phi (persistencia) | sd | IC 90% | sigma_h | ESS |
|---|---|---|---|---|---|
| r_BTCUSDT | 0.751 | 0.138 | [0.50, 0.93] | 0.199 | 12 |
| r_NASDAQ100 | 0.962 | 0.017 | [0.93, 0.99] | 0.039 | 53 |
| d_DGS10 | 0.965 | 0.021 | [0.93, 0.99] | 0.013 | 35 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.468 |
| rho_avg (hoy) | -0.153 |
| rho_avg (mediana historica) | -0.151 |
| persistencia_dcc (a+b) | 0.991 |
| convergio | si |

**persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.**

**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.312 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.256 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.514 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1005 | 4.40 | cola pesada | 0.081 | 2.63 |
| ETH | 1005 | 3.75 | cola pesada | 0.195 | 2.66 |
| BNSOL | 1005 | 6.76 | cola pesada | 0.070 | 2.54 |
| BNB | 1005 | 4.24 | cola pesada | 0.091 | 2.64 |
| PAXG | 1005 | 3.68 | cola pesada | 0.282 | 2.66 |
| SPYB | 8447 | 6.44 | cola pesada | 0.714 | 2.55 |
| SMHB | 6592 | 9.45 | cola pesada | 0.429 | 2.48 |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Errores estandar asintoticos

No calculados en esta corrida. Se estiman los viernes o con `--stderr`: el Hessiano del MS-VAR son ~1700 evaluaciones de la verosimilitud y no cabe en la corrida diaria.

---

## Risk budgeting — reparto del aporte mensual

Contribucion marginal y total al riesgo desde Sigma=D·R·D del cDCC (D de GARCH univariante por posicion, R=`rho_hoy`). Las contribuciones (PCTR) suman 100% del riesgo de cartera por identidad de Euler, no por normalizacion.

> **Regla de asignacion (decision cerrada, auditoria 2026-08-26): 30% del aporte va fijo al cluster cripto (BTC+ETH+BNSOL+BNB), proporcional a los pesos objetivo de `portfolio.yaml` -no por minima varianza, la tesis de inversion vive en la config, no en el optimizador-. El resto (70%) se asigna por minima varianza entre PAXG/SPYB/SMHB, tope 50% del aporte por posicion. Ver `models/risk_budget.py` (docstring del modulo) para el Monte Carlo que motivo la decision -incluye por que se descarto un techo de PCTR cripto. La brecha de la tabla (PCTR vs peso objetivo) sigue siendo diagnostico de capital-vs-riesgo, ya no es lo que determina el aporte de abajo.

| Posicion | Peso actual | Contrib. riesgo (PCTR) | Peso objetivo | Brecha | Aporte sugerido ($) |
|---|---|---|---|---|---|
| BTC | 26.2% | 35.3% | 24.0% | -11.3pp | 24 |
| ETH | 17.1% | 27.3% | 15.0% | -12.3pp | 15 |
| BNSOL | 10.0% | 17.1% | 10.0% | -7.1pp | 10 |
| BNB | 10.4% | 10.3% | 10.0% | -0.3pp | 10 |
| PAXG | 9.5% | 3.5% | 10.0% | +6.5pp | 40 |
| SPYB | 17.6% | 2.8% | 20.0% | +17.2pp | 100 |
| SMHB | 9.2% | 3.5% | 10.0% | +6.5pp | 0 |

sigma_p actual: 2.2789 -> tras el aporte: 1.8605. SOLO-COMPRA: nunca vende (vender antes de 2028-08-18 dispara renta ordinaria).

---

## Growth-at-Risk — riesgo a la baja del crecimiento del PIB

Adrian-Boyarchenko-Giannone (2019): regresion cuantilica de Y_{t+h} (crecimiento anualizado promedio del PIB real a h trimestres) sobre y_t (crecimiento contemporaneo) y NFCI_t (condiciones financieras), una regresion INDEPENDIENTE por cuantil. **No vota, no entra en la concordancia de arriba: mensual, y su estadistico es un cuantil de crecimiento del PIB, no una probabilidad de regimen de mercado** -ver models/gar.py.

> **EN RODAJE** (0.0 de 6 meses desde su propia respec_fecha): se acumula en regime_history.csv para medir falsos positivos, no informa ninguna decision.

**Cuantiles del crecimiento del PIB a h=4 trimestres, dado hoy** (y_t=1.49%, NFCI_t=-0.498, dato mas reciente: 2026Q2)

| Cuantil (tau) | Crecimiento PIB anualizado (%) |
|---|---|
| 0.05 | 0.74 |
| 0.10 | 1.14 |
| 0.25 | 2.10 |
| 0.50 | 2.82 |
| 0.75 | 3.99 |
| 0.90 | 4.70 |
| 0.95 | 5.75 |

**Derivados de la densidad ajustada** (skew-t de Jones-Faddy sobre los 7 cuantiles, por minimos cuadrados)

| Medida | Valor |
|---|---|
| P(crecimiento del PIB < 0% a h=4) | 2.0% |
| Expected shortfall al 5% | -0.10% |

**Asimetria de la distribucion condicional** (el hallazgo central de ABG: la cola izquierda se mueve con las condiciones financieras, la derecha mucho menos)

| Medida | Valor |
|---|---|
| Distancia q0.05 -> mediana | 2.08pp |
| Distancia mediana -> q0.95 | 2.93pp |
| Asimetria de Bowley ((q95-q50)-(q50-q05))/(q95-q05) | 0.170 |

Distancia mayor hacia arriba que hacia abajo -> asimetria al alza (tipico en calma); lo opuesto (cola izquierda mas ancha) en estres financiero.


**Coeficiente de NFCI por cuantil** (en Y_fwd = a + b·y_t + c·NFCI_t; el patron de ABG es c muy negativo en tau bajos, cercano a cero o positivo en tau altos)

| Cuantil (tau) | c_NFCI | p-value | Significativo (5%) |
|---|---|---|---|
| 0.05 | -1.911 | 0.0000 | SI |
| 0.10 | -1.484 | 0.0000 | SI |
| 0.25 | -1.138 | 0.0000 | SI |
| 0.50 | -0.426 | 0.0034 | SI |
| 0.75 | -0.199 | 0.1998 | no |
| 0.90 | 0.245 | 0.1054 | no |
| 0.95 | 0.839 | 0.0065 | SI |

> **Nota sobre tau=0.95:** en esta muestra sale POSITIVO y significativo (c=+0.84, p=0.007; excluyendo el año 2020 completo SUBE a +1.19, p<0.0001), distinto de ABG (muestra hasta ~2015). Lectura: con h=4 (un año vista), condiciones financieras tensas HOY predicen recesion-y-rebote dentro del horizonte de un año, lo que empuja el cuantil alto al alza — es una diferencia real de periodo muestral (esta muestra cubre 2015-2025), no sesgo de variable omitida: y_t siempre estuvo en la regresion. Ver models/gar.py para el detalle completo.

**Comparacion con el mes anterior** (corrida 2026-08-26, dato 2026Q2)

| Cuantil | Valor anterior | Valor actual | Delta (pp) |
|---|---|---|---|
| 0.05 | 0.74 | 0.74 | +0.00 |
| 0.10 | 1.14 | 1.14 | +0.00 |
| 0.25 | 2.10 | 2.10 | +0.00 |
| 0.50 | 2.82 | 2.82 | +0.00 |
| 0.75 | 3.99 | 3.99 | +0.00 |
| 0.90 | 4.70 | 4.70 | +0.00 |
| 0.95 | 5.75 | 5.75 | +0.00 |

Cola izquierda (cuantil 0.05) practicamente estable desde el mes anterior (+0.00pp).


**Diagnostico**

| Diagnostico | Valor |
|---|---|
| Dato GDPC1 mas reciente usado | 2026Q2 |
| Muestra de estimacion | 1971Q1 a 2025Q2 (218 obs; los ultimos 4 trimestres no entran en la estimacion, Y_fwd todavia no se realiza) |
| Cruce de cuantiles en la muestra (no deberia haber) | 3/218 (1.4%) |
| Backtest: gana el condicional vs. incondicional | 7/7 cuantiles |

**Backtest por cuantil** (perdida pinball, condicional -con NFCI- vs. incondicional; cobertura empirica deberia acercarse al cuantil nominal)

| Cuantil | Mejora del condicional | p-value | Cobertura empirica |
|---|---|---|---|
| 0.05 | 0.0491 | 0.0321 | 5.5% |
| 0.10 | 0.1035 | 0.0001 | 10.6% |
| 0.25 | 0.0908 | 0.0086 | 25.7% |
| 0.50 | 0.0411 | 0.0295 | 50.0% |
| 0.75 | 0.0091 | 0.3263 | 74.8% |
| 0.90 | 0.0076 | 0.1641 | 89.4% |
| 0.95 | 0.0092 | 0.3527 | 94.5% |

---

## Indices de referencia (lectura directa, no votan)

Construidos por bancos centrales o academicos sobre decenas o cientos de series primarias. No entran en ningun modelo de este brief ni cuentan rodaje: se leen tal cual.

**Semanales** (cambio vs. ~7 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 7d | N obs |
|---|---|---|---|---|---|
| NFCI | 2026-08-21 | -0.566 | 29% | -0.005 | 2903 |
| ANFCI | 2026-08-21 | -0.576 | 25% | 0.002 | 2903 |
| STLFSI4 | 2026-08-14 | -0.829 | 6% | -0.058 | 1703 |
| CISS (BCE) | 2026-08-25 | 0.016 | 18% | 0.000 | 7221 |

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

- **MS-VAR**: |Sigma| ratio 114.9 (min 3), duracion 2.9d (min 5): sin regimen distinguible
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.42 vs mediana 1.30, persistencia phi=0.75
- **cDCC**: pctl_corr=0.468 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.31, -0.26, -0.51], persistencia_dcc=0.991 — persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.081; proxies: SPYB<-SPY, SMHB<-SMH

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
