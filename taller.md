# Taller 01 — String Interpolation y Data Binding en Angular 21

> **Institución:** Instituto Universitario de la Paz (Unipaz)
> **Programa / Asignatura:** Programación Básica en Cliente — PBC Nivel 1
> **Tecnología:** Angular `21.2.7` (standalone components, signals, nuevo control flow `@for` / `@if`)
> **Duración estimada:** 2 sesiones (≈ 4 horas)
> **Modalidad:** Práctico guiado + entrega individual vía Pull Request
> **Autor docente:** _ing. Alfonso Chermes_

---

## 🎯 Objetivos de aprendizaje

Al finalizar este taller el estudiante estará en capacidad de:

1. Explicar con sus propias palabras qué es la **interpolación de cadenas (String Interpolation)** y en qué se diferencia del **Data Binding**.
2. Identificar y usar los **cuatro tipos de Data Binding** que ofrece Angular: *Interpolation*, *Property Binding*, *Event Binding* y *Two-Way Binding*.
3. Integrar **signals** de Angular 21 en plantillas para lograr reactividad.
4. Aplicar estos conceptos en un proyecto real, versionarlo con **Git** y realizar una contribución mediante **Fork + Pull Request**.

---

## 🧠 1. Marco conceptual

### 1.1 ¿Qué es *Data Binding*?

**Data Binding** (enlace de datos) es el mecanismo mediante el cual Angular **sincroniza los datos** entre la **clase del componente** (TypeScript) y su **plantilla** (HTML).

Antes de Angular (o en JS plano) tendríamos que manipular el DOM a mano:

```javascript
document.getElementById('saludo').innerText = 'Hola ' + nombre;
```

Con Angular, la plantilla **se actualiza sola** cuando los datos cambian. Esa "magia" se llama **binding**.

Angular define **4 tipos de Data Binding** según la **dirección del flujo de datos**:

| Tipo                     | Sintaxis                     | Dirección                         | Ejemplo                        |
| ------------------------ | ---------------------------- | --------------------------------- | ------------------------------ |
| **Interpolation**        | `{{ expresion }}`            | Componente → Vista (solo texto)   | `<h1>{{ titulo }}</h1>`        |
| **Property Binding**     | `[propiedad]="expresion"`    | Componente → Vista (propiedades)  | `<img [src]="urlImagen">`      |
| **Event Binding**        | `(evento)="manejador()"`     | Vista → Componente                | `<button (click)="saludar()">` |
| **Two-Way Binding**      | `[(ngModel)]="variable"`     | Componente ⇄ Vista (bidireccional)| `<input [(ngModel)]="nombre">` |

> 💡 **Regla nemotécnica de los corchetes:**
> - `[ ]` → **entra** dato al elemento (propiedad).
> - `( )` → **sale** evento del elemento.
> - `[( )]` → **"caja de banana"**, entra y sale (bidireccional).

---

### 1.2 String Interpolation en detalle

La **interpolación de cadenas** usa la sintaxis de **dobles llaves** `{{ }}` (llamadas *mustaches* o *handlebars*) para incrustar el valor de una expresión de TypeScript dentro del HTML **como texto**.

**Sintaxis general:**

```html
{{ expresion }}
```

**¿Qué puede ir dentro de `{{ }}`?**

✅ **SÍ se permite:**
- Variables del componente: `{{ nombre }}`
- Propiedades/objetos: `{{ usuario.email }}`
- Llamadas a funciones puras: `{{ calcularTotal() }}`
- Llamadas a signals: `{{ contador() }}` ← *Angular 21 style*
- Operaciones simples: `{{ precio * cantidad }}`
- Operador ternario: `{{ edad >= 18 ? 'Mayor' : 'Menor' }}`
- Concatenación: `{{ 'Hola, ' + nombre }}`
- Template literals: `{{ \`Hola ${nombre}\` }}`

❌ **NO se permite:**
- Asignaciones: `{{ x = 5 }}` 🚫
- Operadores `++` / `--`: `{{ contador++ }}` 🚫
- `new`, `typeof`, `instanceof` 🚫
- Operadores bit a bit `|`, `&` (el `|` se reserva para **pipes**) 🚫
- Sentencias tipo `if`, `for` (para eso existen `@if`, `@for`) 🚫

