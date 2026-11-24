# Guía Práctica de Sintaxis CSS (Temas Críticos)

Esta guía va directo al grano: omite definiciones teóricas básicas y se centra en **cómo escribir la sintaxis, qué hace cada valor y cuándo usar** las propiedades que presentan mayor dificultad en la práctica.

---

## 1. Box Model Real y Comportamiento de Cajas (`box-sizing` y `display`)

### 1.1 `box-sizing: border-box` (Evitar que el `padding` infle tus cajas)

Por defecto, CSS usa `content-box`, donde el `padding` y el `border` **se suman** al `width` declarado (`200px` de ancho + `20px` de padding por lado = `240px` reales). Con `border-box`, el `padding` y el `border` **se incluyen dentro** del `width` declarado.

```css
/* Regla recomendada al inicio de todo archivo CSS */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.card {
  width: 300px;
  padding: 24px;
  border: 2px solid #cbd5e1;
  /* Con border-box, el ancho total en pantalla sigue siendo exactamente 300px */
}
```

### 1.2 `display: block` vs `inline` vs `inline-block`

Muchos errores ocurren al intentar ponerle `width`, `height` o `margin-top` a un `<span>` o a un `<a>` (`display: inline` por defecto).

| Valor de `display` | ¿Salta a nueva línea? | ¿Acepta `width` y `height`? | ¿Acepta `margin` y `padding` verticales (`top`/`bottom`)? | Caso de uso típico |
| :--- | :---: | :---: | :---: | :--- |
| `block` (`div`, `p`, `section`) | ✅ Sí (ocupa el 100% disponible) | ✅ Sí | ✅ Sí | Contenedores, secciones, tarjetas |
| `inline` (`span`, `a`, `strong`) | ❌ No (fluye con el texto) | ❌ **No (los ignora)** | ❌ **No empuja verticalmente** | Fragmentos de texto dentro de un párrafo |
| `inline-block` | ❌ No (se pone uno al lado del otro) | ✅ **Sí** | ✅ **Sí** | Botones, etiquetas (`badges`), píldoras (`pills`) |

```css
/* Convertir un enlace <a> en un botón con dimensiones reales */
.boton {
  display: inline-block;
  padding: 12px 24px;
  min-width: 140px;
  text-align: center;
}
```

### 1.3 Control de límites: `min-width`, `max-width`, `min-height`, `max-height`

En lugar de fijar tamaños rígidos en `px` que se rompen en pantallas pequeñas o cuando hay más texto, combina medidas fluidas con límites:

```css
.contenedor {
  width: 90%;          /* Ocupa el 90% de la pantalla en móviles */
  max-width: 1100px;   /* Pero nunca crece más allá de 1100px en monitores grandes */
  margin: 0 auto;      /* Lo centra horizontalmente */
}

.banner {
  min-height: 400px;   /* Mide al menos 400px, pero si el texto crece, se estira sin desbordar */
}
```

---

## 2. Medidas Relativas, Viewport y `calc()`

### 2.1 Tabla comparativa de unidades (`%`, `em`, `rem`, `vw`, `vh`)

| Unidad | ¿Relativa a qué? | Ejemplo práctico | ¿Cuándo utilizarla? |
| :---: | :--- | :--- | :--- |
| **`rem`** | Tamaño de fuente de la etiqueta raíz `<html>` (por defecto `1rem = 16px`). | `1.5rem` = `24px`<br>`0.875rem` = `14px` | **Tipografía (`font-size`), `padding`, `margin` y `gap`**. Mantiene proporción global sin multiplicarse en cascada. |
| **`em`** | Tamaño de fuente del **propio elemento** (o del padre si se usa en `font-size`). | `padding: 0.5em 1.2em;` en un botón. | Cuando quieres que el `padding` de un botón o icono crezca proporcionalmente si cambias el `font-size` de ese botón. |
| **`%`** | Tamaño del **elemento padre directo**. | `width: 50%;` | Anchos fluidos dentro de un contenedor padre. *(Ojo: `height: 100%` sólo funciona si el padre tiene altura definida).* |
| **`vw`** | **1% del ancho de la ventana** (Viewport Width). | `100vw` = ancho total de ventana. | Elementos a ancho completo de pantalla o tipografía fluida. |
| **`vh`** | **1% del alto de la ventana** (Viewport Height). | `100vh` = alto total de la pantalla visible. | Portadas (`Hero sections`), pantallas de login centradas (`min-height: 100vh`). |

