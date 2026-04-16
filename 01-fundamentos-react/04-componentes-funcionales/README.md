# 04 - Componentes Funcionales y Arquitectura

Si JSX es el lenguaje con el que describimos nuestra interfaz, los **Componentes** son los bloques de construcción. Piensa en ellos como piezas de Lego: creas piezas pequeñas e independientes (un botón, un avatar, un formulario) y las unes para construir algo complejo (una página web entera).

En React, un componente es conceptualmente igual a una función de JavaScript: recibe datos de entrada (llamados `props`) y retorna elementos de React (JSX) que describen lo que debería aparecer en la pantalla.

---

## 🏗️ La Anatomía de un Componente

La forma moderna y recomendada de escribir componentes en React es mediante **Componentes Funcionales** (Functional Components). Se pueden escribir usando funciones tradicionales o *Arrow Functions* (funciones flecha).

### 1. Usando Funciones Tradicionales
```jsx
export function Saludo() {
  return <h1>¡Hola, desarrollador!</h1>;
}
```

### 2. Usando Arrow Functions (El estándar más común)
```jsx
export const Saludo = () => {
  return <h1>¡Hola, desarrollador!</h1>;
};
```
*Ambas son perfectamente válidas y su rendimiento es idéntico. La mayoría de proyectos modernos prefieren las arrow functions por su sintaxis concisa.*

---

## 📜 Reglas Estrictas para los Componentes

Para que React reconozca que tu función es un componente y no una función matemática cualquiera, debes cumplir una regla inquebrantable:

**El nombre de la función debe empezar SIEMPRE con mayúscula (PascalCase).**

❌ **Incorrecto (React pensará que es una etiqueta HTML nativa):**
```jsx
const miBoton = () => <button>Click</button>;
```

✅ **Correcto (React sabe que es un componente personalizado):**
```jsx
const MiBoton = () => <button>Click</button>;
```

---

## 🦖 Clases vs Funciones (Un poco de historia)

Si buscas tutoriales antiguos, verás que los componentes se escribían usando Clases (`class MiComponente extends React.Component`). 

Esto se hacía porque antes, solo las clases podían tener "estado" (memoria). Las funciones eran "tontas" y solo servían para pintar datos estáticos. En la versión 16.8, React introdujo los **Hooks**. Los Hooks le dieron superpoderes a las funciones, permitiéndoles hacer todo lo que hacían las clases, pero con un código mucho más limpio.

> **💡 Regla de oro hoy en día:** No escribas nuevos componentes de Clase. Usa siempre Componentes Funcionales.

---

## 📦 Exportaciones: ¿Default o Named?

Cuando creas un componente, necesitas exportarlo para usarlo en otro archivo. Tienes dos formas de hacerlo:

### 1. Exportación por Defecto (Default Export)
Solo puedes tener un `export default` por archivo.
```jsx
// En Button.jsx
const Button = () => <button>Click</button>;
export default Button;

// Al importarlo, ¡puedes ponerle el nombre que quieras!
import BotoncitoMagico from './Button'; 
```
**Problema:** Al poder cambiarle el nombre al importarlo, distintos desarrolladores del equipo podrían llamarlo de formas diferentes, causando confusión.

### 2. Exportación Nombrada (Named Export) - 🏆 Recomendada
Exportas exactamente la variable que creaste.
```jsx
// En Button.jsx
export const Button = () => <button>Click</button>;

// Al importarlo, ESTÁS OBLIGADO a usar su nombre real entre llaves
import { Button } from './Button';
```
**Ventajas:** Obliga a todo el equipo a usar el mismo nombre, el refactoring en los editores de código es automático y previene errores tipográficos.

---

## 🧩 Anidando Componentes

La verdadera magia de React es que los componentes pueden renderizar a otros componentes en su interior.

```jsx
export const Avatar = () => <img src="gato.jpg" alt="Avatar" />;
export const UserName = () => <h3>Gato Cósmico</h3>;

export const UserProfile = () => {
  return (
    <div className="card-perfil">
      <Avatar />
      <UserName />
      <p>Explorador del universo de la caja de cartón.</p>
    </div>
  );
};
```

---

## 📁 Arquitectura y Buenas Prácticas (Mundo Real)

Saber escribir un componente es solo la mitad del trabajo; la otra mitad es saber **dónde guardarlo**. Cuando trabajas en aplicaciones reales con cientos de componentes, la organización lo es todo.

### 1. La regla: Un componente por archivo
Aunque React te permite crear 20 componentes dentro de `App.jsx`, **no lo hagas**. Salvo que sea un micro-componente (como un pequeño icono que solo se usa dentro de ese mismo archivo), cada componente principal debe tener su propio archivo dedicado.
- **Por qué:** Facilita la lectura, permite que varios desarrolladores trabajen en distintos componentes sin causar conflictos en Git (Merge Conflicts) y hace que los tests sean más fáciles de escribir.

### 2. El Patrón de Carpetas y Barrels (`index.js`)
En proyectos profesionales, no verás los archivos sueltos en la carpeta `src/components`. Verás que cada componente tiene **su propia carpeta**. Esto permite guardar junto al componente sus propios estilos o tests.

```text
src/
 ┗ components/
   ┗ Button/
     ┣ Button.jsx        (El componente lógico)
     ┣ Button.module.css (Sus estilos aislados)
     ┗ index.js          (El archivo "Barrel")
```

**¿Para qué sirve el `index.js`?**
Imagina que desde otro lugar quieres importar el botón. Sin el `index.js`, la ruta se vería redundante:
```jsx
// Feo y redundante ❌
import { Button } from './components/Button/Button';
```

El archivo `index.js` actúa como un "barril" (Barrel) o puerta de enlace. Su único contenido es:
```javascript
// Dentro de src/components/Button/index.js
export * from './Button';
```

Gracias a esto, cuando importas el componente desde otra parte de la aplicación, NodeJS buscará automáticamente el `index.js` de la carpeta, dejándote una importación limpia y profesional:
```jsx
// Limpio y Profesional ✅
import { Button } from './components/Button';
```