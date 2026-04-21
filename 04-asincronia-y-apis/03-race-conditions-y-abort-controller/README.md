# 03 - El Peligro de la Red: Race Conditions y `AbortController`

En el tema anterior aprendimos a hacer peticiones limpias con Axios. Pero, ¿qué pasa si el usuario dispara múltiples peticiones antes de que la primera haya terminado?

Bienvenido al mundo de las **Condiciones de Carrera (Race Conditions)**. Un problema de concurrencia donde el resultado final depende de qué petición gane la "carrera" de vuelta desde el servidor, ignorando el orden en que el usuario hizo clic.

---

## 🏎️ 1. Entendiendo el Bug (El caso de los Perfiles)

Imagina una barra lateral con botones para ver los perfiles de tres usuarios: Ana, Beto y Carlos.

1. El usuario hace clic en **Ana**. React inicia la petición `GET /users/ana`. Supongamos que la base de datos está lenta y esto tardará **4 segundos**.
2. Un segundo después, el usuario se impacienta y hace clic en **Beto**. React inicia la petición `GET /users/beto`. Esta petición es rápida y tarda **1 segundo**.
3. El perfil de Beto se carga y aparece en pantalla. El usuario está feliz leyendo.
4. Pasan 2 segundos más y, de repente, ¡la petición de Ana (la de 4 segundos) termina! El código ejecuta `setUsuario(datosDeAna)`.
5. **Resultado fatal:** El usuario hizo clic en "Beto", la URL dice "Beto", pero la pantalla mágicamente cambia para mostrar los datos de "Ana".

---

## 🛡️ 2. La solución nativa: `AbortController`

En el Tema 1 vimos que podíamos usar una variable `isMounted` para ignorar los datos que llegaban tarde. Sin embargo, eso es solo ponerle una curita al problema: la petición de red se seguía ejecutando en segundo plano, consumiendo los datos móviles del usuario y la memoria del navegador.

La solución profesional (y el estándar moderno) es **cancelar (asesinar) la petición HTTP en vuelo**. Para esto usamos una API nativa del navegador llamada `AbortController`.

Así funciona en JavaScript puro:
```javascript
// 1. Creamos el controlador
const controller = new AbortController();

// 2. Le pasamos su "señal" al fetch (o a Axios)
fetch('/api/datos', { signal: controller.signal });

// 3. Si queremos cancelar la petición antes de que termine, tiramos del cable:
controller.abort();
```

---

## ⚛️ 3. Implementación Perfecta en React + Axios

Para usar esto en React, combinamos el `AbortController` con la **Función de Limpieza (Cleanup)** del `useEffect`. Cada vez que el ID del usuario cambie, React limpiará el efecto anterior, disparando `abort()` y matando la petición vieja antes de iniciar la nueva.

Veamos cómo se hace en un componente real con nuestra arquitectura de Axios:

```jsx
import { useEffect, useState } from 'react';
import axios from 'axios';

export const VisorDePerfil = ({ userId }) => {
  const [perfil, setPerfil] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    // 1. Instanciamos el controlador al inicio del efecto
    const controller = new AbortController();

    const cargarPerfil = async () => {
      setIsLoading(true);
      try {
        // 2. Pasamos la señal en la configuración de Axios
        const { data } = await axios.get(`https://api.ejemplo.com/users/${userId}`, {
          signal: controller.signal 
        });
        
        setPerfil(data);
        setError(null);
      } catch (err) {
        // 3. MANEJO DE ERROR CRÍTICO: 
        // Cuando cancelamos una petición, Axios lanza un error intencionalmente.
        // ¡No queremos mostrarle esto al usuario como si fuera un fallo del servidor!
        if (axios.isCancel(err)) {
          console.log(`Petición del usuario ${userId} cancelada exitosamente.`);
        } else {
          setError("Error real al cargar el perfil");
        }
      } finally {
        setIsLoading(false);
      }
    };

    cargarPerfil();

    // 4. CLEANUP: Tiramos del cable si el userId cambia o el componente se desmonta
    return () => {
      controller.abort();
    };
  }, [userId]); // El efecto se re-ejecuta cuando cambia el userId

  if (isLoading) return <p>Cargando perfil de {userId}...</p>;
  if (error) return <p>{error}</p>;
  if (!perfil) return <p>Selecciona un usuario</p>;

  return (
    <div className="perfil">
      <h3>{perfil.nombre}</h3>
      <p>Email: {perfil.email}</p>
    </div>
  );
};
```

---

## 🧠 4. ¿Cuándo DEBES usar AbortController?

Como Arquitecto, no es necesario que pongas `AbortController` en absolutamente todas las peticiones de tu aplicación. Úsalo estrictamente en estos tres escenarios:

1. **Buscadores con Autocompletado (Debounce no siempre basta):** Si el usuario busca "Rea", luego borra y busca "Vue", la búsqueda de "Rea" podría tardar más y sobrescribir los resultados de "Vue".
2. **Navegación Rápida:** Si tienes listas, pestañas o barras laterales donde el usuario puede cambiar la vista (y el ID) rápidamente.
3. **Peticiones Pesadas:** Si un componente carga reportes, gráficas o archivos grandes y existe la mínima posibilidad de que el usuario cierre ese modal o cambie de página antes de que termine.

### El futuro: ¿Por qué seguimos escribiendo esto a mano?
Si has notado, entre el `isLoading`, el `try/catch`, el chequeo de `axios.isCancel` y el `AbortController`, nuestro componente se ha llenado de lógica de red en lugar de lógica visual. 

En el **Tema 5**, veremos cómo herramientas como TanStack Query (React Query) automatizan absolutamente todo este comportamiento por debajo, dejándonos con componentes de 3 líneas de código. Pero para usar la magia, primero debías entender el truco.