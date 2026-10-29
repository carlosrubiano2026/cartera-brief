# DETECTORES DE REGIMEN

**Generado (UTC, ISO 8601):** 2026-10-29T06:02:57Z
**Caducidad (UTC, ISO 8601):** 2026-10-30T18:02:58Z -- TTL=36h, FIJADO por Carlos el 2026-09-10 (ver src/run_models.py, TTL_BRIEF_HORAS, para el argumento completo y la condicion de reapertura). Pasada esta fecha, este documento NO debe usarse para decidir sin verificar antes que la publicacion de hoy funciono.

## Frescura de los datos

- **MS-VAR**: input mas viejo hace 2 dia(s) (fuente: fred).
- **MS-VAR (largo)**: input mas viejo hace 3 dia(s) (fuente: fred).
- **BVAR-SV**: input mas viejo hace 2 dia(s) (fuente: fred).
- **cDCC**: input mas viejo hace 2 dia(s) (fuente: fred).
- **GARCH-t**: input mas viejo hace 1 dia(s) (fuente: yahoo).
- fuente **binance** (mejor caso): 0 dia(s).
- fuente **fred** (mejor caso): 1 dia(s).
- fuente **yahoo** (mejor caso): 1 dia(s).
- fuente **indices_lectura** (mejor caso): 2 dia(s).

**IMPORTANTE**: la frescura de arriba es POR FUENTE/MODELO, no del panel entero -una fila de hoy de Binance (cotiza 24/7) no implica que las series macro de un modelo esten al dia. Ver `output/model_results.json` (`fuentes`/`modelos.*.frescura_input`) para el detalle completo por serie.

## Pesos reales de la cartera (relativos)

Fuente: API de Binance (src/collect_binance.py), NO portafolio.md. Solo porcentajes: este brief es publico y no publica cantidades, precios ni valores en USD.

**Capturado (UTC):** 2026-10-29T06:00:12Z -- hace 0.0 h. **Caducidad de estos pesos (UTC):** 2026-10-30T18:00:12Z (TTL del brief, contado desde la captura).

| Posicion | Peso real | Peso objetivo | Desviacion |
|---|---|---|---|
| BTC | 16.2% | 24.0% | -7.8pp |
| SPYB | 25.1% | 20.0% | +5.1pp |
| ETH | 8.8% | 15.0% | -6.2pp |
| SMHB | 15.8% | 10.0% | +5.8pp |
| BNSOL | 6.4% | 10.0% | -3.6pp |
| BNB | 5.4% | 10.0% | -4.6pp |
| PAXG | 22.3% | 10.0% | +12.3pp |

