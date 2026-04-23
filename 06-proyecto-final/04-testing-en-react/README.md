# 04 - La Malla de Seguridad: Testing en React 🧪

El código sin pruebas automatizadas es código heredado (legacy) desde el momento en que lo escribes. Escribir tests no es "perder el tiempo", es **comprar tiempo futuro** que no tendrás que gastar buscando bugs en producción.

En el ecosistema moderno de Vite, la herramienta por defecto para ejecutar pruebas es **Vitest** (es muchísimo más rápido que el antiguo Jest). Para probar componentes de React, usamos **React Testing Library (RTL)**.

## 📦 Instalación

```bash
npm install -D vitest @testing-library/react @testing-library/dom @testing-library/jest-dom jsdom @testing-library/user-event
```

---

## 🧠 1. La Filosofía de React Testing Library

Antes de escribir una línea de código, debes entender la regla de oro de RTL:
**"Cuanto más se parezcan tus pruebas a la forma en que el software se usa, más confianza te darán."**

* ❌ **NO probamos detalles de implementación:** No probamos si un `useState` cambió a `true`. Al usuario no le importa tu `useState`.
* ✅ **Probamos el comportamiento visual:** Probamos si, al hacer clic en un botón, aparece un texto específico en la pantalla. Al usuario le importa lo que ve.

---

## 🏗️ 2. Tu Primer Test (Renderizado Básico)

Imagina que tenemos un componente simple:

```jsx
// src/components/Saludo.jsx
export const Saludo = ({ nombre }) => {
  return (
    <div>
      <h1>Bienvenido a la App</h1>
      {nombre ? <p>Hola, {nombre}</p> : <p>Inicia sesión por favor</p>}
    </div>
  );
};
```

Vamos a escribir su prueba. Los archivos de test se nombran igual que el componente pero terminan en `.test.jsx`.

```jsx
// src/components/Saludo.test.jsx
import { render, screen } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { Saludo } from './Saludo';

// 'describe' agrupa un bloque de pruebas
describe('Componente <Saludo />', () => {

  // 'it' o 'test' es una prueba individual
  it('debe mostrar el mensaje por defecto si no hay nombre', () => {
    // 1. Arrange (Preparar): Renderizamos el componente en un DOM virtual (jsdom)
    render(<Saludo />);

    // 2. Act (Actuar): Buscamos elementos en la pantalla
    // Usamos getByText o getByRole, igual que un usuario buscaría con sus ojos
    const mensaje = screen.getByText(/inicia sesión por favor/i); // /i ignora mayúsculas

    // 3. Assert (Afirmar): Comprobamos que el elemento existe
    expect(mensaje).toBeInTheDocument();
  });

  it('debe saludar al usuario si se pasa la prop nombre', () => {
    render(<Saludo nombre="Alex" />);
    
    // Buscamos un encabezado (h1-h6) específico
    const titulo = screen.getByRole('heading', { level: 1 });
    const mensaje = screen.getByText('Hola, Alex');

    expect(titulo).toBeInTheDocument();
    expect(mensaje).toBeInTheDocument();
  });

});
```

---

## 🖱️ 3. Interactividad y Eventos del Usuario

Las UIs son interactivas. Para simular clics o escritura de teclado de forma realista, usamos la librería `@testing-library/user-event`.

```jsx
// src/components/Interruptor.jsx
import { useState } from 'react';

export const Interruptor = () => {
  const [encendido, setEncendido] = useState(false);

  return (
    <div>
      <p>El sistema está: {encendido ? 'ON' : 'OFF'}</p>
      <button onClick={() => setEncendido(!encendido)}>
        Alternar
      </button>
    </div>
  );
};
```

**El Test:**

```jsx
// src/components/Interruptor.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect } from 'vitest';
import { Interruptor } from './Interruptor';

describe('Componente <Interruptor />', () => {
  
  it('debe cambiar de OFF a ON al hacer clic en el botón', async () => {
    // 1. Configuramos el actor (el usuario)
    const user = userEvent.setup();
    render(<Interruptor />);

    // 2. Estado inicial
    expect(screen.getByText('El sistema está: OFF')).toBeInTheDocument();

    // 3. Actuamos: El usuario hace clic en el botón
    const boton = screen.getByRole('button', { name: /alternar/i });
    await user.click(boton); // Siempre es asíncrono

    // 4. Afirmamos el nuevo estado visual
    expect(screen.getByText('El sistema está: ON')).toBeInTheDocument();
  });

});
```

---

## 🌐 4. Nivel Arquitecto: Mocking y Providers

Aquí es donde caen la mayoría de los Juniors. Si intentas renderizar un componente que usa React Router (`<Link>`) o TanStack Query (`useQuery`), **el test va a explotar**.

¿Por qué? Porque el componente asume que está envuelto en un `<BrowserRouter>` o en un `<QueryClientProvider>`. En los tests, renderizamos el componente *aislado*.

### La Solución: El Wrapper Personalizado
Para componentes complejos, no usamos el `render` normal. Creamos un `render` personalizado que envuelve el componente en todos los contextos necesarios de la aplicación real.

```jsx
// src/tests/utils.jsx (Archivo de utilidades de testeo)
import { render } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const createTestQueryClient = () => new QueryClient({
  defaultOptions: { queries: { retry: false } } // Apagamos reintentos en tests
});

// Envolvemos los hijos con los Providers necesarios
const AllTheProviders = ({ children }) => {
  const testQueryClient = createTestQueryClient();
  return (
    <QueryClientProvider client={testQueryClient}>
      <MemoryRouter> {/* MemoryRouter es especial para testear React Router */}
        {children}
      </MemoryRouter>
    </QueryClientProvider>
  );
};

// Sobrescribimos el método render
const customRender = (ui, options) =>
  render(ui, { wrapper: AllTheProviders, ...options });

// Exportamos todo y sobrescribimos el render
export * from '@testing-library/react';
export { customRender as render };
```

A partir de ahora, en tus tests importarás `render` desde `src/tests/utils.jsx` en lugar de la librería original, y tus componentes dejarán de explotar. 

---

### 🏆 Resumen: Reglas de Oro del Testing

1. **Testea Casos de Uso, no código:** Si refactorizas un componente (ej. cambias `useState` por `useReducer`) sin cambiar la interfaz visual, **el test no debería romperse**. Si se rompe, hiciste un mal test.
2. **Prioriza el `getByRole`:** Es la mejor forma de asegurar que tu aplicación es accesible (para lectores de pantalla). Si no puedes seleccionar algo por `role`, tu HTML probablemente esté mal estructurado.
3. **No busques el 100% de cobertura (Coverage):** Testear componentes tontos (ej. un `<h1>` estático) es una pérdida de tiempo. Dedica tus tests a la lógica de negocio compleja, cálculos, permisos y flujos de usuario críticos (ej. proceso de pago).