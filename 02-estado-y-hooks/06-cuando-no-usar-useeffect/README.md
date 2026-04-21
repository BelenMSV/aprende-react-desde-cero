# 06 - Cuándo NO usar `useEffect`

Hace poco, el equipo central de React actualizó su documentación oficial con una sección revolucionaria: **"You Might Not Need an Effect"** (Puede que no necesites un Efecto). ¿Por qué? Porque el uso excesivo de efectos causa aplicaciones lentas, difíciles de depurar y propensas a errores de sincronización.

Existe una broma en la comunidad de React: *Si le das a un Junior un martillo (useEffect), tratará todo como un clavo.*

Como `useEffect` permite sincronizar datos y ejecutar código automáticamente, es muy tentador usarlo para controlar la lógica interna de tu aplicación. Sin embargo, **los efectos son una vía de escape** del paradigma de React. Deben usarse solo para salir de React y sincronizarse con sistemas externos.

Si usas efectos para la lógica que podría resolverse durante el renderizado o en un evento, harás que tu aplicación sea lenta (por re-renderizados dobles) y difícil de mantener.

Aquí tienes los 6 escenarios del mundo real donde debes borrar tus efectos.

---

## ❌ 1. Transformar datos (El costo del Doble Render)

Si tienes un estado y quieres crear una lista filtrada a partir de él, **nunca uses un efecto**.

❌ **El Anti-patrón:**
```jsx
const [usuarios, setUsuarios] = useState([]);
const [activos, setActivos] = useState([]);

useEffect(() => {
  const filtrados = usuarios.filter(u => u.isActive);
  setActivos(filtrados);
}, [usuarios]);
```
**¿Por qué es malo?** Esto provoca una cascada llamada "Doble Renderizado". React pinta todos los usuarios en la pantalla (Render 1). Luego ejecuta el efecto, actualiza el estado, y vuelve a pintar la pantalla completa con los activos (Render 2). El usuario ve un "parpadeo" y el procesador trabaja el doble.

✅ **La Solución (Estado Derivado durante el renderizado):**
Calcula al instante. Es un solo paso y ocurre antes de que la pantalla se pinte.
```jsx
const [usuarios, setUsuarios] = useState([]);

// ✅ BIEN: Se calcula al instante sin re-renderizados extra.
const activos = usuarios.filter(u => u.isActive);
```

---

## ❌ 2. Manejar Eventos del Usuario

Este es un error de lógica de negocio muy común. Imagina un botón de "Comprar". Cuando el usuario compra, quieres mostrar una notificación.

❌ **El Anti-patrón (Usar el efecto como observador):**
```jsx
const [compraExitosa, setCompraExitosa] = useState(false);

useEffect(() => {
  if (compraExitosa) {
    // 🚩 MAL: La notificación es resultado de un CLIC, no del ciclo de vida.
    mostrarNotificacion("¡Gracias por tu compra!");
  }
}, [compraExitosa]);

const manejarCompra = async () => {
  await procesarPago();
  setCompraExitosa(true);
};
```

✅ **La Solución (Lógica dentro del Evento):**
Si una acción es causada directamente por el usuario (un clic, un formulario), la lógica debe ir **dentro de la función manejadora del evento**.
```jsx
const manejarCompra = async () => {
  await procesarPago();
  setCompraExitosa(true);
  // ✅ BIEN: Sabemos exactamente qué causó la notificación.
  mostrarNotificacion("¡Gracias por tu compra!");
};
```

---

## ❌ 3. Efectos en Cadena (El Efecto Dominó)

Cuando tienes variables que dependen unas de otras, es tentador encadenarlas para que se actualicen en cascada.

❌ **El Anti-patrón:**
```jsx
useEffect(() => { setPasoB(pasoA + 1); }, [pasoA]);
useEffect(() => { setPasoC(pasoB * 2); }, [pasoB]);
useEffect(() => { enviarAlServidor(pasoC); }, [pasoC]);
```
**¿Por qué es terrible?** Cada eslabón fuerza un renderizado completo. Es ineficiente y si hay un bug, es casi imposible rastrear qué efecto disparó a los demás.

