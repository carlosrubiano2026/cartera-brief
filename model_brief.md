# DETECTORES DE REGIMEN

**Generado (UTC, ISO 8601):** 2026-09-01T05:02:56Z

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (0.3 meses, EN RODAJE)
- **MS-VAR (largo)**: 2026-08-22 (0.3 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (0.3 meses, EN RODAJE)
- **cDCC**: 2026-08-23 (0.3 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (0.3 meses, EN RODAJE)
- **Growth-at-Risk** (mensual, no vota): 2026-08-26 (0.2 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, MS-VAR (largo), BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 691 | retirado · rodaje |
| MS-VAR (largo) | estres confirmado >=2d (hoy) | 0.000 | 0.50 | no | 9150 | ok · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.074 | 0.35 | no | 687 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.589 | 0.90 | no | 691 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.175 | 0.90 | no | 7 | ok · rodaje |
| Growth-at-Risk | P(crecimiento PIB<0, h=4) — NO prob. de regimen | 0.021 | — | no (mensual, no vota) | 218 | ok · rodaje |

**Concordancia: 0 de 4 modelos operativos.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y VARIOS DE ELLOS NO SON PROBABILIDADES DE REGIMEN COMPARABLES ENTRE SI -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|, y P(crecimiento<0) de Growth-at-Risk es la probabilidad de un evento macro a un anio vista -no de un regimen de mercado hoy, y ni siquiera vota-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

retirado: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.

---

## MS-VAR (largo) — regimen de comovimiento macro

MSH-VAR(1) sobre `r_NASDAQCOM`, `d_VIXCLS`, `d_BAA10Y`, PANEL_LARGO (T=9150d, muestra macro sin BTC). Estimado por EM (Hamilton-Kim), no por MLE directa -ver models/msvar.py.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -6915.7 |
| parametros | 29 |
| AIC / BIC | 13889.4 / 14095.9 |
| convergencia (EM) | si |
| iteraciones EM | 34 |

**Cadena de Markov**

| Regimen | p_ii | Duracion esperada | Prob. ergodica | |Sigma| |
|---|---|---|---|---|
| calma | 0.939 | 16.4d | 0.723 | 1.231e-04 |
| estres | 0.841 | 6.3d | 0.277 | 4.797e-02 |

**Chequeo cruzado de arranques**: dispersion de duracion **0.00% → identificado**, peor entrada de A **0.00%** (0% = las 9 entradas identicas entre los 3 arranques).


Ratio |Sigma| estres/calma: **389.8x** (minimo 3). Duracion del regimen de estres: **6.3d** (minimo 5).

**Voto de hoy (histeresis de 2d):** sin confirmar -probabilidad suavizada de hoy: 0.102. La histeresis es una capa de LECTURA sobre la probabilidad ya calculada (no cambia el filtro ni la estimacion) -motivada por que el 52.23% de las probabilidades no nitidas de este panel son tramos aislados de mediana 2 dias, no ambiguedad estructural sostenida (ver models/msvar.py).

**Limitacion conocida de A (documentada, no oculta):** la entrada NASDAQCOM←BAA10Y sale con t=1.048 en el Hessiano del optimo de EM -identificada (0% de dispersion entre arranques) pero imprecisa, consistente con eficiencia de mercado a frecuencia diaria mas que con un fallo de identificacion. Verificado que no degrada la clasificacion: perturbar A a lo largo de su direccion de menor curvatura (±1 SE) cambia la probabilidad suavizada como maximo 0.033 en toda la muestra (umbral 0.05).

**Desviaciones tipicas por regimen y correlaciones en estres**

| Serie | sd calma | sd estres | ratio |
|---|---|---|---|
| r_NASDAQCOM | 0.882 | 2.365 | 2.68 |
| d_VIXCLS | 0.829 | 2.824 | 3.41 |
| d_BAA10Y | 0.021 | 0.047 | 2.27 |

| Par | corr calma | corr estres |
|---|---|---|
| r_NASDAQCOM / d_VIXCLS | -0.687 | -0.704 |
| r_NASDAQCOM / d_BAA10Y | -0.033 | -0.214 |
| d_VIXCLS / d_BAA10Y | 0.020 | 0.196 |

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 831.9 |
| ESS minimo de phi | 15.8 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h^2) mezcla peor porque phi y sigma_h^2 estan fuertemente correlacionados a posteriori (marginal, entre barridos) con persistencia alta.

**ESS(phi)=15.8, por debajo de 400 (Vehtari et al. 2021) — limitacion CARACTERIZADA, no abierta** (auditoria 2026-08-22 a 2026-08-30, ver docstring de models/bvarsv.py: cuatro intentos de correccion probados y revertidos, mecanismo identificado, palancas restantes fuera de alcance por costo y sin justificacion -phi no alimenta ninguna decision del sistema). **Guardarraiz de publicacion: se publica la media posterior de phi, NO su intervalo de credibilidad** (tabla de abajo).


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.333 |
| sd posterior de sigma_T | 0.365 |
| IC 90% de sigma_T | [0.83, 2.03] |
| mediana de la trayectoria | 1.318 |
| cociente sigma_T / mediana | 1.01 |
| P(sigma_T > q90) +/- MCSE | 0.074 +/- 0.007 |

**Proceso de log-volatilidad por ecuacion** (solo media posterior de phi -sin IC, ver guardarraiz arriba)

| Serie | phi (persistencia, media) | sigma_h^2 | ESS de phi |
|---|---|---|---|
| r_BTCUSDT | 0.863 | 0.104 | 17 |
| r_NASDAQ100 | 0.949 | 0.055 | 66 |
| d_DGS10 | 0.956 | 0.017 | 16 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.589 |
| rho_avg (hoy) | -0.148 |
| rho_avg (mediana historica) | -0.151 |
| persistencia_dcc (a+b) | 0.991 |
| convergio | si |

**persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.**

**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.316 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.245 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.516 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1011 | 4.37 | cola pesada | 0.587 | 2.63 |
| ETH | 1011 | 3.73 | cola pesada | 0.525 | 2.66 |
| BNSOL | 1011 | 6.78 | cola pesada | 0.547 | 2.54 |
| BNB | 1011 | 4.22 | cola pesada | 0.589 | 2.64 |
| PAXG | 1011 | 3.66 | cola pesada | 0.411 | 2.66 |
| SPYB | 0 | — | **historia insuficiente** | — | — |
| SMHB | 0 | — | **historia insuficiente** | — | — |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Errores estandar asintoticos

No calculados en esta corrida. Se estiman los viernes o con `--stderr`: el Hessiano del MS-VAR son ~1700 evaluaciones de la verosimilitud y no cabe en la corrida diaria.

---

## Risk budgeting — reparto del aporte mensual

**Risk budgeting no disponible esta corrida**: sin data/raw/yahoo_*.csv — corre src/collect_yahoo.py

---

## Growth-at-Risk — riesgo a la baja del crecimiento del PIB

Adrian-Boyarchenko-Giannone (2019): regresion cuantilica de Y_{t+h} (crecimiento anualizado promedio del PIB real a h trimestres) sobre y_t (crecimiento contemporaneo) y NFCI_t (condiciones financieras), una regresion INDEPENDIENTE por cuantil. **No vota, no entra en la concordancia de arriba: mensual, y su estadistico es un cuantil de crecimiento del PIB, no una probabilidad de regimen de mercado** -ver models/gar.py.

> **EN RODAJE** (0.2 de 6 meses desde su propia respec_fecha): se acumula en regime_history.csv para medir falsos positivos, no informa ninguna decision.

**Cuantiles del crecimiento del PIB a h=4 trimestres, dado hoy** (y_t=1.47%, NFCI_t=-0.498, dato mas reciente: 2026Q2)

| Cuantil (tau) | Crecimiento PIB anualizado (%) |
|---|---|
| 0.05 | 0.73 |
| 0.10 | 1.14 |
| 0.25 | 2.10 |
| 0.50 | 2.82 |
| 0.75 | 3.99 |
| 0.90 | 4.70 |
| 0.95 | 5.75 |

**Derivados de la densidad ajustada** (skew-t de Jones-Faddy sobre los 7 cuantiles, por minimos cuadrados)

| Medida | Valor |
|---|---|
| P(crecimiento del PIB < 0% a h=4) | 2.1% |
| Expected shortfall al 5% | -0.13% |

**Asimetria de la distribucion condicional** (el hallazgo central de ABG: la cola izquierda se mueve con las condiciones financieras, la derecha mucho menos)

| Medida | Valor |
|---|---|
| Distancia q0.05 -> mediana | 2.10pp |
| Distancia mediana -> q0.95 | 2.93pp |
| Asimetria de Bowley ((q95-q50)-(q50-q05))/(q95-q05) | 0.166 |

Distancia mayor hacia arriba que hacia abajo -> asimetria al alza (tipico en calma); lo opuesto (cola izquierda mas ancha) en estres financiero.


**Coeficiente de NFCI por cuantil** (en Y_fwd = a + b·y_t + c·NFCI_t; el patron de ABG es c muy negativo en tau bajos, cercano a cero o positivo en tau altos)

| Cuantil (tau) | c_NFCI | p-value | Significativo (5%) |
|---|---|---|---|
| 0.05 | -1.872 | 0.0000 | SI |
| 0.10 | -1.484 | 0.0000 | SI |
| 0.25 | -1.137 | 0.0000 | SI |
| 0.50 | -0.426 | 0.0034 | SI |
| 0.75 | -0.199 | 0.1998 | no |
| 0.90 | 0.245 | 0.1054 | no |
| 0.95 | 0.839 | 0.0065 | SI |

> **Nota sobre tau=0.95:** en esta muestra sale POSITIVO y significativo (c=+0.84, p=0.007; excluyendo el año 2020 completo SUBE a +1.19, p<0.0001), distinto de ABG (muestra hasta ~2015). Lectura: con h=4 (un año vista), condiciones financieras tensas HOY predicen recesion-y-rebote dentro del horizonte de un año, lo que empuja el cuantil alto al alza — es una diferencia real de periodo muestral (esta muestra cubre 2015-2025), no sesgo de variable omitida: y_t siempre estuvo en la regresion. Ver models/gar.py para el detalle completo.

**Comparacion con el mes anterior:** primera corrida integrada al brief -sin punto de comparacion todavia.


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
| 0.05 | 0.0491 | 0.0311 | 5.5% |
| 0.10 | 0.1035 | 0.0001 | 10.6% |
| 0.25 | 0.0908 | 0.0086 | 25.2% |
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
| STLFSI4 | 2026-08-21 | -0.811 | 7% | 0.018 | 1704 |
| CISS (BCE) | 2026-08-28 | 0.016 | 18% | -0.001 | 7224 |

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

- **MS-VAR**: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.
- **MS-VAR (largo)**: EM (Hamilton-Kim), identificado: dispersion entre arranques 0.00%, |Sigma| ratio 389.8x, duracion 6.3d. Vota con histeresis de 2d sobre p>0.5 -ver 'REGLA DE HISTERESIS' en models/msvar.py (validado: RCM=17.39, alineacion 6/6 episodios de estres historicos, sensibilidad de A 0.033<0.05). p_suavizada de hoy=0.1018, confirmado_estres_hoy=no (2d consecutivos).
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.33 vs mediana 1.32, persistencia phi=0.86
- **cDCC**: pctl_corr=0.589 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.32, -0.24, -0.52], persistencia_dcc=0.991 — persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.587 — historia insuficiente: SPYB, SMHB

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
