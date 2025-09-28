Quiero que me busques datasets de la red eléctrica argentina. ¿Qué cargas tiene por zona? ¿Cuántas bajas tienen por zona? Intentá buscar datos de EDENOR, EDESUR y del ENRE. La idea es poder, a partir de los datos temporales, temperatura, carga y cantidad de reparaciones, establecer alguna correlatividad de datos.
ChatGPT said:
Pensado durante 3m 5s

¡Buenísimo! Armé un relevamiento de datasets públicos para trabajar “carga (demanda) ↔ temperatura ↔ cortes/reparaciones” en el AMBA (Edenor/Edesur) y a nivel país. Te dejo lo más útil, con links y el tipo de granularidad que ofrecen.

Datasets clave
1) Carga eléctrica (por región / zona)

CAMMESA – Demanda y generación del SADI (horaria, por regiones y total país). Es la base estándar para “carga por zona”: incluye Gran Buenos Aires, Buenos Aires Interior, Litoral, Centro, Cuyo, NOA, NEA, Comahue, Patagonia (las regiones pueden variar levemente según el reporte). API y descargas desde el portal oficial. 
Cammesa
+2
Cammesa
+2

CAMMESA – Informes/estadísticas + base asociada (series en paso mensual/diario/horario con las mismas variables de los informes). Útil para histórico limpio. 
Cammesa

CAMMESA – Programación diaria (tablas “Demanda MEM por Regiones”): visualización rápida de la demanda por región (sirve para chequear consistencia). 
Cammesa

Con esto respondemos “¿qué cargas tiene por zona?”: CAMMESA es la fuente; la zona = región CAMMESA (y para AMBA, el recorte es “Gran Buenos Aires”). 
Cammesa

2) Cortes / “bajas” por zona (Edenor / Edesur)

ENRE – Mapa de Cortes (AMBA): estado del servicio, usuarios sin suministro, tipos de corte BT/MT, filtrable por partido/comuna y distribuidora (Edenor / Edesur). Es la foto “en curso” y permite ver “detalles de cortes” por zona. No publican un histórico masivo descargable; si querés un histórico, suele armarse con capturas periódicas (ver “Limitaciones” más abajo). 
Argentina
+1

ENRE – Calidad del servicio (SAIDI/SAIFI por partido/comuna): resultados semestrales (p.ej. sep/2024–feb/2025) con desempeño por zona en Edenor y Edesur. Es ideal para frecuencia y duración de interrupciones a nivel partido/comuna. 
Argentina

Con esto respondemos “¿cuántas bajas tienen por zona?”: para conteos puntuales en curso, el Mapa del ENRE; para indicadores consolidados por zona, SAIFI/SAIDI del ENRE. 
Argentina
+1

3) Temperatura (series temporales)

SMN – Datos abiertos: listados de estaciones y datasets de variables de superficie (temperatura, presión, viento, humedad). Se pueden usar series del SMN y/o la API de Series de Tiempo de Datos Argentina (cuando corresponda). 
Datos Argentina
+2
Datos Argentina
+2

SMN en AWS (pronósticos WRF hi-res): útil si quisieras características meteorológicas de alta resolución; para correlación histórica conviene observaciones del SMN. 
registry.opendata.aws

4) Capas geográficas de la red / zonas (para mapear cortes y áreas de concesión)

Datos abiertos EDENOR – infraestructura: subestaciones, media tensión, empalmes, etc. (CSV/SHAPE). Sirven para enriquecer análisis espaciales y para validar zonas de influencia. 
Datos Argentina
+2
Datos Argentina
+2

ENRE – Distribución (info institucional + mapa de concesión de Edenor/Edesur): referencia para límites de concesión y partidos/comunas de cada una. 
Argentina

Cómo integrarlo (propuesta de pipeline)

Carga (CAMMESA)

Bajar demanda horaria por región. Para AMBA usar la región Gran Buenos Aires (o total país + otras regiones si querés control). 
Cammesa
+1

Cortes / “bajas” (ENRE)

Corto plazo: snapshot del Mapa de Cortes con Usuarios sin suministro por partido/comuna y por tipo (BT/MT).

Mediano plazo / histórico: usar los reportes semestrales SAIDI/SAIFI por zona como indicador consolidado de interrupciones (frecuencia y duración). 
Argentina
+1

Temperatura (SMN)

Elegir estaciones del SMN que cubran el AMBA (CABA/Conurbano). Construir diario/horario (según disponibilidad) y luego agregar a diario para empatar con demanda/indicadores. 
Datos Argentina

Normalización espacial

Definir “zona” = partido/comuna (ENRE).

Mapear región CAMMESA → zonas AMBA: la región GBA contendrá varios partidos/comunas; al correlacionar, usá una agregación consistente (p. ej. demandar GBA y cruzarla con sumas o promedios poblacionales/usuarios por partidos).

