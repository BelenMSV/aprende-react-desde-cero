# 01 - Introducción al Estado y `useState`

Hasta ahora, hemos construido componentes que reciben datos mediante `props` y los dibujan en la pantalla. Pero las interfaces reales son interactivas: el usuario hace clics, escribe en formularios y abre menús.

Para lograr esto, nuestros componentes necesitan **"memoria"**. Necesitan recordar cosas a lo largo del tiempo (ej. "el usuario abrió el menú", "el contador va por 5"). A esta memoria de los componentes la llamamos **Estado** (*State*).

---

## 🚫 El problema de las variables normales

Si vienes de JavaScript puro (Vanilla JS), tu instinto para hacer un contador podría ser crear una variable `let` y sumarle 1 al hacer clic.

Veamos qué pasa si intentamos eso en React:

```jsx
// ❌ ESTO NO FUNCIONA COMO ESPERAS
export const ContadorRoto = () => {
  let numero = 0;

  const incrementar = () => {
    numero = numero + 1;
    console.log("El número ahora es:", numero);
  };

  return (
    <div>
      <h2>Contador: {numero}</h2>
      <button onClick={incrementar}>Sumar 1</button>
    </div>
  );
};
```

Si ejecutas esto y miras la consola, verás que la variable `numero` sí está sumando. Sin embargo, **la pantalla sigue mostrando un 0 estático**.

**¿Por qué?**
1. React no "escucha" a las variables locales.
2. Incluso si lo supiera, cuando un componente termina de renderizarse, sus variables locales se destruyen.

Necesitamos una herramienta que le avise a React: *"¡Oye, este dato ha cambiado, vuelve a dibujar este componente para mostrar la nueva información!"*.

---

## ⚡ La solución: El Hook `useState`

Para solucionar esto, React nos da nuestro primer **Hook**. Los Hooks son funciones especiales de React (todas empiezan con `use`) que te permiten "engancharte" a sus características internas.

El Hook `useState` hace dos cosas:
1. Retiene el dato entre cada renderizado.
2. Nos da una función especial que, al ejecutarse, **fuerza a React a re-renderizar** el componente con el nuevo dato.

### La Sintaxis Mágica (Desestructuración)

```jsx
import { useState } from 'react';

// const [variable, funcionActualizadora] = useState(valorInicial);
```

Usamos la **desestructuración de Arrays** de JavaScript. Por convención, la función actualizadora siempre empieza con `set`.

```jsx
export const ContadorArreglado = () => {
  const [numero, setNumero] = useState(0);

  const incrementar = () => {
    setNumero(numero + 1); 
  };

  return (
    <div className="tarjeta">
      <h2>Contador: {numero}</h2>
      <button onClick={incrementar}>Sumar 1</button>
    </div>
  );
};
```

---

## 🌙 Variables Booleanas (Dark Mode)

El estado puede guardar cualquier tipo de dato. Un caso clásico es usar un booleano para encender o apagar algo:

```jsx
export const InterruptorTema = () => {
  const [esModoOscuro, setEsModoOscuro] = useState(false);

  return (
    <div className={esModoOscuro ? 'dark-theme' : 'light-theme'}>
      <h1>{esModoOscuro ? 'Noche' : 'Día'}</h1>
      <button onClick={() => setEsModoOscuro(!esModoOscuro)}>
        Cambiar Tema
      </button>
    </div>
  );
};
```

---

## ⚠️ La trampa del `console.log` (Pregunta de Entrevista)

El error más común de un desarrollador Junior es pensar que `setNumero` cambia la variable inmediatamente.

```jsx
const [numero, setNumero] = useState(0);

const hacerClic = () => {
  setNumero(5);
  console.log(numero); // 💥 Imprime 0, ¡NO 5!
};
```
**¿Por qué imprime el valor viejo?**
Las actualizaciones de estado en React no son inmediatas. Piensa en `setNumero(5)` como si estuvieras haciendo un **pedido** a React: *"Para el próximo renderizado, quiero que numero valga 5"*. Pero en la línea de código actual, `numero` sigue valiendo lo que valía al principio del renderizado. 

A esto se le conoce como un comportamiento **por lotes (Batching)**. React agrupa las actualizaciones para ganar rendimiento.

---

## 🚀 Pro Tip: Inicialización Perezosa (Lazy Initialization)

El valor que pones dentro de `useState(valorInicial)` se usa solo la primera vez que el componente se carga. Sin embargo, React sigue evaluando ese código en cada renderizado, aunque luego lo ignore.

Si tu valor inicial requiere un cálculo pesado (como leer el disco duro con `localStorage`), arruinarás el rendimiento:

```jsx
// ❌ MAL: Lee el disco duro en CADA renderizado
const [tema, setTema] = useState(localStorage.getItem('temaPreferido'));
```

**La solución:** Pasa una **función** al `useState` en lugar del valor directo. A esto se le llama "Lazy Initialization". React solo ejecutará esa función la primera vez.

```jsx
// ✅ BIEN: Solo lee el disco duro la primera vez que carga
const [tema, setTema] = useState(() => localStorage.getItem('temaPreferido'));
```

---

## 🧠 Reglas de Oro del Estado

1. **El Estado es Privado:** Si pones dos `<Contador />` en la pantalla, cada uno tiene su propio estado aislado.
2. **El Estado es Inmutable:** NUNCA modifiques la variable directamente (`numero = 5`). Usa siempre el `setNumero(5)`.
3. **El Estado es una "Foto":** El valor del estado no cambia durante un ciclo de renderizado en curso, se actualiza para el siguiente ciclo.