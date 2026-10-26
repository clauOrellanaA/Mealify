# Mealify

Mealify es un proyecto universitario de aplicación web responsive para planificar comidas semanales, organizar recetas y generar una lista de compras descontando los ingredientes disponibles en casa.

El objetivo es construir un MVP pequeño y coherente, y demostrar cómo aplicar **Spec-Driven Development (SDD)**: definir el comportamiento esperado antes de escribir código y mantener requisitos, decisiones técnicas, tareas y verificaciones conectados.

## Funcionalidades del MVP

| Módulo | Idea principal |
| --- | --- |
| Recetas | Registrar, editar, eliminar y consultar recetas con ingredientes, cantidades, unidades, porciones e instrucciones. |
| Planificador | Asignar una receta al desayuno, almuerzo o cena de cada día, de lunes a domingo. |
| Mi cocina | Registrar y actualizar los ingredientes disponibles y sus cantidades. |
| Lista de compras | Consolidar las necesidades de una semana y mostrar únicamente las cantidades faltantes. |

Por ejemplo, programar dos veces una receta con 200 g de pollo y 100 g de arroz requiere 400 g de pollo y 200 g de arroz. Si hay 150 g de pollo y 100 g de arroz en casa, la lista mostrará **250 g de pollo y 100 g de arroz**.

Cada asignación cuenta como una preparación completa. Los ingredientes se agrupan por nombre normalizado y unidad; no se convierten unidades ni se ajustan porciones por comida. El alcance contempla un solo conjunto de datos domésticos, sin autenticación, múltiples usuarios ni integraciones externas.

## Stack previsto

| Parte | Tecnología y responsabilidad |
| --- | --- |
| Frontend | React, TypeScript, Vite y Tailwind CSS para las cuatro vistas y formularios. |
| Backend | Node.js, Express y TypeScript para la API, validaciones y reglas de negocio. |
| Persistencia | Un archivo `server/data/mealify.json`, administrado exclusivamente por el backend. |
| Verificación futura | Vitest, React Testing Library y jsdom; pruebas de dominio, API, archivos e interfaz. |

Los datos se conservarán al recargar la página y reiniciar el backend. El servidor validará el documento y ordenará las operaciones de lectura y escritura para evitar pérdidas. El navegador consultará la API y no persistirá datos de negocio en localStorage. Durante el uso no se necesitarán servicios externos, pero el backend deberá estar disponible.

## Aplicación de SDD

En este proyecto, SDD se organiza en seis etapas:

| Etapa | Aplicación en Mealify | Documento |
| --- | --- | --- |
| 1. Constitución | Fijar seis principios que guían el alcance, la arquitectura, la integridad y las pruebas. | [constitution.md](docs/constitution.md) |
| 2. Especificación | Definir qué necesita el usuario y por qué: ocho requisitos funcionales (RF), 19 criterios de aceptación (CA) en notación EARS y reglas de negocio. | [spec.md](specs/001-mealify-mvp/spec.md) |
| 3. Revisión QA | Resolver ambigüedades, errores y casos límite antes de implementar; registrar las decisiones en la sección 9. | [spec.md](specs/001-mealify-mvp/spec.md) |
| 4. Plan técnico | Explicar cómo construirlo: stack, arquitectura, modelo JSON, API, persistencia y estrategia de pruebas. | [plan.md](specs/001-mealify-mvp/plan.md) |
| 5. Tareas | Dividir el trabajo en 28 tareas de 20–30 minutos, con dependencias y condiciones de finalización. | [tasks.md](specs/001-mealify-mvp/tasks.md) |
| 6. Verificación documental | Revisar la coherencia y relacionar cada RF con sus CA, secciones del plan y tareas. | [tasks.md](specs/001-mealify-mvp/tasks.md) |

La trazabilidad sigue la cadena **RF → CA → plan → tareas → implementación y evidencia futura**. Por ejemplo, RF-8 exige obtener faltantes; CA-17 fija el resultado de pollo y arroz; el plan define el cálculo en el backend; T07, T27, T17 y T20 cubren su lógica, API, vista e integración.

Para continuar con SDD, se identifica el requisito y sus criterios, se revisa el plan y se ejecuta la tarea correspondiente. Si cambia el comportamiento o la arquitectura, primero se actualizan los documentos afectados y su trazabilidad; después se implementa y verifica el cambio.

## Estado actual

Las seis etapas documentales están completadas. La implementación y las pruebas permanecen pendientes: todavía no hay aplicación ejecutable ni comandos de instalación o arranque definidos en el repositorio.

El siguiente paso previsto es T01, siguiendo el orden de aparición y las dependencias de [tasks.md](specs/001-mealify-mvp/tasks.md). Los identificadores T25–T28 se insertaron para incorporar el backend sin renumerar las tareas anteriores.

Las instrucciones para agentes que trabajen en este repositorio están en [AGENTS.md](AGENTS.md).
