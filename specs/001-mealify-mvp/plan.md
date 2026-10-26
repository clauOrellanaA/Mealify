# Plan técnico de Mealify

## P1. Stack y decisiones

Frontend con React, TypeScript, Vite y Tailwind CSS. Backend con Node.js, TypeScript y Express para una API HTTP; persistencia mediante `node:fs/promises` en un único archivo JSON. Vitest y React Testing Library con jsdom serán dependencias de desarrollo para las pruebas futuras. Se usarán `fetch` y las herramientas nativas de Node.js, sin ORM ni librerías de estado, fechas, formularios o validación.

Se elige Express para definir rutas y errores con poca infraestructura, y JSON por el volumen doméstico y valor didáctico. Se descartan base de datos externa, autenticación y servicios de terceros. También se descartan conversiones, copias de recetas por semana y listas persistidas. Los identificadores técnicos estarán en inglés; los textos visibles y documentos, en español. Las rutas siguientes son propuestas futuras: esta entrega solo modifica documentación.

En desarrollo, Vite redirigirá `/api` al backend; en producción, Express servirá la compilación del frontend y la API desde el mismo origen. Una instalación local usará un solo proceso backend, escuchando en `127.0.0.1`, y un conjunto de datos. No se prevén múltiples procesos, despliegues distribuidos ni acceso público sin autenticación. No se requiere Internet durante el uso, pero el backend debe estar disponible.

## P2. Arquitectura y organización

Flujo: vistas → cliente HTTP → rutas Express → servicios de aplicación → dominio y repositorio JSON. Las reglas y los cálculos se ejecutan exclusivamente en el backend. El dominio recibe datos y devuelve resultados sin importar React, Express ni `fs`; el repositorio solo lee, valida estructura y escribe. Los servicios aplican validación completa y coordinan la persistencia. El frontend gestiona navegación, formularios, formato de entrada y estados de solicitud mediante Context, sin recalcular compras ni repetir reglas de negocio.

| Ruta futura | Responsabilidad |
| --- | --- |
| `client/src/api/` | Cliente `fetch`, contratos de respuesta y tratamiento de errores HTTP o de conexión. |
| `client/src/ui/` | Vistas Recipes, Planner, Pantry y Shopping, navegación y formularios. |
| `server/src/domain/` | Tipos, validadores y operaciones puras de recipes, planner, pantry y shopping. |
| `server/src/application/` | Casos de uso y cola de operaciones para coordinar lectura, reglas y guardado. |
| `server/src/http/` | Rutas Express, validación de solicitudes y traducción de resultados a HTTP. |
| `server/src/data/` | Puerto `MealifyRepository` y adaptador `JsonFileRepository`. |
| `server/data/mealify.json` | Único documento persistido; fuera de los archivos estáticos públicos. |
| `client/src/tests/`, `server/src/tests/` | Pruebas de interfaz, dominio, API y archivos temporales. |

En el backend, `MealifyRepository` expone `load()` → promesa de estado validado, y `save(state)` → promesa de confirmación; ambos pueden devolver errores. Una inicialización separada crea el archivo vacío solo si no existe al arrancar. Sustituir JSON por otro repositorio no modifica el dominio ni los contratos HTTP. Una cola única del servidor serializa consultas y mutaciones: cada operación lee el último documento confirmado, y las mutaciones lo reemplazan después de validar. La cola continúa atendiendo solicitudes aunque una operación falle.

## P3. Modelo y validación

Documento UTF-8 único en `server/data/mealify.json`, con ruta absoluta resuelta al arrancar y nunca recibida del cliente. Ejemplo coherente con CA-17; la primera inicialización usa colecciones vacías:

```json
{
  "schemaVersion": 1,
  "recipes": [
    {
      "id": "recipe-1",
      "name": "Pollo con arroz",
      "servings": 2,
      "instructions": "Cocinar el arroz y el pollo; servir juntos.",
      "ingredients": [
        { "name": "pollo", "unit": "g", "quantity": 200 },
        { "name": "arroz", "unit": "g", "quantity": 100 }
      ]
    }
  ],
  "pantry": [
    { "id": "pantry-1", "name": "pollo", "unit": "g", "quantity": 150 },
    { "id": "pantry-2", "name": "arroz", "unit": "g", "quantity": 100 }
  ],
  "weeks": [
    {
      "weekStart": "2026-10-26",
      "assignments": [
        { "date": "2026-10-26", "mealType": "lunch", "recipeId": "recipe-1" },
        { "date": "2026-10-27", "mealType": "lunch", "recipeId": "recipe-1" }
      ]
    }
  ]
}
```

- `Unit`: `g | kg | ml | l | unidad`; `MealType`: `breakfast | lunch | dinner`, con etiquetas españolas. Las ausencias en `assignments` representan espacios vacíos; `weekStart` es único y lunes. No se persisten selección de vista, semana seleccionada ni lista calculada.
- Aplicar RN-1 a todos los nombres de ingredientes; clave de agrupación: par `[name, unit]`, sin concatenaciones ambiguas. Guardar nombres normalizados. Los nombres de receta e instrucciones solo se recortan; dos recetas pueden compartir nombre y tienen identidades distintas.
- Validar estructura completa, versión exacta, campos obligatorios, identificadores únicos de recetas y existencias, porciones enteras positivas, cantidades de RN-2 y duplicados. Cada asignación debe referenciar una receta existente, contener fecha real dentro de su semana y tipo válido; no se repite el par fecha/comida. Las porciones deben ser enteros representables exactamente.
- Generar identificadores mediante `randomUUID()` de `node:crypto` en los servicios del backend y pasarlos al dominio. El cliente no decide identificadores al crear. Editar conserva el existente y exige que exista; no hay actualización que cree registros implícitamente.
- Admitir coma o punto decimal en el formulario, sin separadores de miles ni notación exponencial. Convertir la entrada decimal a milésimas enteras para validar y operar; JSON guarda cantidades numéricas con hasta tres decimales. En lectura, redondear `quantity × 1000` y comprobar que al dividir entre 1000 se obtiene el número original. Esto evita rechazar decimales válidos por representación binaria.
- Sumar y restar milésimas con enteros seguros; si un acumulado deja de ser seguro, devolver `CALCULATION_LIMIT` sin resultado parcial. Formatear hasta tres decimales. Ordenar por nombre normalizado y unidad usando comparación lexicográfica por puntos de código, independiente de la configuración regional.
- Trabajar con fechas `YYYY-MM-DD`: descomponer año, mes y día y efectuar aritmética de calendario sin convertirlas desde cadenas UTC. Obtener la semana inicial desde la fecha local, y probar cruces de mes/año.

## P4. Operaciones y flujos

Las operaciones internas devuelven éxito con valor o error con código y mensaje español; la validación indica el campo cuando corresponda. Las mutaciones de dominio producen un estado candidato sin modificar el recibido. Los servicios guardan antes de confirmar; las rutas no permiten reemplazar el documento entero desde el navegador.

| Operación | Entrada → salida exitosa | Errores relevantes |
| --- | --- | --- |
| `listRecipes`, `getRecipe` | Estado, e identificador para detalle → lista o receta | `NOT_FOUND` en detalle. |
| `createRecipe`, `updateRecipe` | Estado, datos e identificador → estado candidato y receta | `VALIDATION`, `DUPLICATE_INGREDIENT`, `NOT_FOUND` al editar. |
| `deleteRecipe` | Estado, identificador → estado sin receta | `NOT_FOUND`, `RECIPE_IN_USE` si aparece en cualquier semana. |
| `getWeek` | Estado, fecha elegida → lunes y 21 espacios | `INVALID_DATE`. |
| `assignMeal` | Estado, lunes, fecha, tipo, receta → candidato con asignación creada/reemplazada | `NOT_FOUND`, `INVALID_SLOT` para semana, fecha o tipo inválidos. |
| `removeMeal` | Estado, lunes, fecha, tipo → candidato sin asignación | `INVALID_SLOT`; espacio válido ya vacío es éxito sin cambio. |
| `createPantryItem`, `updatePantryItem` | Estado, datos e identificador → candidato y existencia | `VALIDATION`, `DUPLICATE_INGREDIENT`, `NOT_FOUND` al editar. La actualización reemplaza cantidad. |
| `listPantry`, `deletePantryItem` | Estado y, al eliminar, identificador → lista o candidato sin entrada | `NOT_FOUND` al eliminar. |
| `calculateNeeds` | Estado, lunes → cantidades consolidadas de esa semana | `INVALID_DATE`, `INVALID_DATA`, `CALCULATION_LIMIT`. |
| `generateShoppingList` | Estado, lunes → faltantes ordenados y estado `noMeals`, `covered` o `missing` | Los del cálculo; nunca devuelve un resultado parcial. |