```css
html {
  font-size: 16px; /* Base: 1rem = 16px */
}

.hero-fullscreen {
  min-height: 100vh;     /* Alto completo de la pantalla del usuario */
  padding: 2rem;         /* 32px de espacio interno */
}

.hero-titulo {
  font-size: 2.25rem;    /* 36px constantes, sin importar cuántos divs padres tenga */
}
```

### 2.2 La función `calc()`

Permite mezclar unidades distintas (por ejemplo, porcentaje o `vh` menos píxeles o `rem`).

> [!WARNING]
> **Regla de sintaxis obligatoria:** En `calc()`, **siempre debe haber un espacio en blanco antes y después** de los signos `+` y `-`.
> - ❌ Incorrecto: `calc(100vh-64px)`
> - ✅ Correcto: `calc(100vh - 64px)`

```css
/* Ejemplo 1: Sección principal que ocupa toda la pantalla menos los 64px de la barra de navegación */
.main-screen {
  min-height: calc(100vh - 64px);
}

/* Ejemplo 2: 3 columnas con separación exacta */
.columna {
  width: calc((100% - 2rem) / 3);
}
```

---

## 3. Imágenes (`<img>`) y Fondos (`background-*`)

### 3.1 Etiqueta `<img>`: `object-fit`, `object-position` y `aspect-ratio`

Cuando fuerzas un `width` y un `height` en una etiqueta `<img>`, la foto se aplasta o estira. **`object-fit`** soluciona esto comportándose como un recorte inteligente dentro del marco de la imagen:

```css
.card-img {
  width: 100%;
  height: 220px;           /* O bien: aspect-ratio: 16 / 9; sin fijar height */
  object-fit: cover;       /* Llena el marco y recorta sobrantes SIN deformar la foto */
  object-position: center; /* Elige qué zona priorizar: center | top | bottom | 50% 20% */
  display: block;          /* Elimina el pequeño espacio blanco inferior de las imágenes inline */
}
```

- **`object-fit: cover`**: Llena toda la caja sin deformar (recorta los bordes que sobren). **Ideal para fotos de tarjetas, avatares y banners.**
- **`object-fit: contain`**: Muestra la imagen completa sin recortar nada, dejando espacio vacío si la proporción no coincide. **Ideal para logotipos de marcas o fotos de productos con fondo limpio.**
- **`aspect-ratio: 16 / 9`** (o `1 / 1` para cuadrado, `4 / 3`, `3 / 4`): Define la proporción ancho/alto automáticamente sin necesidad de calcular píxeles de altura.

```css
.avatar-cuadrado {
  width: 80px;
  aspect-ratio: 1 / 1;     /* Calcula height: 80px automáticamente */
  object-fit: cover;
  border-radius: 50%;      /* Círculo perfecto */
}
```

### 3.2 Imágenes de Fondo en Contenedores (`background-*`)

Usa `background-image` cuando necesites **escribir texto o colocar contenido HTML encima de la imagen** (por ejemplo, en un banner Hero o tarjeta con texto superpuesto).

```css
.hero-banner {
  /* Puedes combinar un degradado oscuro semitransparente + la imagen para que el texto blanco sea legible */
  background-image:
    linear-gradient(rgba(15, 23, 42, 0.65), rgba(15, 23, 42, 0.65)),
    url("paisaje.jpg");

  background-size: cover;        /* cover: cubre toda la caja | contain: cabe entera */
  background-position: center;   /* Centra el punto focal de la imagen */
  background-repeat: no-repeat;  /* Evita que la imagen se repita como mosaico */
  min-height: 360px;
}
```

---

