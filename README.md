# Orquestación multi-modelo para Claude Code

Configuración para que Claude Code reparta el trabajo entre tres modelos de forma automática:

| Quién | Modelo | Qué hace |
|---|---|---|
| Sesión principal | Opus | Razona, decide, planifica, hace ediciones puntuales y es la única que usa conectores MCP |
| `Explore` | Haiku | Busca y lee código o texto (solo lectura) y devuelve un reporte compacto `ruta:línea` |
| `implementer` | Sonnet | Implementa un plan aprobado y lo valida; también extrae datos de archivos pesados (PDF, Excel, IFC, CSV, logs, `git diff`) con scripts y devuelve solo la síntesis |

Así Opus no gasta su contexto leyendo archivos, procesando documentos grandes ni escribiendo código repetitivo.

## Contenido

- `agents/explore.md`: agente explorador (Haiku, solo lectura).
- `agents/implementer.md`: agente implementador (Sonnet).
- `CLAUDE-bloque.md`: reglas obligatorias que le dicen a la sesión principal cuándo delegar.

## Reglas principales

- **Compuerta en `CLAUDE.md`:** a la sesión principal le está PROHIBIDO leer 3+ archivos, leer un archivo de más de 300 líneas, hacer búsquedas abiertas o escribir en 3+ archivos. Eso va SIEMPRE a `Explore` (lectura) o a `implementer` (escritura y extracción pesada).
- **Extracción pesada en Sonnet:** PDF, Excel, IFC, CSV, modelos de cálculo, logs extensos y `git diff` grandes se procesan con scripts dentro de `implementer`; a Opus solo llega la tabla o el resumen.
- **Tope de 3 reintentos:** si un test, linter o compilación falla, `implementer` intenta corregirlo como máximo 3 veces; luego se detiene y reporta el error exacto.
- **Autonomía táctica:** `implementer` busca por su cuenta los datos menores que están en el repositorio (rutas de import, firmas, comando de test) y solo se detiene ante contradicciones o decisiones de diseño.
- **Reporte adaptable:** `Explore` omite las secciones "Interdependencias" y "Vacíos" cuando la consulta es simple.
- **Opus fijo:** `settings.json` lleva `"model": "opus"`, así toda sesión nueva arranca con Opus como Arquitecto.

## Instalación (unos 2 minutos)

1. **Copia los agentes** a tu carpeta global de agentes:
   - Windows: `C:\Users\<tu-usuario>\.claude\agents\`
   - macOS / Linux: `~/.claude/agents/`

   Crea la carpeta `agents` si no existe.

2. **Añade el bloque de instrucciones.** Pega el contenido de `CLAUDE-bloque.md` al final de tu `CLAUDE.md` global (`~/.claude/CLAUDE.md`). Si ya tenías una versión anterior de la sección `# Orquestación multi-modelo`, reemplázala. Si el archivo no existe, créalo con ese contenido.

3. **Fija Opus por defecto.** En `~/.claude/settings.json`, añade (o cambia) en la raíz la clave `"model": "opus"`, sin tocar el resto de claves. Si el archivo no existe, créalo con:

   ```json
   {
     "model": "opus"
   }
   ```

4. **Abre una sesión nueva** de Claude Code.

5. **Comprueba que se cargaron.** Escribe `/agents` y deberían aparecer `Explore` e `implementer`.

> Nota: `Explore` reemplaza al agente explorador integrado de Claude Code con esta versión, limitada a Haiku y a solo lectura.

## Cómo funciona el disparo automático y qué tener en cuenta

1. **Por qué es casi 100 % automático.** Se combinan tres capas: `settings.json` arranca cada sesión con Opus; el bloque de `CLAUDE.md` le da reglas de tipo PROHIBIDO/SIEMPRE que lo obligan a delegar sin que lo pidas; y la `description` de cada agente empieza con "DEBE USARSE PROACTIVAMENTE (MUST BE USED)", que es lo que Claude lee al elegir agente. Aun así, la decisión final la toma el modelo en cada tarea, así que no es una garantía absoluta. Para forzarla, escríbelo: *"usa Explore para…"* o *"usa implementer para…"*.
2. **Dónde aplica.** En cualquier carpeta de este PC, desde Claude Desktop (pestaña Code), la terminal (CLI) y las extensiones de Cursor/VS Code, porque todas leen `~/.claude/`. No aplica a las sesiones en la nube de `claude.ai` (web), que no ven tu carpeta local.
3. **Precedencia.** Si un proyecto tiene su propia carpeta `.claude/agents/` con un agente del mismo nombre (`Explore` o `implementer`), dentro de ese proyecto manda el del proyecto sobre el global.
4. **Elección de modelo.** Si cambias el modelo a mano en una sesión (con `/model` o el selector de la app), esa sesión usa el que elegiste; `"model": "opus"` solo fija el valor por defecto de las sesiones nuevas. Los sub-agentes usan siempre el modelo de su línea `model:`.

## Prueba rápida

En una sesión nueva, pide:

> Revisa que funcionen los agentes: crea un proyecto de prueba de 3 archivos en una carpeta temporal, usa Explore para encontrar dónde se define y usa una función, e implementer para añadir otra función y validarla ejecutando el código.

Resultado esperado: Explore responde con referencias `ruta:línea` en su formato de reporte, e implementer modifica los archivos y muestra la salida de la validación.

## Requisitos

- Claude Code con acceso a los modelos Opus, Sonnet y Haiku. Si tu plan no incluye alguno, cambia la línea `model:` del agente (valores válidos: `haiku`, `sonnet`, `opus` o `inherit`) o la clave `"model"` de `settings.json`.
- Los sub-agentes solo tienen las herramientas de su línea `tools:` y no acceden a conectores MCP; por eso toda llamada MCP queda en la sesión principal.

## Personalización

- La línea `tools:` del encabezado de cada agente limita qué herramientas puede usar.
- La línea `description:` decide cuándo Claude elige ese agente automáticamente.
- El texto debajo del encabezado son sus reglas. Edítalo con libertad.

## Licencia

MIT. Ver [LICENSE](LICENSE).
