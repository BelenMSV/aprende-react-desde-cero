# 02 - Arquitectura de Servicios con Axios

Aunque la API nativa `fetch()` ha mejorado mucho, **Axios** sigue siendo el rey indiscutible en aplicaciones empresariales React. Axios transforma automáticamente los datos a JSON, maneja los errores HTTP de forma lógica (cayendo directamente en el `catch`) y nos da superpoderes de configuración.

**Instalación:**
```bash
npm install axios
```

---

## 🆚 1. Fetch vs Axios

Mira por qué Axios nos ahorra líneas de código y dolores de cabeza:

❌ **Con Fetch:**
```javascript
try {
  const res = await fetch('[https://api.ejemplo.com/users](https://api.ejemplo.com/users)');
  // Fetch NO lanza error en un 404 o 500, hay que revisarlo a mano
  if (!res.ok) throw new Error('Error HTTP'); 
  const data = await res.json(); // Hay que parsearlo manualmente
  console.log(data);
} catch (error) { ... }
```

✅ **Con Axios:**
```javascript
import axios from 'axios';

try {
  // Axios hace el parseo a JSON automáticamente.
  // Cualquier código HTTP fuera del rango 2xx lanza un error directo al catch.
  const { data } = await axios.get('[https://api.ejemplo.com/users](https://api.ejemplo.com/users)');
  console.log(data);
} catch (error) { ... }
```

---

## 🏗️ 2. Instancias de Axios (Evitando repetir URLs)

Imagina que tu API cambia de dominio, de `api.v1.com` a `api.v2.com`. Si usaste la URL completa en 50 componentes, tendrás que editar 50 archivos. 

El primer paso de nuestra arquitectura es crear un **Cliente Base** configurado. Creamos una carpeta llamada `src/api` y un archivo `apiClient.js`:

```javascript
// src/api/apiClient.js
import axios from 'axios';

export const apiClient = axios.create({
  // En la vida real, esta URL viene de un archivo .env
  baseURL: '[https://api.ejemplo.com/v1](https://api.ejemplo.com/v1)', 
  timeout: 10000, // Aborta la petición si tarda más de 10 segundos
  headers: {
    'Content-Type': 'application/json'
  }
});
```
A partir de ahora, en toda nuestra aplicación importaremos este `apiClient` en lugar del `axios` normal.

---

## 📂 3. La Capa de Servicios (Separation of Concerns)

**Regla de Oro del Arquitecto:** Los componentes de React no deberían saber qué es una URL, ni qué método HTTP se usa (GET, POST). El componente solo debería decir: *"Dame los usuarios"*.

Separamos toda la lógica de la API en archivos llamados **Servicios**. 

```javascript
// src/api/servicios/usuarios.service.js
import { apiClient } from '../apiClient';

// Agrupamos todas las llamadas relacionadas con Usuarios
export const UsuariosService = {
  
  obtenerTodos: async () => {
    const { data } = await apiClient.get('/users');
    return data;
  },

  obtenerPorId: async (id) => {
    const { data } = await apiClient.get(`/users/${id}`);
    return data;
  },

  crear: async (nuevoUsuario) => {
    const { data } = await apiClient.post('/users', nuevoUsuario);
    return data;
  }
};
```

**Míralo en acción dentro de un Componente:**
¡Mira qué limpio, legible y profesional queda tu componente!

```jsx
import { useEffect, useState } from 'react';
import { UsuariosService } from '../api/servicios/usuarios.service';

export const ListaUsuarios = () => {
  const [usuarios, setUsuarios] = useState([]);

  useEffect(() => {
    const cargar = async () => {
      try {
        // El componente no sabe nada de URLs ni de Axios.
        // Solo llama a un servicio.
        const datos = await UsuariosService.obtenerTodos();
        setUsuarios(datos);
      } catch (error) {
        console.error("Error al cargar usuarios");
      }
    };
    cargar();
  }, []);

  return <div>{usuarios.map(u => <p key={u.id}>{u.nombre}</p>)}</div>;
};
```

---

## 🛡️ 4. El Arma Secreta de Axios: Los Interceptores

Aquí es donde Axios demuestra por qué las empresas lo aman. Un Interceptor es como una "aduana" por la que pasan **todas** tus peticiones antes de salir al internet, y **todas** tus respuestas antes de llegar a tus componentes.

### Interceptor de Petición (Request): Inyectando el Token
Si tu app requiere inicio de sesión, necesitas enviar un Token (ej. JWT) en cada petición. En lugar de agregarlo manualmente en cada servicio, usamos un interceptor.

```javascript
// src/api/apiClient.js

apiClient.interceptors.request.use(
  (config) => {
    // Esta función se ejecuta ANTES de que cualquier petición salga
    const token = localStorage.getItem('token_seguridad');
    
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    return config; // Dejamos que la petición continúe su viaje
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

### Interceptor de Respuesta (Response): Manejo de Sesión Expirada
¿Qué pasa si el usuario se queda inactivo y su token expira? El servidor devolverá un error `401 Unauthorized`. 
En lugar de revisar si hay un `401` en cada uno de tus 50 componentes, el Interceptor de Respuesta lo detecta a nivel global y expulsa al usuario al Login automáticamente.

```javascript
// src/api/apiClient.js

apiClient.interceptors.response.use(
  (response) => {
    // Cualquier código de estado 2xx pasa por aquí (Éxito)
    return response;
  },
  (error) => {
    // Cualquier código de estado fuera de 2xx cae aquí (Error)
    if (error.response && error.response.status === 401) {
      console.warn("Sesión expirada. Redirigiendo al Login...");
      // Borramos el token inválido
      localStorage.removeItem('token_seguridad');
      // Redirigimos al usuario (usualmente usando window.location 
      // o un router global de la aplicación)
      window.location.href = '/login';
    }
    
    return Promise.reject(error); // Pasamos el error al catch del componente
  }
);
```

### 🏆 Beneficios de esta Arquitectura:
1. **Escalabilidad:** Si cambias de API, solo cambias `apiClient.js`.
2. **Componentes limpios:** Tu UI solo se preocupa por pintar datos, no por configurar cabeceras HTTP.
3. **Seguridad Centralizada:** Un solo lugar maneja los tokens y las redirecciones.