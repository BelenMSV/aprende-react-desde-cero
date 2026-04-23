# 03 - Patrones de Diseño: Componentes Compuestos 💎

Construir un componente reutilizable no significa hacerlo capaz de adivinar todo lo que el usuario quiere hacer a través de `props`. Significa construir piezas pequeñas que trabajen juntas en armonía.

Piensa en cómo funciona el HTML nativo. No usas `<select options={['A', 'B']} />`. Usas un `<select>` padre y múltiples `<option>` hijos. Ese es el patrón de **Componentes Compuestos**.

---

## ❌ 1. El Anti-patrón: El "Dios de las Props"

Imagina que te piden hacer un componente `<Tarjeta />` (Card). Un desarrollador Junior intentará prever todos los casos de uso posibles pasándole variables:

```jsx
// 🚩 Esto es frágil, difícil de extender y terrible de leer
<Tarjeta 
  titulo="Plan Premium"
  subtitulo="Para equipos grandes"
  imagenUrl="/img/premium.png"
  tieneBotonAccion={true}
  textoBoton="Comprar ahora"
  colorBoton="rojo"
  posicionBoton="derecha"
  tieneFooter={true}
  textoFooter="Cancelación gratuita"
/>
```
¿Qué pasa si mañana te piden que el botón de acción sea en realidad un enlace `<a>` o un menú desplegable? Tu componente colapsará porque fue diseñado para ser rígido.

---

## ✅ 2. La Solución: Compound Components

En lugar de un solo componente monolítico, vamos a crear un "Padre" que comparta su estado, y varios "Hijos" que representen las partes visuales.

### Paso 1: El Contexto y el Padre
Usamos Context API (¿recuerdas el Módulo 5?) pero de forma **local**, solo para este componente y sus hijos directos.

```jsx
// src/components/Card/Card.jsx
import { createContext, useContext } from 'react';

// 1. Creamos un Contexto privado solo para la Tarjeta
const CardContext = createContext();

// 2. El Componente Padre (El Cascarón)
export const Card = ({ children, isHoverable = false }) => {
  return (
    <CardContext.Provider value={{ isHoverable }}>
      <div className={`tarjeta-base ${isHoverable ? 'hover-effect' : ''}`}>
        {children}
      </div>
    </CardContext.Provider>
  );
};
```

### Paso 2: Los Componentes Hijos (Sub-componentes)
Creamos las piezas de Lego que el desarrollador podrá ensamblar.

```jsx
// src/components/Card/Card.jsx (continuación)

// 3. Los Hijos
const Title = ({ children }) => <h2 className="tarjeta-titulo">{children}</h2>;
const Image = ({ src, alt }) => <img className="tarjeta-img" src={src} alt={alt} />;

const ActionButton = ({ onClick, children }) => {
  // Un hijo puede leer el contexto del padre si lo necesita
  const { isHoverable } = useContext(CardContext);
  return (
    <button onClick={onClick} className={`btn ${isHoverable ? 'animado' : ''}`}>
      {children}
    </button>
  );
};

// 4. EL TRUCO SENIOR: Adjuntamos los hijos al objeto Padre
Card.Title = Title;
Card.Image = Image;
Card.ActionButton = ActionButton;
```

---

## 🏗️ 3. El Resultado: Código Nivel Arquitecto

Ahora mira cómo el desarrollador utiliza tu componente. Es expresivo, limpio y **completamente flexible**.

```jsx
import { Card } from './components/Card';

export const PantallaPrecios = () => {
  return (
    <div className="grid">
      {/* Tarjeta 1: Completa */}
      <Card isHoverable>
        <Card.Image src="/img/pro.png" alt="Pro" />
        <Card.Title>Plan Profesional</Card.Title>
        <p>Todo lo que necesitas para crecer.</p> {/* HTML normal convive sin problema */}
        <Card.ActionButton onClick={() => alert('¡Comprado!')}>
          Suscribirse
        </Card.ActionButton>
      </Card>

      {/* Tarjeta 2: Solo Texto. ¡No hay que pasar variables en blanco (tieneBoton={false})! */}
      <Card>
        <Card.Title>Aviso Importante</Card.Title>
        <p>El sistema estará en mantenimiento mañana.</p>
      </Card>
    </div>
  );
};
```

### 🏆 ¿Por qué este patrón es superior?
1. **Inversión de Control:** Tú (el creador de la librería) controlas la lógica interna y el estilo base, pero el usuario final decide el *orden* de los elementos y *qué* elementos mostrar.
2. **Cero Props Inútiles:** Ya no necesitas pasar `tieneImagen={false}`. Si no quieres imagen, simplemente no escribes `<Card.Image />`.
3. **Mantenibilidad:** Si el equipo de diseño pide un nuevo elemento (ej. una etiqueta de "Descuento"), simplemente creas un nuevo hijo `<Card.Badge />` sin romper el código de las miles de tarjetas que ya existen en la aplicación.

*(Nota: Este es el patrón exacto que utilizan librerías como Headless UI, Radix UI y shadcn/ui. Aprender a leerlo y escribirlo te permitirá entender el código fuente de los mejores proyectos del mundo).*