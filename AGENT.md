# AGENT.md - Guidelines & Best Practices for AI Agents

Welcome to the **Freedium** repository. This document defines the architectural overview, development rules, conventions, and operational best practices for AI agents working on this codebase.

---

## 1. Project Overview & Architecture

Freedium is an open-source alternative Medium parser and proxy that renders articles cleanly without paywalls.

### Key Components & Directory Structure
- **[`web/`](file:///D:/pj/me/freedium/web)**: Core FastAPI web server, templates, routing handlers, and background services.
  - `web/server/main.py`: Application entry point and router attachments.
  - `web/server/config.py`: Environment configurations powered by Pydantic Settings.
  - `web/server/handlers/`: Route handlers (`post.py`, `iframe.py`, `miro.py`, `misc.py`).
  - `web/server/templates/`: Jinja2 templates styled with Tailwind CSS.
  - `web/server/utils/`: Caching, logging (`loguru`), error reporting, maintenance schedulers.
- **[`medium-parser/`](file:///D:/pj/me/freedium/medium-parser)**: Core parsing logic responsible for resolving Medium URLs, extracting markup/AST, fetching media/content, and formatting rendering trees.
- **[`freedium-library/`](file:///D:/pj/me/freedium/freedium-library)**: Shared library components across Freedium microservices.
- **[`database-lib/`](file:///D:/pj/me/freedium/database-lib)**: Database access and abstraction layer.
- **[`proxy-balancer/`](file:///D:/pj/me/freedium/proxy-balancer) & [`wgcf/`](file:///D:/pj/me/freedium/wgcf)**: Proxy rotation and Cloudflare WARP proxy integration to bypass upstream rate-limits.
- **[`docker-compose/`](file:///D:/pj/me/freedium/docker-compose)**: Multi-profile orchestration (`min`, `local`, `prod`).
- **[`caddy/`](file:///D:/pj/me/freedium/caddy)**: Reverse proxy configuration and static file delivery.

---

## 2. Agent Workflow Rules & Guidelines

1. **Safety First**:
   - Never commit sensitive keys, real Medium auth cookies (`uid`, `sid`), or production passwords. Always reference or populate [`.env_template`](file:///D:/pj/me/freedium/.env_template).
   - Do not overwrite existing configuration files (`.env`, `docker-compose.yml`) without explicit verification.
2. **Minimal and Targeted Modifications**:
   - Keep changes tightly scoped to the specific issue or feature requested.
   - Preserve existing comments, docstrings, and licensing info.
3. **Submodule / Subpackage Awareness**:
   - The repository is organized into distinct subpackages (e.g., `web/`, `medium-parser/`, `database-lib/`).
   - Notice the separate `pyproject.toml` or `requirements.txt` / `setup.py` files in each subproject. Run dependency and lint commands in the appropriate directory or with appropriate arguments.

---

## 3. Code Style & Standards

### Python (Backend)
- **Python Version**: `>= 3.12` for `web/` (`pyproject.toml`).
- **Formatting & Linting**:
  - **Ruff**: Configured with line-length `120`, rules `E`, `F`, `UP`, `I`, `C4`.
  - **Black**: Used with line-length `220` (in `pyproject.toml`) or conforming to project style.
  - **Mypy**: Strict type-checking enabled (`strict = true`). Always add proper type annotations to function parameters, return values, and data structures.
  - Use `loguru` for logging (`web/server/utils/logger.py`) rather than raw `print()` statements.
- **Async Conventions**:
  - Endpoints and network I/O operations must be `async def`. Use non-blocking HTTP clients (`curl-cffi`, `httpx`, or `aiohttp`) as defined by the submodules.

### Frontend & Templates
- HTML files are Jinja2 templates located in [`web/server/templates/`](file:///D:/pj/me/freedium/web/server/templates/).
- Styling is built using **Tailwind CSS v3**. Maintain responsive, mobile-first utility classes and dark-mode compatibility where applicable.
- HTML linting: `djlint` is configured in `[tool.poetry.group.dev.dependencies]`.

---

## 4. Testing & Verification

Before finalizing any changes or submitting pull requests:

1. **Linting & Code Quality**:
   ```bash
   cd web
   poetry run ruff check .
   poetry run black --check .
   poetry run mypy .
   ```
2. **Automated & Smoke Tests**:
   - Run existing smoke tests:
     ```bash
     python -m unittest tests/smokie_tests.py
     ```
   - When adding new parser rules, test edge cases with various Medium URL schemes (custom domains, user handles `@username/slug`, short links, medium publication paths).

---

## 5. Docker & Deployment Best Practices

- Use Docker Compose profiles appropriately:
  - `min`: Lightweight local testing without auxiliary proxies.
  - `local`: Local testing with exposed host `freedium.local`.
  - `prod`: Full stack including proxies, Plausible analytics, and monitoring.
- Ensure any new environment variables are documented with sensible defaults in [`.env_template`](file:///D:/pj/me/freedium/.env_template).
