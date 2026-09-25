# Diseño de Interfaces Web

## Unidad 18 · Preprocesadores CSS (SASS/SCSS y LESS)

**Módulo 0615 · Diseño de Interfaces Web**  
CFGS Desarrollo de Aplicaciones Web (DAW)

---

## Objetivos de aprendizaje · I

- <span class="fragment">Explicar qué es un **preprocesador de estilos** y por qué se necesita, frente a las variables CSS nativas</span>
- <span class="fragment">Escribir hojas de estilo con **SASS/SCSS**: variables, interpolación, anidación, partials y `@extend`</span>
- <span class="fragment">Reutilizar patrones con **`@mixin`** y encapsular lógica con **funciones**, bucles y condicionales</span>
- <span class="fragment">Organizar un proyecto a escala real con la arquitectura de módulos (`@use`/`@forward`) y una estructura escalable</span>

Note: Esta unidad cierra el bloque de *estilos* del RA2 mostrando cómo los preprocesadores aportan abstracción, reutilización y generación de código que el CSS plano no ofrece de forma cómoda. Pregunta al aula quién ha repetido el mismo valor (un color, un breakpoint) en veinte sitios distintos: ese es el dolor que resuelve SASS.

---

## Objetivos de aprendizaje · II

- <span class="fragment">Compilar **SASS a CSS estándar** con `dart-sass` e integrarlo en el flujo con **Vite**, verificando el resultado en el navegador</span>
- <span class="fragment">Conocer las equivalencias en **LESS** y saber cuándo conviene un preprocesador frente a CSS nativo o Tailwind</span>
- <span class="fragment">Vinculación directa con el **RA2** del módulo 0615 (crea interfaces web homogéneas definiendo y aplicando estilos): cumple el **CE 2.j** «analizar y utilizar preprocesadores de estilos para traducir estilos comunes a un código estándar reconocible por los navegadores»</span>

Note: Insistid en que el navegador NUNCA ve SASS: solo recibe CSS plano. El preprocesador trabaja en tiempo de compilación; las Custom Properties, en cambio, resuelven temas en runtime. Esa distinción es la clave conceptual de toda la unidad y del CE 2.j.

---

## ¿Por qué un preprocesador?

<span class="fragment">El CSS describe **presentación**, no programa: al crecer, repetir valores y bloques se vuelve inmanejable.</span>

<span class="fragment">Un preprocesador añade lo que todo lenguaje ofrece: **variables, funciones, mixins, bucles y condicionales** → código DRY y escalable.</span>

<div style="font-size: 0.85rem; text-align: left;">
| Capa | Cuándo cambia | Ideal para |
|---|---|---|
| **CSS nativo** (Custom Properties) | Runtime (navegador/JS) | Temas claro/oscuro, ajustes dinámicos |
| **Preprocesador** (SASS/LESS) | Compilación | Reutilización y generación de código |
| **Utility-first** (Tailwind) | En el HTML | Productividad por restricciones (U17) |
</div>

<span class="fragment">En la práctica se combinan: *tokens* en SASS + *temas* como Custom Properties.</span>

Note: Tres capas que conviven. El error típico es usar variables SASS para algo que debe cambiar en runtime (modo oscuro): eso toca a las Custom Properties. Recordad que U17 (Tailwind) es la tercera vía, utility-first.

---

## SASS frente a SCSS

<span class="fragment">SASS nació con sintaxis de **indentación** (`.sass`); **SCSS** usa llaves y `;` como el CSS clásico.</span>

```scss
.tarjeta {
  padding: 1rem;
  border-radius: 8px;

  &__titulo { font-weight: 700; }   // & = selector padre
}
```

<span class="fragment">Todo CSS válido es SCSS válido → **SCSS es el estándar de facto** (Vite, Webpack, la mayoría de librerías).</span>

Note: Recomendad trabajar siempre en `.scss` salvo proyecto legacy. El `&` representa al selector padre y es la base de la anidación BEM.

---

## Variables e interpolación

