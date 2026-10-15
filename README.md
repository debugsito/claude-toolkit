# claude-toolkit

Skills y subagentes para [Claude Code](https://docs.claude.com/en/docs/claude-code), empaquetados como **tres plugins independientes**. Instalas los que quieras y puedes quitar cualquiera sin tocar el resto.

| Plugin | Qué es | Qué trae |
|---|---|---|
| `skills-devops` | Skills propias: comandos `/algo` que tú invocas (o Claude cuando detecta el caso) | `/mapa-proyecto`, `/runbook-cambio-prod`, `/trello-tarjetas` |
| `skills-terceros` | Skills de otros autores (MIT) que uso a diario | `drawio-skill`, `find-skills` |
| `agentes-devops` | Subagentes a los que Claude delega tareas pesadas de lectura | `code-dev`, `debug`, `qa`, `po`, `sec-review`, `k8s-triage`, `tf-dev`, `tf-review` |

Las skills de PowerPoint, Word, Excel y PDF, y los plugins de Azure y Terraform, **no están aquí** porque son de Anthropic y no se pueden redistribuir. Se instalan en un minuto desde sus marketplaces oficiales: ver [Lo que uso y no está en este repo](#lo-que-uso-y-no-está-en-este-repo).

## Instalación

Todo se hace **dentro de Claude Code** (no en la terminal normal). Primero registras el marketplace, después instalas lo que quieras:

```
/plugin marketplace add debugsito/claude-toolkit
```

Skills propias:

```
/plugin install skills-devops@carlos-toolkit
```

Skills de terceros (draw.io y buscador de skills):

```
/plugin install skills-terceros@carlos-toolkit
```

Agentes:

```
/plugin install agentes-devops@carlos-toolkit
```

Para ver qué quedó instalado: `/plugin`. Para quitar uno:

```
/plugin uninstall agentes-devops@carlos-toolkit
```

Cuando yo suba cambios, los traes con:

```
/plugin marketplace update carlos-toolkit
```

Si prefieres instalarlos a nivel de un solo proyecto y no global, el menú de `/plugin` te deja elegir el alcance (user o project).

## Skills

Una skill es una carpeta con un `SKILL.md`: instrucciones que Claude carga cuando el caso coincide con la descripción. Se disparan solas o escribiendo `/nombre`.

### `/mapa-proyecto`

Para carpetas con muchos repos o documentos. Genera un índice (`MAPA.md`) con lo que se lee de git o del disco (rama, último commit, stack) más una descripción mantenida a mano por repo. Así Claude lee ~1.500 tokens al abrir sesión en vez de explorar 40 carpetas cada vez.

Dispara con: "haz un mapa del proyecto", "actualiza el mapa", "no encuentro dónde está cada cosa".

### `/runbook-cambio-prod`

Arma el runbook de un cambio en producción en Azure o AWS: ventana medida, precondiciones, pasos, verificación y rollback. Úsala **antes** de resize o migración de base de datos, upgrade de cluster, cambio de instancia, failover, cambio de DNS o de listener.

Dispara con: "arma el runbook", "plan de cambio en producción", "necesito el plan de rollback", "esto toca producción".

### `/trello-tarjetas`

Atiende tarjetas de Trello: ver pendientes, leer una tarjeta, resolverla en el repo correspondiente y moverla o comentarla. **Requiere** un comando local `trello` en el PATH; sin eso no sirve.

Dispara con: "mis tarjetas", "atiende la tarjeta", "pásala a Hecho", o pegando un enlace `trello.com/c/...`.

## Skills de terceros

Van en su propio plugin para que decidas si las quieres. Las dos son MIT; el `LICENSE` original va dentro de cada carpeta.

### `drawio-skill`

Crea y edita diagramas de draw.io / diagrams.net como archivos `.drawio` editables: arquitectura cloud (trae iconos de Azure, AWS y GCP), UML, ERD, secuencia, BPMN, redes, swimlanes. Puede partir de una descripción o convertir desde código, Terraform, SQL o un esquema de API. También sincroniza, revisa y exporta diagramas existentes.

Pide "un diagrama en draw.io" o "un `.drawio` de esta arquitectura". No se dispara sola: hay que invocarla por nombre. Origen: [Agents365-ai/drawio-skill](https://github.com/Agents365-ai/drawio-skill).

### `find-skills`

Busca e instala skills del ecosistema abierto de [skills.sh](https://skills.sh/). Cuando preguntas "¿hay una skill para X?" o "¿cómo hago X?", consulta el catálogo y te propone instalarla con `npx skills add`. Origen: [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).

## Lo que uso y no está en este repo

Son de Anthropic y su licencia no permite copiarlas a otro repo. Se instalan desde los marketplaces oficiales con los mismos comandos de siempre.

**Documentos de oficina** (PowerPoint, Word, Excel, PDF). Es lo que uso para armar presentaciones y reportes:

```
/plugin marketplace add anthropics/skills
```

```
/plugin install document-skills@anthropic-agent-skills
```

Si usas Claude Code logueado con tu cuenta de claude.ai, estas mismas skills ya vienen sincronizadas; revisa `/plugin` o pregúntale a Claude "qué skills tienes" antes de instalarlas dos veces.

**Azure, Terraform y documentación de Microsoft.** Plugins del marketplace oficial de Claude Code, que ya está registrado por defecto:

```
/plugin install azure@claude-plugins-official
```

```
/plugin install terraform@claude-plugins-official
```

```
/plugin install microsoft-docs@claude-plugins-official
```

Yo los instalo con alcance `project` solo en los repos donde aplican, para no cargar tokens en los demás.

**Omarchy.** Tengo dos skills más (`omarchy`, `diagnose-crash`) que trae la distro Omarchy de Linux y solo sirven ahí. Si la usas, ya las tienes.

## Agentes

Un subagente corre en su propio contexto, lee mucho y devuelve una conclusión corta. La idea es que el hilo principal no se ensucie con volcados de logs, tests o diffs enormes.

Reglas comunes a todos:

- **Ninguno commitea.** La identidad y la firma se aplican en el hilo principal.
- **Salvo `tf-dev` y `po`, son de solo lectura:** proponen el cambio, no lo aplican.
- Nunca imprimen valores de secretos; dicen que existen y dónde.

| Agente | Cuándo lo usa Claude | Modelo |
|---|---|---|
| `code-dev` | Preparar un cambio de código que exige leer varios archivos para entender la convención. Devuelve un diff aplicable | heredado |
| `debug` | Encontrar la causa raíz de un error recorriendo capas o muchos logs. Distingue causa de síntoma | sonnet |
| `qa` | Correr la batería de tests y reportar qué falla y por qué, o proponer los que faltan. Detecta pytest, vitest, phpunit, cargo, go | sonnet |
| `sec-review` | Barrer un repo o configuración completa buscando secretos versionados, permisos amplios o exposición pública | heredado |
| `k8s-triage` | Diagnosticar pods, eventos, ingress o HPA en un cluster. Solo `kubectl get/describe/logs` | sonnet |
| `tf-review` | Revisar un diff o un `terraform plan` y señalar lo que destruye, recrea o expone. Nunca `apply` | sonnet |
| `tf-dev` | **Escribir** Terraform en Azure o AWS: implementa en una rama nueva, formatea, valida y entrega para revisión. Nunca `apply` ni `destroy` | opus |
| `po` | Product owner: qué sigue, qué hay pendiente, qué se retrasa. Mantiene `BACKLOG.md` barriendo el plan, PRs abiertos y ramas sin mergear | heredado |

"Heredado" significa que usa el modelo de tu sesión. Puedes cambiarlo editando el campo `model` del frontmatter de cada `.md`.

### Para que Claude delegue solo

Las descripciones ya dicen "USAR PROACTIVAMENTE", pero funciona mucho mejor si tu `~/.claude/CLAUDE.md` tiene una tabla de cuándo delegar. Copia esto:

```markdown
## Delegación a subagentes

Cuando una tarea encaje con uno de estos, delegar en vez de hacerlo en el hilo principal.
Salvo `tf-dev` y `po`, son de solo lectura. Ninguno commitea.

| Agente | Delegar cuando |
|---|---|
| `code-dev` | preparar un cambio de código que exige leer varios archivos para entender la convención |
| `qa` | correr la batería de tests, o proponer los que faltan |
| `debug` | encontrar la causa raíz de un error recorriendo varias capas o muchos logs |
| `tf-review` | revisar un diff o un `plan` de Terraform antes de proponer aplicarlo |
| `sec-review` | barrer un repo o una configuración buscando secretos, permisos amplios o exposición |
| `k8s-triage` | diagnosticar pods, eventos o ingress en un cluster |
| `tf-dev` | escribir Terraform: implementa en una rama nueva, valida y entrega; el hilo principal revisa con `tf-review` y commitea |
| `po` | saber qué sigue o qué hay pendiente; al abrir sesión y al cerrar una tarea |

No delegar lo que se resuelve con un comando: el subagente arranca sin contexto y
explicarle la tarea cuesta más de lo que ahorra.
```

### Ajustes que seguro vas a querer

- **`po`** tiene una tabla de "frentes" apuntando a mis proyectos (repos de `credicorp-internal`, archivos `BACKLOG.md` y `BACKLOG-hcm.md`). Edita `plugins/agentes/agents/po.md` y pon tus repos y tus planes.
- **`tf-dev`** menciona convenciones mías de módulos (`az-mod-*`, carpeta `st/`). Si tu layout es otro, cámbialo en `plugins/agentes/agents/tf-dev.md`.

Para editarlos sin esperar a que yo publique, clona el repo y registra el marketplace desde la ruta local:

```
/plugin marketplace add /ruta/a/claude-toolkit
```

## Estructura

```
claude-toolkit/
├── .claude-plugin/marketplace.json   # define los tres plugins
└── plugins/
    ├── skills/                       # skills-devops
    │   ├── .claude-plugin/plugin.json
    │   └── skills/<nombre>/SKILL.md
    ├── terceros/                     # skills-terceros
    │   ├── .claude-plugin/plugin.json
    │   └── skills/<nombre>/SKILL.md
    └── agentes/                      # agentes-devops
        ├── .claude-plugin/plugin.json
        └── agents/<nombre>.md
```

Para agregar una skill: carpeta nueva en `plugins/skills/skills/` con su `SKILL.md`. Para un agente: un `.md` nuevo en `plugins/agentes/agents/` con frontmatter `name`, `description` y `tools`. Valida con `claude plugin validate .` antes de hacer push.