✅ **La Solución:**
Calcula todo lo que puedas en el evento inicial que disparó el cambio, o deriva los valores durante el renderizado.

---

## ❌ 4. Resetear el estado cuando cambian las Props

Imagina un componente `Perfil` que tiene un estado `comentario`. Cuando cambias del "Usuario A" al "Usuario B", quieres que el comentario se borre.

❌ **El Anti-patrón (Efecto de limpieza tardío):**
```jsx
useEffect(() => {
  setComentario('');
}, [usuarioId]);
```
**El problema:** Por un milisegundo, el Usuario B verá el comentario que escribiste para el Usuario A antes de que el efecto logre borrarlo.

✅ **La Solución (El poder de la `key`):**
No uses efectos. Pasa una prop `key` al componente desde su padre. Cuando la `key` cambia, React destruye el componente y crea uno nuevo con el estado inicial 100% limpio.
```jsx
// En el componente padre:
<Perfil key={usuarioId} id={usuarioId} />
```

---

## 🚀 5. Ajustar el estado durante el render (Patrón Avanzado)

A veces necesitas que un estado cambie *exactamente* cuando una prop cambia, pero no puedes usar una `key` porque quieres conservar el resto de la interfaz. Puedes hacerlo comparando el valor anterior **durante el render**.

✅ **La Solución Pro:**
```jsx
function Lista({ items }) {
  const [seleccionadoId, setSeleccionadoId] = useState(null);
  const [prevItems, setPrevItems] = useState(items);

  // Si los items cambian, ajustamos el estado INMEDIATAMENTE
  if (items !== prevItems) {
    setPrevItems(items);
    setSeleccionadoId(null);
  }
  // ...
}
```
*React detectará el `setState` durante el render y volverá a ejecutar la función inmediatamente ANTES de pintar la pantalla, evitando el temido doble renderizado visual.*

---

## ❌ 6. Inicializar toda la App (La trampa del Fetch manual)

Hasta ahora hemos enseñado que el "Arreglo Vacío" `[]` se usa para pedir datos a una API. 

```jsx
// 🚩 Lo que te enseñan en el 90% de los tutoriales básicos
useEffect(() => {
  fetch('/api/datos').then(res => res.json()).then(setDatos);
}, []);
```

Aunque esto funciona, **la documentación oficial de React hoy desaconseja el fetch manual con useEffect en aplicaciones grandes**. 

**¿Por qué?**
1. **No soporta SSR:** Tu app carga en blanco, luego carga JS, y luego pide datos.
2. **Cascadas de Red:** El componente Hijo no puede empezar su fetch hasta que el Padre termine de cargar.
3. **Falta de caché:** Si cambias de pestaña y vuelves, hace la petición de nuevo innecesariamente.

✅ **El Estándar Moderno de la Industria:**
Hoy en día usamos herramientas especializadas que manejan caché, re-intentos y carga paralela.
1. **Librerías de Fetching:** *React Query (TanStack Query)* o *SWR*. 
2. **Frameworks de React:** *Next.js* o *Remix*.
*(Nota: Aprenderemos React Query en el Módulo de APIs. Por ahora, concéntrate en dominar la lógica pura).*

---

## 💡 Resumen de Arquitecto (La Regla de Oro)

Antes de escribir `useEffect`, recorre esta lista mental:
1. **¿Puedo calcularlo en el render?** -> Borra el efecto (Usa Estado derivado).
2. **¿Es una acción del usuario?** -> Borra el efecto (Usa Manejador de eventos).
3. **¿Quiero resetear todo el componente?** -> Borra el efecto (Usa Prop `key`).

Si intentas sincronizarte con un sistema externo (API, manipulación manual del DOM, cronómetros de JavaScript o WebSockets), **entonces y solo entonces, usa `useEffect`.**