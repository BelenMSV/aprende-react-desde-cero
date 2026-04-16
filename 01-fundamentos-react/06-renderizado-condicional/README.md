# 06 - Renderizado Condicional

Tus componentes casi nunca van a mostrar exactamente la misma interfaz todo el tiempo. Dependiendo de los datos (props o estado), querrás mostrar un componente de carga, un mensaje de error o la interfaz principal.

A esto le llamamos **Renderizado Condicional**, y en React no usamos directivas especiales (como el `v-if` de Vue o el `*ngIf` de Angular). En React, usamos **JavaScript puro**.

Existen 3 formas principales de hacerlo. Cada una tiene su momento y lugar ideal.

---

## 1. El Retorno Temprano (Early Return con `if`)

Es la forma más tradicional. Si se cumple una condición, interrumpimos la ejecución del componente y devolvemos un JSX distinto. Es ideal cuando **toda la pantalla** debe cambiar (por ejemplo, mostrar una pantalla de carga a pantalla completa).

```jsx
export const PerfilUsuario = ({ isLoading, error, usuario }) => {
  // 1. Si está cargando, mostramos solo esto y salimos.
  if (isLoading) {
    return <h2>Cargando perfil... ⏳</h2>;
  }

  // 2. Si hubo un error, mostramos esto y salimos.
  if (error) {
    return <h2>Error: No se pudo cargar el perfil ❌</h2>;
  }

  // 3. Si todo fue bien, renderizamos la interfaz normal.
  return (
    <div>
      <h1>Bienvenido, {usuario.nombre}</h1>
      <p>Email: {usuario.email}</p>
    </div>
  );
};
```
*Ventaja:* Mantiene el código limpio y evita tener que envolver el JSX final en condicionales gigantes.

---

## 2. El Operador Ternario (`condición ? true : false`)

Recuerda que dentro del JSX (dentro de las llaves `{}`) no podemos usar sentencias `if`. Si necesitas cambiar solo **una parte pequeña** de la interfaz (un "esto o lo otro"), el operador ternario es tu mejor amigo.

```jsx
export const BotonSuscripcion = ({ esPremium }) => {
  return (
    <div>
      <h3>Tu plan actual:</h3>
      {esPremium ? (
        <button className="btn-gold">⭐ Cancelar Premium</button>
      ) : (
        <button className="btn-blue">🚀 Mejorar a Premium</button>
      )}
    </div>
  );
};
```
*Cuándo usarlo:* Cuando tienes un caso de "If / Else" claro dentro del marcado.

---

## 3. El Operador Lógico AND (`&&`)

A veces no quieres un "esto o lo otro", simplemente quieres **"mostrar esto o no mostrar NADA"**. Para esos casos, usamos el operador `&&`.

En JavaScript, `true && expresion` siempre evalúa a `expresion`, y `false && expresion` evalúa a `false` (React ignora los booleanos y no pinta nada).

```jsx
export const Notificaciones = ({ tieneMensajesNuevos }) => {
  return (
    <nav>
      <span>Inicio</span>
      <span>Perfil</span>
      
      {/* Si es true, renderiza el globo rojo. Si es false, no renderiza nada. */}
      {tieneMensajesNuevos && <span className="alerta">¡Tienes mensajes!</span>}
    </nav>
  );
};
```

---

## ⚠️ La Trampa del Operador `&&` (Pregunta de Entrevista)

El operador `&&` es peligroso si no conoces a fondo cómo funciona JavaScript con los valores *falsy* (falsos por naturaleza, como `0`, `NaN` o `""`).

Imagina que quieres mostrar una lista de tareas solo si tienes tareas guardadas.

❌ **El Error de Novato:**
```jsx
const tareas = []; // Array vacío (longitud 0)

return (
  <div>
    <h1>Mis Tareas</h1>
    {tareas.length && <ListaTareas tareas={tareas} />}
  </div>
);
```
**¿Qué pasa aquí?** JavaScript evalúa `tareas.length` (que es `0`). Como `0` es un valor *falsy*, el operador `&&` se detiene y **devuelve el `0`**. React renderizará en tu pantalla literalmente un número "0" debajo de tu título.

✅ **La Solución Profesional:**
Asegúrate SIEMPRE de que la parte izquierda del `&&` sea un booleano real.

```jsx
// Opción A: Hacer una comparación explícita (Recomendada)
{tareas.length > 0 && <ListaTareas tareas={tareas} />}

// Opción B: Forzar el booleano con doble negación
{!!tareas.length && <ListaTareas tareas={tareas} />}
```

---

## 🧼 Pro Tips: Escribiendo Código Limpio (Clean Code)

A medida que tus componentes crezcan, inyectar demasiada lógica dentro del `return` hará que tu código sea ilegible. Los desarrolladores Senior aplican estas tres técnicas para mantener el JSX limpio:

### 1. Extraer condiciones complejas a variables
Si una condición tiene múltiples comprobaciones (`&&`, `||`, `!`), sácala del JSX y ponle un nombre descriptivo. Tu código se leerá como un libro.

```jsx
export const PanelAdmin = ({ usuario }) => {
  // ❌ Mal: JSX saturado
  // return {usuario.isLogged && usuario.role === 'ADMIN' && !usuario.isBanned && <BotonesAdmin />}

  // ✅ Bien: Lógica extraída
  const puedeVerPanel = usuario.isLogged && usuario.role === 'ADMIN' && !usuario.isBanned;

  return (
    <div>
      {puedeVerPanel && <BotonesAdmin />}
    </div>
  );
};
```

### 2. El Patrón "Diccionario" (Object Map) para evitar ternarios anidados
Anidar ternarios (`condicion ? esto : otraCondicion ? loOtro : aquello`) es considerado una pésima práctica. Si tienes más de dos opciones, usa un objeto o una función.

```jsx
// ❌ Mal: Ternario anidado (difícil de leer)
const EstadoPedido = ({ estado }) => {
  return estado === 'PENDIENTE' ? <IconoReloj /> : estado === 'ENVIADO' ? <IconoCamion /> : <IconoCheck />;
};

// ✅ Bien: Patrón Diccionario
const EstadoPedido = ({ estado }) => {
  const iconosPorEstado = {
    PENDIENTE: <IconoReloj />,
    ENVIADO: <IconoCamion />,
    ENTREGADO: <IconoCheck />
  };

  // Si el estado no existe en el objeto, mostramos un fallback por defecto
  return <div>{iconosPorEstado[estado] || <IconoDesconocido />}</div>;
};
```

### 3. Clases CSS Condicionales (Template Literals)
Muchas veces el condicional no es para ocultar un componente, sino para cambiar su estilo. En lugar de duplicar código, inyectamos la clase condicionalmente usando *Template Literals* (las comillas invertidas de JavaScript).

```jsx
export const Boton = ({ isActive }) => {
  return (
    <button className={`btn-base ${isActive ? 'btn-activo' : 'btn-inactivo'}`}>
      Hacer clic
    </button>
  );
};
```
*(Nota: En proyectos grandes, es muy común instalar librerías como `clsx` o `classnames` para manejar esto de forma aún más elegante).*