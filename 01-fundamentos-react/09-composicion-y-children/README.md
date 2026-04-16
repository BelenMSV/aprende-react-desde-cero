# 09 - Composición y Arquitectura Avanzada

En el Tema 5 rozamos brevemente la existencia de la prop especial `children`. En este tema, vamos a elevar ese concepto y aprender cómo los desarrolladores Senior utilizan la prop `children` y otras props para crear arquitecturas flexibles. A este patrón de diseño se le conoce como **Composición**.

La regla de oro de la Composición en React es: **Construye componentes que no necesiten saber qué van a renderizar de antemano.**

---

## 1. El Patrón "Layout" (Contenedores)

El uso más común de la Composición es crear "Layouts" o envoltorios genéricos. Piensa en un componente modal, un panel lateral (Sidebar) o el diseño principal de una página web (Navbar arriba, Footer abajo).

El contenedor define la estructura y los estilos, y "delega" el contenido central a quien lo use mediante `children`.

```jsx
// MainLayout.jsx (El Contenedor)
export const MainLayout = ({ children }) => {
  return (
    <div className="layout-app">
      <header>Mi Empresa</header>
      
      <main className="contenido-central">
        {/* Aquí se inyectará lo que pongamos dentro de las etiquetas */}
        {children} 
      </main>
      
      <footer>© 2026 Todos los derechos reservados</footer>
    </div>
  );
};
```

Uso en la aplicación:
```jsx
// App.jsx
import { MainLayout } from './MainLayout';
import { FormularioContacto } from './FormularioContacto';

export const App = () => {
  return (
    <MainLayout>
      {/* Todo esto se convierte en la prop "children" */}
      <h1>Contáctanos</h1>
      <FormularioContacto />
    </MainLayout>
  );
};
```

---

## 2. El Patrón "Slots" (Múltiples huecos)

`children` es genial, pero ¿qué pasa si tu componente necesita recibir inyecciones de código en **varios lugares distintos**? (Por ejemplo, un Navbar donde quieres poner un logo a la izquierda y un botón a la derecha).

Como los componentes de React son solo funciones, y JSX es solo JavaScript, **puedes pasar componentes enteros como props normales**. A esto se le llama el patrón de Slots (Ranuras).

```jsx
// Navbar.jsx
// Recibimos tres "ranuras" listas para ser llenadas
export const Navbar = ({ slotIzquierda, slotCentro, slotDerecha }) => {
  return (
    <nav className="flex-navbar">
      <div className="izq">{slotIzquierda}</div>
      <div className="cen">{slotCentro}</div>
      <div className="der">{slotDerecha}</div>
    </nav>
  );
};
```

Uso en la aplicación:
```jsx
// App.jsx
export const App = () => {
  return (
    <Navbar 
      slotIzquierda={<img src="/logo.png" />}
      slotCentro={<Buscador />}
      slotDerecha={<BotonLogin />}
    />
  );
};
```
Esta es la técnica definitiva para crear componentes de interfaz de usuario altamente reutilizables.

---

## 🧠 Resolviendo el "Prop Drilling" con Composición

En el Tema 5 mencionamos el terrible problema del **Prop Drilling**: tener que pasar la prop `usuario` desde el abuelo, al padre, al hijo, solo para que el nieto dibuje el nombre del usuario.

Muchos recurren inmediatamente a herramientas complejas como Redux o Zustand para solucionar esto. Sin embargo, la propia documentación de React recomienda intentar usar **Composición** (Inversión de Control) primero.

❌ **El problema (Prop Drilling):**
```jsx
const App = ({ usuario }) => <Layout usuario={usuario} />;
const Layout = ({ usuario }) => <Cabecera usuario={usuario} />;
const Cabecera = ({ usuario }) => <Avatar usuario={usuario} />;
const Avatar = ({ usuario }) => <img src={usuario.foto} />;
```

✅ **La solución (Composición):**
El componente padre crea el elemento final directamente, y solo pasa el elemento (el componente instanciado) hacia abajo a través de props o `children`. Los componentes intermedios ya no necesitan saber que existe el dato `usuario`.