Un lunes válido sin registro semanal representa una semana vacía. Las lecturas y cálculos también rechazan un estado no validado mediante `INVALID_DATA`; no se omiten referencias rotas silenciosamente. Los errores de P5 se aplican a todas las operaciones que lean o guarden.

### Contrato HTTP de la API propia

Prefijo `/api`; cuerpos y respuestas JSON salvo respuestas `204`. Recetas reciben `name`, `servings`, `instructions`, `ingredients`; existencias reciben `name`, `unit`, `quantity`. Las actualizaciones reemplazan esos campos completos y reciben el identificador en la ruta. Los parámetros de semana deben ser un lunes `YYYY-MM-DD`.

| Método y ruta | Entrada → respuesta exitosa | RF |
| --- | --- | --- |
| `GET /recipes`, `GET /recipes/:id` | Sin cuerpo → `200` con lista o detalle. | RF-2 |
| `POST /recipes`, `PUT /recipes/:id` | Datos de receta → `201` o `200` con receta guardada. | RF-1 |
| `DELETE /recipes/:id` | Identificador → `204`, solo si no está programada. | RF-1 |
| `GET /weeks/:weekStart` | Lunes → `200` con semana y 21 espacios. | RF-4 |
| `PUT /weeks/:weekStart/meals/:date/:mealType` | `{ "recipeId": "..." }` → `200` con asignación guardada. | RF-3, RF-4 |
| `DELETE /weeks/:weekStart/meals/:date/:mealType` | Espacio → `204`, incluso si ya está vacío. | RF-4 |
| `GET /pantry` | Sin cuerpo → `200` con lista de existencias. | RF-6 |
| `POST /pantry`, `PUT /pantry/:id` | Datos de existencia → `201` o `200` con entrada guardada. | RF-5 |
| `DELETE /pantry/:id` | Identificador → `204`. | RF-6 |
| `GET /shopping-list?weekStart=YYYY-MM-DD` | Lunes → `200` con `{ "items": [], "status": "noMeals" }` o faltantes y estado `covered`/`missing`. | RF-7, RF-8 |

Respuestas con datos usan `{ "data": valor }`. Los errores usan `{ "error": { "code": "...", "message": "...", "field": "..." } }`, omitiendo `field` cuando no aplica. Códigos HTTP: `400` para JSON de solicitud mal formado, `VALIDATION`, `INVALID_DATE` e `INVALID_SLOT`; `404` para `NOT_FOUND`; `409` para `DUPLICATE_INGREDIENT` y `RECIPE_IN_USE`; `422` para `CALCULATION_LIMIT`; `503` para `INVALID_DATA`, `STORAGE_READ` y `STORAGE_WRITE` del servidor. Fallos inesperados producen `500` con mensaje genérico sin exponer rutas ni detalles internos. Todas las respuestas de la API usan `Cache-Control: no-store`.

Pseudocódigo de los flujos principales:

