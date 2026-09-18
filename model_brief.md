# DETECTORES DE REGIMEN

**Generado (UTC, ISO 8601):** 2026-09-18T05:03:45Z
**Caducidad (UTC, ISO 8601):** 2026-09-19T17:03:46Z -- TTL=36h, FIJADO por Carlos el 2026-09-10 (ver src/run_models.py, TTL_BRIEF_HORAS, para el argumento completo y la condicion de reapertura). Pasada esta fecha, este documento NO debe usarse para decidir sin verificar antes que la publicacion de hoy funciono.

## Frescura de los datos

- **MS-VAR**: input mas viejo hace 2 dia(s) (fuente: fred).
- **MS-VAR (largo)**: input mas viejo hace 2 dia(s) (fuente: fred).
- **BVAR-SV**: input mas viejo hace 2 dia(s) (fuente: fred).
- **cDCC**: input mas viejo hace 2 dia(s) (fuente: fred).
- **GARCH-t**: input mas viejo hace 1 dia(s) (fuente: yahoo).
- fuente **binance** (mejor caso): 0 dia(s).
- fuente **fred** (mejor caso): 1 dia(s).
- fuente **yahoo** (mejor caso): 1 dia(s).
- fuente **indices_lectura** (mejor caso): 3 dia(s).

