# 05 - Props y Comunicación entre Componentes

Hemos aprendido a crear componentes como si fueran piezas de Lego. Pero para que una aplicación sea útil, esas piezas necesitan información para mostrar. Aquí es donde entran las **Props** (abreviatura de *Properties* o propiedades).

Si un Componente es como una función de JavaScript, las **Props son los argumentos (parámetros)** que le pasamos a esa función para que su resultado (la interfaz) cambie.

---

## 🚚 Pasando Props (De Padre a Hijo)

En HTML tradicional, usamos atributos como `src` o `href`. En React, podemos inventar nuestros propios atributos y pasar cualquier tipo de dato de JavaScript: *strings*, números, *booleans*, *arrays*, objetos o incluso funciones.

```jsx
// Componente Padre (App.jsx)
import { TarjetaUsuario } from './TarjetaUsuario';

export const App = () => {
  return (
    <main>
      {/* Pasando un String (con comillas) */}
      <TarjetaUsuario nombre="Ana" />
      
      {/* Pasando números o booleanos (con llaves {}) */}
      <TarjetaUsuario nombre="Carlos" edad={30} esPremium={true} />
    </main>
  );
};
```

---

## 📥 Recibiendo Props

Cuando React renderiza un componente, agrupa todos los atributos que le pasaste en un único objeto llamado `props`. 

### 1. La forma clásica (Menos usada hoy en día)
Recibes el objeto completo y accedes a sus propiedades con el punto (`props.nombre`).

```jsx
export const TarjetaUsuario = (props) => {
  return (
    <div className="tarjeta">
      <h2>{props.nombre}</h2>
      <p>Edad: {props.edad}</p>
    </div>
  );
};
```

### 2. La forma Profesional (Desestructuración) 🏆
En proyectos reales, casi siempre desestructuramos el objeto directamente en los parámetros de la función. Es más limpio, más corto y te permite ver de un vistazo qué datos necesita el componente.

```jsx
export const TarjetaUsuario = ({ nombre, edad, esPremium }) => {
  return (
    <div className="tarjeta">
      <h2>{nombre}</h2>
      <p>Edad: {edad}</p>
      {esPremium ? <span>⭐ Usuario VIP</span> : null}
    </div>
  );
};
```

---

## 🛡️ Valores por Defecto (Default Props)

¿Qué pasa si el componente Padre olvida pasar una prop? El valor en el Hijo será `undefined`, lo que podría romper tu diseño o generar errores.

Para evitarlo, usamos la sintaxis de **valores por defecto de ES6** (JavaScript moderno) directamente en la desestructuración:

```jsx
// Si no me pasan la edad, por defecto será "Desconocida"
export const TarjetaUsuario = ({ nombre, edad = "Desconocida", rol = "Invitado" }) => {
  return (
    <div>
      <h2>{nombre} ({rol})</h2>
      <p>Edad: {edad}</p>
    </div>
  );
};
```

---

## 🗣️ Comunicación de Hijo a Padre (Pasando Funciones)

Recordemos la regla de React: **Los datos siempre fluyen hacia abajo** (del Padre al Hijo). Un componente Hijo no puede pasarle variables directamente a su Padre.

Entonces, ¿cómo hace un botón (Hijo) para avisarle a su Padre que ha sido clickeado? **El Padre le pasa una función al Hijo a través de las props.**

```jsx
// 1. EL PADRE crea la función y se la pasa al Hijo
export const App = () => {
  const saludarDesdeHijo = (nombreHijo) => {
    alert(`¡Hola! Mi hijo ${nombreHijo} me ha llamado.`);
  };

  return <BotonMagico alHacerClic={saludarDesdeHijo} />;
};

// 2. EL HIJO recibe la función y la ejecuta cuando ocurre un evento
export const BotonMagico = ({ alHacerClic }) => {
  return (
    <button onClick={() => alHacerClic('Pedrito')}>
      Llamar a Papá
    </button>
  );
};
```
Esta técnica se conoce a menudo como **"Lifting State Up"** (Elevar el estado) o "Callbacks como Props".

---

## 🛑 La Regla de Oro: Las Props son de Solo Lectura

Nunca, bajo ninguna circunstancia, intentes modificar las props dentro de un componente hijo. **Las props son inmutables (read-only).**

❌ **Prohibido:**
```jsx
export const MiComponente = ({ titulo }) => {
  // ERROR: No puedes reasignar una prop
  titulo = "Nuevo Título"; 
  return <h1>{titulo}</h1>;
};
```

Si necesitas que un dato cambie con el tiempo, no debes usar Props, debes usar **Estado (`useState`)** (que aprenderemos en el Módulo 2).

---

## 📦 Concepto Avanzado 1: La prop mágica `children`

Hay una prop especial en React que no necesitas enviar explícitamente como atributo. Se llama `children`. Contiene todo lo que pongas **entre las etiquetas de apertura y cierre** de un componente.

Esto es fundamental para el patrón de diseño llamado **Composición**. Imagina que quieres crear una Tarjeta (`Card`) que a veces contenga texto, y otras veces contenga una imagen o un formulario.

```jsx
// 1. Creamos el componente contenedor que recibe "children"
export const Card = ({ children }) => {
  return (
    <div className="card-estilizada-con-sombra">
      {children}
    </div>
  );
};

// 2. Usamos el componente inyectando contenido dentro
export const App = () => {
  return (
    <main>
      <Card>
        <h2>Título de la tarjeta</h2>
        <p>Este párrafo es el "children".</p>
      </Card>

      <Card>
        <img src="foto.jpg" alt="Otra tarjeta" />
        <button>Comprar</button>
      </Card>
    </main>
  );
};
```
Gracias a `children`, el componente `Card` no necesita saber qué va a contener de antemano; solo se encarga de los estilos y la estructura exterior.

---

## 📨 Concepto Avanzado 2: Reenviando Props (El operador Rest/Spread)

A veces creas un componente que es solo una versión mejorada de un elemento HTML normal (como un botón personalizado). Quieres que acepte `onClick`, `disabled`, `type`, `onMouseEnter` y cualquier otro atributo de botón, pero no quieres escribirlos todos manualmente en la desestructuración.

Usamos el **operador Rest (`...`)** para capturar "el resto de las props" y el **operador Spread (`...`)** para inyectarlas:

```jsx
// Extraemos "texto" y "color", y guardamos el resto en "restProps"
export const BotonPersonalizado = ({ texto, color = "blue", ...restProps }) => {
  return (
    <button 
      style={{ backgroundColor: color }} 
      {...restProps} // Aquí inyectamos onClick, disabled, etc. automáticamente
    >
      {texto}
    </button>
  );
};

// Uso en el Padre:
<BotonPersonalizado 
  texto="Enviar" 
  color="green" 
  type="submit" 
  disabled={false} 
  onClick={() => console.log('Click!')} 
/>
```

---

## 🧠 Glosario: "Prop Drilling"
A medida que tu aplicación crezca, te encontrarás pasando props desde un abuelo, a un padre, a un hijo, a un nieto... solo porque el nieto necesita el dato. A este fenómeno se le llama **Prop Drilling** (Perforación de Props). En módulos posteriores aprenderemos a solucionar esto utilizando la *Context API* o gestores de estado como *Zustand / Redux*.