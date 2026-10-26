# Tareas de implementación y verificación documental

## 1. Cómo seguir el plan

Estas 28 tareas corresponden a una implementación futura: todas quedan pendientes. Seguir el orden de aparición y las dependencias explícitas. Se conservan T01–T24 y se insertan T25–T28 para el backend y la comunicación HTTP. Cada estimación es de 20 a 30 minutos; las pruebas indicadas se escribirán y ejecutarán entonces, no en esta entrega. Los RF transversales se indican como RF-1 a RF-8.

## 2. Lista de tareas

- [ ] **T01 — Configuración mínima (25 min).** Depende de: documentos SDD. RF: RF-1 a RF-8, soporte.
  **Hecho cuando:** la base `client/` con React/TypeScript/Vite/Tailwind funciona y Vitest, Testing Library y jsdom quedan configurados, sin dependencias de servicios externos.
- [ ] **T25 — Base del backend (25 min).** Depende de: T01. RF: RF-1 a RF-8, soporte.
  **Hecho cuando:** `server/` contiene la base Node.js/Express/TypeScript, ejecución local en un solo proceso, Vitest en entorno Node y rutas `/api`; Vite usa un proxy y el servidor puede servir la compilación del cliente sin exponer datos.
- [ ] **T02 — Tipos y validación estructural (30 min).** Depende de: T25. RF: RF-1 a RF-8.
  **Hecho cuando:** el dominio del backend acepta la estructura P3 y rechaza campos ausentes, tipos incorrectos, identificadores duplicados, versiones desconocidas y referencias rotas, sin depender de Express ni archivos.
- [ ] **T03 — Normalización y cantidades exactas (25 min).** Depende de: T02. RF: RF-1, RF-5, RF-7, RF-8.
  **Hecho cuando:** el validador de estado incorpora RN-1/RN-2 y pruebas unitarias cubren espacios/mayúsculas, duplicados, unidades distintas, tres decimales, cero, negativos, exceso de decimales y límite de cantidad.
- [ ] **T04 — Dominio de recetas (30 min).** Depende de: T03. RF: RF-1, RF-2.
  **Hecho cuando:** las funciones del backend para crear, editar, listar, consultar y eliminar cumplen CA-1 a CA-5 en pruebas unitarias, incluyendo receta vacía, ingredientes repetidos y bloqueo por uso en otra semana.
- [ ] **T05 — Dominio del planificador (30 min).** Depende de: T04. RF: RF-3, RF-4.
  **Hecho cuando:** pruebas verifican semana por lunes, 21 espacios, reemplazo, quitar un espacio vacío, rechazo de receta/fecha/tipo inválidos y cruce de año, según CA-6 a CA-10.
- [ ] **T06 — Dominio de existencias (25 min).** Depende de: T03. RF: RF-5, RF-6.
  **Hecho cuando:** pruebas verifican registro, actualización por reemplazo, listado, borrado, cero y rechazo de duplicados o identificadores inexistentes, según CA-11 a CA-14.
- [ ] **T07 — Cálculo de necesidades y faltantes (30 min).** Depende de: T05, T06. RF: RF-7, RF-8.
  **Hecho cuando:** pruebas unitarias cubren CA-15 a CA-19, recetas repetidas y editadas, unidades separadas, precisión decimal, ausencia de mutaciones y fallo sin resultados parciales ante datos inválidos o desbordamiento.
- [ ] **T08 — Repositorio JSON y carga validada (30 min).** Depende de: T02, T03. RF: RF-1 a RF-8.
  **Hecho cuando:** `MealifyRepository` y `JsonFileRepository` leen el archivo UTF-8, validan P3 y distinguen primera inicialización, corrupción, desaparición posterior y error de permisos sin sobrescribir datos.
- [ ] **T09 — Escritura segura del archivo (30 min).** Depende de: T08. RF: RF-1, RF-3, RF-4, RF-5, RF-6.
  **Hecho cuando:** el candidato validado se escribe en temporal, se sincroniza, se cierra y reemplaza el principal antes de confirmar; una escritura o reemplazo fallido conserva el principal anterior.
- [ ] **T10 — Servicios y cola del backend (30 min).** Depende de: T04 a T09. RF: RF-1 a RF-8.
  **Hecho cuando:** cada consulta o mutación se encola con su ciclo completo de lectura/validación/guardado, se generan identificadores en el servidor y un fallo no bloquea la cola ni pierde cambios independientes.
