---
name: mapa-proyecto
description: >
  Crear o refrescar el mapa de un proyecto con muchos repos o muchos documentos, para no
  volver a explorarlo desde cero en cada sesión. Genera un índice con las columnas
  factuales leídas de git o del disco y las descripciones mantenidas a mano. Disparadores:
  "haz un mapa del proyecto", "un índice de los repos", "resumen de qué hay en esta
  carpeta", "actualiza el mapa", "una matriz de los repositorios", "no encuentro dónde
  está cada cosa", o al abrir un proyecto con más de ~10 repos o documentos sin índice.
---

# Mapa de proyecto

Explorar una carpeta con 40 repos cuesta miles de tokens y se repite en cada sesión. Un
mapa de ~1.500 tokens que se lee de una vez lo reemplaza — **siempre que esté al día**. Un
mapa desactualizado es peor que ninguno, porque se confía en él.

De ahí la regla de diseño: **lo que cambia solo se genera; lo que no cambia se escribe a mano.**

| Se genera en cada corrida | Se mantiene a mano |
|---|---|
| Remoto, rama actual, fecha del último commit | Qué es cada repo y para qué sirve |
| Stack detectado por los archivos presentes | Cómo se agrupan |
| Repos nuevos, clones duplicados, ramas fuera de `main` | Advertencias y estado ("en pausa", "sin implementar") |

## Cómo montarlo

1. **Reunir los datos factuales con shell**, no leyendo archivos: `git remote get-url`,
   `git branch --show-current`, `git log -1 --format=%cs`, y la presencia de
   `*.tf` / `package.json` / `pyproject.toml` / `Dockerfile` para inferir el stack.
2. **Escribir un generador** en `.claude/tools/mapa-repos.py` con las descripciones en un
   diccionario al principio del archivo. Que emita el `.md` a la raíz del proyecto.
3. El generador debe **detectar lo que no está descrito** y marcarlo como
   `**SIN DESCRIBIR**` en vez de omitirlo. Un repo nuevo que desaparece del mapa es la
   forma en que un mapa muere.
4. **Referenciar el mapa desde el `CLAUDE.md`** del proyecto, diciendo que se lea antes de
   explorar.
5. El `.md` lleva un encabezado que dice **generado, no editar a mano**, con la fecha y la
   ruta del generador.

Ejemplo ya montado: `~/Documents/local/edp-agentes/.claude/tools/mapa-repos.py` — 43 repos
en ~1.500 tokens, con secciones automáticas de trabajo en curso y clones duplicados.

## Qué incluir

- **Agrupación por función, no alfabética.** El orden alfabético no dice nada; "módulos de
  Terraform" / "agentes" / "tooling" sí.
- **Una línea por repo**, no un párrafo. El mapa se lee entero en cada sesión: cada palabra
  se paga cada vez.
- **Las anomalías, arriba y marcadas.** Clones duplicados, repos que siguen siendo la
  plantilla vacía, repos de otro proyecto mezclados en la carpeta, ramas viejas sin mergear.
  Eso es lo que ahorra tiempo de verdad.
- **Lo que está en pausa o roto**, con la advertencia en negrita. Un mapa que no avisa de
  la trampa induce a pisarla.

## Qué NO incluir

- Descripciones largas: eso va en el `CLAUDE.md` del repo correspondiente.
- Listas de archivos internos: para eso está el repo.
- Nada que se pueda deducir del nombre. `az-mod-redis-cache` crea un Redis Cache; escribirlo
  es gastar tokens en cada sesión para no decir nada.

## Al refrescarlo

Correr el generador y revisar solo lo que cambió: repos nuevos sin describir, ramas que
volvieron a `main`, duplicados nuevos. **No reescribir las descripciones existentes** salvo
que algo haya dejado de ser cierto.

## Variante para carpetas de documentos

El mismo patrón sirve para una carpeta con decenas de `.md`: el generador lista los archivos
con su fecha de modificación y el diccionario dice de qué trata cada uno. Los documentos
vivos (los que cambian seguido) se marcan como tales para que se lean siempre en vez de
confiar en el resumen.
