# Guía de presentación del proyecto EV

## Objetivo

Demostrar cómo se transforman archivos de vehículos eléctricos en datos
limpios y un resumen por fabricante, aplicando calidad, orquestación,
gobernanza, control de versiones y optimización.

La presentación se realiza en vivo desde GitHub y Databricks.
No se incluyen capturas de pantalla en el repositorio.

## 1. Preparación

Antes de presentar:

- Abrir el repositorio público y su README.
- Tener disponibles los notebooks con sus salidas.
- Abrir una corrida exitosa del Job `m4_ev_prod`.
- Confirmar que solo prod tiene programación activa.
- Verificar que el permiso del usuario en `seguridad.acceso_vin`
  termine con `puede_ver_vin = true`.
- Mantener el checkpoint y los archivos originales.
- No ejecutar simultáneamente los Jobs manual, dev y prod.
- No ejecutar todo `03_gold` esperando éxito: contiene una prueba
  que debe fallar intencionalmente.

Un Pull puede limpiar salidas de notebooks actualizados.
Si ocurre, ejecutar únicamente las celdas necesarias para recuperar
las evidencias.

## 2. Orden de presentación

Duración propuesta: 18 minutos, dejando margen hasta el máximo de 20.

| Tiempo | Tema | Qué mostrar |
|---|---|---|
| 0–2 min | Problema y datos | Pregunta de negocio, fuente y alcance del extracto |
| 2–3 min | Arquitectura | Diagrama del README y función de cada capa |
| 3–6 min | Bronze y Silver | Lotes, métricas, deduplicación y JOIN |
| 6–8 min | Gold y calidad | Resumen, restricciones y fila rechazada |
| 8–10 min | Orquestación | Dependencias, reintentos, corrida y horario |
| 10–12 min | Gobernanza | Grants, mapeo, máscara y tags |
| 12–14 min | Git y bundle | Pull request integrado, YAML y despliegues |
| 14–16 min | Observabilidad | Linaje, clustering y ejecución de OPTIMIZE |
| 16–18 min | Aprendizajes y defensa | Decisiones, límites y pregunta del profesor |

## 3. Evidencias que se mostrarán en vivo

### V1 · Bronze incremental

Abrir `01_bronze` y mostrar el conteo por lote:

| Lote | Filas |
|---|---:|
| lote_1 | 15.000 |
| lote_2 | 15.010 |

Explicar que la reejecución sin archivos nuevos conservó el conteo
y las fechas de ingesta durante la prueba realizada.

El conteo actual por lote muestra lo cargado. La prueba de no incremento
se explica con las ejecuciones realizadas, sin afirmar que una sola
consulta demuestra por sí misma toda la incrementalidad.

### V2 · Silver limpia

Abrir `02_silver` y mostrar:

| Capa | Filas | Vehículos únicos | Repetidos | Ciudades sin tratar |
|---|---:|---:|---:|---:|
| Bronze | 30.010 | 30.000 | 10 | 4 |
| Silver | 30.000 | 30.000 | 0 | 0 |

Explicar la llave `vehicle_id`, el uso de `ROW_NUMBER()` y el tratamiento
de ciudad y condado con `SIN DATO`.

Mostrar el JOIN con la referencia de tipos:

- BEV: 23.780 vehículos.
- PHEV: 6.220 vehículos.

### C1 · Gold y calidad

Abrir `03_gold` y mostrar:

- 65 grupos de fabricante y tipo.
- 44 fabricantes.
- 30.000 vehículos sumados.
- Fabricante obligatorio mediante `NOT NULL`.
- Cantidad positiva mediante `CHECK`.

Mostrar el error del INSERT con cantidad `-1`:
`DELTA_VIOLATE_CONSTRAINT_WITH_VALUES`.

No recrear la tabla únicamente para enseñar sus restricciones.
Consultar `SHOW CREATE TABLE` y `SHOW TBLPROPERTIES`.

### C2 · Orquestación

Abrir `m4_ev_prod` y mostrar:

- Dependencias `bronze → silver → gold`.
- Dos reintentos en Bronze, con intervalo mínimo de 60 segundos.
- Una corrida completa con las tres tareas en `Succeeded`.
- Programación activa diaria a las 08:00,
  zona `America/Costa_Rica`.

La programación de dev y del Job manual permanece pausada.

Distinguir una corrida manual exitosa de una corrida iniciada por el
horario. Solo afirmar ejecución programada cuando su origen esté
verificado en el historial.

### C3 · Gobernanza

Abrir `04_gobernanza` y mostrar:

- `SHOW GRANTS` del catálogo, esquema Gold y tabla.
- `SELECT` para `account users` sobre el resumen Gold.
- Tabla `seguridad.acceso_vin`.
- Función `seguridad.enmascarar_vin`.
- Tags de `vehicle_id` y `vin_prefix`.

Para demostrar la máscara:

