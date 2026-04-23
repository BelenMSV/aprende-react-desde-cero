# 04 - El Futuro del Enrutamiento: Loaders y Actions 🚀

Hasta ahora, el ciclo de vida de una pantalla en React era el siguiente:
1. El usuario hace clic en el enlace `/perfil`.
2. React Router cambia la URL y dibuja el componente `<Perfil />`.
3. El componente se da cuenta de que no tiene datos y muestra un `Spinner`.
4. El componente hace un *fetch* a la base de datos.
5. Los datos llegan y la pantalla por fin muestra la información.

A esto se le llama **"Render-then-Fetch"** (Dibujar y luego pedir). Causa parpadeos y hace que la aplicación se sienta lenta. 

React Router v6.4+ introdujo el patrón **"Fetch-then-Render"** (Pedir y luego dibujar) utilizando dos herramientas mágicas: **Loaders** (para leer datos) y **Actions** (para enviar datos).

---

## 📥 1. Loaders: Preparando el terreno

Un `loader` es simplemente una función asíncrona que se ejecuta **antes** de que tu componente se dibuje en la pantalla. React Router pausará la navegación, ejecutará el loader, y solo cuando los datos estén listos, dibujará el componente.

### Paso 1: Crear el Loader y el Componente
En el mismo archivo de tu componente, exportamos una función `loader`.

```jsx
// src/pages/PerfilUsuario.jsx
import { useLoaderData } from 'react-router-dom';

// 1. EL LOADER: Se ejecuta ANTES de que el componente exista
export const perfilLoader = async ({ params }) => {
  // params contiene las variables de la URL (ej. el ID del usuario)
  const respuesta = await fetch(`https://api.ejemplo.com/usuarios/${params.id}`);
  
  if (!respuesta.ok) {
    throw new Error("No se pudo cargar el perfil"); // Esto lo atrapará el ErrorBoundary
  }
  
  return respuesta.json(); // Devolvemos los datos limpios
};

// 2. EL COMPONENTE: Ya no necesita useEffect ni Spinners
export const PerfilUsuario = () => {
  // useLoaderData nos entrega los datos que el loader ya preparó
  const usuario = useLoaderData(); 

  return (
    <div>
      <h1>Perfil de {usuario.nombre}</h1>
      <p>Email: {usuario.email}</p>
      {/* ¡Mira qué limpio! Cero estados de carga, cero errores. */}
    </div>
  );
};
```

### Paso 2: Conectar el Loader en el Enrutador
Ahora debemos decirle a nuestro `createBrowserRouter` que este componente tiene un cargador de datos asociado.

```jsx
// src/main.jsx
import { PerfilUsuario, perfilLoader } from './pages/PerfilUsuario';

const router = createBrowserRouter([
  {
    path: '/usuarios/:id',
    element: <PerfilUsuario />,
    loader: perfilLoader, // 🔗 ¡Conectamos la función aquí!
  }
]);
```

---

## 📤 2. Actions: Formularios sin dolor de cabeza

Si los `loaders` son para pedir datos (GET), los `actions` son para enviar datos (POST, PUT, DELETE). 

React Router revivió la forma en que funcionaba el HTML en los años 90, pero con superpoderes de JavaScript. Nos da un componente `<Form>` (con F mayúscula). Al usarlo, React Router intercepta el envío, previene la recarga de la página, y le pasa los datos a tu función `action`.

```jsx
// src/pages/CrearUsuario.jsx
import { Form, redirect } from 'react-router-dom';

// 1. EL ACTION: Se ejecuta cuando el usuario envía el formulario
export const crearUsuarioAction = async ({ request }) => {
  // Extraemos los datos del formulario nativo de HTML
  const formData = await request.formData();
  const datos = Object.fromEntries(formData);

  // Enviamos a la API
  await fetch('[https://api.ejemplo.com/usuarios](https://api.ejemplo.com/usuarios)', {
    method: 'POST',
    body: JSON.stringify(datos),
  });

  // Redirigimos automáticamente al inicio tras el éxito
  return redirect('/'); 
};

// 2. EL COMPONENTE: Un formulario HTML puro
export const CrearUsuario = () => {
  return (
    // IMPORTANTE: Usar <Form> de react-router-dom, no el <form> minúsculo
    <Form method="post">
      <input type="text" name="nombre" placeholder="Tu nombre" required />
      <input type="email" name="email" placeholder="Tu email" required />
      <button type="submit">Guardar</button>
    </Form>
  );
};
```
*Y por supuesto, debes conectar `crearUsuarioAction` en tu `main.jsx` usando la propiedad `action: crearUsuarioAction`.*

---

## ⏳ 3. Mejorando la UX con `useNavigation`

Con los Loaders, el componente no se dibuja hasta que los datos llegan. Esto es genial para evitar componentes parpadeando, pero crea un nuevo problema: si el internet es lento, el usuario hará clic en un enlace y **parecerá que la aplicación se congeló**, porque la URL no cambiará hasta que el loader termine.

Para solucionar esto, React Router nos da el hook `useNavigation`. Nos permite saber si hay un loader trabajando en segundo plano en cualquier parte de la app.

Lo ideal es poner esto en tu Cascarón Principal (Layout):

```jsx
// src/layouts/AppLayout.jsx
import { Outlet, Link, useNavigation } from 'react-router-dom';

export const AppLayout = () => {
  const navigation = useNavigation();
  
  // 'loading' significa que un loader está buscando datos para la siguiente ruta
  const estaCargando = navigation.state === 'loading';

  return (
    <div>
      <nav>
        <Link to="/">Inicio</Link>
        <Link to="/usuarios/123">Perfil</Link>
      </nav>

      {/* Una barra de progreso global (como la que tiene YouTube arriba del todo) */}
      {estaCargando && <div className="barra-de-carga-global">Cargando...</div>}

      <main style={{ opacity: estaCargando ? 0.5 : 1 }}>
        <Outlet />
      </main>
    </div>
  );
};
```

---

### 🏆 Resumen: ¿Por qué esto te hace mejor programador?
Al usar Loaders y Actions, estás eliminando decenas de `useStates`, `useEffect` y manejadores de eventos (`e.preventDefault()`). Estás separando la **lógica de negocio** (hablar con la API) de la **lógica visual** (pintar divs en pantalla). Esto es exactamente lo que buscan los equipos técnicos de alto nivel.