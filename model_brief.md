# DETECTORES DE REGIMEN

**Generado (UTC, ISO 8601):** 2026-10-08T05:03:15Z
**Caducidad (UTC, ISO 8601):** 2026-10-09T17:03:15Z -- TTL=36h, FIJADO por Carlos el 2026-09-10 (ver src/run_models.py, TTL_BRIEF_HORAS, para el argumento completo y la condicion de reapertura). Pasada esta fecha, este documento NO debe usarse para decidir sin verificar antes que la publicacion de hoy funciono.

## Frescura de los datos

- **MS-VAR**: input mas viejo hace 2 dia(s) (fuente: fred).
- **MS-VAR (largo)**: input mas viejo hace 2 dia(s) (fuente: fred).
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

**Capturado (UTC):** 2026-10-08T05:00:08Z -- hace 0.1 h. **Caducidad de estos pesos (UTC):** 2026-10-09T17:00:08Z (TTL del brief, contado desde la captura).

| Posicion | Peso real | Peso objetivo | Desviacion |
|---|---|---|---|
| BTC | 15.9% | 24.0% | -8.1pp |
| SPYB | 24.0% | 20.0% | +4.0pp |
| ETH | 9.2% | 15.0% | -5.8pp |
| SMHB | 16.9% | 10.0% | +6.9pp |
| BNSOL | 6.9% | 10.0% | -3.1pp |
| BNB | 5.2% | 10.0% | -4.8pp |
| PAXG | 21.8% | 10.0% | +11.8pp |

