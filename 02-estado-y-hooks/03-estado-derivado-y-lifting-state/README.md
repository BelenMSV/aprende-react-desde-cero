# 03 - Estado Derivado y Lifting State Up

Cuando descubres el Hook `useState`, es muy tentador meter toda la información de tu aplicación dentro de él. Sin embargo, en React, **menos estado es mejor estado**.

Tener demasiadas variables de estado hace que tu código sea difícil de leer, propenso a errores de desincronización y, lo más importante, provoca renderizados innecesarios. En este tema aprenderemos dos técnicas vitales de arquitectura para mantener nuestro estado al mínimo.

---

## 🧮 1. Estado Derivado (Derived State)

La regla de oro del estado derivado es: **Si un valor se puede calcular a partir de las `props` o de otra variable de `estado` existente, NO debes meterlo en un nuevo `useState`.**

### El problema: Variables redundantes (Anti-patrón)

Imagina que tienes una lista de tareas y quieres mostrar cuántas están completadas.

❌ **El error del Junior:**
```jsx
export const ListaTareas = () => {
  const [tareas, setTareas] = useState([
    { id: 1, texto: 'Aprender React', completada: true },
    { id: 2, texto: 'Ir al gimnasio', completada: false }
  ]);
  
  // 🚩 ERROR: Crear un estado extra para algo que ya sabemos
  const [tareasCompletadas, setTareasCompletadas] = useState(1);

  const agregarTarea = (nueva) => {
    setTareas([...tareas, nueva]);
    // 🚩 Ahora tienes la pesadilla de mantener DOS estados sincronizados a mano.
    // Si olvidas actualizar 'setTareasCompletadas' aquí, tu app mostrará datos falsos.
  };

  return (
    <div>
      <p>Completadas: {tareasCompletadas}</p>
    </div>
  );
};
```

### La solución: Calcular al vuelo

Recuerda que cada vez que el estado `tareas` cambie, React volverá a ejecutar toda la función del componente de arriba a abajo. ¡Aprovecha eso!

✅ **El enfoque Profesional (Estado Derivado):**
Calcula el valor directamente usando JavaScript puro justo antes del `return`.

```jsx
export const ListaTareas = () => {
  const [tareas, setTareas] = useState([
    { id: 1, texto: 'Aprender React', completada: true },
    { id: 2, texto: 'Ir al gimnasio', completada: false }
  ]);

  // ✅ CORRECTO: Se recalcula automáticamente en cada renderizado.
  // Es imposible que se desincronice. Si las tareas cambian, esto se actualiza solo.
  const tareasCompletadas = tareas.filter(tarea => tarea.completada).length;

  return (
    <div>
      <p>Completadas: {tareasCompletadas}</p>
    </div>
  );
};
```

> ⚠️ **Nota de rendimiento:** Si la lista tiene 10 elementos, el `.filter` es instantáneo. Si tiene 10,000 elementos, calcularlo en cada renderizado podría ser lento. En el Módulo de Optimización aprenderemos a usar el hook `useMemo` para cachear estos cálculos pesados, pero la regla de no usar `useState` se mantiene.

---

## 🏗️ 2. Elevando el Estado (Lifting State Up)

A menudo te encontrarás en una situación donde **dos componentes hermanos necesitan compartir la misma información**. 

Sabemos que en React los datos solo fluyen hacia abajo (de Padre a Hijo a través de *props*). Un componente hermano no puede pasarle datos directamente a otro.

### El problema: Hermanos incomunicados

Imagina un componente `Acordeon` que tiene dos paneles (`PanelA` y `PanelB`). Queremos que cuando abras un panel, el otro se cierre automáticamente (solo uno abierto a la vez).

Si cada panel tiene su propio estado `[isOpen, setIsOpen]`, no tienen forma de avisarle al otro que se han abierto.

### La Solución: Subir el estado al Padre común

Para que los hermanos se comuniquen, debes "arrancar" el estado de los componentes hijos y moverlo a su ancestro común más cercano (el Padre). Luego, el Padre les pasará el estado a los hijos a través de las *props*.