```text
GUARDAR CAMBIO
  encolar en el backend:
    estado ← repository.load(); validar estado y entrada
    candidato ← operación de dominio(estado, entrada)
    si hay error: responder error sin guardar
    validar candidato; esperar repository.save(candidato)
    si falla: responder error sin anunciar éxito
    si tiene éxito: responder con datos guardados o 204
  en la interfaz: invalidar compras y refrescar vistas tras confirmación

ASIGNAR O QUITAR COMIDA
  obtener lunes de la semana; validar fecha y tipo
  al asignar: exigir receta existente y reemplazar el único espacio
  al quitar: retirar ese espacio si existe
  devolver candidato para GUARDAR CAMBIO

ELIMINAR RECETA
  exigir que exista; buscar su identificador en todas las semanas
  si está asignada: devolver RECIPE_IN_USE
  devolver candidato sin receta para GUARDAR CAMBIO

CALCULAR COMPRAS
  dentro de la cola del backend: cargar y validar estado y lunes
  obtener asignaciones de esa semana
  si no hay asignaciones: devolver lista vacía y noMeals
  para cada asignación: buscar receta vigente
    para cada ingrediente: sumar sus milésimas por par (nombre, unidad)
  para cada par requerido:
    disponible ← milésimas de la existencia del mismo par, o cero
    faltante ← máximo(0, requerido - disponible)
    incluir solo faltantes positivos, convertidos a cantidad decimal
  ordenar resultado; devolver missing o covered sin modificar existencias
```

## P5. Persistencia e integridad

Al arrancar, crear el directorio de datos si falta. Solo si `mealify.json` no existe (`ENOENT`) en esa inicialización, crear el documento vacío mediante el mismo procedimiento seguro de escritura. Archivo vacío, JSON mal formado, versión desconocida o referencias rotas producen `INVALID_DATA`; permisos y fallos de acceso producen `STORAGE_READ`. Un arranque con datos inválidos o inaccesibles no habilita el servicio ni sobrescribe el archivo. Un archivo desaparecido durante el funcionamiento es error de lectura, nunca reinicialización silenciosa.

Para cada mutación, dentro de la cola única: cargar el documento, aplicar el caso de uso, validar y serializar el candidato. Escribirlo en un archivo temporal único del mismo directorio, sincronizar su contenido, cerrarlo y reemplazar el archivo principal mediante `rename`. No borrar previamente el principal. Confirmar la operación solo después del reemplazo. Si falla la escritura o el reemplazo, devolver `STORAGE_WRITE` y mantener el principal anterior; limpiar el temporal de esa operación cuando sea posible. Un temporal sobrante tras interrumpir el proceso no sustituye al archivo principal en el siguiente arranque.

La serialización cubre todo el ciclo leer/modificar/escribir, evitando actualizaciones perdidas entre solicitudes. Las consultas también pasan por la cola y calculan sobre una sola versión completa. No se admite editar el archivo manualmente mientras el proceso esté activo ni compartirlo entre procesos. El archivo no se sirve como recurso público. El formato tiene versión, sin migraciones ni recuperación automática; tampoco se implementan copias históricas.

El navegador no guarda datos de negocio en localStorage. Borrar sus datos no elimina el archivo del servidor. Después de un guardado confirmado, invalidar compras y refrescar los datos afectados. Si falla solo ese refresco, indicar «Guardado realizado; no se pudo actualizar la vista». Ante fallo HTTP confirmado, conservar formulario y estado visible anterior, marcado como desactualizado cuando corresponda. Ante pérdida de conexión durante una mutación, su resultado puede ser desconocido: no reintentar automáticamente; bloquear nuevos cambios hasta consultar los datos y permitir al usuario verificar si se aplicó. Una consulta fallida no se representa como una colección vacía.

## P6. Vistas y responsabilidades

Navegación con cuatro botones, cliente HTTP común y mensajes de carga/error; sin enrutador adicional. Deshabilitar envíos pendientes y mostrar acciones para volver a consultar ante conexión perdida. Estilos consistentes, formularios etiquetados, foco visible, mensajes asociados a campos y cantidades con unidades explícitas. Las respuestas del backend son la fuente de datos confirmados.

