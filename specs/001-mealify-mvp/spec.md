# Especificación del MVP de Mealify

## 1. Contexto, objetivo y usuarios

Mealify permite a una persona organizar sus comidas semanales, conservar recetas y conocer qué ingredientes le faltan. El MVP reduce la suma manual de ingredientes y evita comprar cantidades que ya tiene en casa. Se dirige a estudiantes y personas que planifican su alimentación doméstica, sin cuentas ni colaboración.

## 2. Historias de usuario

- **HU-1:** como usuario, quiero guardar y consultar mis recetas para reutilizar su preparación.
- **HU-2:** como usuario, quiero organizar desayuno, almuerzo y cena de lunes a domingo para saber qué cocinar.
- **HU-3:** como usuario, quiero mantener mis existencias para conocer lo disponible en casa.
- **HU-4:** como usuario, quiero obtener los faltantes de una semana para preparar mis compras.

## 3. Requisitos y criterios de aceptación EARS

Los criterios se interpretan junto con las reglas RN-1 a RN-7 y los requisitos no funcionales.

### Recetario

**RF-1. Gestionar recetas:** crear, editar y eliminar recetas con nombre, ingredientes, cantidades, unidades, porciones e instrucciones.

- **CA-1:** Cuando el usuario guarde una receta válida, el sistema deberá conservar todos sus campos; cuando edite esa receta, deberá actualizarla conservando su identidad.
- **CA-2:** Si faltan nombre, instrucciones o ingredientes, las porciones son inválidas, una cantidad o unidad incumple RN-2, o se repite un ingrediente según RN-1, el sistema deberá rechazar el guardado, señalar el campo y conservar la versión anterior.
- **CA-3:** Cuando el usuario elimine una receta sin asignaciones, el sistema deberá retirarla; si está programada en cualquier semana, deberá impedir la eliminación e indicar que primero debe quitar sus asignaciones.

**RF-2. Consultar recetas:** listar las recetas guardadas y consultar su detalle completo.

- **CA-4:** Cuando el usuario abra el recetario o seleccione una receta existente, el sistema deberá mostrar, respectivamente, los nombres guardados o todos los campos de la receta seleccionada.
- **CA-5:** Si no existen recetas, el sistema deberá mostrar un estado vacío con acceso a su registro; si una receta solicitada no existe, deberá informar que no está disponible.

### Planificador semanal

**RF-3. Asignar recetas:** asignar una receta existente a una fecha y tipo de comida de la semana seleccionada.

- **CA-6:** Cuando el usuario seleccione una fecha, el sistema deberá mostrar la semana de lunes a domingo que la contiene; cuando asigne una receta existente a un espacio válido, deberá conservar esa asignación.
- **CA-7:** Si el espacio ya está ocupado, el sistema deberá reemplazar su receta al guardar; si la receta no existe, la fecha queda fuera de la semana o el tipo de comida no es válido, deberá rechazar la asignación sin alterar el plan.

**RF-4. Consultar y modificar el plan:** consultar las 21 comidas de la semana, reemplazar recetas y quitar asignaciones.

- **CA-8:** Cuando el usuario consulte una semana, el sistema deberá mostrar sus siete días con desayuno, almuerzo y cena, incluyendo los espacios vacíos; cuando cambie de semana, deberá mostrar únicamente las asignaciones de esa semana.
- **CA-9:** Cuando el usuario quite una asignación, el sistema deberá dejar libre ese espacio; si ya estaba vacío, deberá mantenerlo vacío. Cuando guarde un reemplazo válido, deberá conservar una única receta en ese espacio.
- **CA-10:** Cuando se guarden cambios de una receta programada, el sistema deberá reflejar sus datos vigentes en todas sus asignaciones y en el siguiente cálculo de compras.

### Ingredientes disponibles: Mi cocina

**RF-5. Registrar y actualizar existencias:** guardar un ingrediente con su nombre, unidad y cantidad disponible; actualizar esos campos.

- **CA-11:** Cuando el usuario registre o edite una existencia válida, el sistema deberá conservar sus datos y tratar la cantidad editada como el nuevo total disponible, sin sumarla al anterior.
- **CA-12:** Si el nombre está vacío, la cantidad o unidad es inválida o ya existe otra entrada con la misma identidad, el sistema deberá rechazar el cambio, explicar el motivo y conservar las existencias anteriores.

**RF-6. Consultar y eliminar existencias:** listar los ingredientes disponibles y eliminar entradas.

- **CA-13:** Cuando el usuario abra Mi cocina, el sistema deberá mostrar nombre, cantidad y unidad de cada entrada, incluyendo cantidades cero; si no existen entradas, deberá mostrar un estado vacío con acceso al registro.
- **CA-14:** Cuando el usuario elimine una entrada existente, el sistema deberá retirarla sin modificar recetas ni planes; si la entrada ya no existe, deberá informar el error sin modificar las demás.