- [ ] **T26 — API del recetario y errores comunes (30 min).** Depende de: T10. RF: RF-1, RF-2.
  **Hecho cuando:** las rutas de recetas de P4 devuelven cuerpos y estados acordados; pruebas HTTP comprueban validación directa, duplicados, no encontrado, borrado bloqueado y fallos de almacenamiento sin confirmar éxito.
- [ ] **T27 — API del planificador, cocina y compras (30 min).** Depende de: T26. RF: RF-3 a RF-8.
  **Hecho cuando:** las rutas restantes cumplen P4 y pruebas HTTP comprueban asignaciones, reemplazos, existencias y compras, incluyendo errores, semana vacía y el ejemplo CA-17.
- [ ] **T28 — Cliente HTTP y estado de solicitudes (30 min).** Depende de: T27. RF: RF-1 a RF-8.
  **Hecho cuando:** un cliente `fetch` común procesa datos, errores y respuestas 204; distingue guardado rechazado, resultado desconocido y refresco fallido después de éxito, conserva formularios y no guarda datos en localStorage ni reintenta mutaciones automáticamente.
- [ ] **T11 — Navegación y elementos compartidos (25 min).** Depende de: T28. RF: RF-1 a RF-8, soporte de interfaz.
  **Hecho cuando:** las cuatro vistas consultan la API, muestran carga/errores sin falsos vacíos y comparten etiquetas y foco visible sin replicar reglas del backend.
- [ ] **T12 — Listado y detalle de recetas (25 min).** Depende de: T11. RF: RF-1, RF-2.
  **Hecho cuando:** se consultan todos los campos, se muestran ausencia/no encontrado y se elimina o explica el bloqueo por uso conforme a CA-3 a CA-5.
- [ ] **T13 — Formulario de recetas (30 min).** Depende de: T12. RF: RF-1.
  **Hecho cuando:** se crean y editan recetas e ingredientes, se muestran errores por campo y se conservan los datos introducidos ante un fallo de guardado, según CA-1/CA-2 y RNF-2.
- [ ] **T14 — Semana y calendario (25 min).** Depende de: T11. RF: RF-3, RF-4.
  **Hecho cuando:** elegir una fecha muestra su lunes y siete días con 21 espacios, y cambiar de semana conserva las asignaciones anteriores, según CA-6/CA-8.
- [ ] **T15 — Acciones del calendario (25 min).** Depende de: T13, T14. RF: RF-3, RF-4.
  **Hecho cuando:** se asigna, reemplaza y quita una receta; no se asigna sin recetas y la edición de una receta se refleja en sus espacios, según CA-6 a CA-10.
- [ ] **T16 — Vista Mi cocina (30 min).** Depende de: T11. RF: RF-5, RF-6.
  **Hecho cuando:** listado y formulario permiten registrar, editar y eliminar, muestran cero, estado vacío y errores de CA-11 a CA-14.
- [ ] **T17 — Vista de compras (25 min).** Depende de: T15, T16. RF: RF-7, RF-8.
  **Hecho cuando:** comparte semana con el planificador, solicita al backend los faltantes y distingue `noMeals`, `covered`, `missing`, resultado invalidado y error, sin calcular ni alterar existencias en el cliente.
- [ ] **T18 — Integración de recetas y cocina (30 min).** Depende de: T13, T16. RF: RF-1, RF-2, RF-5, RF-6.
  **Hecho cuando:** pruebas de interfaz con HTTP simulado cubren CA-1 a CA-5 y CA-11 a CA-14, incluyendo errores, conexión perdida sin repetición automática y conservación de formularios.
- [ ] **T19 — Integración del planificador (25 min).** Depende de: T15, T18. RF: RF-1, RF-3, RF-4.
  **Hecho cuando:** pruebas cubren CA-6 a CA-10 y el bloqueo de borrado en uso, incluyendo edición de receta y cambio de semana sin pérdida de asignaciones.
- [ ] **T20 — Integración de compras (25 min).** Depende de: T17, T19. RF: RF-7, RF-8.
  **Hecho cuando:** el flujo HTTP de dos comidas y existencias obtiene CA-17, cubre CA-15 a CA-19 y exige regenerar tras editar recetas, existencias o plan; la interfaz distingue fallo de guardado de fallo del refresco posterior.
