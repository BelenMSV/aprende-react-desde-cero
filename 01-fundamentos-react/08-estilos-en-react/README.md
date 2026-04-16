# 08 - Manejo de Estilos en React

React no es un framework dogmático, lo que significa que no te obliga a usar una forma específica para dar estilo a tus componentes. Como desarrollador, debes conocer los métodos nativos y las herramientas modernas que dominan la industria.

---

## 1. CSS Global (Hoja de estilos tradicional)

Es la forma más parecida a la maquetación web clásica. Creas un archivo `.css` y lo importas en tu componente.

```jsx
import '../styles/Boton.css';

export const Boton = () => {
  return <button className="btn-primario">Aceptar</button>;
};
```
- **Peligro:** En React, todo el CSS global se inyecta en el `<head>` del documento. Si dos componentes usan la clase `.tarjeta`, los estilos colisionarán. Es difícil de escalar.

---

## 2. Estilos en Línea (Inline Styles)

El atributo `style` en React no acepta un string, acepta un **objeto de JavaScript**. Las propiedades con guiones pasan a `camelCase` (`backgroundColor`).

```jsx
export const Alerta = ({ tipo }) => {
  return (
    <div style={{ 
      backgroundColor: tipo === 'error' ? 'red' : 'green',
      borderRadius: '8px'
    }}>
      ¡Operación completada!
    </div>
  );
};
```
- **Ventaja:** Excelente para estilos altamente dinámicos calculados al vuelo.
- **Desventaja:** No soportan `:hover` ni *Media Queries*. El JSX se ensucia rápido.

---

## 3. CSS Modules (El estándar nativo) 🏆

Vite soporta "CSS Modules" por defecto. Añades `.module.css` a tu archivo y el compilador genera **nombres de clase únicos**, garantizando que nunca haya colisiones.

```css
/* Tarjeta.module.css */
.contenedor { border: 1px solid gray; padding: 20px; }
```

```jsx
import styles from './Tarjeta.module.css'; 

export const Tarjeta = () => {
  return <article className={styles.contenedor}>Contenido</article>;
};
```

---

## 🌪️ 4. Tailwind CSS (El Rey de la Industria)

Cuando entres a trabajar en un proyecto real moderno, es muy probable que no escribas CSS a mano. Lo más seguro es que uses **Tailwind CSS**.

Tailwind es un framework "Utility-First". En lugar de crear clases semánticas (como `.tarjeta-usuario`), aplicas clases pequeñas que hacen una sola cosa directamente en el JSX.

```jsx
// Un botón estilizado con Tailwind CSS puro
export const BotonTailwind = () => {
  return (
    <button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg shadow-md transition-colors">
      Enviar
    </button>
  );
};
```
**¿Por qué es tan popular?**
- Escribes la interfaz muchísimo más rápido sin salir del archivo `.jsx`.
- Nunca hay colisiones de clases.
- El archivo CSS final en producción es enano (solo compila las clases que has usado).

---

## 🌼 5. El Siguiente Nivel: Tailwind + daisyUI

El único "problema" de Tailwind puro es que tu HTML puede llenarse de clases larguísimas (como en el botón de arriba). Para solucionar esto, los profesionales usan librerías de componentes. La más querida en el ecosistema de Tailwind es **daisyUI**.

daisyUI es un plugin para Tailwind que añade clases semánticas de alto nivel. Bajo el capó sigue siendo Tailwind, pero te ahorra escribir 15 clases distintas.

```jsx
// El MISMO botón de arriba, pero usando daisyUI
export const BotonDaisy = () => {
  return (
    <button className="btn btn-primary shadow-md">
      Enviar
    </button>
  );
};
```

### El poder de la semántica con daisyUI
Además de botones, te permite crear componentes complejos en segundos usando su sistema de clases:

```jsx
export const TarjetaProducto = () => {
  return (
    <div className="card w-96 bg-base-100 shadow-xl">
      <figure><img src="zapatillas.jpg" alt="Zapatillas" /></figure>
      <div className="card-body">
        <h2 className="card-title">Zapatillas Nike!</h2>
        <p>Ideales para correr por la ciudad.</p>
        <div className="card-actions justify-end">
          <button className="btn btn-primary">Comprar Ahora</button>
        </div>
      </div>
    </div>
  );
};
```
---

## 🛠️ 6. Nivel Senior: Clases Dinámicas y la utilidad `cn`

Cuando empiezas a usar Tailwind o CSS Modules, rápidamente te encuentras con la necesidad de aplicar clases de forma condicional (Ej: Un botón que cambia de color si está deshabilitado).

❌ **El problema (Concatenar strings a mano):**
```jsx
// Feo, propenso a errores y difícil de leer
<button className={`btn ${isActive ? 'bg-blue-500' : 'bg-gray-300'} ${isLarge ? 'text-lg p-4' : 'text-sm p-2'}`}>
```

Para solucionar esto de forma profesional, la industria utiliza dos librerías clave que suelen combinarse en una utilidad maestra:

### 1. `clsx` (o `classnames`)
Es una mini-librería que te permite construir cadenas de clases condicionales usando objetos de JavaScript. Si el valor es `true`, aplica la clase; si es `false`, la ignora.

```jsx
import clsx from 'clsx';

// Mucho más limpio
<button className={clsx(
  'btn', 
  {
    'bg-blue-500': isActive,
    'bg-gray-300': !isActive,
    'text-lg p-4': isLarge
  }
)}>
```

### 2. `tailwind-merge` (El salvador de Tailwind)
En Tailwind, si le pasas a un elemento dos clases que hacen lo mismo (ej. `p-4 p-8`), el resultado es **impredecible** por cómo funciona el motor de CSS. `tailwind-merge` lee tus clases, detecta los conflictos y se asegura de que la última clase que escribas siempre gane de forma segura.

### 🏆 El Estándar de la Industria: La función `cn`

Hoy en día, en proyectos avanzados (y en librerías famosas como *shadcn/ui*), combinamos ambas herramientas creando una función de utilidad global llamada `cn` (por *classNames*).

Normalmente se crea un archivo `src/utils/cn.js`:

```javascript
// src/utils/cn.js
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

**¿Cómo se usa en tus componentes?**
¡Es magia pura! Te permite mezclar strings normales, condicionales lógicos y asegurarte de que Tailwind nunca se rompa, todo al mismo tiempo.

```jsx
import { cn } from '../utils/cn';

export const TarjetaAlerta = ({ isError, classNameProps }) => {
  return (
    <div className={cn(
      "p-4 rounded-md shadow-md bg-white border-2", // Clases base fijas
      isError ? "border-red-500 text-red-700" : "border-green-500 text-green-700", // Clases condicionales
      classNameProps // Clases extra que vengan del componente Padre (sobreescribirán a las anteriores si hay conflicto gracias a twMerge)
    )}>
      Mensaje del sistema
    </div>
  );
};
```
Aplicar este patrón desde el principio hará que tu código sea infinitamente más mantenible y te preparará para arquitecturas de nivel empresarial.


**💡 Resumen del Ecosistema:**
1. Aprende **CSS nativo** para entender cómo funciona la web.
2. Usa **CSS Modules** si quieres mantener tu CSS en archivos separados sin colisiones.
3. Domina **Tailwind CSS + daisyUI** si quieres velocidad, eficiencia y el estándar actual del mercado laboral.