## 4. Flexbox: Alineación y Distribución en 1 Dimensión

Flexbox se activa poniendo `display: flex` en el **contenedor padre**. A partir de ese momento, sus **hijos directos** se convierten en *Flex Items*.

### 4.1 Los dos ejes de Flexbox (`Main Axis` vs `Cross Axis`)

Todo en Flexbox gira alrededor de qué dirección tiene `flex-direction`:

| `flex-direction` | Eje Principal (`Main Axis`) → controlado por `justify-content` | Eje Transversal (`Cross Axis`) → controlado por `align-items` |
| :--- | :--- | :--- |
| **`row`** *(por defecto)* | **Horizontal (→)** (de izquierda a derecha) | **Vertical (↓)** (de arriba hacia abajo) |
| **`column`** | **Vertical (↓)** (de arriba hacia abajo) | **Horizontal (→)** (de izquierda a derecha) |

### 4.2 Propiedades del Contenedor Padre (`Flex Container`)

```css
.flex-container {
  display: flex;

  /* 1. Dirección del eje principal */
  flex-direction: row;           /* row | column | row-reverse | column-reverse */

  /* 2. ¿Qué pasa si los hijos no caben en una sola línea? */
  flex-wrap: wrap;               /* nowrap (por defecto, los aplasta) | wrap (saltan de línea) */

  /* 3. Distribución en el EJE PRINCIPAL (Main Axis) */
  justify-content: space-between;
  /* Valores clave:
     - flex-start    : agrupados al inicio
     - center        : agrupados al centro
     - flex-end      : agrupados al final
     - space-between : primer hijo al inicio, último al final, espacio igual entre ellos
     - space-around  : espacio alrededor de cada hijo
     - space-evenly  : espacio exactamente idéntico entre todos y los bordes */

  /* 4. Alineación en el EJE TRANSVERSAL (Cross Axis) de cada línea */
  align-items: center;
  /* Valores clave:
     - stretch       : (por defecto) estira los hijos para igualar la altura del más alto
     - center        : centrados verticalmente (cuando es row)
     - flex-start    : alineados arriba
     - flex-end      : alineados abajo */

  /* 5. Separación limpia entre hijos (sin usar márgenes manuales) */
  gap: 1.5rem;                   /* o row-gap: 1rem; column-gap: 2rem; */
}
```

> [!TIP]
> **Centrado perfecto absoluto en 3 líneas:**
> ```css
> .centrado {
>   display: flex;
>   justify-content: center;
>   align-items: center;
> }
> ```

### 4.3 Propiedades de los Elementos Hijos (`Flex Items`)

Estas propiedades **no se ponen en el padre**, sino directamente en las tarjetas o elementos hijos:

```css
.flex-item {
  /* Tamaño base ideal antes de repartir el espacio sobrante */
  flex-basis: 240px;

  /* ¿Cuánto puede CRECER si sobra espacio en la fila? (0 = no crece, 1 = crece equitativamente) */
  flex-grow: 1;

  /* ¿Cuánto puede ENCOGERSE si falta espacio? (1 = se encoge, 0 = prohibido encogerse) */
  flex-shrink: 1;

  /* ATAJO RECOMENDADO (flex: grow shrink basis) */
  flex: 1 1 240px; /* Crece, se encoge y mide mínimo/base 240px */
}

/* Atajo rápido para que todas las columnas midan exactamente lo mismo: */
.columna-igual {
  flex: 1; /* Equivalente a flex: 1 1 0% */
}

/* Desalinear UN solo hijo respecto a sus hermanos en el eje transversal: */
.item-destacado {
  align-self: flex-start; /* stretch | center | flex-start | flex-end */
  order: -1;              /* Por defecto todos tienen order: 0. Un valor menor lo mueve al principio */
}
```

---

## 5. CSS Grid: Maquetación en 2 Dimensiones (Filas y Columnas)

Mientras Flexbox ordena elementos en una fila o columna a la vez, **CSS Grid** crea una cuadrícula bidimensional donde controlas columnas y filas simultáneamente.

