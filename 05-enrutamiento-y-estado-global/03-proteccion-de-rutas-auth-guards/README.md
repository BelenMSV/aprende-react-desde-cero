# 03 - Protección de Rutas: Auth Guards

En el tema anterior aprendimos a crear rutas anidadas. Ahora aprenderemos a poner "guardaespaldas" a esas rutas. Un **Auth Guard** es un componente de lógica que decide si permite al usuario ver una ruta o si lo redirige a otra parte (usualmente al Login).

---

## 🛡️ 1. El Componente `<Navigate />`

Para proteger rutas de forma declarativa, React Router nos ofrece el componente `<Navigate />`. A diferencia de `useNavigate` (que es un hook para funciones), `<Navigate />` es un componente que, al ser renderizado, dispara una redirección inmediata.

```jsx
// Si el usuario no tiene permiso, lo mandamos al login
if (!usuarioAutenticado) {
  return <Navigate to="/login" replace />;
}
```
*(Nota: Usamos `replace` para que el usuario no pueda volver atrás con el botón del navegador y quedar atrapado en un bucle).*

---

## 🏗️ 2. Creando el Componente Protector (Ruta Privada)

La forma más profesional y escalable de proteger rutas es crear un componente envoltorio (Wrapper) llamado `ProtectedRoute`. Este componente actuará como un filtro.

```jsx
// src/components/ProtectedRoute.jsx
import { Navigate, Outlet } from 'react-router-dom';

export const ProtectedRoute = ({ isAllowed, redirectTo = "/login", children }) => {
  // 1. Si no está permitido, redirigimos
  if (!isAllowed) {
    return <Navigate to={redirectTo} replace />;
  }

  // 2. Si está permitido, mostramos los hijos (children) o el Outlet (si es anidada)
  return children ? children : <Outlet />;
};
```

---

## 🗺️ 3. Implementación en el Enrutador

Ahora, en nuestro `createBrowserRouter`, envolvemos las rutas que queremos proteger dentro de nuestro `ProtectedRoute`. 

Imagina que tenemos un estado de usuario (que luego aprenderemos a manejar con Context o Zustand):

```jsx
const user = { loggedIn: false, role: 'user' };

const router = createBrowserRouter([
  {
    path: "/",
    element: <AppLayout />,
    children: [
      { index: true, element: <Home /> },
      
      // 🔒 RUTAS PROTEGIDAS (Solo para logueados)
      {
        element: <ProtectedRoute isAllowed={user.loggedIn} />,
        children: [
          { path: "dashboard", element: <Dashboard /> },
          { path: "perfil", element: <Perfil /> },
        ]
      },

      // 👑 RUTAS DE ADMINISTRADOR (Filtro por Rol)
      {
        path: "admin",
        element: (
          <ProtectedRoute 
            isAllowed={user.loggedIn && user.role === 'admin'} 
            redirectTo="/dashboard" 
          />
        ),
        children: [
          { index: true, element: <PanelAdmin /> }
        ]
      }
    ]
  },
  { path: "/login", element: <Login /> }
]);
```

---

## 🚀 4. Nivel Arquitecto: ¿De dónde viene la verdad?

En este ejemplo, usamos una variable `user` estática, pero en la vida real, el estado de autenticación debe persistir. Un error de principiante es guardar el estado solo en un `useState`. Si el usuario recarga la página, el estado se borra y la app lo redirigirá al Login aunque ya estuviera logueado.

**Estrategia Senior:**
1. **Verificar Token:** Al arrancar la app, revisamos si existe un token en `localStorage`.
2. **Estado Global:** Cargamos ese usuario en un Contexto o en una Store de Zustand (lo veremos en el Tema 5).
3. **Sincronización:** El `ProtectedRoute` escuchará ese estado global. Si el token expira (detectado por un interceptor de Axios, como vimos en el Módulo 4), el estado cambiará a `false` y el Guard lo expulsará automáticamente de la ruta privada.

### Beneficios de este patrón:
* **Centralización:** Toda la lógica de seguridad de la navegación está en un solo componente.
* **Flexibilidad:** Puedes proteger por login, por roles, por suscripciones premium o por cualquier condición booleana.
* **Legibilidad:** Al mirar el enrutador, queda clarísimo qué partes de la app son públicas y cuáles no.