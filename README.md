# Protección contra prompt injection en OpenCode

Tres archivos, dos capas:

| Archivo | Qué hace | Límite |
|---|---|---|
| `AGENTS.md` | Le explica al modelo qué es una orden y qué es un dato, y cuándo debe preguntar. | Es una capa blanda: el modelo puede ignorarlo, sobre todo ante una injection bien hecha. |
| `opencode.json` | Permisos para **OpenCode 1**: OpenCode hace cumplir lo que el modelo olvide. | Un `opencode.json` dentro de un repositorio puede sobrescribir estos permisos. |
| `v2/opencode.json` | Lo mismo para **OpenCode 2** (otro formato: `permissions` + `policies`). | Las reglas `permissions` también se pueden sobrescribir desde el proyecto; las `policies` globales no. |

## 1. Saber qué versión tienes

```bash
opencode --version
```

Si empieza por `1.`, usa `opencode.json`. Si empieza por `2.`, usa `v2/opencode.json`. Homebrew y el paquete de Arch ya instalan la 2.

## 2. Dónde copiarlos

- `AGENTS.md` → `~/.config/opencode/AGENTS.md` (global, vale para todos los proyectos). También puede ir en la raíz de un proyecto.
- El `opencode.json` de tu versión → `~/.config/opencode/opencode.json`. Si ya tienes uno, combina la sección `permission` (v1) o `permissions` y `policies` (v2) en vez de reemplazar el archivo.
- En Windows, `~` es `C:\Users\<tu usuario>`.

**OpenCode 1, versión más firme:** la configuración gestionada del sistema tiene prioridad sobre la del proyecto y el usuario no la puede cambiar. Para que un repositorio no pueda desactivar los permisos, copia el `opencode.json` ahí (requiere permisos de administrador):

- Linux: `/etc/opencode/opencode.json`
- macOS: `/Library/Application Support/opencode/opencode.json`
- Windows: `%ProgramData%\opencode\opencode.json`

## 3. Qué hace la configuración

- Todo lo que no esté listado **pregunta** (`ask`), incluidas las herramientas de servidores MCP.
- Leer, buscar y editar código normal: permitido sin preguntar.
- Leer `.env`, `.pem`, `.key`, `.npmrc`: pregunta. Claves SSH, `.aws`, `.kube`, credenciales de `gh` y de Docker, y las claves de OpenCode (`auth.json`): **bloqueado**.
- Editar configuración de OpenCode, `AGENTS.md`, CI, hooks de git, `.vscode/` y archivos de la shell: pregunta.
- Comandos de shell: preguntan todos, salvo `git status/diff/log/show`, `git branch`, `ls` y `pwd`. `sudo`, `su` y `doas` están bloqueados.
- `webfetch`, `websearch`, skills y acceso fuera de la carpeta del proyecto: preguntan.

## 4. Lo que ninguna configuración resuelve

- **La shell es la puerta grande.** Si apruebas `python script.py` o `npm install`, ese proceso puede leer `.env` y enviarlo fuera aunque la herramienta de lectura lo tenga bloqueado. Lee cada comando antes de aprobarlo.
- **"Permitir siempre" es peligroso.** En OpenCode 2 guarda la aprobación para el proyecto. Úsalo solo con comandos inofensivos.
- **No uses `--auto`** en repositorios que no conoces: aprueba todo lo que no esté en `deny`.
- **Antes de abrir OpenCode en un repo ajeno**, revisa si trae `opencode.json`, `opencode.jsonc`, `.opencode/` o `AGENTS.md`: pueden cambiar permisos o meter instrucciones.
- Si algo legítimo pregunta demasiado, añade una regla `allow` concreta para eso, no un `allow` general.

Documentación oficial: [Permisos v1](https://opencode.ai/docs/permissions/) · [Permisos v2](https://opencode.ai/v2/docs/permissions/) · [Políticas v2](https://opencode.ai/v2/docs/policies/) · [Reglas / AGENTS.md](https://opencode.ai/docs/rules/)
