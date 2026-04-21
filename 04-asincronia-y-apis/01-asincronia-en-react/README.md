# 01 - Asincronía en React: Promesas y `useEffect`

React está diseñado para ser **síncrono y predecible**. Cuando el estado cambia, React calcula la nueva interfaz y la dibuja en la pantalla de forma inmediata.

Sin embargo, el mundo real (la red, las bases de datos, las APIs) es **asíncrono e impredecible**. Una petición puede tardar 10 milisegundos, 5 segundos, o fallar por completo.

En este tema aprenderemos cómo reconciliar el mundo síncrono de React con el mundo asíncrono de JavaScript utilizando el hook `useEffect` sin romper las reglas de la librería.

---

## ❌ 1. El Anti-patrón Mortal: `async` directo en `useEffect`

Este es, sin duda, el error más común que cometen los desarrolladores al empezar a conectar React con APIs.

❌ **El instinto Junior:**
```jsx
import { useEffect, useState } from 'react';

export const PerfilUsuario = () => {
  const [usuario, setUsuario] = useState(null);

  // 🚩 ERROR CRÍTICO: Pasar una función 'async' directamente al useEffect
  useEffect(async () => {
    const respuesta = await fetch('[https://api.ejemplo.com/usuario](https://api.ejemplo.com/usuario)');
    const datos = await respuesta.json();
    setUsuario(datos);
  }, []);

  return <div>...</div>;
};
```

**¿Por qué React prohíbe esto?**
El contrato del hook `useEffect` dicta que su función interna solo puede devolver **una cosa**: una función de limpieza sincrónica (Cleanup Function, vista en el Módulo 2). 

Sin embargo, en JavaScript, cualquier función que lleve la palabra `async` **devuelve automáticamente una Promesa**.
Al pasar un `async` directamente, le estás entregando una Promesa a React en lugar de una función de limpieza. Esto rompe el ciclo de vida, causa fugas de memoria y genera errores extraños en la consola.

---

## ✅ 2. La Solución Profesional: La Función Interna

Para usar `async/await` dentro de un efecto, debemos declarar la función asíncrona *dentro* del `useEffect` y luego invocarla inmediatamente.

```jsx
export const PerfilUsuario = () => {
  const [usuario, setUsuario] = useState(null);

  useEffect(() => {
    // 1. Declaramos la función asíncrona internamente
    const cargarUsuario = async () => {
      try {
        const respuesta = await fetch('[https://api.ejemplo.com/usuario](https://api.ejemplo.com/usuario)');
        const datos = await respuesta.json();
        setUsuario(datos);
      } catch (error) {
        console.error("Error al cargar:", error);
      }
    };

    // 2. La invocamos inmediatamente
    cargarUsuario();

    // 3. (Opcional) Ahora sí podemos devolver nuestra función de limpieza normal
    return () => {
      console.log("Limpiando componente...");
    };
  }, []); // Se ejecuta solo al montar

  return <div>{usuario ? usuario.nombre : 'Cargando...'}</div>;
};
```

*(Nota Pro: También puedes usar una IIFE - Immediately Invoked Function Expression - pero la función interna nombrada es más legible y fácil de depurar en los Stack Traces de errores).*

---

## ⚖️ 3. La Trinidad del Estado (Loading, Data, Error)

Hacer un simple `setUsuario(datos)` no es suficiente para una experiencia de usuario (UX) real. Toda petición HTTP tiene tres fases, y nuestra interfaz debe reflejarlas usando tres estados (o un estado compuesto).

✅ **El Patrón Estándar:**
```jsx
export const ListaArticulos = () => {
  const [datos, setDatos] = useState(null);
  // 1. Empezamos asumiendo que estamos cargando
  const [isLoading, setIsLoading] = useState(true); 
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        // Por si acaso la petición se vuelve a disparar, reseteamos el estado
        setIsLoading(true); 
        setError(null);
        
        const res = await fetch('[https://api.ejemplo.com/articulos](https://api.ejemplo.com/articulos)');
        if (!res.ok) throw new Error(`Error HTTP: ${res.status}`);
        
        const json = await res.json();
        setDatos(json);
      } catch (err) {
        // 2. Si falla, guardamos el error
        setError(err.message); 
      } finally {
        // 3. Ocurra lo que ocurra (éxito o fallo), apagamos el loading
        setIsLoading(false); 
      }
    };

    fetchData();
  }, []);

  // --- Renderizado Condicional Defensivo ---
  if (isLoading) return <h2>Cargando artículos... ⏳</h2>;
  if (error) return <h2 className="error">Hubo un problema: {error} ❌</h2>;
  if (!datos) return null;

  return (
    <ul>
      {datos.map(art => <li key={art.id}>{art.titulo}</li>)}
    </ul>
  );
};
```

---

## 👻 4. La Trampa del "Componente Fantasma" (Unmounted Component)

Imagina el siguiente escenario:
1. El usuario entra a la página "Perfil".
2. React dispara el `fetch` (que tarda 3 segundos).
3. Al segundo 1, el usuario se aburre, hace clic en el menú y **se va a otra página**. El componente "Perfil" es destruido (Desmontado).
4. Al segundo 3, el `fetch` termina y tu código intenta hacer `setUsuario(datos)`.

**El resultado:** React intentará actualizar el estado de un componente que ya no existe en la pantalla. Esto genera un error clásico en la consola:
`Warning: Can't perform a React state update on an unmounted component. This indicates a memory leak.`

### La Solución Clásica: La variable `isMounted`

Para evitar que una promesa "zombie" intente actualizar un estado destruido, usamos una variable booleana dentro del `useEffect` en combinación con la función de limpieza (Cleanup).

```jsx
useEffect(() => {
  // 1. Bandera para saber si el componente sigue vivo
  let isMounted = true; 

  const cargarDatos = async () => {
    const res = await fetch('/api/datos');
    const json = await res.json();
    
    // 2. SOLO actualizamos el estado si el componente sigue montado
    if (isMounted) {
      setDatos(json);
    }
  };

  cargarDatos();

  return () => {
    // 3. Cuando el componente muere, cambiamos la bandera.
    // Esto evita que promesas pendientes actualicen el estado en el futuro.
    isMounted = false; 
  };
}, []);
```

**💡 El futuro es mejor:**
Usar la variable booleana (`isMounted` o `ignore`) arregla el error de React, **pero NO cancela la petición HTTP**. El navegador sigue descargando los datos inútilmente en segundo plano, consumiendo ancho de banda. 

En el **Tema 3**, aprenderemos cómo usar el `AbortController` nativo de JavaScript para literalmente asesinar la petición HTTP en vuelo y ahorrar megas al usuario.