| Vista / elementos principales | Acciones y estados |
| --- | --- |
| **Recetas:** `RecipeList`, `RecipeForm`, `RecipeDetail` | Listar, crear, editar campos y filas de ingredientes, ver detalle y eliminar; explicar bloqueo por uso; estado vacío con «Crear receta». |
| **Planificador:** `WeekSelector`, `WeeklyPlanner`, `MealSlot` | Selector de fecha compartido con compras; siete días y tres comidas; seleccionar receta, guardar reemplazo o quitar; mostrar espacios vacíos y acceso a Recetas cuando no hay opciones. |
| **Mi cocina:** `PantryList`, `PantryForm` | Listar, registrar, editar nombre/unidad/cantidad y eliminar; mostrar ceros, errores de duplicado y estado vacío. |
| **Lista de compras:** `WeekSelector`, `ShoppingList` | Mostrar semana, botón «Generar lista» que consulta la API, faltantes con cantidades y unidades; distinguir semana vacía, todo cubierto, resultado invalidado y error. |

A 1280 px el plan muestra siete columnas; a 360 px muestra los siete días apilados con sus comidas. Formularios y navegación se adaptan al ancho; no se dibujan mockups ni se incorporan imágenes. La selección de semana es compartida entre planificador y compras y vuelve a la semana actual al recargar.

## P7. Estrategia de pruebas futuras

No se ejecutan pruebas en esta entrega. En la implementación se prevén:

- **Unitarias de dominio:** normalización y duplicados (CA-2, CA-12), límites y precisión, cantidades cero/negativas, porciones y unidades; borrado bloqueado en otra semana (CA-3), reemplazo y eliminación idempotente (CA-7, CA-9), fechas entre años (CA-6, CA-8), recetas editadas (CA-10).
- **Unitarias de cálculo:** ejemplo CA-17, receta repetida, ingredientes compartidos entre recetas, tres decimales, g/kg separados, existencias ausentes/cero/insuficientes/iguales/sobrantes, otra semana, semana vacía y estado inválido; asegurar entradas intactas y resultado determinista (CA-15 a CA-19).
- **Integración de archivos:** usar directorios temporales por prueba; comprobar primera inicialización, guardar/cargar y reiniciar, archivo vacío/corrupto, versión desconocida, referencias rotas y desaparición durante ejecución. Simular fallos de lectura, escritura y reemplazo, así como interrupción antes de reemplazar: conservar un principal completo y no promover temporales automáticamente (RNF-1/RNF-2).
- **Integración HTTP:** iniciar Express en un puerto temporal, usar `fetch` contra la API y un repositorio JSON temporal. Verificar cuerpos, estados HTTP y todos los contratos P4, rechazo de entradas directas inválidas y guardados concurrentes independientes sin pérdidas. Comprobar confirmación posterior al guardado y que no se expone el archivo de datos.
- **Integración de interfaz:** simular HTTP para CA-1 a CA-19: CRUD, planes, compras y estados vacíos. Incluir carga fallida sin falso vacío, guardado rechazado, respuesta perdida sin repetición automática y guardado confirmado con refresco fallido (RNF-5). La lógica de negocio se verifica en backend, no mediante una copia en la interfaz.
- **Verificación manual futura:** guardar datos, recargar, reiniciar el backend conservando el mismo archivo y abrir desde otro navegador; comprobar desconexión y recuperación, teclado y anchos de 360/1280 px. Completar comprobación de tipos y compilación de cliente y servidor junto con las pruebas anteriores.

## P8. Trazabilidad de decisiones

| Decisión | Requisitos que satisface |
| --- | --- |
| P2: dominio puro en backend y coordinación única; P3: validación en servidor | RF-1 a RF-8. |
| P3: referencias por identificador y recetas vigentes; P4: bloqueo de borrado | RF-1, RF-2, RF-3, RF-4, RF-7. |
| P3: normalización y milésimas; P4: suma previa al descuento único | RF-1, RF-5, RF-7, RF-8. |
| P3/P4: semana por lunes y 21 espacios únicos | RF-3, RF-4, RF-7, RF-8. |
| P4: API propia y contratos de solicitudes/respuestas | RF-1 a RF-8; RNF-5. |
| P5: repositorio JSON reemplazable, cola, carga validada y reemplazo completo antes de confirmar | RF-1 a RF-8; RNF-1/RNF-2, persistencia de RF-1, RF-3, RF-4, RF-5, RF-6. |
| P6: cuatro vistas; P7: comprobaciones por criterios | RF-1 a RF-8. |