### Lista de compras

**RF-7. Calcular necesidades:** sumar los ingredientes de todas las recetas asignadas a la semana seleccionada.

- **CA-15:** Cuando el usuario genere la lista semanal, el sistema deberá contar una preparación completa por asignación y sumar las cantidades de los ingredientes con igual identidad, aunque procedan de recetas diferentes.
- **CA-16:** Si hay asignaciones en otras semanas, el sistema deberá excluirlas; si la semana seleccionada no tiene asignaciones, deberá mostrar una lista vacía y el mensaje «No hay comidas programadas»; si los datos son inválidos o no permiten un cálculo exacto, deberá informar un error sin mostrar una lista parcial.

**RF-8. Obtener faltantes:** descontar las existencias de las necesidades consolidadas y mostrar únicamente cantidades pendientes de compra.

- **CA-17:** Cuando las necesidades semanales sean 400 g de pollo y 200 g de arroz y haya 150 g de pollo y 100 g de arroz disponibles, el sistema deberá mostrar 250 g de pollo y 100 g de arroz.
- **CA-18:** Si una existencia cubre o supera una necesidad, el sistema deberá omitir ese ingrediente; si falta una existencia, deberá considerar disponible cero; si las unidades difieren, deberá mantener cantidades separadas sin convertirlas ni descontarlas entre sí.
- **CA-19:** Cuando se genere la lista, el sistema deberá mostrar nombre, unidad y cantidad positiva de cada faltante sin cambiar las existencias; si hay comidas programadas y todas sus necesidades están cubiertas, deberá mostrar «No necesitas comprar ingredientes».

## 4. Reglas de negocio

- **RN-1. Identidad:** el nombre de ingrediente se normaliza quitando espacios de los extremos, reduciendo espacios interiores consecutivos a uno y pasando a minúsculas. Su identidad es ese nombre más la unidad. Se conservan tildes y plurales: «papa» y «papas» son distintos. Se rechazan identidades repetidas dentro de una receta o entre existencias; entre recetas se permiten y se consolidan al calcular.
- **RN-2. Valores:** nombres e instrucciones deben contener texto tras quitar espacios extremos. Las porciones son un entero positivo. Las cantidades admiten hasta tres decimales, deben ser finitas y no superar 1 000 000; en recetas deben ser mayores que cero y en existencias deben ser mayores o iguales a cero. Unidades permitidas: g, kg, ml, l y unidad. Solo se suman o descuentan identidades iguales, incluso cuando dos unidades podrían convertirse físicamente.
- **RN-3. Preparaciones:** las cantidades de una receta corresponden a todas sus porciones. Cada asignación utiliza esas cantidades completas; no se eligen porciones por comida ni se escalan ingredientes. Editar las porciones no ajusta cantidades automáticamente.
- **RN-4. Semanas:** cada semana comienza el lunes y termina el domingo, según fechas del calendario local; admite 21 espacios, una receta por fecha y tipo de comida. Inicialmente se selecciona la semana actual. Se conservan las semanas previamente guardadas.
- **RN-5. Referencias:** el plan utiliza siempre la versión vigente de cada receta. Se impide eliminar recetas utilizadas en cualquier semana para evitar comidas sin receta; quitar o reemplazar asignaciones permite liberarlas.
- **RN-6. Cálculo:** por cada identidad, faltante = máximo(0, suma de necesidades de la semana − existencia actual). Las existencias se descuentan una sola vez del total semanal. Generar compras no consume existencias, no reserva ingredientes para otras semanas y no guarda una lista histórica. El resultado se ordena por nombre normalizado y luego unidad, en orden lexicográfico.
- **RN-7. Actualidad:** editar recetas, asignaciones o existencias invalida el resultado mostrado; se debe volver a generar la lista antes de presentar cantidades como actuales.

## 5. Requisitos no funcionales

- **RNF-1. Persistencia:** después de un guardado exitoso, recargar la página o reiniciar la aplicación debe conservar recetas, existencias y asignaciones. Abrir la misma instalación desde otro navegador debe permitir consultar los mismos datos, sin depender del almacenamiento del navegador.
- **RNF-2. Integridad:** los datos recuperados deben cumplir todas las reglas y referencias; ante datos inválidos o fallos de lectura se debe informar el problema e impedir cambios, sin reemplazarlos por datos vacíos. Ante fallo de guardado se conserva el estado anterior y no se anuncia éxito. Las operaciones recibidas casi simultáneamente deben ejecutarse sobre el último estado confirmado, sin perder cambios por sobrescritura.
- **RNF-3. Uso:** las cuatro vistas, mensajes y validaciones estarán en español; a anchos de 360 y 1280 píxeles deben permitir los mismos flujos sin desbordamiento horizontal de la página. Formularios y acciones tendrán etiquetas, uso por teclado y foco visible.
- **RNF-4. Autonomía y exactitud:** las operaciones no necesitarán servicios externos; entradas iguales producirán resultados iguales y los cálculos respetarán tres decimales, sin faltantes negativos ni residuos de redondeo visibles.
- **RNF-5. Disponibilidad y confirmación:** si no se puede comunicar con la aplicación, se deberá informar el error y conservar los formularios, sin mostrar datos vacíos como si fueran reales. Si se pierde la respuesta de un guardado, se deberá indicar que su resultado es desconocido y solicitar consultar de nuevo los datos antes de repetirlo; no se anunciará éxito ni se repetirá automáticamente.