> ⚠️ **Importante:** La interpolación convierte el resultado a **string** automáticamente. Si necesitas asignar un valor **no-string** a una propiedad (ej. un booleano a `disabled`), **debes usar Property Binding**, no interpolación.

**Diferencia clave entre Interpolation y Property Binding:**

```html
<!-- ❌ MAL: 'true' y 'false' son strings, ambos son "truthy" -->
<button disabled="{{ estaDeshabilitado }}">Enviar</button>

<!-- ✅ BIEN: estaDeshabilitado se evalúa como booleano real -->
<button [disabled]="estaDeshabilitado">Enviar</button>
```

---

### 1.3 Property Binding `[propiedad]`

Enlaza una **propiedad del DOM** (no un atributo HTML) con una expresión del componente.

```html
<img [src]="urlFoto" [alt]="descripcion">
<input [value]="nombreInicial" [disabled]="bloqueado">
<div [class.activo]="estaActivo">...</div>
<div [style.color]="colorTexto">...</div>
```

> 🎓 **Propiedad vs Atributo:** Los atributos son los que escribes en el HTML estático; las propiedades son las del objeto DOM en memoria. Por eso para enlazar atributos ARIA se usa `[attr.aria-label]="..."`.

---

### 1.4 Event Binding `(evento)`

Escucha un **evento del DOM** (o uno personalizado) y ejecuta un método del componente.

```html
<button (click)="incrementar()">+1</button>
<input (input)="onInput($event)">
<form (submit)="guardar($event)">...</form>
```

La variable especial **`$event`** contiene el objeto del evento nativo del navegador.

---

### 1.5 Two-Way Binding `[(ngModel)]`

Combina property binding y event binding en **una sola sintaxis**. Se usa típicamente con inputs de formularios.

```html
<input [(ngModel)]="nombre">
<p>Hola, {{ nombre }}</p>
```

Equivale a:

```html
<input [value]="nombre" (input)="nombre = $event.target.value">
```

> ⚠️ **Requisito:** Para usar `ngModel` debes **importar `FormsModule`** en el componente standalone (lo veremos en el Paso 5).

---

### 1.6 Signals (Angular 21) + Interpolation

Desde Angular 16+ (y como estándar en Angular 21), la forma recomendada de manejar estado reactivo son los **signals**.

```typescript
import { Component, signal } from '@angular/core';

@Component({...})
export class App {
  protected readonly title = signal('angular-base');
  protected readonly contador = signal(0);

  incrementar() {
    this.contador.update(v => v + 1);
  }
}
```

En la plantilla, un signal se **lee invocándolo como función**:

```html
<h1>{{ title() }}</h1>
<p>Contador: {{ contador() }}</p>
<button (click)="incrementar()">+1</button>
```

> 🔎 **Observa la línea 235 del archivo `src/app/app.html` del repositorio**: ya contiene un ejemplo real: `<h1>Hello, {{ title() }}</h1>`.

---

## 🛠️ 2. Preparación del entorno

### 2.1 Requisitos previos

