# Instrucciones para trabajar en Mealify

## Alcance y estado

Estas instrucciones se aplican a todo el repositorio. Mealify es un MVP universitario con cuatro módulos: recetas, planificador semanal, ingredientes disponibles y lista de compras.

Actualmente el proyecto contiene documentación SDD; el código y las pruebas están pendientes. Cuando la solicitud sea documental, modifica documentación sin crear código, configuraciones ejecutables ni datos de ejemplo fuera de los documentos. Cuando el usuario solicite implementar, continúa con las tareas correspondientes sin requerir una aprobación adicional por este archivo.

## Documentos de referencia

Lee los documentos pertinentes antes de modificar el proyecto:

1. [docs/constitution.md](docs/constitution.md): seis principios innegociables del proyecto.
2. [specs/001-mealify-mvp/spec.md](specs/001-mealify-mvp/spec.md): comportamiento, RF, CA, reglas, errores y alcance.
3. [specs/001-mealify-mvp/plan.md](specs/001-mealify-mvp/plan.md): arquitectura, datos, API y decisiones de implementación.
4. [specs/001-mealify-mvp/tasks.md](specs/001-mealify-mvp/tasks.md): dependencias, condiciones de finalización y trazabilidad.

El [README.md](README.md) presenta el proyecto; los detalles de comportamiento se consultan en la especificación. Si el usuario cambia el alcance o una decisión técnica, actualiza primero los documentos afectados, conserva su coherencia y refleja el cambio en el README cuando corresponda.

## Flujo de trabajo SDD

- Identifica los RF y CA relacionados con cada cambio; utiliza las decisiones del plan y la tarea correspondiente.
- Mantén el QUÉ y el POR QUÉ en la especificación, y el CÓMO en el plan técnico.
- Sigue las tareas en su orden de aparición y respeta sus dependencias; la numeración no representa por sí sola el orden de ejecución.
- Conserva los identificadores existentes. Si añades o modificas requisitos, criterios o tareas, actualiza sus referencias y la matriz de trazabilidad.
- Marca una tarea como completada únicamente cuando se cumpla su condición «Hecho cuando» y exista evidencia de la verificación correspondiente.
- Realiza cambios acotados al objetivo solicitado y preserva el trabajo existente del usuario.

## Arquitectura acordada

- Frontend: React, TypeScript, Vite y Tailwind CSS, bajo `client/`.
- Backend: Node.js, Express y TypeScript, bajo `server/`, con API propia bajo `/api`.
- Ejecuta las reglas, validaciones y cálculos en funciones de dominio del backend, separadas de HTTP, interfaz y acceso a archivos.
- El frontend consume la API y gestiona presentación y solicitudes; evita duplicar las reglas o calcular compras en componentes React.
- Persiste los datos en `server/data/mealify.json` mediante `MealifyRepository` y `JsonFileRepository`, separados del dominio. No uses localStorage para datos de negocio.
- Valida los datos recuperados. Serializa el ciclo de lectura/modificación/escritura y reemplaza el principal mediante un temporal completo antes de confirmar el guardado, según el plan.
- Conserva los datos ante errores, distingue ausencia inicial de corrupción y trata una respuesta perdida como resultado desconocido, sin repetir automáticamente una mutación.
- Mantén un solo proceso backend y un conjunto de datos por instalación. Evita dependencias y patrones innecesarios.

## Reglas que deben preservarse

- Agrupa ingredientes por nombre normalizado y unidad; no conviertas unidades. Rechaza duplicados dentro de una receta o entre existencias y consolida los de recetas diferentes al calcular.
- Cuenta una preparación completa por asignación; cada semana tiene 21 espacios y cada espacio admite una receta.
- Descuenta las existencias una sola vez del total semanal y muestra solo faltantes positivos, sin consumir existencias al generar compras.
- Impide eliminar recetas programadas en cualquier semana; sus ediciones deben reflejarse en las asignaciones y el siguiente cálculo.
- Aplica los límites y la precisión definidos en la especificación; invalida las compras después de cambios relevantes.
- Mantén fuera de alcance autenticación, múltiples usuarios, integraciones externas y los demás elementos excluidos por la especificación.

## Idioma y verificación

Escribe documentación, interfaz y mensajes en español; usa inglés para identificadores técnicos. Evita copiar reglas extensas entre documentos: referencia su fuente para reducir inconsistencias.

Para cambios documentales, revisa enlaces relativos, identificadores, dependencias y coherencia entre documentos. No ejecutes pruebas de aplicación por una edición exclusivamente documental ni afirmes que existen resultados funcionales.

Durante una implementación autorizada, añade y ejecuta las verificaciones pertinentes del plan: unitarias de dominio, integración de API y archivos, interfaz, tipos y compilación. Usa los comandos que existan realmente en el proyecto; no inventes scripts ni resultados. Informa qué cambió, qué se verificó y qué quedó pendiente.