- [ ] **T21 — Integración del almacenamiento (30 min).** Depende de: T09, T10, T27. RF: RF-1 a RF-8.
  **Hecho cuando:** pruebas con archivos temporales de P7 comprueban guardar/cargar/reiniciar, corrupción, permisos, fallo de escritura/reemplazo y temporal sobrante; solicitudes concurrentes conservan cambios y no dejan JSON parcial.
- [ ] **T22 — Revisión responsive y teclado (25 min).** Depende de: T17. RF: RF-1 a RF-8, soporte de interfaz.
  **Hecho cuando:** las cuatro vistas permiten sus acciones a 360 y 1280 px sin desbordamiento de página y con teclado, etiquetas y foco visible según RNF-3.
- [ ] **T23 — Recarga y recorrido completo (25 min).** Depende de: T20, T21, T22. RF: RF-1 a RF-8.
  **Hecho cuando:** se guardan receta, existencias y dos semanas, se recarga, se reinicia el backend y otro navegador recupera los mismos datos; se comprueban desconexión/recuperación, compras y borrado tras quitar asignaciones.
- [ ] **T24 — Verificación final de implementación (25 min).** Depende de: T18 a T23. RF: RF-1 a RF-8.
  **Hecho cuando:** tipos, compilación y pruebas de cliente/servidor pasan, CA-1 a CA-19 y RNF-1 a RNF-5 tienen evidencia, y se revisa esta matriz sin introducir alcance adicional.

## 3. Matriz de trazabilidad del SDD

Las tareas T01–T03, T08–T11, T21–T25 y T28 aportan soporte transversal según sus RF declarados; la tabla destaca dominio, API e interfaz por requisito. P1 a P8 son secciones de [plan.md](plan.md).

| RF y comportamiento | Criterios de aceptación | Secciones del plan | Tareas específicas |
| --- | --- | --- | --- |
| RF-1: gestionar recetas | CA-1, CA-2, CA-3 | P3, P4, P5, P6, P7 | T04, T26, T12, T13, T18, T19 |
| RF-2: listar y consultar detalle | CA-4, CA-5 | P4, P5, P6, P7 | T04, T26, T12, T18 |
| RF-3: asignar recetas | CA-6, CA-7 | P3, P4, P5, P6, P7 | T05, T27, T14, T15, T19 |
| RF-4: consultar y modificar plan | CA-8, CA-9, CA-10 | P3, P4, P5, P6, P7 | T05, T27, T14, T15, T19 |
| RF-5: registrar y actualizar existencias | CA-11, CA-12 | P3, P4, P5, P6, P7 | T06, T27, T16, T18 |
| RF-6: consultar y eliminar existencias | CA-13, CA-14 | P4, P5, P6, P7 | T06, T27, T16, T18 |
| RF-7: consolidar necesidades semanales | CA-15, CA-16 | P3, P4, P5, P6, P7 | T07, T27, T17, T20 |
| RF-8: descontar existencias y mostrar faltantes | CA-17, CA-18, CA-19 | P3, P4, P5, P6, P7 | T07, T27, T17, T20 |

## 4. Verificación final documental — etapa 6

| Revisión | Resultado |
| --- | --- |
| Constitución | Seis principios verificables, dentro de 15 líneas; decisiones y tareas compatibles. |
| Cobertura | Cuatro módulos, ocho RF, 19 CA únicos y 28 tareas; cada RF tiene criterios, dominio, contrato HTTP e interfaz previstos. |
| Flujos e interfaz | Las cuatro vistas permiten registrar, consultar, modificar y eliminar lo previsto, planificar y generar compras. |
| Cálculo | Misma identidad por nombre/unidad; una preparación por asignación; consolidación previa al descuento único; faltantes positivos. |
| Integridad y cambios | Recetas vigentes, borrado bloqueado en todas las semanas, compras invalidadas y errores explícitos; sin referencias huérfanas aceptadas. |
| Persistencia | Backend único con archivo JSON validado, cola y reemplazo completo antes de confirmar; primera inicialización distinta de corrupción; conservación tras reiniciar; compras derivadas. |
| Comunicación | API propia y cliente HTTP sin persistencia en navegador; conexión perdida y resultado desconocido explícitos, sin reintentos automáticos. |
| Separación y alcance | Especificación de comportamiento; plan de implementación; cuatro documentos sin código, configuraciones, imágenes ni módulos adicionales. |

**Veredicto:** documentación consistente y suficiente para comenzar la implementación del MVP siguiendo las tareas. Se completaron constitución, especificación, revisión QA, plan, descomposición y verificación documental; no se implementó la aplicación ni se ejecutaron pruebas.
