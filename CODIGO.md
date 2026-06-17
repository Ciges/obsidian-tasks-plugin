# Estudio del código: Obsidian Tasks v8.2.1

Análisis del bundle `main.js` (archivo minificado, ~880 KB).

---

## Información básica

- **ID:** `obsidian-tasks-plugin`
- **Nombre:** Tasks
- **Versión:** 8.2.1
- **Requisito mínimo de Obsidian:** 1.8.7
- **Autores originales:** Clare Macrae, Ilyas Landikov (creado por Martin Schenck)
- **Repo oficial:** https://github.com/obsidian-tasks-group/obsidian-tasks

---

## Funcionalidad principal

Gestiona tareas definidas como checkboxes `- [ ]` en toda la vault. Extrae metadatos de cada tarea (fechas, prioridad, estado, dependencias) y permite filtrarlas y agruparlas mediante un lenguaje de queries propio en bloques de código.

---

## Componentes principales

| Clase (minificada) | Rol |
|---|---|
| `bf` (Plugin) | Clase principal, gestiona el ciclo de vida |
| `ws` (TaskCache) | Mantiene todas las tareas en memoria |
| `lf` (QueryRenderer) | Renderiza bloques ` ```tasks ``` ` |
| `ef` (InlineRenderer) | Renderiza tareas inline en el editor |
| `jd` (TasksEvents) | Sistema de eventos custom del plugin |
| `mf` (SettingTab) | Panel de configuración |
| `at` (StatusRegistry) | Registro de estados de tarea |

---

## Secuencia de inicialización (onload)

1. Registrar logger de consola
2. Cargar configuración (settings)
3. Inicializar almacenamiento local
4. Crear gestor de eventos (`TasksEvents`)
5. Registrar panel de settings
6. Inicializar i18n con metadata de Obsidian
7. Cargar estados de tareas personalizados
8. Crear caché de tareas con listeners de archivos
9. Crear renderizadores inline y de queries
10. Configurar tipos de propiedades Obsidian
11. Registrar extensión de editor (syntax highlighting)
12. Registrar editor suggest (autocompletado)
13. Registrar comandos

---

## Eventos que escucha

```javascript
this.app.metadataCache.on("changed", ...)  // Cambios en metadata de archivos
this.app.vault.on("rename", ...)           // Renombrado de archivos
```

---

## Comandos registrados

| ID | Nombre | Tipo |
|---|---|---|
| `edit-task` | Create or edit task | Editor |
| `toggle-done` | Toggle task done | Editor |
| `add-query-file-defaults-properties` | Add all Query File Defaults properties | Global |

Además genera comandos dinámicos por cada estado personalizado registrado.

---

## Metadatos soportados en tareas

- Fechas: vencimiento, inicio, programación, creación, finalización
- Recurrencia (via rrule.js)
- Prioridad: highest, high, medium, low, normal
- Dependencias entre tareas
- Estados personalizables (símbolo, nombre, tipo)
- Tags en la descripción
- Urgencia calculada automáticamente

---

## Lenguaje de queries

Los bloques ` ```tasks ``` ` aceptan filtros, ordenación y agrupación:

- Filtros por fecha, estado, prioridad, tags, ruta de archivo...
- `filterByFunction`, `sortByFunction`, `groupByFunction` para lógica custom en JavaScript
- Filtro global aplicable a toda la vault desde settings

---

## Estados de tarea

Cuatro tipos base: `TODO`, `IN_PROGRESS`, `DONE`, `CANCELLED`.

Cada estado tiene un símbolo de checkbox (ej: `x` para done), un nombre y un tipo. Se pueden crear estados custom con CSS.

Flujo de estados visualizable como diagrama Mermaid desde el panel de settings.

---

## API pública (v1)

Accesible desde otros plugins via `plugin.apiV1()`:

```javascript
{
    createTaskLineModal()              // Abre modal para crear tarea
    editTaskLineModal(line)            // Abre modal para editar línea
    executeToggleTaskDoneCommand(line, options)  // Toglea estado
}
```

---

## Librerías incluidas en el bundle

| Librería | Uso |
|---|---|
| rrule.js | Cálculo de fechas recurrentes |
| EventEmitter2 | Sistema de eventos |
| mustache.js | Templates para i18n |

---

## Ficheros de la instalación

| Fichero | Descripción |
|---|---|
| `main.js` | Bundle minificado con todo el código |
| `manifest.json` | Metadatos del plugin (ID, versión, autor...) |
| `styles.css` | Estilos CSS (~27 KB) |
| `data.json` | Configuración guardada por el usuario |
