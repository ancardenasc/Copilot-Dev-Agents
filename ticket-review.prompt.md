---
agent: 'agent'
description: 'Revisión completa de un ticket: cambios del MR + code review + UX review'
---

Todos los outputs, incluyendo los generados al invocar agentes externos, deben presentarse en español.

Antes de comenzar, pregunta al usuario las siguientes dos cosas a la vez:
1. En cuál de los proyectos del workspace debe trabajar.
   Antes de preguntar, lista las carpetas raíz del workspace actual de VS Code y preséntaselas como opciones numeradas.
2. ¿Cuál es la rama o el ID del MR que quieres revisar?

Si el usuario indica un proyecto que no está en el workspace, responde: "El proyecto indicado no está disponible. Por favor selecciona uno de los proyectos listados." No continúes hasta recibir una selección válida.

Si algún paso no puede completarse (por falta de acceso al MR, al agente, o a los archivos), detén la ejecución, informa al usuario del bloqueo y espera instrucciones antes de continuar.

Una vez el usuario confirme el proyecto y la rama/MR, realiza los siguientes 3 pasos en orden enfocándote únicamente en ese proyecto:

## Paso 1: Resumen de cambios del MR
Revisa el MR de la rama indicada en el proyecto seleccionado y explícame todos los cambios que se realizaron.
Incluye: archivos modificados, lógica nueva o modificada, dependencias añadidas y propósito general del cambio.
Si no se encuentra un MR abierto para la rama especificada, notifica al usuario con el mensaje: "No se encontró un MR abierto para la rama [nombre]. Verifica el nombre de la rama o el proyecto e inténtalo de nuevo." Detén la ejecución hasta recibir confirmación.

## Paso 2: Code Review
Usa el agente [code-review.agent](code-review.agent.md) y aplica sus instrucciones para hacer un code review completo del MR.
Si el agente code-review.agent no está disponible, notifica al usuario y detén la ejecución hasta recibir confirmación de cómo proceder.

## Paso 3: UX Review
Usa el agente [code-review-ux.agent](code-review-ux.agent.md) y aplica sus instrucciones para hacer un UX review completo del MR.
Si el proyecto seleccionado no tiene componentes de interfaz de usuario, indica explícitamente que el UX review no aplica y omite este paso.
Si el agente code-review-ux.agent no está disponible, notifica al usuario y detén la ejecución hasta recibir confirmación de cómo proceder.

---
Al finalizar los pasos, presenta un resumen ejecutivo estructurado con las siguientes secciones:
1. **Top 3 hallazgos críticos del code review**
2. **Top 3 hallazgos críticos del UX review** (omitir si el UX review no aplicó)
3. **Recomendación final** sobre si el MR está listo para merge o requiere cambios.

Ordena cada sección por severidad descendente.