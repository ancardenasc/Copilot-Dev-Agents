---
agent: 'agent'
description: 'Inicializar una nueva tarea: checkout, nueva rama, tests y MR en GitLab'
tools: ['jira/*', 'GitLab-MCP', 'terminal']
---

Antes de comenzar pregunta al usuario los siguientes datos:
1. **ID del ticket de Jira** (formato BDD-XXXX)
2. **Nombre del proyecto** del workspace donde va a trabajar.
   Antes de preguntar, lista las carpetas raíz del workspace actual de VS Code y preséntaselas al usuario como opciones numeradas.
3. **Assignee del MR** *(opcional)*: usuario de GitLab a asignar (p. ej. `@ncardenas`). Si no se especifica, deja el MR sin asignar.

Una vez confirmados, sigue estos pasos en orden:

---

## Paso 1: Consultar Jira y determinar rama base

Usa el MCP **jira** para obtener los detalles del ticket `BDD-XXXX`:
- Busca si el ticket tiene un **ticket padre o épica**.

Si el ticket no se encuentra o la llamada devuelve un error, detente inmediatamente e informa al usuario: "No se pudo obtener el ticket [ID]: [error]. Por favor, verifica el ID del ticket y tus permisos en Jira antes de reintentar."

**¿Tiene padre?**
- ✅ **SÍ:** Usa el MCP **GitLab-MCP** para listar todas las ramas del proyecto y encontrar la que comienza con el ID del ticket padre en minúsculas (p. ej. `bdd-100-`). Si se encuentra exactamente una coincidencia, úsala como rama base. Si no se encuentra ninguna o hay más de una, lista las candidatas y pide al usuario que elija antes de continuar.
  ```bash
  git checkout <rama-padre>
  git pull
  ```
- ❌ **NO:** La rama base es `master`.
  ```bash
  git checkout master
  git pull
  ```

---

## Paso 2: Generar y crear la nueva rama

Con los datos obtenidos de Jira:
1. Extrae el **ID del ticket** → `BDD-XXXX`
2. Usa el **título del ticket** de Jira para generar un nombre:
   - En **inglés**
   - **Semántico** y relacionado con la tarea
   - Máximo **50 caracteres**
   - En **kebab-case**
3. Nombre final: `bdd-xxxx-nombre-de-la-rama`

Muestra el nombre generado al usuario y **pide confirmación** antes de continuar. Si el usuario rechaza el nombre, pídele que proporcione directamente su nombre preferido. Valida que siga kebab-case y no supere los 50 caracteres (sin contar el prefijo del ID del ticket), luego confirma una vez más antes de ejecutar el comando.

Una vez confirmado:
```bash
git new bdd-xxxx-nombre-de-la-rama
```

---

## Paso 3: Ejecutar tests

Antes de ejecutar los tests, genera el siguiente bloque de estado para poder recuperar el contexto si la ejecución se interrumpe:

```
[Estado]
- Ticket: BDD-XXXX
- Rama base: <rama-base>
- Nueva rama: <nombre-de-la-rama>
```

```bash
yarn test --watchAll=false
```

- ✅ **Tests pasan:** Continúa al Paso 4.
- ❌ **Tests fallan:** Detente y muestra los errores. NO continúes hasta que el usuario los resuelva.

---

## Paso 4: Push de la rama

```bash
git push -u origin bdd-xxxx-nombre-de-la-rama
```

Si el comando termina con un código distinto de cero, detente y muestra la salida del error. NO continúes al Paso 5. Pide al usuario que resuelva el problema (p. ej. force-push si la rama ya existe en remoto, o verificar permisos) y que confirme antes de reintentar.

---

## Paso 5: Crear MR en GitLab

Antes de crear el MR, verifica si ya existe un MR abierto con la misma rama fuente. Si existe, omite la creación e incluye su URL en el resumen final en lugar de crear uno nuevo.

Usa el MCP **GitLab-MCP** para crear el Merge Request:
- **Source branch:** la rama recién creada
- **Target branch:** la rama base del Paso 1
- **Título:** `[BDD-XXXX]` seguido del nombre de la rama en Title Case (primera letra de cada palabra en mayúscula), p. ej. `[BDD-1234] Add User Authentication Flow`
- **Assignee:** el usuario indicado en el Paso 0 (si se proporcionó); si no, omite este campo.

---

## Resumen final

Muestra un resumen con:
- 📌 Ticket: `BDD-XXXX`
- 📋 Título en Jira: `<título del ticket>`
- 🌿 Rama base: `<rama-base>`
- 🔀 Nueva rama: `<nombre-de-la-rama>`
- ✅ Tests: passed/failed
- 🚀 MR: `<url-del-mr>`