```scss
$color-primario: #667eea;
$breakpoints: ("sm": 640px, "md": 768px, "lg": 1024px);

.btn {
  background: $color-primario;
  padding: 1rem (1rem * 2);        // expresiones aritméticas
}

$icono: "buscar";
.icono-#{$icono} { }               // interpolación en selectores
```

<span class="fragment">`!default` solo asigna si la variable no estaba definida → parciales reutilizables.</span>

Note: Las variables se resuelven en compilación y NO pueden cambiar en runtime. La interpolación `#{}` inyecta valores donde no se espera un valor simple (selectores, nombres de propiedad).

---

## Partials y sistema de módulos

<span class="fragment">Un **partial** (`_botones.scss`) no se compila solo: se incorpora a otros.</span>

```scss
// botones.scss
@use "variables" as v;

.btn { background: v.$color-primario; }  // namespace evita colisiones
```

<span class="fragment">`@use`/`@forward` cargan **una sola vez** y sin emitir CSS extra (el legado `@import` duplicaba reglas).</span>

Note: Migrad siempre de `@import` a `@use`: es el sistema de módulos moderno y evita la duplicación de reglas. Los namespaces (`as v`) son lo que hace mantenible un proyecto grande.

---

## Reutilización: `@extend` y `@mixin`

```scss
%enlace-base { color: $color-primario; text-decoration: none; }
a, .btn-enlace { @extend %enlace-base; }   // herencia de selectores

@mixin bp($punto) {
  @media (min-width: map-get($breakpoints, $punto)) { @content; }
}
.hero { @include bp("md") { font-size: 2rem; } }  // mixin con @content
```

<span class="fragment">`@extend` hereda reglas; los **mixins** (con `@content`) generan bloques condicionales → breakpoints DRY (CE 2.i).</span>

Note: Los mixins con `@content` actúan como plantillas que envuelven el código que les pasamos. Es la herramienta central para generar media queries sin repetirlas.

---

## Funciones y control de flujo

```scss
@function rem($px) { @return ($px / 16) * 1rem; }
.titulo { font-size: rem(32); }   // -> 2rem

@for $i from 1 through 6 {
  .p-#{$i} { padding: ($i * 0.25rem); }   // escala de espaciado generada
}
```

<span class="fragment">`@for`, `@each` y `@if` **generan CSS programáticamente** → adiós a escribir decenas de clases a mano.</span>

Note: Esto elimina la escritura manual de patrones repetitivos. Los mapas organizan *design tokens* por categoría (color, tipografía, espaciado) y se consumen con funciones como `token()`.

---

## Arquitectura del proyecto de estilos

