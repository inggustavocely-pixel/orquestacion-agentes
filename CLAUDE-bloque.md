# Orquestación multi-modelo (Opus + Sonnet + Haiku)
Reglas obligatorias: delega sin esperar a que el usuario lo pida. Antes de usar `Read`, `Grep`, `Glob`, `Write`, `Edit` o `Bash` en la sesión principal, evalúa el volumen:
1. **PROHIBIDO en la sesión principal:** leer 3+ archivos, leer un archivo de >300 líneas, hacer búsquedas abiertas en el repositorio o escribir cambios en 3+ archivos.
2. **SIEMPRE `Explore` (Haiku):** búsqueda de símbolos/rutas, mapeo de carpetas, lectura de 3+ archivos o de 1 archivo >300 líneas. Lanza varios en paralelo si los módulos son independientes. No releas lo que ya resumió.
3. **SIEMPRE `implementer` (Sonnet):**
   - Implementación o refactor multi-archivo o de varios pasos tras aprobar el plan, con prompt autocontenido: rutas, firmas/contratos, decisiones y cómo validar. Varios en paralelo solo si tocan archivos 100 % disjuntos.
   - Extracción pesada por script/consola: PDF, Excel, IFC, CSV, modelos de cálculo, logs extensos o `git diff` grandes vía Python/`Bash`; a Opus solo llega la tabla o síntesis limpia.
4. **Solo la sesión principal (Opus):** razonamiento arquitectónico, planes, ediciones puntuales (1–2 archivos cortos) y toda llamada a herramientas o conectores MCP (los sub-agentes tienen `tools` cerrado y no acceden a MCPs).