Usar capas EDENOR para enriquecer (p. ej. distancia a subestación, densidad de MT/BT por zona). 
Datos Argentina

Features & correlación

Variables:

Demanda: hora/día (GBA).

Temperatura: T2m (media/máxima/mínima), sensación térmica, grados-día (cooling/heating degree days).

Cortes:

ENRE “cortes en curso” → conteos por zona/fecha.

SAIFI/SAIDI → indicadores semestrales por zona (pueden repartirse a diario como “target suave” o usarse en modelos de panel). 
Argentina
+1

Técnicas: correlación de Pearson/Spearman, lags (p. ej. T de hoy y de días previos vs. cortes de mañana), modelos Poisson/NegBin para conteos, o panel data con efectos fijos por zona.

Limitaciones y cómo sortearlas

Histórico detallado de cortes por zona (evento a evento): el ENRE no publica (al menos de forma visible) un CSV histórico del Mapa. Para investigación, suele muestrearse periódicamente el Mapa (verificar términos de uso) para construir una serie propia; como alternativa, pedir vía Ley de Acceso a la Información el volcado de la “Base de Datos para el Seguimiento de Cortes” creada por Res. ENRE 267/2021. 
Enre
+1

Diferencias de granularidad: Demanda está por región CAMMESA, mientras que cortes del ENRE están por partido/comuna. Solución: trabajar con AMBA aglutinando partidos y compararlo contra la región GBA de CAMMESA, o construir un índice de cortes AMBA (población/usuarios-ponderado) para matchear. 
Cammesa

Temperatura: priorizar observaciones del SMN (no solo pronóstico). Si alguna serie falta, podés completar con estaciones cercanas o con reanálisis, documentando supuestos. 
Datos Argentina

Enlaces directos (referencia rápida)

CAMMESA – Portal / API web / Informes. 
Cammesa
+2
Cammesa
+2

CAMMESA – Programación diaria (Demanda por regiones). 
Cammesa

ENRE – Mapa de Cortes (AMBA). 
Enre

ENRE – Calidad del servicio (SAIDI/SAIFI por zona). 
Argentina

SMN – Datasets y estaciones; API Series de Tiempo. 
Datos Argentina
+2
Datos Argentina
+2

EDENOR – Datos abiertos de infraestructura (subestaciones/MT/AT). 
Datos Argentina
+2
Datos Argentina
+2

ENRE – Distribución (info y mapa concesión). 
Argentina

Siguiente paso (si te sirve)

Puedo dejarte un notebook base que: (a) baja demanda horaria de CAMMESA (región GBA), (b) arma temperatura diaria del SMN para CABA/Conurbano, y (c) incorpora SAIFI/SAIDI del último semestre por partido para un primer análisis de correlación (con lags). Decime y te lo preparo ya con estructura de tablas.

You said:
Armemoslo
ChatGPT said:
Pensado durante 1m 13s
Pensado durante 9s

¡Listo! Te dejé un proyecto base con notebook + datos de ejemplo para correr el flujo end-to-end y, cuando quieras, cambiar a datos reales de CAMMESA/SMN/ENRE.

📓 Notebook: pipeline_correlacion_amba.ipynb

🗂️ Proyecto completo (ZIP): amba_correlacion_pipeline.zip

Entra al notebook y corré las celdas. Viene con:

Datos de ejemplo en data/raw/ para que funcione offline.

Secciones de feature engineering, correlaciones (Pearson/Spearman), un gráfico de dispersión demanda–temperatura y un modelo Poisson simple para conteos de cortes.

Stubs para reemplazar por fetch reales (CAMMESA/SMN/ENRE) cuando quieras conectarte a las fuentes oficiales.

¿Querés que lo adapte para descargar automáticamente desde:

CAMMESA (demanda GBA),

SMN (temperaturas CABA/Conurbano),

ENRE (SAIDI/SAIFI por partido/comuna),


---
2025-09-28 Codex
- Se generaron datasets de ejemplo en data/raw/ (demanda horaria, temperatura diaria, cortes diarios, SAIDI/SAIFI) para que el notebook corra sin depender de fuentes externas.
- Proximo paso: reemplazar los samples con extractos reales de CAMMESA, SMN y ENRE usando el mismo esquema de columnas.
- Para el dashboard de mantenimiento conviene sumar dimensiones de costos (capex/opex, tipo de trabajo, cuadrilla), inventario de activos (subestacion, alimentador, tension, antiguedad), y backlog de ordenes correctivas vs preventivas.
- Complementar clima con viento, lluvia y rayos del SMN o reanalisis, y estimar criticidad economica (energia no suministrada, multas) por zona para priorizar inversions.
