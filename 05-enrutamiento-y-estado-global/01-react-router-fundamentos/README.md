# 01 - Fundamentos de React Router v6

En HTML tradicional, si quieres ir de `inicio.html` a `contacto.html`, usas una etiqueta `<a href="contacto.html">`. Esto provoca que el navegador destruya la página actual, parpadee en blanco y descargue la nueva página desde el servidor. En React, esto es un crimen: recargar la página destruye todo nuestro estado (`useState`), nuestro caché (TanStack Query) y arruina la Experiencia de Usuario.

Para navegar sin recargar la página, la industria utiliza **React Router DOM**.

## 📦 Instalación

Asegúrate de instalar la versión orientada a la web (`dom`):
```bash
npm install react-router-dom
```

---

## 🏗️ 1. La Arquitectura Moderna: `createBrowserRouter`

Si ves tutoriales antiguos (incluso de la primera versión de v6), verás que usan un componente llamado `<BrowserRouter>`. Aunque sigue funcionando, **ya no es el estándar de la industria**.

A partir de la versión 6.4, React Router introdujo los enrutadores basados en objetos (`createBrowserRouter`). Esta es la forma oficial y obligatoria si en el futuro quieres usar características avanzadas de Arquitectura (como *Loaders* o *Actions*).

Configuramos esto en la raíz de nuestra aplicación, normalmente en `main.jsx`:

```jsx
// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Importamos nuestras "Pantallas" (Componentes)
import { PaginaInicio } from './pages/PaginaInicio';
import { PaginaContacto } from './pages/PaginaContacto';
import { PaginaError } from './pages/PaginaError';

// 1. Definimos el árbol de rutas como un Array de Objetos
const enrutador = createBrowserRouter([
  {
    path: '/', // La URL raíz (ej. [midominio.com/](https://midominio.com/))
    element: <PaginaInicio />,
    errorElement: <PaginaError />, // Atrapa errores y páginas 404
  },
  {
    path: '/contacto', // (ej. [midominio.com/contacto](https://midominio.com/contacto))
    element: <PaginaContacto />,
  }
]);

// 2. Inyectamos el enrutador en la aplicación
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={enrutador} />
  </React.StrictMode>
);
```

*(Nota Pro: Observa la propiedad `errorElement`. Si el usuario escribe una URL que no existe, como `/rutafalsa`, React Router no romperá la aplicación, sino que mostrará automáticamente el componente `<PaginaError />`).*

---

## 🔗 2. Navegación Declarativa (`<Link>`)

Como dijimos, usar etiquetas `<a>` está prohibido internamente porque provocan recargas completas (Full Page Reload). React Router nos da el componente `<Link>`.

Bajo el capó, `<Link>` renderiza una etiqueta `<a>` (para que el SEO y la accesibilidad funcionen perfectamente), pero **intercepta el clic** y hace el cambio de vista usando JavaScript.

```jsx
// src/pages/PaginaInicio.jsx
import { Link } from 'react-router-dom';

export const PaginaInicio = () => {
  return (
    <div>
      <h1>Bienvenido a la App</h1>
      <nav>
        {/* ❌ ESTO ESTÁ PROHIBIDO (Recarga la página y borra el estado) */}
        {/* <a href="/contacto">Ir a Contacto</a> */}

        {/* ✅ LA FORMA CORRECTA (Navegación instantánea SPA) */}
        <Link to="/contacto">Ir a Contacto</Link>
      </nav>
    </div>
  );
};
```

---

## 🚀 3. Navegación Programática (`useNavigate`)

A veces no queremos navegar porque el usuario hizo clic en un enlace explícito, sino **como resultado de una acción en nuestro código**. 

Por ejemplo: El usuario llena un formulario de inicio de sesión, hacemos un *fetch* al servidor, el servidor responde "OK", y entonces queremos redirigirlo al "Panel de Control".

Para esto usamos el hook `useNavigate`:

```jsx
// src/pages/PaginaLogin.jsx
import { useNavigate } from 'react-router-dom';

export const PaginaLogin = () => {
  // 1. Instanciamos el hook
  const navigate = useNavigate();

  const manejarLogin = async (e) => {
    e.preventDefault();
    
    // Simulamos que enviamos datos y el login es exitoso
    const exito = await simularLlamadaAPI();

    if (exito) {
      // 2. Redirigimos al usuario por código
      navigate('/dashboard');
      
      // Tip Senior: Si quieres que el usuario NO pueda usar el botón 
      // "Atrás" del navegador para volver al login, usa replace:
      // navigate('/dashboard', { replace: true });
    }
  };

  return (
    <form onSubmit={manejarLogin}>
      <h2>Iniciar Sesión</h2>
      <button type="submit">Entrar</button>
    </form>
  );
};
```

---

## 🛑 4. El Manejo de Errores 404 (Not Found)

Gracias a que configuramos el `errorElement` en nuestro enrutador principal, construir una página de error 404 es extremadamente sencillo. 

React Router nos provee el hook `useRouteError` para saber exactamente por qué caímos en esta pantalla (útil para saber si fue un 404 porque la URL no existe, o un 500 porque un componente explotó).

```jsx
// src/pages/PaginaError.jsx
import { useRouteError, Link } from 'react-router-dom';

export const PaginaError = () => {
  // Obtenemos los detalles del error
  const error = useRouteError();
  console.error(error);

  return (
    <div className="pantalla-error">
      <h1>¡Ups! Ruta no encontrada.</h1>
      <p>Lo sentimos, ha ocurrido un error inesperado.</p>
      
      {/* Mostramos el texto del error técnico si existe */}
      <p><i>{error.statusText || error.message}</i></p>

      {/* Le damos una salida segura al usuario */}
      <Link to="/">Volver al Inicio</Link>
    </div>
  );
};
```