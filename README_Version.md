# 🤖 Copilot Dev Agents

> Agentes y prompts para GitHub Copilot que automatizan el ciclo completo de desarrollo: desde la inicialización de una tarea hasta la revisión de código, seguridad, rendimiento, accesibilidad y experiencia de usuario.

---

## 📦 Contenido

| Archivo | Tipo | Propósito |
|---|---|---|
| `code-review.agent.md` | Agente | Revisión de código: seguridad (OWASP), SOLID, rendimiento, tests y requisitos |
| `code-review-ux.agent.md` | Agente | Revisión UX/UI/Accesibilidad: WCAG 2.1 AA, NTC 5854 y heurísticas de Nielsen |
| `start-task.prompt.md` | Prompt | Inicializar tarea: detecta proyectos, determina rama base, crea rama, ejecuta tests y abre el MR |
| `ticket-review.prompt.md` | Prompt | Revisión completa de un ticket: resumen del MR + code review + UX review |

---

## ✨ Características

### `code-review.agent.md`
- ✅ Verificación de requisitos contra criterios de aceptación de Jira
- 🔐 Análisis de seguridad basado en **OWASP Top 10** (inyección, XSS, exposición de datos, control de acceso, etc.)
- ⚡ Detección de problemas de rendimiento (N+1, memory leaks, caché)
- 🏗️ Evaluación de principios **SOLID** y buenas prácticas (DRY, KISS, YAGNI)
- 🛡️ Robustez: manejo de excepciones, validación de entradas, fallos silenciosos
- 🧪 Cobertura de tests: unitarios, integración y casos de borde
- 📋 Reporte estructurado con severidad: 🔴 Crítico / 🟡 Importante / 🔵 Menor

### `code-review-ux.agent.md`
- ♿ Conformidad con **WCAG 2.1 Nivel AA** y la norma colombiana **NTC 5854**
- 🖥️ Evaluación de las **10 Heurísticas de Usabilidad de Nielsen**
- 🌍 Principios de **diseño equitativo e inclusivo** (11 criterios)
- 🎨 Análisis de consistencia visual con el sistema de diseño
- 📱 Estados UX: carga, vacío, error, éxito, hover, focus, disabled
- 🏃 Preferencias de movimiento (`prefers-reduced-motion`) y esquema de color (`prefers-color-scheme`)
- 📋 Reporte estructurado con nivel de conformidad estimado AA / NTC 5854

### `start-task.prompt.md`
- 🗂️ **Detección dinámica de proyectos**: lista las carpetas raíz del workspace actual y las presenta como opciones numeradas — no hay una lista de proyectos hardcodeada
- 👤 **Assignee configurable**: parámetro opcional al inicio; si no se especifica, el MR queda sin asignar
- 🔍 Consulta automática del ticket en **Jira** (título, descripción, épica padre) con manejo de error si el ticket no existe
- 🌿 Determina la rama base: busca la rama del ticket padre por prefijo (`bdd-100-`) o usa `master`; si hay ambigüedad, pide confirmación
- 🏷️ Genera nombre de rama **semántico en inglés, kebab-case**, máximo 50 caracteres (sin contar el prefijo del ID); permite que el usuario proponga su propio nombre con validación automática
- 💾 Genera un **bloque de estado** antes de ejecutar tests para permitir recuperación de contexto si la ejecución se interrumpe
- 🧪 Ejecuta `yarn test` antes del push; detiene el flujo si los tests fallan
- 🚀 Valida el resultado del `git push`; detiene el flujo si el comando termina con error
- 🔁 Verifica si ya existe un MR abierto para la misma rama antes de crear uno nuevo; si existe, reutiliza su URL

### `ticket-review.prompt.md`
- 🗂️ **Detección dinámica de proyectos**: igual que `start-task`, lista las carpetas del workspace sin depender de una lista estática
- 📄 Resumen completo de todos los cambios del MR
- 🔁 Invoca `code-review.agent` para el análisis técnico
- 🔁 Invoca `code-review-ux.agent` para el análisis de accesibilidad y UX
- 📊 Resumen ejecutivo con **Top 3 hallazgos críticos** de cada revisión y recomendación de merge

---

## 🛠️ Requisitos

