# Guía para agentes de IA — `angular-base`

Este documento resume el contexto del repositorio para que asistentes automatizados (IDE, CI, bots) trabajen de forma coherente, segura y alineada con el stack real del proyecto.

## Propósito del proyecto

Aplicación web **Angular** generada con **Angular CLI 21.x**, pensada como **plantilla o base** (`angular-base`). El estado actual es un esqueleto funcional: componente raíz con plantilla de bienvenida, **enrutador configurado** pero **sin rutas definidas** (`routes` vacío).

## Stack y versiones (referencia)

- **Angular**: 21.x (`@angular/core`, router, forms, etc.).
- **TypeScript**: ~5.9, `target` ES2022, `module` preserve (configuración típica de Angular moderno).
- **Tests unitarios**: **Vitest** vía `@angular/build:unit-test` (`ng test`).
- **Gestor de paquetes**: **npm** (campo `packageManager`: `npm@11.11.0`).
- **Formateo**: **Prettier** (`.prettierrc`: comillas simples, `printWidth` 100, parser Angular para HTML).

Comprueba `package.json` y `angular.json` si necesitas versiones exactas o nombres de proyecto.

## Estructura relevante

| Ruta | Rol |
|------|-----|
| `src/main.ts` | Bootstrap standalone: `bootstrapApplication(App, appConfig)`. |
| `src/app/app.ts` | Componente raíz (`selector: 'app-root'`), `imports: [RouterOutlet]`, `title` como **signal**. |
| `src/app/app.html` | Plantilla principal (contenido placeholder de Angular + `<router-outlet />`). |
| `src/app/app.config.ts` | `ApplicationConfig`: `provideBrowserGlobalErrorListeners()`, `provideRouter(routes)`. |
| `src/app/app.routes.ts` | Exporta `routes: Routes` (actualmente **array vacío**). |
| `src/app/app.spec.ts` | Pruebas del componente `App` con `TestBed`. |
| `src/styles.css` | Estilos globales. |
| `angular.json` | Proyecto `angular-base`, builder `@angular/build:application`, assets desde `public/`. |
| `tsconfig.json` / `tsconfig.app.json` | TypeScript estricto + `strictTemplates` en compilador de Angular. |

**Convención de prefijo**: `app` (definido en `angular.json`).

## Comandos habituales

Desde la raíz del repo (tras `npm install` si hace falta):

- **Servidor de desarrollo**: `npm start` → `ng serve` (por defecto suele ser `http://localhost:4200/`).
- **Compilación producción**: `npm run build` → `ng build` (salida en `dist/` según configuración CLI).
- **Tests**: `npm test` → `ng test` (Vitest integrado en el builder de Angular).

Si añades dependencias, respeta **npm** y el lockfile existente salvo instrucción contraria.

## Convenciones de código

1. **Standalone**: la app arranca sin `NgModule` raíz; nuevos artefactos deberían seguir el estilo **standalone** salvo que el equipo decida lo contrario.
2. **Signals**: el título en `App` usa `signal()`; al extender estado reactivo, considera signals o flujos idiomáticos de Angular 19+.
3. **Plantillas estrictas**: `strictTemplates: true` — los bindings y tipos en plantillas deben ser válidos en compilación.
4. **TypeScript estricto**: `strict`, `noImplicitReturns`, etc. Evita atajos que rompan estas banderas.
5. **Estilos por componente**: el componente raíz declara `styleUrl: './app.css'`; si creas estilos, mantén coherencia con rutas relativas y el patrón del CLI.

## Qué debe priorizar un agente

- **Cambios mínimos y enfocados**: no refactorizar plantillas placeholder ni estructura del CLI salvo petición explícita.
- **Rutas**: cualquier nueva pantalla debería pasar por `app.routes.ts` y, si aplica, **lazy loading** según buenas prácticas del equipo.
- **Verificación**: tras cambios sustantivos, ejecutar `npm run build` y/o `npm test` cuando sea posible en el entorno.
- **Seguridad y calidad**: enlaces externos ya usan `rel="noopener"` donde aplica; mantener prácticas similares en nuevo markup.

## Límites y avisos

- El componente raíz declara `styleUrl: './app.css'`. Si en tu copia del árbol **no existe** `src/app/app.css`, el build puede fallar hasta que se añada el archivo o se corrija la ruta.
- No hay configuración **Nx** ni monorepo: es un **proyecto Angular único** en la raíz.
- **E2E**: el `README` menciona `ng e2e`, pero **no hay framework e2e configurado por defecto** en este esqueleto; no asumas Cypress/Playwright hasta ver dependencias y targets en `angular.json`.
- Archivos locales no versionados (por ejemplo notas de taller) pueden existir; no los trates como contrato del producto salvo que el usuario los cite.

## Objetivo de este archivo

Reducir alucinaciones sobre el stack, acelerar el onboarding de asistentes y fijar **comandos, rutas de código y convenciones** verificables en el repositorio. Si el proyecto evoluciona (SSR, i18n, capa HTTP, estado global), actualiza esta guía en el mismo cambio que introduzca la nueva arquitectura.
