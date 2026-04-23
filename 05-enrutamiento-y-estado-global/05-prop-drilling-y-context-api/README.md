# 05 - El Infierno del Prop Drilling y Context API

A medida que tu aplicación crece, el árbol de componentes se hace más profundo. Imagina que tienes el nombre del usuario guardado en tu componente principal (`App`) y necesitas mostrarlo en el componente `<Avatar />`. 

El problema es que `<Avatar />` está dentro de `<Navbar />`, que está dentro de `<Header />`, que está dentro de `<Layout />`.

---

## 😫 1. El Problema: Prop Drilling (Perforación de Props)

Para que el nombre del usuario llegue al `<Avatar />`, tienes que pasarlo como *prop* a través de todos los componentes intermedios. A esto se le llama **Prop Drilling**.

Es como pasar un cubo de agua en una cadena humana para apagar un incendio: las personas del medio se mojan aunque no querían el agua; solo actúan como intermediarios.

❌ **El Anti-patrón en código:**
Fíjate cómo `Layout`, `Header` y `Navbar` reciben la prop `usuario` pero **no la usan para nada**, solo la pasan hacia abajo.

```jsx
// 1. EL ORIGEN DE LOS DATOS
export const App = () => {
  const usuario = { nombre: "Ana", rol: "Admin" };
  // Pasa el dato al Layout
  return <Layout usuario={usuario} />; 
};

// 2. INTERMEDIARIO 1
const Layout = ({ usuario }) => {
  return <div><Header usuario={usuario} /></div>;
};

// 3. INTERMEDIARIO 2
const Header = ({ usuario }) => {
  return <header><Navbar usuario={usuario} /></header>;
};

// 4. INTERMEDIARIO 3
const Navbar = ({ usuario }) => {
  return <nav><Avatar usuario={usuario} /></nav>;
};

// 5. EL DESTINO FINAL (¡Por fin!)
const Avatar = ({ usuario }) => {
  return <div className="avatar">{usuario.nombre.charAt(0)}</div>;
};
```

**¿Por qué es esto terrible?**
Si mañana cambias el nombre de la variable de `usuario` a `cliente`, ¡tendrás que editar 5 componentes distintos! Tu código se vuelve frágil y difícil de mantener.

---

## 📡 2. La Solución Nativa: Context API

React nos da una herramienta nativa para solucionar esto: la **Context API**. 
Imagina que Context es como una **estación de radio** o una red Wi-Fi. El componente padre emite una señal con los datos, y cualquier componente hijo (sin importar qué tan profundo esté) puede "sintonizar" esa señal y usar los datos directamente, saltándose a todos los intermediarios.

Implementarlo requiere **3 pasos sencillos**:

### Paso 1: Crear el Contexto (La emisora de radio)
Creamos un archivo separado para mantener el orden.

```jsx
// src/context/UsuarioContext.jsx
import { createContext } from 'react';

// createContext crea la caja vacía. Por convención, se nombra con mayúscula.
export const UsuarioContext = createContext();
```

### Paso 2: El Provider (Emitir la señal)
Envolvemos nuestra aplicación (o la parte que necesite los datos) con el proveedor del contexto.

```jsx
// src/App.jsx
import { UsuarioContext } from './context/UsuarioContext';
import { Layout } from './components/Layout';

export const App = () => {
  const usuario = { nombre: "Ana", rol: "Admin" };

  return (
    // Todo lo que esté DENTRO de esta etiqueta tendrá acceso a 'value'
    <UsuarioContext.Provider value={usuario}>
      <Layout />
    </UsuarioContext.Provider>
  );
};
```

### Paso 3: El hook `useContext` (Sintonizar la señal)
¡Aquí ocurre la magia! Vamos directamente al componente final y usamos los datos. Los intermediarios (`Layout`, `Header`, `Navbar`) ya no necesitan recibir props.

```jsx
// src/components/Avatar.jsx
import { useContext } from 'react';
import { UsuarioContext } from '../context/UsuarioContext';

export const Avatar = () => {
  // Leemos los datos directamente de la "nube"
  const usuario = useContext(UsuarioContext);

  return <div className="avatar">{usuario.nombre.charAt(0)}</div>;
};
```
**Resultado:** Código limpio, los intermediarios no se enteran de nada, y si cambias la variable, solo editas el origen y el destino.

---

## 🛠️ 3. Modificando el Contexto (Pasando funciones)

Context no solo sirve para leer datos estáticos; también sirve para modificarlos. Si pasamos el estado (`useState`) completo, cualquier hijo puede actualizar a su padre.

```jsx
// En el App.jsx (Padre)
const [tema, setTema] = useState('claro');

return (
  // Podemos pasar un objeto que contenga tanto el valor como la función para cambiarlo
  <TemaContext.Provider value={{ tema, setTema }}>
    <Layout />
  </TemaContext.Provider>
);
```

```jsx
// En el BotonTema.jsx (Hijo lejano)
const BotonTema = () => {
  // Extraemos la función desestructurando el objeto
  const { tema, setTema } = useContext(TemaContext);

  return (
    <button onClick={() => setTema(tema === 'claro' ? 'oscuro' : 'claro')}>
      Cambiar a {tema === 'claro' ? 'Oscuro' : 'Claro'}
    </button>
  );
};
```

---

## ⚠️ 4. Nivel Arquitecto: El lado oscuro de Context API

Si Context es tan bueno, ¿por qué no metemos todo el estado de la aplicación ahí dentro? **Porque destruirá el rendimiento de tu aplicación.**

**La regla de hierro de Context API:**
*Cuando el `value` de un Provider cambia, TODOS los componentes que usen `useContext` de ese provider se volverán a renderizar inmediatamente.*

* **Cuándo usar Context:** Datos que cambian MUY POCO y son de lectura global.
  * Tema Claro/Oscuro.
  * Idioma de la página (Español/Inglés).
  * Datos de sesión del usuario (Logueado o no).
* **Cuándo NO usar Context:** Datos que cambian frecuentemente o estructuras complejas.
  * Lo que el usuario escribe en un input (cada letra forzaría un re-render de todo el sistema).
  * Coordenadas del ratón.
  * Listados gigantes de una API.

Para solucionar este problema de rendimiento en estados globales complejos, la industria abandonó `Context` (y `Redux`) y adoptó una nueva herramienta mágica. La veremos en el próximo tema: **Zustand**.