- [VS Code](https://code.visualstudio.com/) con la extensión **GitHub Copilot** instalada y activa
- Acceso a **GitHub Copilot Chat** con soporte para agentes y prompts personalizados
- **MCP Tools** configuradas en el workspace:

| Herramienta | Uso |
|---|---|
| `GitLab-MCP` | Acceso a proyectos, ramas, MRs y diffs de GitLab |
| `jira/*` | Consulta de tickets, criterios de aceptación y estructura de épicas |

- Alias de git `new` configurado (usado en `start-task` para crear y hacer checkout de la nueva rama):
  ```bash
  git config --global alias.new 'checkout -b'
  ```

> ⚠️ Si el alias `git new` no está disponible en tu entorno, reemplaza `git new <rama>` por `git checkout -b <rama>` en `start-task.prompt.md`.

---

## 🚀 Instalación

1. **Clona este repositorio** o descarga los archivos.

2. **Copia los archivos** a tu workspace de VS Code con la siguiente estructura:

   ```
   .github/
   ├── agents/
   │   ├── code-review.agent.md
   │   └── code-review-ux.agent.md
   └── prompts/
       ├── start-task.prompt.md
       └── ticket-review.prompt.md
   ```

3. **Verifica que las MCP Tools estén configuradas** en tu `settings.json` o en el archivo de configuración MCP de tu workspace.

4. **Configura el alias `git new`** si no lo tienes (ver sección de Requisitos).

5. **Reinicia VS Code** para que Copilot detecte los nuevos agentes y prompts.

---

## 💡 Uso

### Inicializar una nueva tarea

Abre el chat de Copilot y escribe:

```
/start-task
```

El agente te pedirá, en una sola interacción:

1. El **ID del ticket de Jira** (ej. `BDD-1234`)
2. El **proyecto** — detectado automáticamente desde las carpetas raíz del workspace y presentado como lista numerada
3. El **assignee del MR** *(opcional)* — usuario de GitLab (ej. `@ncardenas`); si no se indica, el MR queda sin asignar

Luego ejecutará todos los pasos automáticamente con validaciones en cada uno.

---

### Revisión completa de un ticket

```
/ticket-review
```

El agente detecta los proyectos disponibles en el workspace, te pide la rama o ID del MR y ejecuta los tres pasos en orden:

1. Resumen de cambios del MR
2. Code Review completo
3. UX Review completo

Finaliza con un resumen ejecutivo y recomendación de merge.

---

### Solo Code Review

Activa el agente directamente desde el chat con cualquiera de estas frases:

```
code review BDD-1234
revisar código https://tirantmodulos.atlassian.net/browse/BDD-1234
review MR 42
```

**Frases de activación reconocidas:**
`code review` · `revisar código` · `review MR` · `review merge request` · `revisar tarea jira` · `revisar PR` · `analizar cambios`

---

### Solo UX Review

```
ux review BDD-1234
revisar accesibilidad https://tirantmodulos.atlassian.net/browse/BDD-1234
revisar ntc5854 BDD-1234
```

**Frases de activación reconocidas:**
`ux review` · `revisar ux` · `review ui` · `revisar accesibilidad` · `revisar frontend` · `analizar experiencia de usuario` · `revisar ntc5854`

---

## 📋 Formatos de entrada aceptados

Los agentes de revisión aceptan cualquiera de estos formatos:

| Formato | Ejemplo |
|---|---|
| ID de ticket Jira | `BDD-1234` |
| URL completa de Jira | `https://tirantmodulos.atlassian.net/browse/BDD-1234` |
| ID de MR de GitLab | `42` |

---

## 📊 Estándares aplicados

### Seguridad

| Estándar | Categorías cubiertas |
|---|---|
| **OWASP Top 10** | Inyección, Autenticación rota, Exposición de datos, Control de acceso, Mala configuración |

### Accesibilidad

| Estándar | Nivel mínimo requerido |
|---|---|
| **WCAG 2.1** | AA |
| **NTC 5854** | AA (norma colombiana; adopta WCAG 2.1 como referencia normativa) |

### Usabilidad

| Framework | Criterios evaluados |
|---|---|
| **Heurísticas de Nielsen** | 10 heurísticas |
| **Diseño equitativo** | 11 principios |

---

## 🔄 Flujo de trabajo completo

```
┌──────────────────────────────┐
│       Ticket en Jira         │
│         BDD-XXXX             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│              /start-task                 │
│                                          │
│  0. Detecta proyectos del workspace      │
│     (carpetas raíz — dinámico)           │
│  0. Assignee del MR (opcional)           │
│                                          │
│  1. Consulta Jira → manejo de errores    │
│  2. Detecta rama base                    │
│     (rama padre por prefijo o master)    │
│  3. Genera nombre de rama                │
│     (inglés, kebab-case, ≤50 chars)      │
│     → confirmación / nombre propio       │
│  4. Bloque de estado para recuperación   │
│     yarn test --watchAll=false           │
│  5. git push → valida código de salida   │
│  6. Crea MR (o reutiliza si ya existe)  │
└──────────────┬───────────────────────────┘
               │
               │  (ciclo de desarrollo)
               │
               ▼
┌─────────────────────────────────────────────────┐
│               /ticket-review                    │
│                                                 │
│  0. Detecta proyectos del workspace (dinámico)  │
│                                                 │
│  Paso 1: Resumen de cambios del MR              │
│                                                 │
│  ┌──────────────────┐   ┌─────────────────────┐ │
│  │    Paso 2        │   │      Paso 3         │ │
│  │  Code Review     │   │    UX Review        │ │
│  │                  │   │                     │ │
│  │ • OWASP Top 10   │   │ • WCAG 2.1 AA       │ │
│  │ • SOLID          │   │ • NTC 5854          │ │
│  │ • Rendimiento    │   │ • Nielsen ×10       │ │
│  │ • Tests          │   │ • Diseño equitativo │ │
│  │ • Requisitos     │   │ • Estados UX        │ │
│  └──────────────────┘   └─────────────────────┘ │
│                                                 │
│  ┌─────────────────────────────────────────┐    │
│  │          Resumen ejecutivo              │    │
│  │  Top 3 críticos code · Top 3 UX         │    │
│  │  Recomendación: merge / cambios         │    │
│  └─────────────────────────────────────────┘    │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌──────────────────────────┐
│      Merge → Deploy      │
└──────────────────────────┘
```

---

## 🌐 Idioma de los reportes

Los agentes **responden siempre en el mismo idioma en que está escrito el ticket de Jira**. Para este equipo, los reportes se generan normalmente en **español**.

---

## ⚠️ Limitaciones conocidas

| Limitación | Detalle |
|---|---|
| Dependencia de MCP | Los agentes requieren conexión activa a `GitLab-MCP` y `jira/*`. Sin ellas, el flujo se detiene con mensaje de error. |
| Alias `git new` | `start-task` usa `git new` para crear la rama. Si el alias no está configurado, el comando fallará (ver sección de Requisitos). |
| UX Review condicional | Si el proyecto no tiene componentes de interfaz de usuario, el UX Review se omite automáticamente. |
| Diffs grandes | Para MRs con más de 50 archivos, los agentes priorizan: NTC 5854 blockers → seguridad → requisitos → calidad. |
| Proyectos disponibles | Ambos prompts detectan proyectos dinámicamente desde el workspace; solo funcionan con proyectos que el agente pueda listar. |

---

## 📁 Estructura del repositorio

```
.
├── .github/
│   ├── agents/
│   │   ├── code-review.agent.md        # Agente: revisión de código
│   │   └── code-review-ux.agent.md     # Agente: revisión UX/UI/accesibilidad
│   └── prompts/
│       ├── start-task.prompt.md        # Prompt: inicializar tarea
│       └── ticket-review.prompt.md     # Prompt: revisión completa de ticket
└── README.md
```

---

## 🤝 Cómo contribuir

1. Abre un ticket en Jira describiendo la mejora o corrección propuesta.
2. Ejecuta `/start-task` con ese ticket para crear la rama y el MR automáticamente.
3. Edita el archivo correspondiente con tus cambios.
4. Al terminar, ejecuta `/ticket-review` sobre tu propio MR para validarlo antes de pedir revisión.
5. Asigna el MR al equipo de revisión.

---

## 📄 Licencia

[MIT](LICENSE) — libre para uso, modificación y distribución.