```jsx
// 1. EL PADRE (Acordeon) controla la memoria global
export const Acordeon = () => {
  // Guardamos qué panel está activo actualmente (0 o 1)
  const [panelActivo, setPanelActivo] = useState(0);

  return (
    <div className="acordeon">
      <Panel 
        titulo="Panel 1" 
        // Le decimos si está abierto o no
        isActive={panelActivo === 0} 
        // Le pasamos la función para que le avise al padre que quiere abrirse
        onShow={() => setPanelActivo(0)} 
      >
        Contenido del primer panel.
      </Panel>

      <Panel 
        titulo="Panel 2" 
        isActive={panelActivo === 1} 
        onShow={() => setPanelActivo(1)} 
      >
        Contenido del segundo panel.
      </Panel>
    </div>
  );
};

// 2. EL HIJO (Panel) se vuelve un componente "tonto" (Controlado)
// Ya no tiene useState. Hace exactamente lo que el padre le dice.
export const Panel = ({ titulo, children, isActive, onShow }) => {
  return (
    <section className="panel">
      <h3>{titulo}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Mostrar</button>
      )}
    </section>
  );
};
```

### 🧠 Componentes Controlados vs No Controlados
Lo que acabamos de hacer tiene nombre formal en React:
* Cuando un componente tiene su propio estado interno, se le llama **No Controlado** (él toma sus propias decisiones).
* Cuando le quitas el estado y lo manejas mediante *props* pasadas por su padre, se convierte en un componente **Controlado** (el padre toma las decisiones).

## 🚦 Reglas de Supervivencia para el Estado

Antes de usar `useState`, hazte estas tres preguntas:
1. **¿Permanece inalterado con el tiempo?** Si la respuesta es sí, no es estado. Es una constante normal (`const`).
2. **¿Se pasa desde un componente padre mediante props?** Si es así, no es estado.
3. **¿Puedes calcularlo a partir de cualquier otro estado o prop en tu componente?** Si es así, ¡definitivamente no es estado!

---

## 🚀 Pro Tip Senior: Reseteando el estado con la prop `key`

Este es uno de los secretos mejor guardados de React. Imagina que tienes un componente `<PerfilUsuario />` con un estado interno para un formulario de edición.

Si cambias de usuario (por ejemplo, pasas de editar a "Ana" a editar a "Carlos"), React **reutilizará** el componente `<PerfilUsuario />` para ganar rendimiento. El problema es que el estado interno del formulario (lo que estabas escribiendo para Ana) ¡se quedará guardado y aparecerá en el perfil de Carlos!

❌ **El Anti-patrón (Usar useEffect para limpiar):**
Muchos Juniors intentan arreglar esto "vigilando" el cambio con un efecto:
```jsx
// 🚩 NO HAGAS ESTO: Es lento y propenso a bugs
useEffect(() => {
  setFormularioTexto(''); 
}, [usuarioId]);
```

✅ **La forma React (Usar la prop `key`):**
En el Tema 7 vimos que la prop `key` sirve para las listas. Pero también tiene un superpoder: **Si la `key` de un componente cambia, React destruye el componente viejo y crea uno completamente nuevo**, reseteando TODO su estado interno al instante.

```jsx
// En el componente Padre:
export const PanelEdicion = ({ usuarioSeleccionado }) => {
  return (
    // Al cambiar el ID, React destruye el perfil viejo y monta uno limpio
    <PerfilUsuario 
      key={usuarioSeleccionado.id} 
      usuario={usuarioSeleccionado} 
    />
  );
};
```
Esta es la forma recomendada oficialmente por el equipo de React para resetear estados.

---

## ⚠️ La otra cara de la moneda: El "Prop Drilling"

La técnica de **Lifting State Up** (Subir el estado) es maravillosa para componentes que están cerca (como hermanos compartiendo un padre común). 

Pero, ¿qué pasa si dos componentes necesitan compartir el mismo estado, pero uno está en la Cabecera de la app y otro en el Footer? Si subes el estado hasta `App.jsx`, tendrás que pasar las *props* hacia abajo a través de 10 componentes intermedios que no necesitan esa información.

A este problema se le llama **Prop Drilling** (Perforación de Props).

```jsx
// 🚩 Esto es Prop Drilling: Pasar "temaOscuro" por sitios donde no importa
<App>
  <Layout temaOscuro={temaOscuro}>
    <Header temaOscuro={temaOscuro}>
      <Menu temaOscuro={temaOscuro}>
        <BotonTema temaOscuro={temaOscuro} />
      </Menu>
    </Header>
  </Layout>
</App>
```

**¿La solución?** Si tienes que pasar *props* a través de más de 3 niveles de componentes, "Lifting State Up" ya no es la mejor opción. Para esos casos, utilizaremos el **Context API** o gestores de estado globales como **Zustand** o **Redux** (Conceptos que exploraremos a fondo en módulos posteriores).