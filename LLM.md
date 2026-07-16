# LLM.md — Adaptador universal de arranque

Adaptador de arranque para cualquier LLM (Codex, Claude, modelos locales, Cursor, Continue u otros). `AGENTS.md` (no publicado en este repo público — guía interna gitignorada) es la autoridad; este documento no la reemplaza ni la relaja.

## Orden de arranque

1. `..\AGENTS.md` (charter raíz del paraguas Homelab).
2. `AGENTS.md` de este repo, si tenés acceso local a él (está gitignorado, no viaja con el clon público).
3. [CLAUDE.md](./CLAUDE.md) — reglas duras de sanitización y estructura bilingüe (válidas también para cualquier otro LLM, no solo Claude).
4. [README.md](./README.md).
5. `git status --short` antes de cualquier cambio.

## Regla crítica de este repo

**Este repositorio es público.** Por defecto, solo lectura: escribir o publicar contenido requiere pedido explícito del usuario en la sesión en curso. Cero datos reales (IPs, hostnames, credenciales, rutas internas) — ver la política de sanitización en `CLAUDE.md`. `git push` lo ejecuta únicamente el usuario.

Restricciones de seguridad, privacidad, trazabilidad y no invención de datos no se relajan por ningún documento de menor prioridad que `AGENTS.md`.
