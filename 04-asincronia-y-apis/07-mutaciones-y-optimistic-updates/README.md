# 07 - Mutaciones y Optimistic Updates 💎

En el desarrollo frontend, no solo consumimos datos, también los modificamos. Crear un usuario, borrar un producto o dar un "Me Gusta" son acciones que alteran la base de datos. 

TanStack Query nos proporciona el hook `useMutation` para manejar estos casos. A diferencia de `useQuery` (que se ejecuta automáticamente al montar el componente), `useMutation` **solo se ejecuta cuando tú se lo ordenas**.

---

## 🛠️ 1. Anatomía de una Mutación (`useMutation`)

Imagina que tenemos un botón para añadir un nuevo proyecto.

```jsx
import { useMutation } from '@tanstack/react-query';
import { ProyectosService } from '../api/servicios/proyectos.service';

export const CrearProyecto = () => {
  // Configuración de la mutación
  const mutacion = useMutation({
    // La función asíncrona que hace el POST/PUT/DELETE
    mutationFn: (nuevoProyecto) => ProyectosService.crear(nuevoProyecto),
    
    // Callbacks del ciclo de vida
    onSuccess: (data) => {
      console.log("¡Proyecto creado con éxito en el servidor!", data);
    },
    onError: (error) => {
      console.error("Falló la creación:", error.message);
    }
  });

  const manejarEnvio = () => {
    // Disparamos la mutación pasándole los datos necesarios
    mutacion.mutate({ nombre: "Nuevo Proyecto React", estado: "activo" });
  };

  return (
    <div>
      <button 
        onClick={manejarEnvio} 
        disabled={mutacion.isPending} // isPending es true mientras la red trabaja
      >
        {mutacion.isPending ? 'Creando...' : 'Crear Proyecto'}
      </button>

      {mutacion.isError && <p className="error">{mutacion.error.message}</p>}
    </div>
  );
};
```

---

## 🔄 2. Invalidador de Caché (Query Invalidation)

Aquí surge un problema de arquitectura clásico: 
Tienes una pantalla con la **Lista de Proyectos** y el botón **Crear Proyecto**. Si el usuario crea un proyecto nuevo con éxito, la mutación termina, pero... **la lista de proyectos no se actualiza**. Sigue mostrando los datos cacheados antiguos (Stale).

En lugar de intentar meter el nuevo proyecto manualmente en el array local (lo cual es propenso a errores), la forma profesional es decirle a TanStack Query: *"Oye, la lista de proyectos ya no es válida. Ve al servidor y tráela de nuevo"*.

A esto se le llama **Invalidar Consultas**.

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

export const CrearProyecto = () => {
  // Obtenemos acceso al cliente global del caché
  const queryClient = useQueryClient(); 

  const mutacion = useMutation({
    mutationFn: ProyectosService.crear,
    onSuccess: () => {
      // 🪄 MAGIA: Marcamos la llave ['proyectos'] como inválida.
      // Si el componente de la lista está en pantalla, TanStack Query 
      // hará un refetch invisible automáticamente y la UI se actualizará.
      queryClient.invalidateQueries({ queryKey: ['proyectos'] });
    }
  });

  // ... render del botón
};
```

---

## ⚡ 3. UX Nivel Dios: Actualizaciones Optimistas (Optimistic Updates)

Invalidar el caché está muy bien, pero requiere hacer una petición de red. Si el servidor tarda 2 segundos en responder, el usuario tendrá que esperar 2 segundos viendo un botón de "Cargando" al dar un "Me Gusta". 

**En aplicaciones top tier (como Twitter o Instagram), el "Me Gusta" se marca en rojo INSTANTÁNEAMENTE.** ¿Cómo lo hacen si el servidor aún no ha respondido?

Engañando al usuario. Asumen de forma "Optimista" que el servidor no va a fallar, actualizan la interfaz al instante, y mandan la petición en segundo plano. Si por desgracia el servidor falla (ej. se cae el WiFi), deshacen el cambio y muestran un error.

### Implementando la Magia (El Patrón de Rollback)

Para lograr esto con TanStack Query, usamos el método `onMutate` para "tomar una foto" del estado antes del cambio, y modificar el caché a la fuerza.

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

export const BotonLike = ({ postId, likesActuales }) => {
  const queryClient = useQueryClient();

  const likeMutation = useMutation({
    mutationFn: () => PostsService.darLike(postId),

    // 1. onMutate se ejecuta EXACTAMENTE en el momento del clic, ANTES de la red
    onMutate: async () => {
      // a) Cancelamos cualquier petición de lectura (fetch) en curso para este post
      // para evitar que datos viejos sobreescriban nuestro optimismo.
      await queryClient.cancelQueries({ queryKey: ['post', postId] });

      // b) Tomamos una FOTO (Snapshot) del estado actual del caché por si algo sale mal
      const estadoPrevio = queryClient.getQueryData(['post', postId]);

      // c) ACTUAMOS DE FORMA OPTIMISTA: Modificamos el caché a la fuerza.
      // ¡El usuario ve el nuevo like al instante en la pantalla!
      queryClient.setQueryData(['post', postId], (viejoPost) => ({
        ...viejoPost,
        likes: viejoPost.likes + 1,
        hasLiked: true
      }));

      // d) Retornamos la foto antigua. Esto pasará al 'onError' como contexto.
      return { estadoPrevio };
    },

    // 2. Si el servidor falla (ej. Error 500)
    onError: (err, variables, context) => {
      // RESTAURACIÓN (Rollback): Usamos la foto para dejar todo como estaba
      queryClient.setQueryData(['post', postId], context.estadoPrevio);
      alert("Error de conexión. Se deshizo tu Me Gusta.");
    },

    // 3. Ocurra éxito o error, SIEMPRE re-sincronizamos con el servidor al final
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['post', postId] });
    }
  });

  return (
    <button onClick={() => likeMutation.mutate()}>
      ❤️ {likesActuales}
    </button>
  );
};
```

### ¿Por qué esto te convierte en Senior?
Escribir este patrón demuestra que no solo sabes cómo enviar datos a una base de datos, sino que tienes un profundo respeto por la psicología del usuario. Eliminar la latencia percibida es el secreto mejor guardado de las grandes empresas de tecnología.