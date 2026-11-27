# DETECTORES DE REGIMEN

**Generado (UTC, ISO 8601):** 2026-11-27T06:04:32Z
**Caducidad (UTC, ISO 8601):** 2026-11-28T18:04:32Z -- TTL=36h, FIJADO por Carlos el 2026-09-10 (ver src/run_models.py, TTL_BRIEF_HORAS, para el argumento completo y la condicion de reapertura). Pasada esta fecha, este documento NO debe usarse para decidir sin verificar antes que la publicacion de hoy funciono.

**Código en producción:** cc0246e7d5b7600730e399e28ece93b535764e38 (SIN_CAMBIOS cc0246e7d5b7600730e399e28ece93b535764e38)

**Corrector vs producción:** la corrida del 2026-11-26 no cambió la huella medida del checkout (rama, HEAD, status, contenido modificado, .env*, data/**, .git/config, refs/heads y stash)

**Revisor vs producción:** desconocido (el último estado es de la corrida del 2026-11-25; no hay comprobación de una corrida posterior)

> **FALLO_LOCK_PRODUCCION (2026-11-26T08:43:18Z):** auditor no obtuvo el lock de produccion en 60 min y no corrio; dueno del lock: pid=24085 agente=corrector desde=2026-11-26T07:00:00Z.

> **FALLO_LOCK_PRODUCCION (2026-11-26T11:18:15Z):** revisor no obtuvo el lock de produccion en 60 min y no corrio; dueno del lock: pid=24085 agente=corrector desde=2026-11-26T07:00:00Z.

> **FALLO_LOCK_PRODUCCION (2026-11-26T16:03:58Z):** informe_modelos no obtuvo el lock de produccion en 60 min y no corrio; dueno del lock: pid=24085 agente=corrector desde=2026-11-26T07:00:00Z.

**Revisor:** desconocido (la ultima corrida es del 2026-11-25; no hay una posterior)

## Frescura de los datos

- **MS-VAR**: input mas viejo hace 3 dia(s) (fuente: fred).
- **MS-VAR (largo)**: input mas viejo hace 3 dia(s) (fuente: fred).
- **BVAR-SV**: input mas viejo hace 3 dia(s) (fuente: fred).
- **cDCC**: input mas viejo hace 3 dia(s) (fuente: fred).
- **GARCH-t**: input mas viejo hace 2 dia(s) (fuente: yahoo).
- fuente **binance** (mejor caso): 0 dia(s).
- fuente **fred** (mejor caso): 2 dia(s).
- fuente **yahoo** (mejor caso): 2 dia(s).
- fuente **indices_lectura** (mejor caso): 2 dia(s).

**IMPORTANTE**: la frescura de arriba es POR FUENTE/MODELO, no del panel entero -una fila de hoy de Binance (cotiza 24/7) no implica que las series macro de un modelo esten al dia. Ver `output/model_results.json` (`fuentes`/`modelos.*.frescura_input`) para el detalle completo por serie.

## Pesos reales de la cartera (relativos)

Fuente: API de Binance (src/collect_binance.py), NO portafolio.md. Solo porcentajes: este brief es publico y no publica cantidades, precios ni valores en USD.

**Capturado (UTC):** 2026-11-27T06:00:13Z -- hace 0.1 h. **Caducidad de estos pesos (UTC):** 2026-11-28T18:00:13Z (TTL del brief, contado desde la captura).

| Posicion | Peso real | Peso objetivo | Desviacion |
|---|---|---|---|
| BTC | 15.1% | 24.0% | -8.9pp |
| SPYB | 25.1% | 20.0% | +5.1pp |
| ETH | 8.1% | 15.0% | -6.9pp |
| SMHB | 13.4% | 10.0% | +3.4pp |
| BNSOL | 6.5% | 10.0% | -3.5pp |
| BNB | 5.0% | 10.0% | -5.0pp |
| PAXG | 26.7% | 10.0% | +16.7pp |

Resto no declarado (p.ej. USDT): 0.2%.

Reespecificacion por modelo (el rodaje de cada uno cuenta desde la suya, no de una fecha unica):
- **MS-VAR**: 2026-08-22 (3.2 meses, EN RODAJE)
- **MS-VAR (largo)**: 2026-08-22 (3.2 meses, EN RODAJE)
- **BVAR-SV**: 2026-08-22 (3.2 meses, EN RODAJE)
- **cDCC**: 2026-11-17 (0.3 meses, EN RODAJE)
- **GARCH-t**: 2026-08-23 (3.2 meses, EN RODAJE)

> **EN RODAJE**: MS-VAR, MS-VAR (largo), BVAR-SV, cDCC, GARCH-t siguen acumulando historial (umbral 6 meses desde su propia respec_fecha). Mientras cualquiera este en rodaje, **ninguna salida informa una decision** -se registran para medir la tasa de falsos positivos antes de darles voz.

## Resumen

| Modelo | Estadistico | Valor | Umbral | Vota | Obs | Estado |
|---|---|---|---|---|---|---|
| MS-VAR | frac 20d en estres | — | 0.50 | no | 747 | retirado · rodaje |
| MS-VAR (largo) | estres confirmado >=2d (hoy) | 1.000 | 0.50 | SI | 9209 | ok · rodaje |
| BVAR-SV | P(sigma_T > q90) | 0.083 | 0.35 | no | 741 | ok · rodaje |
| cDCC | pctl_corr (NO prob.) | 0.059 | 0.90 | no | 747 | ok · rodaje |
| GARCH-t | extremeza BTC (2 colas) | 0.839 | 0.90 | no | 7 | ok · rodaje |

**Concordancia: 1 de 4 modelos evaluables.** Cada estadistico tiene una nula DISTINTA (MS-VAR ~0.01, BVAR-SV 0.10 por construccion, cDCC ~0.50, GARCH-t ~0.0 bajo H0) y VARIOS DE ELLOS NO SON PROBABILIDADES DE REGIMEN COMPARABLES ENTRE SI -pctl_corr de cDCC es un rango percentil, la extremeza de GARCH-t es |2*percentil-1|-: no compares las cifras entre si.

---

## MS-VAR — regimen de comovimiento

retirado: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.

---

## MS-VAR (largo) — regimen de comovimiento macro

MSH-VAR(1) sobre `r_NASDAQCOM`, `d_VIXCLS`, `d_BAA10Y`, PANEL_LARGO (T=9209d, muestra macro sin BTC). Estimado por EM (Hamilton-Kim), no por MLE directa -ver models/msvar.py.

| Ajuste | Valor |
|---|---|
| log-verosimilitud | -6975.2 |
| parametros | 29 |
| AIC / BIC | 14008.4 / 14215.1 |
| convergencia (EM) | si |
| iteraciones EM | 32 |

**Cadena de Markov**

| Regimen | p_ii | Duracion esperada | Prob. ergodica | |Sigma| |
|---|---|---|---|---|
| calma | 0.939 | 16.5d | 0.724 | 1.243e-04 |
| estres | 0.841 | 6.3d | 0.276 | 4.839e-02 |

**Chequeo cruzado de arranques**: dispersion de duracion **0.00% → identificado**, peor entrada de A **0.00%** (0% = las 9 entradas identicas entre los 3 arranques).


Ratio |Sigma| estres/calma: **389.4x** (minimo 3). Duracion del regimen de estres: **6.3d** (minimo 5).

**Voto de hoy (histeresis de 2d):** ESTRES CONFIRMADO -probabilidad suavizada de hoy: 0.997. La histeresis es una capa de LECTURA sobre la probabilidad ya calculada (no cambia el filtro ni la estimacion) -motivada por que el 52.23% de las probabilidades no nitidas de este panel son tramos aislados de mediana 2 dias, no ambiguedad estructural sostenida (ver models/msvar.py).

**Limitacion conocida de A (documentada, no oculta):** la entrada NASDAQCOM←BAA10Y sale con t=1.048 en el Hessiano del optimo de EM -identificada (0% de dispersion entre arranques) pero imprecisa, consistente con eficiencia de mercado a frecuencia diaria mas que con un fallo de identificacion. Verificado que no degrada la clasificacion: perturbar A a lo largo de su direccion de menor curvatura (±1 SE) cambia la probabilidad suavizada como maximo 0.033 en toda la muestra (umbral 0.05).

**Desviaciones tipicas por regimen y correlaciones en estres**

| Serie | sd calma | sd estres | ratio |
|---|---|---|---|
| r_NASDAQCOM | 0.884 | 2.371 | 2.68 |
| d_VIXCLS | 0.832 | 2.844 | 3.42 |
| d_BAA10Y | 0.021 | 0.047 | 2.26 |

| Par | corr calma | corr estres |
|---|---|---|
| r_NASDAQCOM / d_VIXCLS | -0.688 | -0.707 |
| r_NASDAQCOM / d_BAA10Y | -0.033 | -0.218 |
| d_VIXCLS / d_BAA10Y | 0.019 | 0.200 |

---

## BVAR-SV — volatilidad y cola

VAR(1) + SV multivariante por Gibbs sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_DGS10`.

| Diagnostico MCMC | Valor |
|---|---|
| extracciones retenidas | 7000 |
| muestreo de la trayectoria | Kim-Shephard + FFBS (extraccion exacta) |
| ESS de sigma_T | 3030.3 |
| ESS minimo de phi | 40.7 |
| fiabilidad de p(estres) | ok |

El ESS que decide es el de sigma_T, que es la cantidad de la que sale p(estres). El bloque de parametros (mu, phi, sigma_h^2) mezcla peor porque phi y sigma_h^2 estan fuertemente correlacionados a posteriori (marginal, entre barridos) con persistencia alta.

**ESS(phi)=40.7, por debajo de 400 (Vehtari et al. 2021) — limitacion CARACTERIZADA, no abierta** (auditoria 2026-08-22 a 2026-08-30, ver docstring de models/bvarsv.py: cuatro intentos de correccion probados y revertidos, mecanismo identificado, palancas restantes fuera de alcance por costo y sin justificacion -phi no alimenta ninguna decision del sistema). **Guardarraiz de publicacion: se publica la media posterior de phi, NO su intervalo de credibilidad** (tabla de abajo).


**Volatilidad actual**

| Medida | Valor |
|---|---|
| sigma_T (media posterior) | 1.375 |
| sd posterior de sigma_T | 0.362 |
| IC 90% de sigma_T | [0.87, 2.02] |
| mediana de la trayectoria | 1.305 |
| cociente sigma_T / mediana | 1.05 |
| P(sigma_T > q90) +/- MCSE | 0.083 +/- 0.004 |

**Proceso de log-volatilidad por ecuacion** (solo media posterior de phi -sin IC, ver guardarraiz arriba)

| Serie | phi (persistencia, media) | sigma_h^2 | ESS de phi |
|---|---|---|---|
| r_BTCUSDT | 0.869 | 0.100 | 41 |
| r_NASDAQ100 | 0.962 | 0.046 | 172 |
| d_DGS10 | 0.956 | 0.016 | 90 |

En datos financieros reales phi debe salir entre 0.9 y 0.99. Cerca de cero significa que la cadena no ha convergido o que no hay agrupamiento de volatilidad.

---

## cDCC — correlacion dinamica

cDCC (Aielli 2013) sobre `r_BTCUSDT`, `r_NASDAQ100`, `d_BAMLH0A0HYM2`. `pctl_corr` es un **rango percentil, NO una probabilidad**: dice en que parte de su propia historia cae la correlacion promedio de hoy.

| Diagnostico | Valor |
|---|---|
| pctl_corr (hoy) | 0.059 |
| rho_avg (hoy) | -0.202 |
| rho_avg (mediana historica) | -0.161 |
| persistencia_dcc (a+b) | 0.508 |
| convergio | si |
**Correlacion condicional de hoy, por par**

| Par | rho_hoy |
|---|---|
| r_BTCUSDT / r_NASDAQ100 | 0.578 |
| r_BTCUSDT / d_BAMLH0A0HYM2 | -0.450 |
| r_NASDAQ100 / d_BAMLH0A0HYM2 | -0.736 |

---

## GARCH-t — cola condicional por posicion

GARCH(1,1)-t (MLE conjunta de nu) por posicion de config/portfolio.yaml. SPYB/SMHB usan el proxy del subyacente (SPY/SMH via Yahoo, src/collect_yahoo.py): sus propias series (46/24 obs) no llegan al minimo de 250. Por debajo de 250 obs, el modelo se niega a reportar nu ("historia insuficiente") en vez de dar un numero poco fiable — ver models/garch_evt.py.

| Posicion | n_obs | nu | categoria | hoy_percentil | VaR99 (sigma) |
|---|---|---|---|---|---|
| BTC | 1090 | 4.31 | cola pesada | 0.919 | 2.64 |
| ETH | 1090 | 3.75 | cola pesada | 0.938 | 2.66 |
| BNSOL | 1090 | 6.93 | cola pesada | 0.930 | 2.54 |
| BNB | 1090 | 4.09 | cola pesada | 0.903 | 2.65 |
| PAXG | 1090 | 3.68 | cola pesada | 0.965 | 2.66 |
| SPYB | 8514 | 6.43 | cola pesada | 0.431 | 2.55 |
| SMHB | 6659 | 9.33 | cola pesada | 0.422 | 2.48 |

`hoy_percentil` es donde cae el retorno de HOY en la distribucion t ajustada (0.5=mediana, cerca de 0 o 1=movimiento extremo). `nu` por encima de ~10 se reporta como categoria ("cola moderada o gaussiana"), no como numero puntual -la informacion de Fisher sobre nu decae ahi y el valor exacto deja de ser fiable, aunque el VaR/ES que se deriva de el casi no cambia en esa zona.

---

## Indices de referencia (lectura directa, no votan)

Construidos por bancos centrales o academicos sobre decenas o cientos de series primarias. No entran en ningun modelo de este brief ni cuentan rodaje: se leen tal cual.

**Semanales** (cambio vs. ~7 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 7d | N obs |
|---|---|---|---|---|---|
| NFCI | 2026-11-20 | -0.444 | 45% | 0.006 | 2916 |
| ANFCI | 2026-11-20 | -0.443 | 45% | 0.010 | 2916 |
| STLFSI4 | 2026-11-20 | -0.070 | 61% | 0.796 | 1717 |
| CISS (BCE) | 2026-11-25 | 0.038 | 33% | 0.029 | 7287 |

**Mensuales** (cambio vs. ~30 dias antes)

| Indice | Fecha | Valor | Percentil hist. | Cambio 30d | N obs |
|---|---|---|---|---|---|
| EPU (Baker-Bloom-Davis) | 2026-10-01 | 157.043 | 83% | -11.542 | 502 |
| GPR (Caldara-Iacoviello) | 2026-10-01 | 150.520 | 91% | -5.420 | 502 |
| JLN 1M (Jurado-Ludvigson-Ng) | 2026-06-01 | 0.675 | 75% | 0.014 | 792 |
| JLN 3M | 2026-06-01 | 0.824 | 77% | 0.017 | 792 |
| JLN 12M | 2026-06-01 | 0.931 | 74% | 0.009 | 792 |

---

## Lectura conjunta

Sin concordancia. Nada que evaluar por esta via.

- **MS-VAR**: negativo informativo -ver 'CIERRE DE PANEL_CORTO' en el docstring de models/msvar.py: cinco vias independientes (backfill de BTC a 3x la muestra sin cambio material, sustitucion del spread HY truncado por BAA10Y, cuarta serie de oro sin senal de refugio, benchmark independiente de persistencia dos ordenes de magnitud mas lento, prueba de falsacion del mecanismo de conflacion varianza/correlacion) apuntan a que el comovimiento no forma regimenes sostenidos a esta frecuencia -no es un problema de datos ni de metodo. El codigo de estimacion (models/msvar.py: fit/fit_em) sigue intacto, invocable a mano.
- **MS-VAR (largo)**: EM (Hamilton-Kim), identificado: dispersion entre arranques 0.00%, |Sigma| ratio 389.4x, duracion 6.3d. Vota con histeresis de 2d sobre p>0.5 -ver 'REGLA DE HISTERESIS' en models/msvar.py (validado: RCM=17.39, alineacion 6/6 episodios de estres historicos, sensibilidad de A 0.033<0.05). p_suavizada de hoy=0.9967, confirmado_estres_hoy=si (2d consecutivos).
- **BVAR-SV**: P(sigma_T > q90 de su propia trayectoria); nula=0.10. sigma_T=1.38 vs mediana 1.31, persistencia phi=0.87
- **cDCC**: pctl_corr=0.059 (rango percentil, NO probabilidad); rho_hoy(pares)=[0.58, -0.45, -0.74], persistencia_dcc=0.508
- **GARCH-t**: p_stress = extremeza de dos colas de BTC (|2*hoy_percentil-1|); hoy_percentil BTC=0.919; proxies: SPYB<-SPY, SMHB<-SMH

---
Los modelos no emiten senal de compra ni de venta. Estiman el estado latente de las variables que ya se vigilan. La decision sigue gobernada por los cinco gatillos de las instrucciones del proyecto.
