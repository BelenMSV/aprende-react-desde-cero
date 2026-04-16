# 02 - `useState` Profundo: Objetos y Arrays

En el tema anterior vimos que la regla de oro del estado es: **El Estado es Inmutable**. Esto significa que nunca debemos modificar la variable directamente, sino usar la función actualizadora (`set...`).

Cuando trabajamos con valores primitivos (números, *strings*, booleanos), esto es fácil de entender: cambias un `5` por un `6`. Pero, ¿qué pasa cuando nuestro estado es un Objeto o un Array?

---

## 📦 Trabajando con Objetos

Imagina que tienes un formulario con un objeto de usuario y quieres actualizar solo su nombre.

❌ **La trampa del Junior (Mutación Directa):**
```jsx
export const Perfil = () => {
  const [usuario, setUsuario] = useState({ nombre: 'Ana', edad: 25 });

  const cambiarNombre = () => {
    // 🚩 ERROR 1: Modificar la propiedad directamente
    usuario.nombre = 'Beatriz'; 
    
    // 🚩 ERROR 2: Pasar el MISMO objeto a la función set
    setUsuario(usuario); 
  };

  return <button onClick={cambiarNombre}>Llamarse Beatriz</button>;
};
```
**¿Por qué falla esto?** React decide si debe re-renderizar la pantalla comparando el estado viejo con el nuevo. Pero en JavaScript, los objetos se comparan por **referencia** (su espacio en memoria), no por su contenido. 

Como modificaste el objeto original y se lo volviste a pasar, React dice: *"Es exactamente el mismo objeto en memoria, así que no ha cambiado nada. No voy a actualizar la pantalla"*.

✅ **La forma Profesional (Operador Spread):**
Para que React se entere del cambio, debes crear **un objeto completamente nuevo** en memoria. Usamos el operador *spread* (`...`) de JavaScript para copiar las propiedades viejas y luego sobreescribir solo la que queremos cambiar.

```jsx
const cambiarNombre = () => {
  setUsuario({
    ...usuario,         // Copiamos todo lo que ya tenía (edad: 25)
    nombre: 'Beatriz'   // Sobreescribimos solo el nombre
  });
};
```

---

## 📚 Trabajando con Arrays

Las mismas reglas aplican a los Arrays. **NUNCA** debes usar métodos que muten el Array original como `.push()`, `.pop()`, `.splice()` o `.sort()` directamente sobre el estado.

Debes usar métodos que devuelvan **un nuevo Array**:
- Para añadir: Operador `...spread`
- Para eliminar: `.filter()`
- Para modificar: `.map()`

### 1. Añadir elementos a un Array
```jsx
const [tareas, setTareas] = useState(['Estudiar', 'Dormir']);

const agregarTarea = (nuevaTarea) => {
  // ❌ INCORRECTO: tareas.push(nuevaTarea); setTareas(tareas);

  // ✅ CORRECTO: Creamos un nuevo array con los elementos anteriores + el nuevo
  setTareas([...tareas, nuevaTarea]);
};
```

### 2. Eliminar elementos
```jsx
const eliminarTarea = (tareaABorrar) => {
  // ✅ CORRECTO: filter devuelve un array nuevo sin el elemento indeseado
  setTareas(tareas.filter(tarea => tarea !== tareaABorrar));
};
```

### 3. Modificar un elemento específico
```jsx
const marcarComoCompletada = (id) => {
  // ✅ CORRECTO: map devuelve un array nuevo alterando solo lo que necesitamos
  setTareas(tareas.map(tarea => {
    if (tarea.id === id) {
      return { ...tarea, completada: true }; // Retorna la tarea modificada
    }
    return tarea; // Retorna la tarea intacta
  }));
};
```

---

## 🔄 El Estado Previo (Previous State)

Hasta ahora, hemos actualizado el estado usando la variable del estado actual (`setNumero(numero + 1)`). Esto funciona el 90% de las veces, pero **tiene una falla crítica** debido a cómo React procesa las actualizaciones por lotes (*Batching*).

Mira este ejemplo. Queremos sumar 3 puntos al contador al hacer clic:

```jsx
const [puntos, setPuntos] = useState(0);

const sumarTres = () => {
  // ❌ Esto NO sumará 3. Solo sumará 1.
  setPuntos(puntos + 1);
  setPuntos(puntos + 1);
  setPuntos(puntos + 1);
};
```
Como vimos en el Tema 1, React agrupa estas llamadas. En ese momento, `puntos` vale `0`. Así que React lee: `setPuntos(0 + 1)`, luego `setPuntos(0 + 1)` y finalmente `setPuntos(0 + 1)`. 

✅ **La Solución: Funciones de Actualización (Updater Functions)**

Si el nuevo valor de tu estado **depende del valor inmediatamente anterior**, NO debes pasarle un valor directo a tu `set`. Debes pasarle una **función** (Arrow Function).