Resto no declarado (p.ej. USDT): 0.1%.

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (2.2 meses, EN RODAJE)
- **MS-VAR (largo)**: 2026-08-22 (2.2 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (2.2 meses, EN RODAJE)
- **cDCC**: 2026-08-23 (2.2 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (2.2 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, MS-VAR (largo), BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 728 | retirado · rodaje |
| MS-VAR (largo) | estres confirmado >=2d (hoy) | 0.000 | 0.50 | no | 9189 | ok · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.003 | 0.35 | no | 723 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.659 | 0.90 | no | 728 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.472 | 0.90 | no | 7 | ok · rodaje |

**Concordancia: 0 de 4 modelos evaluables.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y VARIOS DE ELLOS NO SON PROBABILIDADES DE REGIMEN COMPARABLES ENTRE SI -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

retirado: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.

---

## MS-VAR (largo) — regimen de comovimiento macro

MSH-VAR(1) sobre `r_NASDAQCOM`, `d_VIXCLS`, `d_BAA10Y`, PANEL_LARGO (T=9189d, muestra macro sin BTC). Estimado por EM (Hamilton-Kim), no por MLE directa -ver models/msvar.py.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -6922.9 |
| parametros | 29 |
| AIC / BIC | 13903.9 / 14110.5 |
| convergencia (EM) | si |
| iteraciones EM | 34 |

**Cadena de Markov**

| Regimen | p_ii | Duracion esperada | Prob. ergodica | |Sigma| |
|---|---|---|---|---|
| calma | 0.939 | 16.5d | 0.725 | 1.241e-04 |
| estres | 0.841 | 6.3d | 0.275 | 4.819e-02 |

**Chequeo cruzado de arranques**: dispersion de duracion **0.00% → identificado**, peor entrada de A **0.00%** (0% = las 9 entradas identicas entre los 3 arranques).


Ratio |Sigma| estres/calma: **388.4x** (minimo 3). Duracion del regimen de estres: **6.3d** (minimo 5).

**Voto de hoy (histeresis de 2d):** sin confirmar -probabilidad suavizada de hoy: 0.013. La histeresis es una capa de LECTURA sobre la probabilidad ya calculada (no cambia el filtro ni la estimacion) -motivada por que el 52.23% de las probabilidades no nitidas de este panel son tramos aislados de mediana 2 dias, no ambiguedad estructural sostenida (ver models/msvar.py).

**Limitacion conocida de A (documentada, no oculta):** la entrada NASDAQCOM←BAA10Y sale con t=1.048 en el Hessiano del optimo de EM -identificada (0% de dispersion entre arranques) pero imprecisa, consistente con eficiencia de mercado a frecuencia diaria mas que con un fallo de identificacion. Verificado que no degrada la clasificacion: perturbar A a lo largo de su direccion de menor curvatura (±1 SE) cambia la probabilidad suavizada como maximo 0.033 en toda la muestra (umbral 0.05).

**Desviaciones tipicas por regimen y correlaciones en estres**

| Serie | sd calma | sd estres | ratio |
|---|---|---|---|
| r_NASDAQCOM | 0.883 | 2.367 | 2.68 |
| d_VIXCLS | 0.832 | 2.827 | 3.40 |
| d_BAA10Y | 0.021 | 0.047 | 2.27 |

| Par | corr calma | corr estres |
|---|---|---|
| r_NASDAQCOM / d_VIXCLS | -0.688 | -0.704 |
| r_NASDAQCOM / d_BAA10Y | -0.033 | -0.214 |
| d_VIXCLS / d_BAA10Y | 0.019 | 0.197 |

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 729.4 |
| ESS minimo de phi | 16.0 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h^2) mezcla peor porque phi y sigma_h^2 estan fuertemente correlacionados a posteriori (marginal, entre barridos) con persistencia alta.

**ESS(phi)=16.0, por debajo de 400 (Vehtari et al. 2021) — limitacion CARACTERIZADA, no abierta** (auditoria 2026-08-22 a 2026-08-30, ver docstring de models/bvarsv.py: cuatro intentos de correccion probados y revertidos, mecanismo identificado, palancas restantes fuera de alcance por costo y sin justificacion -phi no alimenta ninguna decision del sistema). **Guardarraiz de publicacion: se publica la media posterior de phi, NO su intervalo de credibilidad** (tabla de abajo).


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 0.988 |
| sd posterior de sigma_T | 0.262 |
| IC 90% de sigma_T | [0.61, 1.48] |
| mediana de la trayectoria | 1.293 |
| cociente sigma_T / mediana | 0.76 |
| P(sigma_T > q90) +/- MCSE | 0.003 +/- 0.001 |

**Proceso de log-volatilidad por ecuacion** (solo media posterior de phi -sin IC, ver guardarraiz arriba)

| Serie | phi (persistencia, media) | sigma_h^2 | ESS de phi |
|---|---|---|---|
| r_BTCUSDT | 0.906 | 0.066 | 34 |
| r_NASDAQ100 | 0.949 | 0.051 | 27 |
| d_DGS10 | 0.954 | 0.019 | 16 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.659 |
| rho_avg (hoy) | -0.142 |
| rho_avg (mediana historica) | -0.149 |
| persistencia_dcc (a+b) | 0.990 |
| convergio | si |

**persistencia_dcc=0.990 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.**

**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.299 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.202 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.524 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1063 | 4.33 | cola pesada | 0.264 | 2.64 |
| ETH | 1063 | 3.73 | cola pesada | 0.284 | 2.66 |
| BNSOL | 1063 | 6.84 | cola pesada | 0.264 | 2.54 |
| BNB | 1063 | 4.22 | cola pesada | 0.785 | 2.64 |
| PAXG | 1063 | 3.57 | cola pesada | 0.348 | 2.66 |
| SPYB | 8494 | 6.47 | cola pesada | 0.687 | 2.55 |
| SMHB | 6639 | 9.46 | cola pesada | 0.767 | 2.48 |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Indices de referencia (lectura directa, no votan)

Construidos por bancos centrales o academicos sobre decenas o cientos de series primarias. No entran en ningun modelo de este brief ni cuentan rodaje: se leen tal cual.

**Semanales** (cambio vs. ~7 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 7d | N obs |
|---|---|---|---|---|---|
| NFCI | 2026-10-23 | -0.487 | 41% | 0.007 | 2912 |
| ANFCI | 2026-10-23 | -0.501 | 39% | 0.008 | 2912 |
| STLFSI4 | 2026-10-23 | -0.584 | 23% | 0.050 | 1713 |
| CISS (BCE) | 2026-10-27 | 0.019 | 21% | 0.009 | 7266 |

**Mensuales** (cambio vs. ~30 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 30d | N obs |
|---|---|---|---|---|---|
| EPU (Baker-Bloom-Davis) | 2026-09-01 | 175.193 | 89% | 1.829 | 501 |
| GPR (Caldara-Iacoviello) | 2026-09-01 | 146.210 | 90% | 20.589 | 501 |
| JLN 1M (Jurado-Ludvigson-Ng) | 2026-06-01 | 0.675 | 75% | 0.014 | 792 |
| JLN 3M | 2026-06-01 | 0.824 | 77% | 0.017 | 792 |
| JLN 12M | 2026-06-01 | 0.931 | 74% | 0.009 | 792 |

---

## Lectura conjunta

Sin concordancia. Nada que evaluar por esta via.

- **MS-VAR**: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.
- **MS-VAR (largo)**: EM (Hamilton-Kim), identificado: dispersion entre arranques 0.00%, |Sigma| ratio 388.4x, duracion 6.3d. Vota con histeresis de 2d sobre p>0.5 -ver 'REGLA DE HISTERESIS' en models/msvar.py (validado: RCM=17.39, alineacion 6/6 episodios de estres historicos, sensibilidad de A 0.033<0.05). p_suavizada de hoy=0.0133, confirmado_estres_hoy=no (2d consecutivos).
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=0.99 vs mediana 1.29, persistencia phi=0.91
- **cDCC**: pctl_corr=0.659 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.3, -0.2, -0.52], persistencia_dcc=0.990 — persistencia_dcc=0.990 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.264; proxies: SPYB<-SPY, SMHB<-SMH

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
