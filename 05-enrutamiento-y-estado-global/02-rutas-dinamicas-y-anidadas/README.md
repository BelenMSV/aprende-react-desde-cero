# 02 - Arquitectura de Layouts: Rutas Dinámicas y Anidadas

En el mundo real, la mayoría de las aplicaciones comparten una estructura común (un "cascarón"). Piensa en Twitter o YouTube: la barra lateral y la barra de navegación superior nunca desaparecen; solo cambia el contenido del centro.

En React Router v6, logramos esto utilizando el poderoso componente `<Outlet>`.

---

## 🏗️ 1. El Patrón Layout y el `<Outlet>`

Un **Layout** (Diseño/Plantilla) es un componente que envuelve a otros componentes. El `<Outlet>` actúa como un "agujero" o "marcador de posición" donde React Router inyectará la pantalla que corresponda a la URL actual.

Vamos a crear nuestro cascarón principal:

```jsx
// src/layouts/AppLayout.jsx
import { Outlet, Link } from 'react-router-dom';

export const AppLayout = () => {
  return (
    <div className="layout-principal">
      {/* 1. Este Header es fijo. Se renderizará UNA SOLA VEZ */}
      <header>
        <h2>Mi Empresa</h2>
        <nav>
          <Link to="/">Inicio</Link>
          <Link to="/usuarios">Usuarios</Link>
        </nav>
      </header>

      {/* 2. El Outlet es el agujero dinámico. 
          Aquí se inyectará <PaginaInicio> o <PaginaUsuarios> */}
      <main className="contenido-dinamico">
        <Outlet /> 
      </main>

      {/* 3. Este Footer también es fijo */}
      <footer>© 2024 Todos los derechos reservados</footer>
    </div>
  );
};
```

---

## 🗺️ 2. Configurando las Rutas Anidadas (`children`)

Para que el `<Outlet>` sepa qué mostrar, debemos actualizar nuestro enrutador (`createBrowserRouter`) utilizando la propiedad `children`. 

Le decimos a React Router: *"El `AppLayout` es el padre. Dependiendo de la URL, inyecta a uno de sus hijos en su Outlet"*.

```jsx
// src/main.jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { AppLayout } from './layouts/AppLayout';
import { PaginaInicio } from './pages/PaginaInicio';
import { ListaUsuarios } from './pages/ListaUsuarios';
import { PaginaError } from './pages/PaginaError';

const enrutador = createBrowserRouter([
  {
    path: '/',
    element: <AppLayout />, // El Cascarón Padre
    errorElement: <PaginaError />,
    children: [
      {
        index: true, // Ruta por defecto (cuando la URL es exactamente '/')
        element: <PaginaInicio />,
      },
      {
        path: 'usuarios', // La URL será '/usuarios'
        element: <ListaUsuarios />,
      }
    ]
  }
]);
```
*¡Magia! Ahora puedes tener 50 pantallas y nunca más tendrás que importar el Navbar manualmente.*

---

## 🪪 3. Rutas Dinámicas (URL Parameters)

A menudo necesitamos URLs que cambian dependiendo del dato que queremos ver. Por ejemplo, ver el perfil del usuario 123 (`/usuarios/123`) o del usuario 456 (`/usuarios/456`).

Para capturar ese número dinámico, usamos los **Parámetros de URL** declarándolos con dos puntos (`:`) en nuestra configuración de rutas.

```jsx
// Añadimos esta ruta dentro del arreglo 'children' de arriba:
{
  path: 'usuarios/:id', // Los dos puntos indican que 'id' es una variable
  element: <PerfilUsuario />,
}
```

### Leyendo el Parámetro con `useParams`

Para extraer ese `123` de la URL y usarlo (por ejemplo, para hacer un *fetch* a la base de datos), React Router nos da el hook `useParams`.

```jsx
// src/pages/PerfilUsuario.jsx
import { useParams } from 'react-router-dom';
import { useQuery } from '@tanstack/react-query';

export const PerfilUsuario = () => {
  // 1. Extraemos el 'id' exactamente con el mismo nombre que usamos en la ruta
  const { id } = useParams();

  // 2. Usamos el ID para pedir los datos (Patrón Nivel Senior)
  const { data, isLoading } = useQuery({
    queryKey: ['usuario', id],
    queryFn: () => fetch(`/api/usuarios/${id}`).then(res => res.json())
  });

  if (isLoading) return <p>Cargando perfil...</p>;

  return (
    <div>
      <h1>Perfil de {data.nombre}</h1>
      <p>ID de Empleado: {id}</p>
    </div>
  );
};
```

---

## 🎨 4. UX Avanzada: Enlaces Activos (`<NavLink>`)

Si el usuario está en la página `/usuarios`, el enlace "Usuarios" del menú lateral debería estar resaltado (en negrita o de otro color) para que sepa dónde está.

Hacer esto manualmente comparando URLs es tedioso. React Router nos ofrece una versión mejorada de `<Link>` llamada **`<NavLink>`**. Este componente detecta automáticamente si su ruta coincide con la URL actual y nos permite inyectar estilos condicionales.

```jsx
import { NavLink } from 'react-router-dom';

export const MenuLateral = () => {
  return (
    <nav>
      {/* isActive es inyectado automáticamente por NavLink */}
      <NavLink 
        to="/dashboard"
        className={({ isActive }) => isActive ? 'enlace-activo' : 'enlace-normal'}
      >
        Panel de Control
      </NavLink>

      <NavLink 
        to="/usuarios"
        className={({ isActive }) => isActive ? 'enlace-activo' : 'enlace-normal'}
      >
        Usuarios
      </NavLink>
    </nav>
  );
};
```
*(Nota: Si usas Tailwind CSS, esto se vuelve aún más limpio: `className={({ isActive }) => isActive ? 'font-bold text-blue-500' : 'text-gray-500'}`).*