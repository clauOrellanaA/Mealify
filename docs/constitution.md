# Constitución de Mealify

1. **Simplicidad:** usar React, TypeScript, Vite y Tailwind CSS, con backend Node.js/Express y persistencia en un archivo JSON; excluir bases de datos externas y dependencias sin necesidad documentada.
2. **Correspondencia:** cada comportamiento implementado debe responder a un RF y sus criterios; actualizar la especificación antes de cambiar el alcance.
3. **Separación:** ejecutar reglas, validaciones y cálculos en el dominio del backend, sin dependencias de React, HTTP ni del almacenamiento concreto.
4. **Pruebas:** en la implementación futura, verificar reglas con pruebas unitarias y flujos con pruebas de integración trazables a los CA; esta entrega solo documenta su planificación.
5. **Integridad:** conservar cambios tras recargar y reiniciar el backend; validar el JSON, serializar operaciones y confirmar solo escrituras completas, sin sobrescribir datos dañados.
6. **Idioma:** escribir documentación, interfaz y mensajes en español; usar inglés para identificadores técnicos del código.