```jsx
const App = ({ usuario }) => {
  // Instanciamos el Avatar AQUÍ, donde tenemos el dato.
  const miAvatar = <img src={usuario.foto} />;
  
  // Se lo pasamos a Layout como un componente (Slot)
  return <Layout elementoAvatar={miAvatar} />;
};

// Layout y Cabecera se vuelven "tontos" y reutilizables. Solo renderizan lo que les den.
const Layout = ({ elementoAvatar }) => <Cabecera elementoDerecho={elementoAvatar} />;
const Cabecera = ({ elementoDerecho }) => <div className="header">{elementoDerecho}</div>;
```

Esta técnica mantiene tus componentes intermedios completamente limpios y desacoplados de la lógica de negocio.

---

## 🚀 3. Nivel Senior: Dot Notation (Namespaces)

Cuando creas componentes complejos basados en composición (como una Tarjeta que siempre necesita un Header, un Body y un Footer), importar los cuatro componentes sueltos puede ser molesto y propenso a errores.

El patrón "Dot Notation" (o Componentes Compuestos básicos) agrupa los sub-componentes dentro del componente principal. Como en JavaScript las funciones son objetos, podemos añadirles propiedades.

```jsx
// 1. Creamos las piezas individuales (pueden estar en el mismo archivo)
const CardContenedor = ({ children }) => <div className="card">{children}</div>;
const CardHeader = ({ children }) => <div className="card-header">{children}</div>;
const CardBody = ({ children }) => <div className="card-body">{children}</div>;

// 2. "Enganchamos" las piezas al contenedor principal usando notación de punto
CardContenedor.Header = CardHeader;
CardContenedor.Body = CardBody;

// 3. Exportamos SOLO el contenedor principal
export const Card = CardContenedor;
```

**Uso en la aplicación (La magia):**
Fíjate en lo limpia que queda la importación y el uso. El desarrollador que usa tu código sabe inmediatamente qué componentes están diseñados para trabajar juntos.

```jsx
import { Card } from './Card'; // ¡Solo una importación!

export const App = () => {
  return (
    <Card>
      <Card.Header>
        <h2>Título de la Tarjeta</h2>
      </Card.Header>
      <Card.Body>
        <p>Este es el contenido principal.</p>
      </Card.Body>
    </Card>
  );
};
```

---

## 🤯 4. Nivel Experto: Funciones como `children`

Hasta ahora, siempre hemos pasado JSX o strings dentro de `children`. Pero recuerda: `children` es solo una prop de React, y en React puedes pasar *cualquier* tipo de dato por props, **incluso funciones**.

A esto se le conoce como el patrón **"Render Props" (Función como hijo)**. 

Imagina un componente contenedor que tiene cierta información (por ejemplo, saber si la pantalla es de móvil o de escritorio gracias a un *event listener* interno). El componente contenedor no sabe qué dibujar, pero le puede **pasar esos datos hacia abajo** al `children` si el `children` es una función.

```jsx
// El Contenedor "Inteligente"
export const Responsivo = ({ children }) => {
  // (Imagina que aquí hay lógica para detectar el ancho de pantalla)
  const esMovil = true; 

  // En lugar de renderizar children normalmente, lo EJECUTAMOS como una función
  // y le pasamos los datos que hemos calculado.
  return children(esMovil); 
};
```

**Uso en la aplicación:**
En lugar de pasar HTML normal entre las etiquetas, pasamos una *Arrow Function*. El componente `Responsivo` ejecutará esa función pasándole el valor `esMovil`.

```jsx
import { Responsivo } from './Responsivo';

export const App = () => {
  return (
    <Responsivo>
      {/* El children es esta función: */}
      {(esMovil) => (
        <div>
          {esMovil ? <MenuHamburguesa /> : <MenuNavegacionCompleto />}
        </div>
      )}
    </Responsivo>
  );
};
```

Esta técnica es brutalmente poderosa para separar la "Lógica" (el componente Responsivo) de la "Interfaz" (el componente App). Es un patrón que verás muchísimo en librerías avanzadas.