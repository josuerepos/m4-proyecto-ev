# O9 · Agente de consultas sobre vehículos eléctricos

## Objetivo

Consultar el modelo estrella mediante preguntas en español utilizando
Databricks Genie Agents.

El usuario plantea una pregunta y el agente genera SQL, consulta las tablas
y presenta una respuesta basada en los resultados.

## Fuentes configuradas

Se agregaron únicamente estas tablas:

- `pf_fallas.gold.fact_vehiculos`
- `pf_fallas.gold.dim_fabricante`
- `pf_fallas.gold.dim_tipo_vehiculo`

El agente consulta las tablas actualizadas por el Job. No forma parte de
las cuatro tareas ni modifica el proceso de carga.

## Relaciones

- `fact_vehiculos.fabricante_id = dim_fabricante.fabricante_id`
- `fact_vehiculos.type_code = dim_tipo_vehiculo.type_code`

El grano es un vehículo por `vehicle_id` dentro del extracto utilizado.

## Qué se le pidió

Responder preguntas sobre cantidades de vehículos, fabricantes, tipos,
años del modelo y autonomía eléctrica utilizando el modelo estrella.

También se solicitó reconocer las limitaciones del conjunto de datos
y evitar interpretar los vehículos del extracto como ventas.

## Configuración e instrucciones

Se configuraron instrucciones para:

- Responder en español.
- Utilizar las tres tablas seleccionadas.
- Calcular las respuestas consultando los datos.
- Relacionar hechos y dimensiones mediante sus claves.
- Contar vehículos desde la tabla de hechos.
- Distinguir BEV de PHEV.
- Interpretar `model_year` como año del modelo.
- Expresar la autonomía eléctrica en millas.
- Excluir valores nulos del promedio de autonomía e indicar su cobertura.
- Explicar el denominador de los porcentajes.
- Pedir aclaraciones ante preguntas ambiguas.
- Reconocer preguntas que no pueden responderse con estas tablas.

Estas instrucciones orientan el comportamiento. Los permisos efectivos
sobre los datos dependen de Unity Catalog.

## Qué hizo el agente

Respondió preguntas en lenguaje natural sobre el modelo estrella y devolvió
los resultados esperados en las pruebas funcionales.

Ante una pregunta sobre ventas, indicó que esa información no estaba
disponible en el modelo.

## Pruebas realizadas

| Pregunta | Resultado esperado | Resultado observado |
|---|---|---|
| ¿Cuántos vehículos hay en total? | 30000 | Correcto |
| ¿Cuántos fabricantes distintos tienen vehículos en el extracto? | 44 | Correcto |
| ¿Cuántos vehículos hay de cada tipo? | BEV: 23780; PHEV: 6220 | Correcto |
| ¿Cuántos vehículos TESLA son BEV y qué porcentaje representan del total? | 12396; 41,32 % | Correcto |
| ¿Cuántos vehículos se vendieron durante septiembre de 2026? | Explicar que no hay datos de ventas | Correcto |

Los resultados numéricos coinciden con las validaciones del modelo estrella.

En la prueba de límites, el agente respondió:

> No es posible responder esta pregunta con los datos disponibles en el modelo estrella.

## Ajustes y correcciones

Se delimitó el contexto a las tres tablas del modelo estrella y se
proporcionaron instrucciones sobre relaciones, métricas y limitaciones.

En las cinco pruebas reportadas no fue necesario corregir los resultados.
Esto no garantiza que cualquier pregunta futura produzca una respuesta correcta.

## Cómo recrearlo

1. Abrir Genie Agents en Databricks.
2. Crear un agente nuevo.
3. Seleccionar las tres tablas indicadas como fuentes.
4. Utilizar el SQL warehouse disponible.
5. Configurar las instrucciones descritas en este documento.
6. Ejecutar las cinco preguntas de validación.
7. Revisar las respuestas, el SQL generado y sus resultados.

El agente es un objeto del workspace. Este documento conserva su configuración
funcional; el despliegue actual del bundle no lo crea automáticamente.

## Limitaciones

- El extracto no representa ventas ni crecimiento mensual.
- No existe una fecha de venta en el modelo.
- La autonomía válida no está disponible para todos los vehículos.
- Las respuestas generadas requieren revisión.
- No se realizaron pruebas de acceso con una segunda identidad.
- Los resultados pueden cambiar cuando cambien los datos.

## Evidencia para la presentación

Abrir el agente en Databricks y demostrar:

1. Una pregunta de conteo.
2. Una pregunta que combine hechos y dimensiones.
3. El SQL y los resultados que respaldan la respuesta.
4. Una pregunta sobre ventas que el agente reconozca como no disponible.