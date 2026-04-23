# 05 - PROYECTO FINAL: ReactBoard (Clon de Trello) 🚀

Ha llegado el momento de combinar todo tu conocimiento en una sola aplicación. ReactBoard no es un proyecto de juguete; su arquitectura es idéntica a la que usarías en un entorno corporativo real.

## 🛠️ El Stack Tecnológico Definitivo

Para este proyecto, utilizaremos la "Santísima Trinidad" del stack moderno de React:
1. **Enrutamiento:** React Router v6.4+ (Loaders y Actions).
2. **Estado del Servidor:** TanStack Query v5 (Caché y Mutaciones).
3. **Estado del Cliente:** Zustand (Filtros de UI y estado temporal).
4. **Formularios:** React Hook Form + Zod (Validación estricta).
5. **Drag and Drop:** `@hello-pangea/dnd` (El estándar moderno y accesible para listas arrastrables).

---

## 🏗️ 1. Arquitectura del Proyecto (Feature-Sliced Design)

Aplicaremos lo aprendido en el Tema 1. Nuestra aplicación tendrá dos "Features" principales: Tableros (`boards`) y Tareas (`tasks`).

```text
src/
 ┣ 📂 app/              <-- main.jsx, AppLayout.jsx, router.jsx
 ┣ 📂 pages/            <-- BoardPage.jsx (Une las piezas)
 ┣ 📂 features/
 ┃ ┣ 📂 boards/         <-- Lógica para crear y listar tableros
 ┃ ┗ 📂 tasks/          <-- EL CORAZÓN DEL PROYECTO
 ┃   ┣ 📂 api/          <-- Peticiones Axios (getTasks, moveTask)
 ┃   ┣ 📂 components/   <-- Column, TaskCard, CreateTaskForm
 ┃   ┣ 📂 hooks/        <-- useTaskMutations.js
 ┃   ┗ 📜 index.js
 ┗ 📂 shared/           <-- Button, Modal, Input
```

---

## 🧠 2. El Desafío del Estado: ¿Quién guarda las tareas?

En un tablero Kanban, mover una tarjeta de "Pendiente" a "Terminado" es un desafío arquitectónico. Tienes dos opciones, y solo una te hace Senior:

❌ **La opción Junior (Doble Fuente de la Verdad):**
Guardar las tareas en Zustand Y en TanStack Query. Intentar sincronizarlas manualmente con `useEffect`. Esto causa bugs de duplicación y estados desactualizados.

✅ **La opción Senior (Single Source of Truth):**
Las tareas **solo viven en el caché de TanStack Query**. Zustand solo se usa para guardar cosas de la Interfaz, como: `isCardDragging: true` o `activeFilter: 'alta-prioridad'`. Cuando movemos una tarjeta, manipulamos directamente el caché de Query.

---

## 🏎️ 3. El Motor del Tablero: Actualizaciones Optimistas

Cuando un usuario arrastra una tarea y la suelta en una nueva columna, espera que se quede ahí **inmediatamente**. Si le mostramos un *Spinner* de carga mientras el servidor guarda la nueva posición, la experiencia será torpe y horrible.

Usaremos el **Patrón de Rollback (Actualización Optimista)** que vimos en el Módulo 4, pero a un nivel masivo.

### El Hook Maestro: `useMoveTask`

```jsx
// src/features/tasks/hooks/useMoveTask.js
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { tasksApi } from '../api/tasksApi';

export const useMoveTask = (boardId) => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ taskId, newStatus }) => tasksApi.updateStatus(taskId, newStatus),

    // 1. EL USUARIO SUELTA LA TARJETA (Se ejecuta antes de la red)
    onMutate: async ({ taskId, newStatus }) => {
      // Cancelamos peticiones en vuelo para no sobreescribir nuestro optimismo
      await queryClient.cancelQueries({ queryKey: ['tasks', boardId] });

      // Tomamos una "foto" de cómo estaba el tablero por si hay que deshacer
      const snapshotPrevio = queryClient.getQueryData(['tasks', boardId]);

      // Modificamos el caché a la fuerza para que la UI reaccione AL INSTANTE
      queryClient.setQueryData(['tasks', boardId], (tareasViejas) => {
        return tareasViejas.map(tarea => 
          tarea.id === taskId ? { ...tarea, status: newStatus } : tarea
        );
      });

      // Retornamos la foto para el Rollback
      return { snapshotPrevio };
    },

    // 2. ¡EL SERVIDOR FALLÓ! (ej. Se cayó el internet)
    onError: (err, variables, context) => {
      // Devolvemos las tarjetas a su posición original
      queryClient.setQueryData(['tasks', boardId], context.snapshotPrevio);
      alert("Error al mover la tarea. Sincronizando tablero...");
    },

    // 3. Ocurra éxito o error, re-sincronizamos silenciosamente al final
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks', boardId] });
    }
  });
};
```

---

## 🖱️ 4. Implementando el Drag and Drop

Usar `@hello-pangea/dnd` requiere envolver nuestro tablero en un `<DragDropContext>`. Este componente nos avisa exactamente cuándo y dónde se soltó una tarjeta.

```jsx
// src/features/tasks/components/KanbanBoard.jsx
import { DragDropContext } from '@hello-pangea/dnd';
import { useQuery } from '@tanstack/react-query';
import { Column } from './Column';
import { useMoveTask } from '../hooks/useMoveTask';

export const KanbanBoard = ({ boardId }) => {
  // Traemos las tareas del servidor
  const { data: tasks } = useQuery({ queryKey: ['tasks', boardId], ... });
  
  // Traemos nuestra mutación optimista
  const moveTaskMutation = useMoveTask(boardId);

  // Esta función se dispara cuando el usuario suelta el click del ratón
  const onDragEnd = (result) => {
    const { destination, source, draggableId } = result;

    // Si soltó la tarjeta fuera del tablero, no hacemos nada
    if (!destination) return;

    // Si la soltó en la misma columna y misma posición, no hacemos nada
    if (destination.droppableId === source.droppableId && destination.index === source.index) return;

    // ¡Acción! Disparamos la mutación optimista.
    // draggableId es el ID de la tarea. destination.droppableId es el estado de la nueva columna (ej. 'DONE').
    moveTaskMutation.mutate({ 
      taskId: draggableId, 
      newStatus: destination.droppableId 
    });
  };

  return (
    <DragDropContext onDragEnd={onDragEnd}>
      <div className="tablero-grid">
        <Column title="Pendiente" statusId="TODO" tasks={tasks.filter(t => t.status === 'TODO')} />
        <Column title="En Progreso" statusId="IN_PROGRESS" tasks={tasks.filter(t => t.status === 'IN_PROGRESS')} />
        <Column title="Hecho" statusId="DONE" tasks={tasks.filter(t => t.status === 'DONE')} />
      </div>
    </DragDropContext>
  );
};
```

---

### 🏆 ¿Por qué este proyecto te define como Senior?
Cualquiera puede hacer un TODO List. Pero crear un sistema de *Drag & Drop* que se comunique con el servidor en segundo plano, controle los errores de red, manipule el caché manualmente para evitar "spinners" de carga y mantenga el código organizado en "Features", es **Ingeniería de Software real**.