### 5.1 Crear la Cuadrícula en el Padre (`display: grid`, `fr`, `repeat`, `minmax`)

- **La unidad `fr` (fracción):** Representa una fracción del espacio libre disponible en el contenedor Grid.
  - `grid-template-columns: 1fr 1fr 1fr;` → 3 columnas iguales (33.3% cada una, descontando el `gap` automáticamente).
  - `grid-template-columns: 240px 1fr;` → Barra lateral fija de `240px` y contenido principal que ocupa todo el resto (`1fr`).

```css
.grid-container {
  display: grid;

  /* Forma explícita con repeat(cantidad, tamaño) */
  grid-template-columns: repeat(3, 1fr);

  /* Altura de las filas (opcional, o automática según contenido) */
  grid-template-rows: 180px 180px;

  /* Separación entre filas y columnas */
  gap: 1.25rem;
}
```

### 5.2 El patrón maestro Responsive con `repeat()` + `minmax()`

Si quieres que las tarjetas se acomoden solas en 1, 2, 3 o 4 columnas según el ancho de pantalla **sin escribir Media Queries**:

```css
.galeria-responsive {
  display: grid;
  /* Crea tantas columnas como quepan; cada una mide mínimo 240px y máximo 1fr */
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}
```

### 5.3 Expandir Elementos Hijos por Varias Columnas o Filas (`grid-column`, `grid-row`, `span`)

En CSS Grid, un hijo puede ocupar más de una celda usando **`span N`** (expandirse N celdas) o indicando las líneas de inicio y fin (`inicio / fin`):

```css
/* Hacer que una tarjeta ocupe 2 columnas de ancho */
.card-ancha {
  grid-column: span 2;
}

/* Hacer que una tarjeta ocupe 2 filas de alto */
.card-alta {
  grid-row: span 2;
}

/* Hacer que un Header o Footer ocupe TODAS las columnas de lado a lado */
.header,
.footer {
  grid-column: 1 / -1; /* Desde la línea 1 hasta la última línea (-1) */
}
```

### 5.4 Alineación dentro de las Celdas de Grid

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* Alinea los hijos DENTRO de cada celda: start | center | end | stretch */
  justify-items: stretch; /* Horizontal dentro de su celda */
  align-items: center;    /* Vertical dentro de su celda */
}
```

---

## 6. Responsive Design (`@media`) y Pseudoclases (`:hover`, `:focus`)

### 6.1 Sintaxis de Media Queries (`min-width` vs `max-width`)

Una Media Query aplica un bloque de reglas CSS **sólo cuando la pantalla cumple la condición**.

- **`@media (min-width: 768px)` (Enfoque Mobile First):**
  Se lee como *"A partir de 768px hacia arriba (tablets y escritorio), aplica estos estilos"*.
- **`@media (max-width: 768px)` (Enfoque Desktop First):**
  Se lee como *"Hasta un máximo de 768px hacia abajo (móviles), aplica estos estilos"*.

```css
/* Diseño base (Móvil: 1 sola columna) */
.tarjetas {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

/* A partir de 768px (Tablet / Laptop): pasa a 3 columnas */
@media (min-width: 768px) {
  .tarjetas {
    grid-template-columns: repeat(3, 1fr);
    gap: 1.5rem;
  }
}
```

### 6.2 Estados Interactivos: `:hover` y `:focus`

- **`:hover`**: Se activa cuando el usuario pasa el cursor del mouse por encima del elemento.
- **`:focus`**: Se activa cuando el usuario hace clic o navega con la tecla `Tab` hacia un campo de formulario (`<input>`, `<textarea>`) o botón.

```css
.boton {
  background-color: #2563eb;
  color: #ffffff;
  transition: background-color 0.2s ease, transform 0.2s ease;
}

/* Al pasar el cursor por encima */
.boton:hover {
  background-color: #1d4ed8;
  transform: translateY(-2px);
}

/* Cuando el usuario está escribiendo dentro de un input */
.input-texto:focus {
  outline: 2px solid #2563eb;
  border-color: transparent;
}
```
