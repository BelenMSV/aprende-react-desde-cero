# 06 - Zustand: El Estándar Moderno de Estado Global 🐻

Zustand es una librería de manejo de estado global que elimina toda la complejidad de Redux y los problemas de rendimiento de Context API. 

¿Su filosofía? Usar Hooks. Si sabes usar `useState`, ya sabes usar Zustand.

## 📦 Instalación

```bash
npm install zustand
```

---

## 🏗️ 1. Tu Primera Store (Tienda de Datos)

En Zustand, no necesitas envolver tu aplicación con `<Providers>` gigantescos (como hacíamos en Context). Simplemente creas un archivo, defines tu estado y tus funciones para modificarlo, y lo exportas como un Hook.

Vamos a crear una "Store" para manejar el estado de autenticación y el perfil de un usuario.

```javascript
// src/store/useAuthStore.js
import { create } from 'react';

// 'create' recibe una función que nos da el método 'set' para actualizar el estado
export const useAuthStore = create((set) => ({
  // 1. EL ESTADO (Las variables)
  usuario: null,
  isLoggedIn: false,

  // 2. LAS ACCIONES (Las funciones que modifican el estado)
  iniciarSesion: (nombre) => {
    // 'set' actualiza el estado. ¡Zustand une (mergea) el objeto automáticamente!
    set({ usuario: nombre, isLoggedIn: true });
  },

  cerrarSesion: () => {
    set({ usuario: null, isLoggedIn: false });
  }
}));
```
¡Eso es todo! Ya tienes un estado global listo para usarse en cualquier parte de tu aplicación. Ni Reducers, ni Actions, ni Providers.

---

## 🔌 2. Consumiendo la Store en los Componentes

Ahora vamos a usar nuestro nuevo hook `useAuthStore` en dos componentes que están muy lejos el uno del otro en el árbol de React.

**Componente A: El Navbar (Muestra el estado)**
```jsx
// src/components/Navbar.jsx
import { useAuthStore } from '../store/useAuthStore';

export const Navbar = () => {
  // Extraemos las variables que necesitamos
  const isLoggedIn = useAuthStore((state) => state.isLoggedIn);
  const usuario = useAuthStore((state) => state.usuario);

  return (
    <nav>
      {isLoggedIn ? <span>Hola, {usuario}</span> : <span>Invitado</span>}
    </nav>
  );
};
```

**Componente B: El Login (Modifica el estado)**
```jsx
// src/pages/Login.jsx
import { useAuthStore } from '../store/useAuthStore';

export const Login = () => {
  // Extraemos SOLO la función que necesitamos
  const iniciarSesion = useAuthStore((state) => state.iniciarSesion);

  return (
    <button onClick={() => iniciarSesion("Alex Pro")}>
      Entrar a la App
    </button>
  );
};
```

---

## 🚀 3. El Secreto Senior: Los Selectores y el Rendimiento

Presta mucha atención al código de arriba. Notarás que dentro del hook pasamos una pequeña función: `(state) => state.isLoggedIn`. 

A esto se le llama **Selector**. Le estás diciendo a Zustand: *"Oye, a este componente **SOLO** le importa la variable `isLoggedIn`"*.

**¿Por qué esto te convierte en Arquitecto?**
Imagina que añadimos un "contador de clics" a nuestro store. Si el contador cambia de 1 a 100, **el Navbar no se volverá a renderizar**. Zustand es lo suficientemente inteligente para saber que el Navbar solo está "suscrito" a `isLoggedIn` y `usuario`. 

Con Context API, el Navbar se habría renderizado 100 veces de forma innecesaria. ¡Por esto Zustand es el rey del rendimiento!

---

## ⏳ 4. Acciones Asíncronas (Peticiones a APIs)

En Redux, para hacer una petición asíncrona (como un *fetch*), tenías que instalar librerías extrañas como Redux Thunk o Redux Saga. 

En Zustand, la asincronía es nativa y natural. Simplemente pones `async/await` en tu acción.

```javascript
// src/store/useUsuariosStore.js
import { create } from 'zustand';

export const useUsuariosStore = create((set) => ({
  usuarios: [],
  isLoading: false,
  error: null,

  // ACCIÓN ASÍNCRONA
  cargarUsuarios: async () => {
    // 1. Encendemos el estado de carga
    set({ isLoading: true, error: null });

    try {
      // 2. Hacemos la petición
      const res = await fetch('[https://api.ejemplo.com/usuarios](https://api.ejemplo.com/usuarios)');
      const data = await res.json();
      
      // 3. Guardamos los datos
      set({ usuarios: data, isLoading: false });
    } catch (error) {
      // 4. Manejamos el error
      set({ error: error.message, isLoading: false });
    }
  }
}));
```

Para usarlo en un componente, simplemente lo llamas con un `useEffect`:

```jsx
import { useEffect } from 'react';
import { useUsuariosStore } from '../store/useUsuariosStore';

export const ListaAsincrona = () => {
  const { usuarios, isLoading, cargarUsuarios } = useUsuariosStore((state) => ({
    usuarios: state.usuarios,
    isLoading: state.isLoading,
    cargarUsuarios: state.cargarUsuarios
  }));

  useEffect(() => {
    cargarUsuarios();
  }, [cargarUsuarios]);

  if (isLoading) return <p>Cargando...</p>;
  return <div>{usuarios.map(u => <p key={u.id}>{u.nombre}</p>)}</div>;
};
```

---

## 🏆 Resumen: ¿Cuándo usar qué?

Como profesional, tu cinturón de herramientas para el estado queda así:

1. **`useState` / `useReducer`:** Para estado local que solo le importa a un componente (ej. abrir un acordeón, escribir en un input).
2. **Context API:** Para datos globales estáticos que casi nunca cambian (ej. El tema Claro/Oscuro o el idioma de la app).
3. **Zustand:** Para estado global complejo, asíncrono o de alta frecuencia de actualización (ej. Carrito de compras, estado del reproductor de música, sesión del usuario).
4. **TanStack Query (React Query):** Exclusivamente para sincronizar datos con tu base de datos externa (Estado del Servidor).