Resto no declarado (p.ej. USDT): 0.1%.

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (1.5 meses, EN RODAJE)
- **MS-VAR (largo)**: 2026-08-22 (1.5 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (1.5 meses, EN RODAJE)
- **cDCC**: 2026-08-23 (1.5 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (1.5 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, MS-VAR (largo), BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 716 | retirado · rodaje |
| MS-VAR (largo) | estres confirmado >=2d (hoy) | 0.000 | 0.50 | no | 9176 | ok · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.025 | 0.35 | no | 712 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.950 | 0.90 | SI | 716 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.700 | 0.90 | no | 7 | ok · rodaje |

**Concordancia: 1 de 4 modelos evaluables.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y VARIOS DE ELLOS NO SON PROBABILIDADES DE REGIMEN COMPARABLES ENTRE SI -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

retirado: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.

---

## MS-VAR (largo) — regimen de comovimiento macro

MSH-VAR(1) sobre `r_NASDAQCOM`, `d_VIXCLS`, `d_BAA10Y`, PANEL_LARGO (T=9176d, muestra macro sin BTC). Estimado por EM (Hamilton-Kim), no por MLE directa -ver models/msvar.py.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -6918.7 |
| parametros | 29 |
| AIC / BIC | 13895.3 / 14101.9 |
| convergencia (EM) | si |
| iteraciones EM | 32 |

**Cadena de Markov**

| Regimen | p_ii | Duracion esperada | Prob. ergodica | |Sigma| |
|---|---|---|---|---|
| calma | 0.940 | 16.6d | 0.724 | 1.238e-04 |
| estres | 0.841 | 6.3d | 0.276 | 4.810e-02 |

**Chequeo cruzado de arranques**: dispersion de duracion **0.00% → identificado**, peor entrada de A **0.00%** (0% = las 9 entradas identicas entre los 3 arranques).


Ratio |Sigma| estres/calma: **388.4x** (minimo 3). Duracion del regimen de estres: **6.3d** (minimo 5).

**Voto de hoy (histeresis de 2d):** sin confirmar -probabilidad suavizada de hoy: 0.004. La histeresis es una capa de LECTURA sobre la probabilidad ya calculada (no cambia el filtro ni la estimacion) -motivada por que el 52.23% de las probabilidades no nitidas de este panel son tramos aislados de mediana 2 dias, no ambiguedad estructural sostenida (ver models/msvar.py).

**Limitacion conocida de A (documentada, no oculta):** la entrada NASDAQCOM←BAA10Y sale con t=1.048 en el Hessiano del optimo de EM -identificada (0% de dispersion entre arranques) pero imprecisa, consistente con eficiencia de mercado a frecuencia diaria mas que con un fallo de identificacion. Verificado que no degrada la clasificacion: perturbar A a lo largo de su direccion de menor curvatura (±1 SE) cambia la probabilidad suavizada como maximo 0.033 en toda la muestra (umbral 0.05).

**Desviaciones tipicas por regimen y correlaciones en estres**

| Serie | sd calma | sd estres | ratio |
|---|---|---|---|
| r_NASDAQCOM | 0.882 | 2.366 | 2.68 |
| d_VIXCLS | 0.830 | 2.826 | 3.40 |
| d_BAA10Y | 0.021 | 0.047 | 2.27 |

| Par | corr calma | corr estres |
|---|---|---|
| r_NASDAQCOM / d_VIXCLS | -0.687 | -0.704 |
| r_NASDAQCOM / d_BAA10Y | -0.032 | -0.214 |
| d_VIXCLS / d_BAA10Y | 0.018 | 0.197 |

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 1800 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 849.8 |
| ESS minimo de phi | 24.4 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h^2) mezcla peor porque phi y sigma_h^2 estan fuertemente correlacionados a posteriori (marginal, entre barridos) con persistencia alta.

**ESS(phi)=24.4, por debajo de 400 (Vehtari et al. 2021) — limitacion CARACTERIZADA, no abierta** (auditoria 2026-08-22 a 2026-08-30, ver docstring de models/bvarsv.py: cuatro intentos de correccion probados y revertidos, mecanismo identificado, palancas restantes fuera de alcance por costo y sin justificacion -phi no alimenta ninguna decision del sistema). **Guardarraiz de publicacion: se publica la media posterior de phi, NO su intervalo de credibilidad** (tabla de abajo).


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.157 |
| sd posterior de sigma_T | 0.309 |
| IC 90% de sigma_T | [0.74, 1.76] |
| mediana de la trayectoria | 1.313 |
| cociente sigma_T / mediana | 0.88 |
| P(sigma_T > q90) +/- MCSE | 0.025 +/- 0.004 |

**Proceso de log-volatilidad por ecuacion** (solo media posterior de phi -sin IC, ver guardarraiz arriba)

| Serie | phi (persistencia, media) | sigma_h^2 | ESS de phi |
|---|---|---|---|
| r_BTCUSDT | 0.884 | 0.083 | 24 |
| r_NASDAQ100 | 0.954 | 0.045 | 53 |
| d_DGS10 | 0.964 | 0.015 | 41 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.950 |
| rho_avg (hoy) | -0.117 |
| rho_avg (mediana historica) | -0.147 |
| persistencia_dcc (a+b) | 0.991 |
| convergio | si |

**persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.**

**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.313 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.202 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.462 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1047 | 4.35 | cola pesada | 0.150 | 2.64 |
| ETH | 1047 | 3.74 | cola pesada | 0.219 | 2.66 |
| BNSOL | 1047 | 6.88 | cola pesada | 0.160 | 2.54 |
| BNB | 1047 | 4.24 | cola pesada | 0.487 | 2.64 |
| PAXG | 1047 | 3.60 | cola pesada | 0.438 | 2.66 |
| SPYB | 8479 | 6.46 | cola pesada | 0.319 | 2.55 |
| SMHB | 6624 | 9.44 | cola pesada | 0.237 | 2.48 |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Indices de referencia (lectura directa, no votan)

Construidos por bancos centrales o academicos sobre decenas o cientos de series primarias. No entran en ningun modelo de este brief ni cuentan rodaje: se leen tal cual.

**Semanales** (cambio vs. ~7 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 7d | N obs |
|---|---|---|---|---|---|
| NFCI | 2026-09-25 | -0.548 | 32% | 0.005 | 2908 |
| ANFCI | 2026-10-02 | -0.504 | 38% | 0.020 | 2909 |
| STLFSI4 | 2026-10-02 | -0.468 | 30% | 0.341 | 1710 |
| CISS (BCE) | 2026-10-06 | 0.016 | 18% | 0.003 | 7251 |

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
- **MS-VAR (largo)**: EM (Hamilton-Kim), identificado: dispersion entre arranques 0.00%, |Sigma| ratio 388.4x, duracion 6.3d. Vota con histeresis de 2d sobre p>0.5 -ver 'REGLA DE HISTERESIS' en models/msvar.py (validado: RCM=17.39, alineacion 6/6 episodios de estres historicos, sensibilidad de A 0.033<0.05). p_suavizada de hoy=0.0043, confirmado_estres_hoy=no (2d consecutivos).
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.16 vs mediana 1.31, persistencia phi=0.88
- **cDCC**: pctl_corr=0.950 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.31, -0.2, -0.46], persistencia_dcc=0.991 — persistencia_dcc=0.991 > 0.98: correlacion casi integrada (analogo del IGARCH). Puede senalar un cambio de regimen en la correlacion no modelado, o ser artefacto de muestra corta -no hay evidencia aqui de cual; no se corrige. Leer rho_hoy/pctl_corr con cautela.
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.150; proxies: SPYB<-SPY, SMHB<-SMH

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
