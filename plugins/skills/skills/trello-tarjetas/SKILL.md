---
name: trello-tarjetas
description: >
  Atender tarjetas de Trello con el comando local `trello`: ver qué hay pendiente o
  asignado, leer una tarjeta, resolverla en el repo que corresponda, y moverla o
  comentarla según las indicaciones del usuario. Disparadores: "mis tarjetas", "qué
  tengo en Trello", "atiende la tarjeta", "toma la de arriba de Por hacer", "muévela a",
  "pásala a Hecho", "comenta en la tarjeta", un enlace trello.com/c/..., o el nombre de un
  tablero de soporte como Verificativa.
---

# Tarjetas de Trello

Todo pasa por el comando `trello` (`trello` sin argumentos imprime la ayuda). **Solo alcanza
los tableros listados en `~/.config/trello-claude/boards`** — ese archivo lo mantiene el
usuario; no lo edites, y si una tarjeta es de otro tablero, dilo y no busques rodeos.

Para crear una tarjeta nueva — una mejora o un hallazgo que merece su propio ticket:
`trello new <tablero> <lista> "<titulo>" [--desc TEXTO]`.

## Tableros y repos

| Alias | Tablero | Se resuelve en |
|---|---|---|
| `verificativa` | SOPORTE TÉCNICO - Verificativa | `~/Documents/local/verificativa-dark` (panel Laravel), `verificativa-reload-ws` (API Node/TS), `verificativa-deploy` (manifiestos k8s) |

### Listas de `verificativa` (en orden)

`POR HACER` → `HACIENDO` → `PARA DESPLIEGUE EN QA` → `DESPLEGAR EN PRD` → `TERMINADO`.
Aparte: `REGRESADO` (QA lo devolvió; lo mueve QA), `stand by` (pausa), `Documentaciones`
(archivo con **más de 300 tarjetas**) y `espacio` (vacía).

**Nunca `trello cards verificativa` sin `--mine` o `--list`**: vuelca Documentaciones entera
al contexto.

Movimientos: a `HACIENDO` al empezar y a `PARA DESPLIEGUE EN QA` al subir el cambio, con
confirmación. A `DESPLEGAR EN PRD`, `TERMINADO` o `stand by`, **solo por orden explícita del
usuario para esa tarjeta**. A `REGRESADO` no se mueve: eso lo hace QA.

Crear una tarjeta (`trello new`) también pide confirmación antes de ejecutarse, igual que
`move`, `comment` y `check`: decir antes el tablero, la lista y el título.

Leer el `CLAUDE.md` del repo antes de tocarlo: cuenta AWS, perfil `default_veriprod`, y qué
comandos operan sobre producción.

## Flujo para atender una tarjeta

1. **Leerla entera**: `trello card <ref>` — descripción, checklist y últimos comentarios.
   El contexto suele estar en los comentarios, no en el título.
2. **Decir en el chat qué se entiende que se pide y en qué repo.** Si la tarjeta es ambigua,
   preguntar antes de tocar código.
3. **Moverla a la lista de trabajo en curso** al empezar — antes, confirmar con el usuario.
4. Resolver en el repo con las reglas de siempre: rama corta desde `main`, identidad de
   commit según el remote, nada destructivo sin confirmación.
5. **Comentar lo que se hizo** — corto y factual: qué se cambió, en qué repo, y el enlace al
   MR o al commit. Sin recomendaciones: eso va en el chat.
6. **Moverla** a la lista que corresponda según el paso en que quedó.

## Reglas

- `move`, `comment`, `check` y `new` piden confirmación siempre — es una regla del harness,
  no hace falta preguntar dos veces. Pero **antes de ejecutarlos, decir qué se va a hacer y
  dónde**, para que la confirmación sea informada.
- **Nunca mover a la lista final de terminado sin que el usuario lo diga explícitamente**
  para esa tarjeta. Resolver no es cerrar: puede faltar validación del cliente.
- No tocar tarjetas asignadas a otras personas salvo que el usuario lo pida.
- Todo comentario queda visible para el equipo del tablero y firmado como el usuario.
- Los nombres de las listas se leen con `trello lists <tablero>`; no asumirlos de memoria.
- Cada escritura queda registrada en `~/.local/state/trello-claude/acciones.log`.