## 6. Casos límite y errores

| Situación | Respuesta observable |
| --- | --- |
| Receta sin ingredientes, duplicados o valores inválidos | Rechazar el guardado y señalar la causa; CA-2. |
| No hay recetas o existencias | Mostrar el estado vacío; sin recetas no se puede asignar; sin existencias se compra toda la necesidad. |
| Existencias insuficientes, exactas o sobrantes | Mostrar la diferencia positiva, o excluir el ingrediente cuando está cubierto. |
| Mismo nombre con g y kg | Mantener dos identidades; no hay equivalencias automáticas. |
| Cambio de semana en un límite de mes o año | Mantener siete fechas consecutivas desde su lunes, sin mezclar semanas. |
| Receta editada o eliminada tras programarla | Usar su versión vigente; impedir eliminar mientras tenga asignaciones. |
| Guardado fallido, datos dañados o referencia inexistente recuperada | Informar el error y conservar datos; no presentar un guardado exitoso ni una compra parcial. |
| Interrupción durante un guardado o pérdida de su respuesta | Conservar una versión completa de los datos; si no llega confirmación, indicar resultado desconocido y volver a consultar antes de repetir. |
| Varias operaciones pendientes | Procesarlas sobre los datos ya confirmados; no perder cambios independientes. |

## 7. Fuera de alcance

Autenticación, múltiples usuarios, integraciones externas, inteligencia artificial, calorías, macronutrientes, recomendaciones nutricionales, precios, presupuestos, pagos, notificaciones y reconocimiento de imágenes. También quedan fuera conversiones de unidades, ajuste de porciones por comida, consumo automático de existencias, historial de listas de compras y edición sin conexión con la aplicación. Cada instalación maneja un solo conjunto de datos domésticos.

## 8. Criterios de finalización

La futura implementación estará completa cuando RF-1 a RF-8 y CA-1 a CA-19 se verifiquen, se cumplan RNF-1 a RNF-5, las cuatro vistas permitan todos los flujos y no queden referencias inválidas ni diferencias en el ejemplo de compras. La conservación tras reiniciar y los errores de comunicación también deben verificarse. Esta entrega termina con la documentación consistente y trazable; no implica implementación ni pruebas ejecutadas.

## 9. Revisión QA y decisiones resueltas

| Hallazgo en el planteamiento | Decisión mínima y justificación |
| --- | --- |
| Identidad y repetidos sin definición precisa | RN-1: normalizar nombre y usar unidad; rechazar duplicados internos evita cantidades accidentales, sumar entre recetas permite reutilización. |
| Unidades «compatibles» y decimales ambiguos | RN-2: catálogo cerrado, hasta tres decimales y ninguna conversión, incluso g/kg; evita resultados no verificables. |
| Porciones y repetición de recetas sin regla de escalado | RN-3: una preparación completa por asignación; CA-15 permite verificar el total sin reglas adicionales. |
| Borrado o edición de recetas utilizadas | RN-5: bloquear borrado en todas las semanas y usar la versión vigente; evita referencias rotas y copias históricas. |
| Posible doble descuento y cantidades negativas | RN-6: consolidar primero, restar una vez y limitar a cero; CA-17 a CA-19 cubren faltantes y sobrantes. |
| Estados vacíos y lista desactualizada | CA-5, CA-13, CA-16 y CA-19 distinguen ausencia de datos y cobertura total; RN-7 exige regenerar tras cambios. |
| Pérdida silenciosa al recuperar o guardar | RNF-1 y RNF-2 exigen persistencia, validación y fallo explícito sin sobrescritura ni éxito aparente. |
| Datos dependientes del navegador y reinicio sin definir | RNF-1 exige un conjunto de datos por instalación, conservado tras reiniciar y consultable desde otro navegador. |
| Guardados superpuestos o respuesta perdida | RNF-2 exige operaciones ordenadas; RNF-5 distingue fallo confirmado de resultado desconocido, sin repetición automática. |
| «Responsive» y semana sin límites verificables | RNF-3 fija dos anchos y uso por teclado; RN-4 fija lunes, domingo y 21 espacios, incluso entre años. |

Revisión documental resuelta: no se detectaron conflictos pendientes con los seis principios; se mantuvieron ocho RF y cuatro módulos. Las comprobaciones funcionales quedan para la implementación futura.