React tomará esa función y le inyectará el estado más reciente y garantizado de forma automática (al que solemos llamar `prev` o `prevState`).

```jsx
const sumarTresCorrectamente = () => {
  // ✅ React ejecutará esto en cola, usando el valor más reciente en cada paso
  setPuntos(prev => prev + 1); // Aquí prev es 0. Retorna 1.
  setPuntos(prev => prev + 1); // Aquí prev es 1. Retorna 2.
  setPuntos(prev => prev + 1); // Aquí prev es 2. Retorna 3.
};
```

> 💡 **Regla de Oro Senior:**
> Si escribes un `setState` y dentro de sus paréntesis estás usando la variable original del estado (ej. `setContador(contador + 1)`), detente. Cámbialo por `setContador(prev => prev + 1)`. Es muchísimo más seguro a prueba de bugs asíncronos.

---

## ⚠️ La trampa del Anidamiento Profundo

El operador `...spread` hace una **copia superficial** (*shallow copy*). Si tienes un objeto dentro de otro objeto, ten mucho cuidado:

```jsx
const [perfil, setPerfil] = useState({
  nombre: 'Alex',
  direccion: { ciudad: 'Madrid', cp: 28001 }
});

const cambiarCiudad = () => {
  // ❌ INCORRECTO: Borraría el Código Postal
  // setPerfil({ ...perfil, direccion: { ciudad: 'Barcelona' } });

  // ✅ CORRECTO: Hay que hacer spread también del objeto anidado
  setPerfil({
    ...perfil,
    direccion: {
      ...perfil.direccion,
      ciudad: 'Barcelona'
    }
  });
};
```
*(Nota: Si tus estados están demasiado anidados, es una señal de que debes aplanar tu estructura de datos o separar el estado en múltiples `useState`).*

---

## 🛠️ Caso de Uso Real: Formularios y el "Manejador Universal"

El lugar donde más vas a utilizar objetos en el estado es al crear formularios.

Imagina un formulario de registro. Un desarrollador Junior instintivamente crearía un estado para cada *input*:

❌ **El enfoque tedioso:**
```jsx
const [nombre, setNombre] = useState("");
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");
// Si hay 15 campos, tendrás 15 funciones distintas...
```

✅ **El Patrón Profesional (Manejador Universal):**
Agrupamos todo en un solo objeto. Luego, usamos el atributo `name` de los inputs HTML y las **Propiedades Computadas de JavaScript** (los corchetes `[]` en las claves de los objetos) para crear una sola función que sirva para TODOS los inputs.

```jsx
export const FormularioRegistro = () => {
  const [form, setForm] = useState({
    nombre: "",
    email: "",
    password: ""
  });

  const handleChange = (e) => {
    // e.target.name será "nombre", "email" o "password" según el input que toques
    // e.target.value será lo que el usuario haya escrito
    
    setForm({
      ...form, // Copiamos el resto del formulario
      [e.target.name]: e.target.value // Sobreescribimos solo el campo que cambió
    });
  };

  return (
    <form>
      <input 
        type="text" 
        name="nombre" /* El name DEBE coincidir con la clave del estado */
        value={form.nombre} 
        onChange={handleChange} 
      />
      <input 
        type="email" 
        name="email" 
        value={form.email} 
        onChange={handleChange} 
      />
      <button type="submit">Registrarse</button>
    </form>
  );
};
```
Este patrón te ahorrará escribir cientos de líneas de código a lo largo de tu carrera.

---

## 🚀 Nivel Experto: La librería `Immer`

En la sección anterior vimos que el anidamiento profundo con el operador spread (`...`) puede volverse ilegible rápidamente. 

```jsx
// Código difícil de leer (React nativo)
setPerfil(prev => ({
  ...prev,
  configuracion: {
    ...prev.configuracion,
    notificaciones: {
      ...prev.configuracion.notificaciones,
      email: false
    }
  }
}));
```

Cuando trabajas en aplicaciones empresariales con estados gigantes, la comunidad de React utiliza una librería casi obligatoria llamada **Immer**.

Immer te permite escribir código "mutando" los objetos directamente (como harías en Vanilla JS) y, bajo el capó, intercepta esos cambios y devuelve un estado perfectamente inmutable para React. Su hook `useImmer` es un reemplazo directo de `useState`.

```jsx
// El mismo código anterior, pero usando useImmer
import { useImmer } from "use-immer";

const [perfil, setPerfil] = useImmer(perfilInicial);

const apagarNotificaciones = () => {
  // ¡Parece mutación directa, pero Immer lo hace inmutable!
  setPerfil(draft => {
    draft.configuracion.notificaciones.email = false;
  });
};
```
*(Nota: No necesitas instalar Immer para proyectos pequeños, pero saber que existe te dará muchísimos puntos en una entrevista técnica para un puesto Mid/Senior).*