# 05 - TanStack Query: El Estándar de la Industria 🚀

Si recuerdas el Tema 3, para hacer una simple petición HTTP robusta tuvimos que crear 3 estados (`data`, `loading`, `error`), usar un `useEffect`, instanciar un `AbortController` y manejar excepciones. Escribimos casi 40 líneas de código.

**TanStack Query** elimina todo ese código. Es un gestor de "Estado del Servidor" que se encarga automáticamente del caché, las sincronizaciones en segundo plano, los re-intentos si la red falla y la deduplicación de peticiones.

## 📦 Instalación

Instalaremos la librería principal y sus DevTools (una herramienta visual para ver el caché en tiempo real).

```bash
npm install @tanstack/react-query @tanstack/react-query-devtools
```

---

## 🏗️ 1. Configuración Global (El Proveedor)

Al igual que muchas herramientas globales en React, TanStack Query necesita envolver nuestra aplicación en un "Proveedor" (`QueryClientProvider`). Esto suele hacerse en el archivo principal (`main.jsx` o `App.jsx`).

```jsx
// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import App from './App';

// 1. Creamos el cliente que guardará el caché de toda la app
const queryClient = new QueryClient();

ReactDOM.createRoot(document.getElementById('root')).render(
  <QueryClientProvider client={queryClient}>
    <App />
    {/* 2. Añadimos las DevTools (Solo serán visibles en desarrollo) */}
    <ReactQueryDevtools initialIsOpen={false} />
  </QueryClientProvider>
);
```

---

## ⚡ 2. Tu primer `useQuery`

Para pedir datos, usamos el hook `useQuery`. Este hook necesita obligatoriamente dos cosas:
1. `queryKey`: Un Array que actúa como el **nombre único** de esta petición en el caché.
2. `queryFn`: Una función asíncrona que devuelva los datos (¡Aquí usamos nuestra capa de servicios de Axios del Tema 2!).

Mira cómo se reduce el componente:

```jsx
import { useQuery } from '@tanstack/react-query';
import { UsuariosService } from '../api/servicios/usuarios.service';

export const ListaUsuarios = () => {
  // TanStack Query nos da los estados empaquetados y listos para usar
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['usuarios'], // El identificador en el caché
    queryFn: UsuariosService.obtenerTodos, // La promesa que trae los datos
  });

  // Manejo de UI declarativo
  if (isLoading) return <h2>Cargando desde la API... ⏳</h2>;
  if (isError) return <h2>Error: {error.message} ❌</h2>;

  return (
    <ul>
      {data.map(usuario => <li key={usuario.id}>{usuario.nombre}</li>)}
    </ul>
  );
};
```
**¡Adiós `useEffect`! ¡Adiós `useState`!** Tu componente ahora solo se preocupa por mostrar la información.

---

## 🪄 3. La Magia del Caché (¿Por qué es nivel Senior?)

Si usas el componente de arriba, y el usuario navega a otra pantalla y luego vuelve a la lista de usuarios... **no verá un spinner de carga**. Los datos aparecerán instantáneamente.

¿Por qué? Porque TanStack Query guardó los datos bajo la llave `['usuarios']` en la memoria caché.
El ciclo de vida interno es el siguiente:
1. El usuario vuelve al componente.
2. TanStack Query dice: *"Tengo datos en el caché para `['usuarios']`, muéstralos inmediatamente"* (Pantalla instantánea).
3. En segundo plano, de forma invisible, hace un *fetch* silencioso a la API.
4. Si hay datos nuevos en la base de datos, actualiza la pantalla en tiempo real sin molestar al usuario.

### Parámetros de Optimización Críticos

Por defecto, TanStack Query considera que tus datos son "Obsoletos" (*Stale*) inmediatamente (0 milisegundos). Puedes cambiar esto para ahorrarle peticiones a tu servidor:

```jsx
const { data } = useQuery({
  queryKey: ['usuarios'],
  queryFn: UsuariosService.obtenerTodos,
  // staleTime: El tiempo en milisegundos que los datos se consideran "Frescos".
  // Durante este tiempo, la librería NO hará background fetching.
  staleTime: 1000 * 60 * 5, // 5 minutos de frescura garantizada
});
```

---

## 🔑 4. Arrays de Dependencias Dinámicos (El poder del `queryKey`)

¿Recuerdas cómo en `useEffect` poníamos `[userId]` en el arreglo de dependencias para que se volviera a ejecutar si el ID cambiaba?

En TanStack Query, el `queryKey` actúa exactamente igual. Si añades variables al array del `queryKey`, el caché se aislará para cada combinación y la petición se re-disparará sola cuando la variable cambie.

```jsx
export const PerfilDetalle = ({ userId }) => {
  const { data, isLoading } = useQuery({
    // El caché creará cajas distintas: ['usuario', 1], ['usuario', 2], etc.
    queryKey: ['usuario', userId], 
    
    // Le pasamos el ID a nuestro servicio de Axios
    queryFn: () => UsuariosService.obtenerPorId(userId), 
    
    // Podemos decirle que NO haga la petición si no hay un ID válido
    enabled: !!userId 
  });

  if (isLoading) return <p>Cargando...</p>;
  return <div>{data?.nombre}</div>;
};
```

---

## 🥇 Resumen de Superpoderes

Al usar `useQuery`, obtienes gratis:
* **Background Refetching:** Se actualiza solo si el usuario cambia de pestaña en el navegador y vuelve.
* **Auto-Retries:** Si el WiFi parpadea y la petición falla, lo reintentará silenciosamente 3 veces antes de mostrar el `isError`.
* **Deduplicación:** Si 5 componentes hijos piden `useQuery({ queryKey: ['usuarios'] })` al mismo tiempo, la librería es inteligente y solo hará **UNA** petición HTTP, repartiendo el resultado a los 5.
* **Caché Global:** Los datos viven fuera de tus componentes, accesibles instantáneamente.