<div style="font-size: 0.85rem; text-align: left;">
| Capa | Contenido |
|---|---|
| **settings/** | variables, tokens, configuración global |
| **tools/** | mixins y funciones (no emiten CSS) |
| **generic/** | reset, box-sizing, estilos base |
| **elements/** | selectores de etiquetas HTML |
| **objects/** | patrones de layout reutilizables |
| **components/** | componentes UI concretos |
| **utilities/** | clases utilitarias |
</div>

<span class="fragment">Orden de carga de lo genérico a lo específico (patrones **ITCSS / 7-1**) + `@use` = base de estilos mantenible.</span>

Note: La arquitectura por capas separa responsabilidades y hace predecible el orden de cascada. Es la diferencia entre un `style.css` de 3000 líneas y un proyecto que un equipo puede mantener.

---

## Compilación e integración con Vite

```jsonc
// package.json
{ "scripts": { "dev": "vite", "build": "vite build" } }
```

```scss
// main.scss (punto de entrada)
@use "generic/base";
@use "objects/contenedor";
@use "utilities/espaciado";
```

<span class="fragment">**`dart-sass`** es el compilador oficial (Ruby SASS en desuso). Vite compila al vuelo; en producción emite CSS minificado + source maps.</span>

Note: Activad source maps en desarrollo para depurar el SCSS original en las DevTools. El navegador solo recibe el CSS resultante: cumplido el CE 2.j.

---

## Preprocesador, CSS nativo o Tailwind?

<div style="font-size: 0.85rem; text-align: left;">
| Enfoque | Cuándo |
|---|---|
| **CSS nativo** | El valor cambia en runtime (temas) o el proyecto es pequeño |
| **SASS/SCSS** | Mucha lógica reutilizable (mixins, escalas por bucles, tokens compilados) |
| **Tailwind (U17)** | Velocidad y consistencia con utilidades atómicas |
</div>

<span class="fragment">La elección responde a **escalabilidad, mantenibilidad y productividad** del equipo, no a modas.</span>

Note: Cerrad con esta decisión: muchas bases combinan SASS para tokens/lógica y Custom Properties para temas. El criterio es profesional, no ideológico.

---

## Ejemplo guiado · Tokens + mixins responsive

```scss
$breakpoints: ("sm": 640px, "md": 768px, "lg": 1024px);

@mixin bp($p) { @media (min-width: map-get($breakpoints, $p)) { @content; } }

.contenedor {
  width: min(100% - 2rem, 72rem);
  margin-inline: auto;
  @include bp("md") { width: min(100% - 3rem, 80rem); }
}

@for $i from 1 through 6 { .p-#{$i} { padding: ($i * 0.25rem); } }
```

<span class="fragment">Al compilar: media queries resueltas + clases `.p-1`…`.p-6` generadas → CSS plano para el navegador.</span>

Note: Pedid al aula que inspeccione la hoja de estilos generada y vea que no queda ni rastro de SASS. Eso demuestra el flujo completo del CE 2.j.

---

## Casos reales

- <span class="fragment"><strong>Bootstrap</strong>: desde la v4 construye utilidades, variantes y breakpoints con **SASS** (bucles + mixins).</span>
- <span class="fragment"><strong>Sistemas de diseño corporativos</strong>: definen *design tokens* en SASS/LESS y los distribuyen a varios productos.</span>
- <span class="fragment"><strong>Ant Design</strong>: migró de **LESS** a CSS-in-JS para soportar *theming* dinámico por cliente (white-label).</span>

Note: El caso de Ant Design ilustra que la elección del preprocesador está ligada a necesidades de temas dinámicos. En proyectos legados es habitual encontrar LESS: saber leerlo y migrarlo a SCSS o Tailwind es tarea profesional recurrente.

---

## Actividades propuestas

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 0.6rem; font-size: 0.8rem; text-align: left;">
  <div class="fragment"><strong>1. Refactorización DRY (RA2 / CE 2.j)</strong><br>Convierte una hoja CSS plana (~300 líneas) en SCSS: variables + 3 mixins + función `rem()`. Documenta el ahorro.</div>
  <div class="fragment"><strong>2. Generación por bucles</strong><br>Genera con `@for`/`@each` las utilidades de espaciado y un sistema de columnas `.col-1`…`.col-12`. Verifica en el navegador.</div>
  <div class="fragment"><strong>3. Arquitectura 7-1</strong><br>Reorganiza el proyecto en settings/tools/generic/elements/objects/components/utilities con `@use` y explica el orden de carga.</div>
  <div class="fragment"><strong>Ampliación: comparativa</strong><br>Implementa la misma tarjeta 3 veces (CSS nativo, SASS, Tailwind) e informa sobre productividad y tamaño final del CSS.</div>
</div>

Note: La actividad 1 es la que evalúa directamente el CE 2.j. En la ampliación, el informe debe comparar no solo el código sino el CSS emitido en producción.

---

## Buenas prácticas

- <span class="fragment">✅ Usa **SCSS** (`.scss`) salvo que el proyecto exija `.sass`</span>
- <span class="fragment">✅ Prefiere `@use`/`@forward` sobre el legado `@import`</span>
- <span class="fragment">✅ Limita la anidación a 2-3 niveles (no inflar la especificidad)</span>
- <span class="fragment">✅ Centraliza valores en **tokens** (mapas) y consúmelos con funciones</span>
- <span class="fragment">✅ Genera patrones repetitivos con **bucles**, no a mano</span>

Note: Cinco hábitos que separan un `style.css` inmanejable de una base de estilos profesional. El más subestimado es el de los tokens: si un valor se repite, va a un token.

---

## Errores frecuentes

- <span class="fragment">❌ **Anidar en exceso**: selectores muy específicos difíciles de sobrescribir</span>
- <span class="fragment">❌ **`@import` en vez de `@use`**: duplica reglas y rompe el sistema de módulos</span>
- <span class="fragment">❌ **Compilar con Ruby SASS** (en desuso): usa `dart-sass`</span>
- <span class="fragment">❌ **Confundir variables SASS con Custom Properties**: las primeras no cambian en runtime</span>
- <span class="fragment">❌ **Olvidar el `&`** al anidar pseudoestados (`&:hover`)</span>

Note: El error más caro es usar variables SASS para un tema que debe cambiar en runtime: no funciona porque se resuelve en compilación. Para eso, Custom Properties.

---

## Resumen · Conceptos clave

- <span class="fragment">🎯 **Preprocesador** = CSS + variables, mixins, funciones, bucles → DRY y escalable</span>
- <span class="fragment">🎯 **SCSS**: sintaxis con llaves; `&` padre; `#{}` interpolación; `!default`</span>
- <span class="fragment">🎯 **Módulos**: partials `_` + `@use`/`@forward` (nada de `@import`)</span>
- <span class="fragment">🎯 **Mixins** con `@content` → breakpoints DRY · **funciones** y **bucles** generan clases</span>
- <span class="fragment">🎯 **Arquitectura 7-1/ITCSS** + orden de carga predecible</span>
- <span class="fragment">🎯 **`dart-sass` + Vite**: compila a CSS estándar (CE 2.j)</span>
- <span class="fragment">🎯 Decisión: SASS vs CSS nativo vs Tailwind según escalabilidad y productividad</span>

Note: Repaso rápido: si solo recordáis una cosa, es que el preprocesador trabaja en compilación y el navegador solo ve CSS. Todo lo demás (mixins, tokens, arquitectura) está al servicio de bases de estilos mantenibles.

---

## Recursos clave

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 0.6rem; font-size: 0.75rem; text-align: left;">
  <div><strong>Documentación</strong><br>SASS (dart-sass) · sass-lang.com/documentation<br>Guía de módulos `@use`/`@forward`<br>Vite + SASS · vitejs.dev/guide/features<br>LESS · lesscss.org</div>
  <div><strong>Arquitecturas y herramientas</strong><br>ITCSS · itcss.io<br>7-1 Architecture · 71project.com<br>Dart Sass (compilador oficial)<br>SassMe / CodePen para probar SCSS en vivo</div>
</div>

Note: La documentación oficial de SASS es la referencia; para experimentar sin montar proyecto, CodePen soporta SCSS directamente. ITCSS y 7-1 son las dos arquitecturas de referencia para organizar hojas de estilo grandes.

---

## Próximos pasos

<span class="fragment">A continuación: **Unidad 19 · Marco legal del contenido multimedia**</span>

- <span class="fragment">Derecho de autor y **licencias** (Creative Commons, dominio público)</span>
- <span class="fragment">Fuentes legales de imágenes, audio, vídeo e iconos + registro de activos (RA3 / CE 3.a)</span>
- <span class="fragment"><strong>Proyecto final integrador:</strong> aplicar lo aprendido en las unidades 01-19</span>

<div style="font-size: 0.8rem; text-align: left;"><span class="mini">Antes de la próxima sesión: traed una base de estilos CSS plana para refactorizar a SCSS (actividad 1).</span></div>

Note: La próxima unidad aporta la dimensión legal que el RA3 exige: qué puedes y no puedes usar en cuanto a contenido multimedia. Pedid que traigan un CSS plano para practicar la refactorización DRY.

---

## ¿Preguntas?

Unidad 18 · Preprocesadores CSS (SASS/SCSS y LESS)

0615 · DAW · Curso 2025/2026

Note: Cierre de la unidad. Recoged dudas sobre `@use`, mixins y compilación con Vite antes de pasar al marco legal del contenido multimedia.
