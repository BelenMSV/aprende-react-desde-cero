# 08 - Custom Hooks y las Reglas de Oro

A medida que tu aplicación crece, te darás cuenta de que escribes la misma lógica una y otra vez. Por ejemplo: abrir y cerrar modales, leer datos del servidor o escuchar el tamaño de la ventana.

En JavaScript clásico, cuando repites código, creas una función. En React, cuando repites lógica que involucra estado o efectos, creas un **Custom Hook**.

---

## ⚖️ Las 2 Reglas de Oro de los Hooks

Antes de escribir tu primer Custom Hook, debes entender las dos leyes absolutas de React. Si las rompes, tu aplicación colapsará.

### Regla 1: Llama a los Hooks siempre en el Nivel Superior
NUNCA llames a un Hook dentro de condicionales (`if`), bucles (`for`, `map`), o funciones anidadas.

❌ **El Anti-patrón:**
```jsx
export const Perfil = ({ id }) => {
  // 🚩 ERROR: React explotará si `id` cambia a nulo
  if (id !== null) {
    const [datos, setDatos] = useState(null);
  }
  return <div>...</div>;
};
```

**🧠 ¿Por qué existe esta regla?** React no sabe cómo se llaman tus variables. Internamente, guarda tus Hooks en un arreglo oculto `[hook1, hook2]`. Se basa **estrictamente en el orden de ejecución** para saber qué estado pertenece a qué `useState`. Si metes un Hook en un `if` y ese `if` no se ejecuta, el orden se rompe y los datos se mezclan.

### Regla 2: Llama a los Hooks solo desde funciones de React
Solo puedes llamar Hooks desde:
1. Componentes de React (funciones en Mayúscula).
2. Custom Hooks (funciones que empiezan por `use`).

---

## 🛠️ Creando tu primer Custom Hook (`useToggle`)

Un Custom Hook es una función de JavaScript cuyo nombre **empieza por "use"** y que llama a otros Hooks en su interior.

```js
// src/hooks/useToggle.js
import { useState } from 'react';

export const useToggle = (valorInicial = false) => {
  const [estado, setEstado] = useState(valorInicial);

  const toggle = () => setEstado(prev => !prev);
  const abrir = () => setEstado(true);
  const cerrar = () => setEstado(false);

  return { estado, toggle, abrir, cerrar };
};
```

**Cómo usarlo:**
```jsx
import { useToggle } from '../hooks/useToggle';

export const Modal = () => {
  // ¡Mira qué limpio queda el componente!
  const { estado: isOpen, toggle } = useToggle(false);

  return (
    <>
      <button onClick={toggle}>Abrir Modal</button>
      {isOpen && <div className="modal">Soy un modal</div>}
    </>
  );
};
```

---

## 📐 Patrones de Arquitectura: ¿Retornar Array u Objeto?

Cuando creas un Custom Hook, puedes elegir qué retornar. Existe un estándar en la industria para esto:

* **Retorna un Array `[valor, funcion]` si:** Tu hook devuelve exactamente 2 cosas (como `useState`). Esto facilita renombrar las variables al desestructurar.
* **Retorna un Objeto `{ a, b, c }` si:** Tu hook devuelve 3 o más cosas (como nuestro `useToggle`). Si devolvieras un array de 4 posiciones, el desarrollador tendría que recordar el orden exacto de los elementos, lo cual es propenso a errores.

---

## ⚠️ La trampa del Estado Global (Pregunta de Entrevista)

*Pregunta: Si tengo un componente `<Header>` y un `<Footer>` y ambos usan mi hook `useTema()`, ¿si el Header cambia el tema, el Footer se entera?*

**Respuesta: NO.**
Los Custom Hooks **comparten la lógica, NO el estado**. Cada vez que llamas a un Custom Hook dentro de un componente, React crea un espacio de memoria completamente aislado y nuevo para ese componente. 
*(Si necesitas compartir memoria real entre componentes lejanos, necesitas el "Context API", que veremos en el Módulo 4).*

---

## 🚀 El Examen Final: El Mega-Hook `useFetch`

Vamos a unir todo lo que hemos aprendido en el Módulo 2 (Estado, Efectos, Cleanup, AbortController y Custom Hooks) para crear la herramienta definitiva que usarás en casi todos tus proyectos reales.

Este Hook se encargará de pedir datos a cualquier URL, manejar el estado de carga (`loading`), capturar errores y limpiar la memoria si el usuario cancela.

```js
// src/hooks/useFetch.js
import { useState, useEffect } from 'react';

export const useFetch = (url) => {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // 1. Preparamos el controlador de cancelación (Tema 5)
    const controller = new AbortController();
    
    // Reiniciamos los estados al cambiar de URL
    setIsLoading(true);
    setError(null);

    const fetchData = async () => {
      try {
        const res = await fetch(url, { signal: controller.signal });
        if (!res.ok) throw new Error('Error en la petición HTTP');
        
        const json = await res.json();
        setData(json);
      } catch (err) {
        // Ignoramos el error si fue provocado intencionalmente por el Cleanup
        if (err.name === 'AbortError') return;
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };

    fetchData();

    // 2. Función de Limpieza (Cleanup)
    return () => {
      controller.abort();
    };
  }, [url]); // 3. Dependencia: Se re-ejecuta si la URL cambia

  // 4. Retornamos un objeto porque son 3 valores
  return { data, isLoading, error };
};
```

**El resultado en tus componentes:**
Tu código ahora tiene nivel de Senior. Toda la complejidad asíncrona está escondida.

```jsx
import { useFetch } from '../hooks/useFetch';

export const PerfilUsuario = ({ id }) => {
  const { data, isLoading, error } = useFetch(`https://api.ejemplo.com/users/${id}`);

  if (isLoading) return <h2>Cargando usuario...</h2>;
  if (error) return <h2>Hubo un problema: {error}</h2>;

  return <div>Bienvenido, {data.nombre}</div>;
};
```

---

### 🎉 ¡Felicidades! Has completado el Módulo 2.
Has pasado de maquetar pantallas estáticas a dominar la memoria, el ciclo de vida, la mutabilidad y la abstracción de arquitectura en React. Estás listo para enfrentarte a la verdadera interacción de usuario y gestión de eventos en el próximo módulo.