1. Consultar el VIN con autorización activa.
2. Cambiar `puede_ver_vin` a false para el usuario actual.
3. Repetir la consulta: debe mostrar `RESTRINGIDO`.
4. Restaurar `puede_ver_vin` a true.
5. Confirmar que los valores originales vuelven a mostrarse.

La prueba utiliza la misma identidad cambiando su autorización.
No presentarla como una prueba desde otra cuenta.

Los tags son descriptivos; la máscara es la que controla
la visualización.

### C4 · Git y despliegue

Mostrar en GitHub:

- Código en el repositorio público.
- Ramas `dev` y `main`.
- Un pull request con estado `Merged`.
- `databricks.yml` y `resources/ev_job.yml`.

Mostrar en Databricks la evidencia disponible del despliegue y
las corridas exitosas de los Jobs dev y prod.

Explicar que los despliegues se realizan manualmente desde Databricks.
No se configuró GitHub Actions ni despliegue automático al merge.

### C5 · Observabilidad y optimización

Abrir `05_observabilidad` y mostrar el linaje:

| Origen | Destino |
|---|---|
| bronze.vehiculos_raw | silver.vehiculos |
| silver.tipos_vehiculo | silver.vehiculos |
| silver.vehiculos | gold.vehiculos_por_fabricante |

Mostrar `DESCRIBE DETAIL` de Bronze:

- `clusteringColumns = ["make"]`.
- Un archivo.
- Tamaño de 510.208 bytes después de optimizar.

Conservar y mostrar la salida de la ejecución de `OPTIMIZE`.
No es necesario repetir la operación para presentar.

Antes de optimizar había dos archivos y 528.580 bytes.
Los conteos permanecieron en 30.010 filas y 30.000 vehículos únicos.

No afirmar una mejora de velocidad: no se realizó un benchmark.

## 4. Explicación breve del proyecto

El proyecto utiliza un extracto de 30.000 vehículos eléctricos.
Los datos se dividieron en dos lotes y se agregaron diez duplicados
controlados para probar la limpieza.

Bronze conserva los registros recibidos. Silver corrige tipos,
trata valores vacíos, elimina duplicados y agrega la referencia
del tipo de vehículo. Gold resume la cantidad y participación
por fabricante y tipo.

El proceso se ejecuta mediante un Job, aplica restricciones de calidad,
protege el VIN parcial con una máscara y conserva el código en GitHub.
El linaje permite observar las dependencias y liquid clustering
se configura por fabricante en la tabla más grande.

## 5. Preguntas de defensa

### ¿Auto Loader elimina vehículos duplicados?

No. Reconoce archivos procesados mediante su estado y checkpoint.
Los vehículos repetidos entre archivos se eliminan en Silver.

### ¿Por qué se usa vehicle_id como llave?

Porque identifica los 30.000 vehículos únicos del extracto.
El VIN parcial no es único.

### ¿Por qué se usa LEFT JOIN?

Para conservar los vehículos aunque no encuentren una coincidencia
en la referencia. En esta carga, ambos tipos encontraron coincidencia.

### ¿Qué significa SIN DATO?

Indica que la ubicación estaba vacía. Conserva el registro sin inventar
una ciudad o condado.

### ¿Por qué conservar la autonomía original?

Permite mantener el dato recibido. La columna `electric_range_valid`
aplica una regla adicional para excluir valores no positivos
de futuros promedios.

### ¿Qué demuestra el INSERT con -1?

Que Databricks hace cumplir el CHECK y rechaza la fila.
La prueba se mantiene fuera del Job porque debe fallar.

### ¿El CHECK reemplaza NOT NULL?

No. El CHECK de cantidad no impide por sí mismo un valor nulo.
En este resumen, COUNT(*) genera cantidades no nulas.
La otra regla explícita es make NOT NULL.

### ¿Los tags ocultan información?

No. Documentan la clasificación. La función aplicada como máscara
decide si devuelve el valor o RESTRINGIDO.

### ¿Qué muestra el linaje?

Las tablas que aportan datos a otras: Bronze y la referencia alimentan
Silver; Silver alimenta Gold.

### ¿Por qué se eligió Bronze para clustering?

Era la tabla de mayor tamaño físico antes de optimizar.
Se eligió make por las consultas filtradas por fabricante.

### ¿clusterByAuto=false significa clustering desactivado?

No. Significa que la columna se eligió manualmente mediante
CLUSTER BY (make), en lugar de selección automática.

### ¿Dev y prod tienen datos separados?

No. Separan Jobs y despliegues, pero comparten el catálogo
y el checkpoint en esta implementación académica.

### ¿Por qué GitHub Actions está vacío?

Porque no se configuraron workflows de Actions.
El despliegue del bundle se realiza desde Databricks.

## 6. Cierre

El proyecto demuestra un pipeline incremental con datos limpios,
reglas aplicadas por Databricks, ejecución orquestada, acceso gobernado,
código versionado y evidencias de linaje y optimización.

Las limitaciones se documentan: extracto sin fecha registrada,
duplicados controlados, datos compartidos entre targets y ausencia
de mediciones comparativas de rendimiento.