- **Node.js** 20 LTS o superior → [https://nodejs.org](https://nodejs.org)
- **Angular CLI 21**
  ```bash
  npm install -g @angular/cli@21
  ng version
  ```
- **Git** instalado y configurado con tu cuenta de GitHub.
- **Cuenta en GitHub** (si no tienes, créala en [https://github.com](https://github.com)).
- Editor recomendado: **VS Code** o **Cursor**.

### 2.2 Clonar y levantar el proyecto

```bash
git clone <URL-DEL-REPO-ORIGINAL>
cd angular-base
npm install
npm start
```

Abre `http://localhost:4200` — deberías ver la pantalla de bienvenida de Angular con el texto **"Hello, angular-base"**.

---

## 📝 3. Paso a paso guiado

En esta sección vamos a **modificar progresivamente** el archivo `src/app/app.ts` y `src/app/app.html` para practicar cada tipo de binding.

### Paso 1 — Reemplazar el contenido placeholder

Abre `src/app/app.html`. Entre las líneas 1 y 342 hay plantilla de bienvenida autogenerada. **Selecciónala completa y bórrala**, dejando únicamente:

```html
<main>
  <h1>Taller: Data Binding en Angular 21</h1>
</main>

<router-outlet />
```

Guarda y verifica que la app se recarga en el navegador (HMR) mostrando solo el título.

---

### Paso 2 — String Interpolation con signals

Edita `src/app/app.ts` para agregar nuevas propiedades:

```typescript
import { Component, signal } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class App {
  protected readonly title = signal('Taller Data Binding');
  protected readonly estudiante = signal('Nombre del estudiante');
  protected readonly anio = signal(2026);

  nombreCompleto(): string {
    return `${this.estudiante()} — Unipaz ${this.anio()}`;
  }
}
```

Ahora en `src/app/app.html`:

```html
<main>
  <h1>{{ title() }}</h1>
  <p>Bienvenido, {{ estudiante() }}</p>
  <p>Año académico: {{ anio() }}</p>
  <p>{{ nombreCompleto() }}</p>
  <p>¿Es mayor de edad?: {{ anio() >= 2026 ? 'Sí' : 'No' }}</p>
</main>
```

✅ **Comprobación:** deberían verse los 5 párrafos con los valores correctos.

---

### Paso 3 — Property Binding

Agrega al componente una URL y un estado:

```typescript
protected readonly logoUrl = signal('https://angular.dev/assets/images/press-kit/angular_icon_gradient.gif');
protected readonly deshabilitado = signal(true);
```

En la plantilla:

```html
<img [src]="logoUrl()" [alt]="'Logo de ' + title()" width="120">

<button [disabled]="deshabilitado()">
  Este botón está {{ deshabilitado() ? 'bloqueado' : 'activo' }}
</button>

<div [style.color]="deshabilitado() ? 'gray' : 'green'"
     [class.activo]="!deshabilitado()">
  Estado visual dinámico
</div>
```

✅ **Comprobación:** la imagen debe cargar y el botón debe verse deshabilitado.

---

### Paso 4 — Event Binding

Añade un signal contador y sus métodos:

```typescript
protected readonly contador = signal(0);

incrementar(): void {
  this.contador.update(v => v + 1);
}

decrementar(): void {
  this.contador.update(v => v - 1);
}

reiniciar(): void {
  this.contador.set(0);
}

toggleBoton(): void {
  this.deshabilitado.update(v => !v);
}
```

En la plantilla:

```html
<section>
  <h2>Contador: {{ contador() }}</h2>
  <button (click)="incrementar()">➕ Sumar</button>
  <button (click)="decrementar()">➖ Restar</button>
  <button (click)="reiniciar()">🔄 Reiniciar</button>
  <button (click)="toggleBoton()">Alternar botón deshabilitado</button>
</section>
```

✅ **Comprobación:** al hacer clic, el número cambia en tiempo real. ¡Eso es reactividad con signals + event binding!

---

### Paso 5 — Two-Way Binding con `ngModel`

Para usar `[(ngModel)]` debemos importar `FormsModule` en el componente standalone.

**`src/app/app.ts`:**

```typescript
import { Component, signal } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule } from '@angular/forms'; // 👈 AÑADIR

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, FormsModule], // 👈 AÑADIR AQUÍ
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class App {
  // ... lo anterior
  protected nombreUsuario = ''; // variable simple (no signal) para ngModel clásico
}
```

**`src/app/app.html`:**

```html
<section>
  <h2>Two-Way Binding</h2>
  <label>
    Escribe tu nombre:
    <input type="text" [(ngModel)]="nombreUsuario" name="nombre">
  </label>
  <p>Hola, <strong>{{ nombreUsuario || '(escribe algo...)' }}</strong></p>
</section>
```

✅ **Comprobación:** al escribir en el `<input>`, el `<p>` se actualiza automáticamente en cada pulsación de tecla.

---

### Paso 6 — Combinando todo

Crea un pequeño bloque que use los 4 tipos juntos:

```html
<section [class.resaltado]="contador() > 5">
  <h2>Resumen</h2>
  <p>Usuario: {{ nombreUsuario }}</p>           <!-- Interpolation -->
  <p>Clicks totales: {{ contador() }}</p>        <!-- Interpolation (signal) -->
  <button [disabled]="!nombreUsuario"            <!-- Property Binding -->
          (click)="incrementar()">              <!-- Event Binding -->
    Registrar click
  </button>
</section>
```

Y en `src/app/app.css`:

```css
.resaltado {
  background: #fff3cd;
  border: 2px solid #ffc107;
  padding: 1rem;
  border-radius: 8px;
}
```

✅ **Comprobación:** cuando el contador pase de 5, la sección se resalta en amarillo. ¡Felicidades, dominas los 4 tipos de binding! 🎓

---

## 🚀 4. Taller práctico evaluable

### 🎯 Enunciado

Vas a construir una **"Ficha Interactiva del Estudiante Unipaz"** dentro de este mismo repositorio. La aplicación debe demostrar el uso correcto de los **4 tipos de Data Binding**.

### 📋 Requisitos funcionales

La ficha debe contener un formulario con los siguientes campos enlazados con `[(ngModel)]`:

| Campo              | Tipo de input            | Variable sugerida     |
| ------------------ | ------------------------ | --------------------- |
| Nombre completo    | `text`                   | `nombre`              |
| Código estudiantil | `number`                 | `codigo`              |
| Programa académico | `select` (PBC, Ing. Sistemas, Ing. Ambiental, Derecho) | `programa` |
| Semestre (1-10)    | `range` o `number`       | `semestre`            |
| Correo             | `email`                  | `correo`              |
| ¿Beca Icetex?      | `checkbox`               | `tieneBeca`           |
| Color favorito     | `color`                  | `colorFavorito`       |

Y debe mostrar en tiempo real (sección **"Vista Previa"**):

1. **Interpolation:** un saludo tipo `"Hola, {{ nombre }}, estudiante de {{ programa }} — Unipaz"`.
2. **Property Binding:**
   - Un `<div>` cuyo `[style.backgroundColor]` sea el color favorito del usuario.
   - Un `[class.semestre-avanzado]="semestre >= 7"` que cambie el estilo si está en semestres superiores.
   - Un botón **"Guardar"** con `[disabled]` que se active solo cuando **nombre y correo** tengan valor.
3. **Event Binding:**
   - Un botón `(click)="guardar()"` que muestre con `console.log` o `alert` los datos capturados.
   - Un botón `(click)="limpiar()"` que reinicie todos los campos.
   - Un contador de veces que se ha pulsado "Guardar".
4. **Signals:**
   - El contador de guardados debe ser un `signal<number>`.
   - Debe existir al menos un `signal` con un mensaje de estado (ej: `"Sin guardar"` / `"Guardado ✓"`).
5. **Bonus (opcional, +0.5):**
   - Validar que el correo termine en `@unipaz.edu.co` usando una función llamada desde interpolation: `{{ correoValido() ? '✓' : '✗' }}`.
   - Usar `@if` de Angular 21 para mostrar la Vista Previa solo cuando haya un nombre escrito.

### 📁 Estructura sugerida

Crea la funcionalidad como un **componente nuevo**:

```bash
ng generate component ficha-estudiante
```

Y úsalo desde `src/app/app.html`:

```html
<app-ficha-estudiante />
```

### 🎨 Requisitos no funcionales

- Código formateado con **Prettier** (`npx prettier --write .`).
- Nombres de variables y funciones en **español** y con estilo `camelCase`.
- **Sin** manipulación directa del DOM (`document.getElementById`, etc.). Todo debe hacerse con bindings.
- Comentarios mínimos y útiles; evita comentarios redundantes.

---

## 📤 5. Instrucciones de entrega (Fork + Pull Request)

### Paso 1 — Forkear el repositorio

1. Entra a `<URL-DEL-REPO-ORIGINAL>` en GitHub.
2. Haz clic en el botón **"Fork"** (esquina superior derecha).
3. Selecciona tu cuenta como destino → GitHub creará una copia en `https://github.com/<tu-usuario>/angular-base`.

### Paso 2 — Clonar TU fork

```bash
git clone https://github.com/<tu-usuario>/angular-base.git
cd angular-base
npm install
```

### Paso 3 — Crear una rama con tu entrega

> ⚠️ **Nunca trabajes directo en `main`.** Usa una rama con tu identificación.

```bash
git checkout -b taller-01/<tu-codigo>-<tu-apellido>
# Ejemplo: git checkout -b taller-01/2024001-perez
```

### Paso 4 — Desarrollar, commitear y pushear

```bash
# Trabaja en el código...
npm start  # para verificar en el navegador

git add .
git commit -m "feat(taller-01): ficha interactiva del estudiante con 4 tipos de data binding"
git push origin taller-01/<tu-codigo>-<tu-apellido>
```

### Paso 5 — Crear el Pull Request

1. En GitHub, entra a tu fork.
2. Aparecerá un banner **"Compare & pull request"** — haz clic.
3. Asegúrate de que:
   - **Base repository:** `<repo-del-docente>/angular-base` · **base:** `main`
   - **Head repository:** `<tu-usuario>/angular-base` · **compare:** `taller-01/<tu-codigo>-<tu-apellido>`
4. Título del PR:
   ```
   Taller 01 - <Tu Nombre Completo> - <Código>
   ```
5. Descripción del PR (plantilla mínima):
   ```markdown
   ## Estudiante
   - Nombre: ...
   - Código: ...
   - Programa: ...

   ## ¿Qué implementé?
   - [x] String Interpolation (ejemplos: ...)
   - [x] Property Binding (ejemplos: ...)
   - [x] Event Binding (ejemplos: ...)
   - [x] Two-Way Binding con ngModel
   - [ ] Bonus (si aplica)

   ## Capturas de pantalla
   (arrastra aquí 1-2 imágenes de la app corriendo)

   ## Dificultades encontradas
   ...
   ```
6. Clic en **"Create pull request"**.

### Paso 6 — Esperar revisión

El docente revisará tu PR. Si hay comentarios, responde en la misma conversación, haz los ajustes en tu rama local y vuelve a hacer `git push` — el PR se actualiza automáticamente. **No crees un PR nuevo.**

---

## ✅ 6. Criterios de evaluación

| Criterio                                                    | Peso |
| ----------------------------------------------------------- | :--: |
| Uso correcto de **String Interpolation**                    | 15%  |
| Uso correcto de **Property Binding** (al menos 3 ejemplos)  | 15%  |
| Uso correcto de **Event Binding** (al menos 3 ejemplos)     | 15%  |
| Uso correcto de **Two-Way Binding** con `FormsModule`       | 15%  |
| Uso de **signals** (Angular 21)                             | 10%  |
| Organización del código y componente `ficha-estudiante`     | 10%  |
| Commit(s) claros, rama bien nombrada y **PR abierto correctamente** | 15%  |
| Bonus (validaciones, `@if`, estilos)                        | +5%  |

**Nota mínima aprobatoria: 3.0 / 5.0**
**Fecha límite de entrega (PR abierto):** _a definir por el docente_.

---

## 📚 7. Recursos recomendados

- 📖 [Angular — Template syntax](https://angular.dev/guide/templates)
- 📖 [Angular — Signals](https://angular.dev/guide/signals)
- 📖 [Angular — Forms](https://angular.dev/guide/forms)
- 🎥 Angular DevTools (extensión de navegador) para inspeccionar bindings en tiempo real.
- 🎥 [Canal oficial de Angular](https://www.youtube.com/@Angular)

---

## 🤝 8. Reglas del juego

- ❌ **Está prohibido** copiar código de otro compañero. Usar IA está permitido **solo como herramienta de consulta**, no para generar la entrega completa.
- ✅ Si te bloqueas, abre un *issue* en el repositorio original con tu pregunta — así se resuelve para todos.
- ✅ Pregunta, explora, rompe el código y vuelve a construirlo. **Así se aprende Angular.**

---

> _"Dime y lo olvido, enséñame y lo recuerdo, involúcrame y lo aprendo."_ — Benjamin Franklin

**¡Éxitos con tu primer Pull Request, futuro ingeniero Unipaz! 🚀**