**IMPORTANTE**: la frescura de arriba es POR FUENTE/MODELO, no del panel entero -una fila de hoy de Binance (cotiza 24/7) no implica que las series macro de un modelo esten al dia. Ver `output/model_results.json` (`fuentes`/`modelos.*.frescura_input`) para el detalle completo por serie.

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (0.9 meses, EN RODAJE)
- **MS-VAR (largo)**: 2026-08-22 (0.9 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (0.9 meses, EN RODAJE)
- **cDCC**: 2026-08-23 (0.9 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (0.9 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, MS-VAR (largo), BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 703 | retirado · rodaje |
| MS-VAR (largo) | estres confirmado >=2d (hoy) | 0.000 | 0.50 | no | 9162 | ok · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.019 | 0.35 | no | 699 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.701 | 0.90 | no | 703 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.574 | 0.90 | no | 7 | ok · rodaje |

**Concordancia: 0 de 4 modelos evaluables.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y VARIOS DE ELLOS NO SON PROBABILIDADES DE REGIMEN COMPARABLES ENTRE SI -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

retirado: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.

---

## MS-VAR (largo) — regimen de comovimiento macro

MSH-VAR(1) sobre `r_NASDAQCOM`, `d_VIXCLS`, `d_BAA10Y`, PANEL_LARGO (T=9162d, muestra macro sin BTC). Estimado por EM (Hamilton-Kim), no por MLE directa -ver models/msvar.py.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -6916.4 |
| parametros | 29 |
| AIC / BIC | 13890.8 / 14097.4 |
| convergencia (EM) | si |
| iteraciones EM | 32 |

**Cadena de Markov**

| Regimen | p_ii | Duracion esperada | Prob. ergodica | |Sigma| |
|---|---|---|---|---|
| calma | 0.939 | 16.5d | 0.724 | 1.235e-04 |
| estres | 0.841 | 6.3d | 0.276 | 4.806e-02 |

**Chequeo cruzado de arranques**: dispersion de duracion **0.00% → identificado**, peor entrada de A **0.00%** (0% = las 9 entradas identicas entre los 3 arranques).


Ratio |Sigma| estres/calma: **389.2x** (minimo 3). Duracion del regimen de estres: **6.3d** (minimo 5).

**Voto de hoy (histeresis de 2d):** sin confirmar -probabilidad suavizada de hoy: 0.043. La histeresis es una capa de LECTURA sobre la probabilidad ya calculada (no cambia el filtro ni la estimacion) -motivada por que el 52.23% de las probabilidades no nitidas de este panel son tramos aislados de mediana 2 dias, no ambiguedad estructural sostenida (ver models/msvar.py).

**Limitacion conocida de A (documentada, no oculta):** la entrada NASDAQCOM←BAA10Y sale con t=1.048 en el Hessiano del optimo de EM -identificada (0% de dispersion entre arranques) pero imprecisa, consistente con eficiencia de mercado a frecuencia diaria mas que con un fallo de identificacion. Verificado que no degrada la clasificacion: perturbar A a lo largo de su direccion de menor curvatura (±1 SE) cambia la probabilidad suavizada como maximo 0.033 en toda la muestra (umbral 0.05).

**Desviaciones tipicas por regimen y correlaciones en estres**

| Serie | sd calma | sd estres | ratio |
|---|---|---|---|
| r_NASDAQCOM | 0.882 | 2.366 | 2.68 |
| d_VIXCLS | 0.830 | 2.825 | 3.40 |
| d_BAA10Y | 0.021 | 0.047 | 2.27 |

| Par | corr calma | corr estres |
|---|---|---|
| r_NASDAQCOM / d_VIXCLS | -0.687 | -0.704 |
| r_NASDAQCOM / d_BAA10Y | -0.033 | -0.214 |
| d_VIXCLS / d_BAA10Y | 0.019 | 0.197 |

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 648.7 |
| ESS minimo de phi | 13.5 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h^2) mezcla peor porque phi y sigma_h^2 estan fuertemente correlacionados a posteriori (marginal, entre barridos) con persistencia alta.

**ESS(phi)=13.5, por debajo de 400 (Vehtari et al. 2021) — limitacion CARACTERIZADA, no abierta** (auditoria 2026-08-22 a 2026-08-30, ver docstring de models/bvarsv.py: cuatro intentos de correccion probados y revertidos, mecanismo identificado, palancas restantes fuera de alcance por costo y sin justificacion -phi no alimenta ninguna decision del sistema). **Guardarraiz de publicacion: se publica la media posterior de phi, NO su intervalo de credibilidad** (tabla de abajo).


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.174 |
| sd posterior de sigma_T | 0.298 |
| IC 90% de sigma_T | [0.75, 1.69] |
| mediana de la trayectoria | 1.312 |
| cociente sigma_T / mediana | 0.89 |
| P(sigma_T > q90) +/- MCSE | 0.019 +/- 0.003 |

**Proceso de log-volatilidad por ecuacion** (solo media posterior de phi -sin IC, ver guardarraiz arriba)

| Serie | phi (persistencia, media) | sigma_h^2 | ESS de phi |
|---|---|---|---|
| r_BTCUSDT | 0.870 | 0.101 | 13 |
| r_NASDAQ100 | 0.957 | 0.043 | 45 |
| d_DGS10 | 0.971 | 0.012 | 46 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.701 |
| rho_avg (hoy) | -0.142 |
| rho_avg (mediana historica) | -0.150 |
| persistencia_dcc (a+b) | 0.990 |
| convergio | si |

**persistencia_dcc=0.990 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.**

**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.297 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.209 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.515 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1028 | 4.34 | cola pesada | 0.787 | 2.64 |
| ETH | 1028 | 3.73 | cola pesada | 0.744 | 2.66 |
| BNSOL | 1028 | 6.81 | cola pesada | 0.962 | 2.54 |
| BNB | 1028 | 4.21 | cola pesada | 0.952 | 2.64 |
| PAXG | 1028 | 3.69 | cola pesada | 0.943 | 2.66 |
| SPYB | 8465 | 6.46 | cola pesada | 0.955 | 2.55 |
| SMHB | 6610 | 9.47 | cola pesada | 0.893 | 2.48 |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Indices de referencia (lectura directa, no votan)

Construidos por bancos centrales o academicos sobre decenas o cientos de series primarias. No entran en ningun modelo de este brief ni cuentan rodaje: se leen tal cual.

**Semanales** (cambio vs. ~7 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 7d | N obs |
|---|---|---|---|---|---|
| NFCI | 2026-09-11 | -0.560 | 30% | -0.002 | 2906 |
| ANFCI | 2026-09-11 | -0.582 | 25% | -0.006 | 2906 |
| STLFSI4 | 2026-09-11 | -0.848 | 5% | -0.058 | 1707 |
| CISS (BCE) | 2026-09-15 | 0.034 | 31% | 0.012 | 7236 |

**Mensuales** (cambio vs. ~30 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 30d | N obs |
|---|---|---|---|---|---|
| EPU (Baker-Bloom-Davis) | 2026-08-01 | 175.350 | 90% | -10.341 | 500 |
| GPR (Caldara-Iacoviello) | 2026-08-01 | 117.919 | 77% | -49.613 | 500 |
| JLN 1M (Jurado-Ludvigson-Ng) | 2026-06-01 | 0.675 | 75% | 0.014 | 792 |
| JLN 3M | 2026-06-01 | 0.824 | 77% | 0.017 | 792 |
| JLN 12M | 2026-06-01 | 0.931 | 74% | 0.009 | 792 |

---

## Lectura conjunta

Sin concordancia. Nada que evaluar por esta via.

- **MS-VAR**: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.
- **MS-VAR (largo)**: EM (Hamilton-Kim), identificado: dispersion entre arranques 0.00%, |Sigma| ratio 389.2x, duracion 6.3d. Vota con histeresis de 2d sobre p>0.5 -ver 'REGLA DE HISTERESIS' en models/msvar.py (validado: RCM=17.39, alineacion 6/6 episodios de estres historicos, sensibilidad de A 0.033<0.05). p_suavizada de hoy=0.0430, confirmado_estres_hoy=no (2d consecutivos).
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.17 vs mediana 1.31, persistencia phi=0.87
- **cDCC**: pctl_corr=0.701 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.3, -0.21, -0.51], persistencia_dcc=0.990 — persistencia_dcc=0.990 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.787; proxies: SPYB<-SPY, SMHB<-SMH

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
