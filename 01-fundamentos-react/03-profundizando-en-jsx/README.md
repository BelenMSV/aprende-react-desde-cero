# 03 - Profundizando en JSX

A primera vista, JSX parece HTML escrito dentro de un archivo JavaScript. Sin embargo, **JSX no es HTML**. Es una extensión de la sintaxis de JavaScript que nos permite escribir marcas de manera declarativa.

Bajo el capó, herramientas como Vite (usando Babel o SWC) traducen todo ese JSX en llamadas a funciones regulares de JavaScript.

```jsx
// Lo que tú escribes:
const elemento = <h1 className="titulo">Hola</h1>;

// Lo que el navegador realmente ejecuta (históricamente):
const elemento = React.createElement('h1', { className: 'titulo' }, 'Hola');
```

Entender que JSX se convierte en objetos de JavaScript es la clave para entender sus reglas.

---

## 🕰️ El "Misterio" del `import React` (React 17+)

Si ves tutoriales o código escrito antes del año 2020, notarás que **absolutamente todos** los archivos que tienen JSX empiezan con esta línea:

```jsx
import React from 'react'; // Obligatorio antes de React 17
```

**¿Por qué antes sí y ahora no?** Como vimos arriba, antes el JSX se traducía literalmente a la función `React.createElement`. Si la palabra `React` no estaba importada en ese archivo, la aplicación fallaba.

A partir de React 17, se introdujo el **Nuevo Transformador de JSX**. Ahora, compiladores modernos como el que usa Vite importan automáticamente las funciones especiales necesarias bajo el capó. Ya no necesitas escribir `import React from 'react'` solo para usar JSX. *(Nota: Sí tendrás que importarlo si vas a usar Hooks, pero lo veremos más adelante).*

---

## 🥇 Las 3 Reglas de Oro de JSX

Si vienes de HTML tradicional, el compilador de React te va a gritar mucho al principio. Para evitarlo, respeta estas tres reglas:

### Regla 1: Devuelve siempre un único elemento raíz

Un componente debe devolver **un solo elemento**. Si intentas devolver dos etiquetas hermanas al mismo nivel, React lanzará un error.

❌ **Incorrecto:**
```jsx
export const MiComponente = () => {
  return (
    <h1>Título</h1>
    <p>Párrafo</p>
  ); // 💥 ERROR: Adjacent JSX elements must be wrapped in an enclosing tag
};
```

✅ **El enfoque Profesional (Usando Fragmentos):**
Si no quieres ensuciar tu HTML final con `<div>` innecesarios (lo cual puede romper tu Flexbox o Grid), usa un **Fragmento** (`<> </>`). Un fragmento agrupa elementos sin dejar rastro en el DOM real.

```jsx
export const MiComponente = () => {
  return (
    <>
      <h1>Título</h1>
      <p>Párrafo</p>
    </>
  );
};
```

### Regla 2: Cierra absolutamente TODAS las etiquetas

En HTML, puedes dejar etiquetas abiertas como `<img>`, `<input>` o `<br>`. En JSX, eso es un delito. **Toda etiqueta debe cerrarse explícitamente.**

✅ **Correcto (Etiquetas de auto-cierre):**
```jsx
<img src="foto.jpg" />
<input type="text" />
<br />
```

### Regla 3: Usa `camelCase` para la mayoría de los atributos

Dado que JSX se transforma en JavaScript, React adopta la convención de JavaScript (`camelCase`) en lugar de la convención de HTML. Además, evita chocar con palabras reservadas de JS.

- `class` pasa a ser `className` (porque `class` es reservada para crear clases en JS).
- `for` pasa a ser `htmlFor` (porque `for` se usa en bucles).
- `onclick` pasa a ser `onClick`.

---

## 🪄 Inyectando JavaScript con `{ }`

La verdadera potencia de JSX es que te permite escapar del marcado e inyectar cualquier **expresión** de JavaScript directamente en la interfaz. Para hacerlo, solo tienes que abrir llaves `{}`.

### 1. Mostrando variables y operaciones
```jsx
const nombre = "Ana";
const urlBase = "[https://miservidor.com](https://miservidor.com)";

return (
  <>
    <h1>Hola, mi nombre es {nombre.toUpperCase()}</h1>
    <p>El doble de 4 es: {4 * 2}</p>
    <img src={`${urlBase}/avatar.png`} alt="Avatar" />
  </>
);
```

### ⚠️ Limitación crucial: Solo Expresiones, NO Declaraciones
Dentro de las llaves `{}` solo puedes poner código que devuelva un valor. **NO puedes poner `if`, `for`, o `let/const`** dentro del JSX. Para los condicionales usaremos ternarios o el operador `&&` (lo veremos a fondo en el Tema 6).

---

## 🧰 Sintaxis del día a día (Pro Tips)

Cuando trabajas en proyectos reales, estas tres prácticas te ahorrarán mucho tiempo y dolores de cabeza:

### 1. Comentarios en JSX
No puedes usar `//` o `` libremente dentro de las etiquetas porque React lo interpretará como texto normal y lo imprimirá en la pantalla. Debes abrir llaves de JS y usar el comentario de bloque:

```jsx
return (
  <div>
    {/* Esto es un comentario correcto en JSX */}
    <h1>Hola</h1>
  </div>
);
```

### 2. Atributos Booleanos implícitos
Si un atributo es `true`, no necesitas escribir `={true}`. Simplemente poner el nombre del atributo es suficiente (igual que en HTML).

```jsx
// ❌ Redundante
<button disabled={true} autoFocus={true}>Enviar</button>

// ✅ Profesional
<button disabled autoFocus>Enviar</button>
```

### 3. El Operador Spread (`...`) para props masivas
A veces tienes un objeto con muchos datos y quieres pasarlos todos como atributos HTML. En lugar de escribirlos uno por uno, puedes "esparcirlos" usando la sintaxis Spread de JavaScript:

```jsx
const configInput = {
  type: "text",
  placeholder: "Escribe tu nombre...",
  maxLength: 50,
  required: true
};

// En lugar de: <input type={configInput.type} placeholder={configInput.placeholder} ... />
// Hacemos esto:
return <input {...configInput} />;
```

---

## 🛡️ JSX y la Seguridad (Protección XSS)

Por defecto, React DOM escapa (limpia) cualquier valor incrustado en JSX antes de renderizarlo. 

```jsx
// Si un hacker intenta inyectar un script en un comentario de tu blog:
const comentarioHacker = "<script>alert('Robando contraseñas...')</script>";

// React lo renderizará literalmente como texto plano, neutralizando el ataque.
return <div>{comentarioHacker}</div>;
```
Esto ayuda a prevenir ataques **XSS (Cross-Site Scripting)** de manera automática, haciendo de React una librería muy segura por defecto.