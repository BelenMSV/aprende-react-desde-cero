# 05 - Dependencias y Limpieza (Cleanup) en `useEffect`

En el tema anterior aprendimos qué es un efecto secundario. Ahora aprenderemos a controlarlo con precisión quirúrgica. 

El Hook `useEffect` tiene un lenguaje propio basado en su segundo parámetro: el **Arreglo de Dependencias**. Este arreglo le dice a React exactamente *cuándo* debe volver a ejecutar tu código. Sin él, tus aplicaciones consumirán demasiada memoria y se comportarán de forma impredecible.

---

## 🚦 1. El Arreglo de Dependencias: Los 3 Escenarios

### Escenario A: Sin Arreglo (Peligro ⚠️)
Si no pones el segundo parámetro, el efecto se ejecuta **después de CADA renderizado**.
```jsx
useEffect(() => {
  console.log("Me ejecuto al nacer y CADA VEZ que cambia CUALQUIER estado/prop");
}); // 🚩 Sin corchetes
```
*Uso real:* Casi nulo. Suele causar problemas de rendimiento o bucles infinitos.

### Escenario B: Arreglo Vacío `[]` (Solo Montaje 📦)
El efecto se ejecuta **una sola vez**, justo después de que el componente aparece en pantalla por primera vez.
```jsx
useEffect(() => {
  console.log("Solo me ejecuto al nacer (Mount)");
}, []); // ✅ Corchetes vacíos
```
*Uso real:* Hacer la primera petición a una API (Fetch), inicializar un mapa de Google Maps, leer del `localStorage` al cargar la página.

### Escenario C: Con Variables `[variable]` (Sincronización 🔄)
El efecto se ejecuta al inicio y **solo cuando una de esas variables cambia**.
```jsx
useEffect(() => {
  console.log("Solo me ejecuto si el ID del usuario o el nombre han cambiado");
}, [userId, nombre]); // 🔍 Vigilando variables específicas
```
*Uso real:* Volver a hacer un fetch cuando cambia el ID en la URL, auto-guardar un borrador cuando el usuario modifica un campo.

---

## 🧹 2. La Función de Limpieza (Cleanup Function)

A veces, un efecto "ensucia" el navegador. Por ejemplo, si creas un cronómetro con `setInterval`, ese cronómetro seguirá corriendo en la memoria del navegador ¡incluso si el usuario se va a otra página y el componente desaparece!

Para evitar esta fuga de memoria (*Memory Leak*), `useEffect` te permite devolver una función (`return () => {}`). React ejecutará esta función en dos momentos exactos:
1. **Justo antes** de volver a ejecutar el efecto (para limpiar el rastro del efecto anterior).
2. Cuando el componente se **desmonta** (muere).

```jsx
export const Cronometro = () => {
  useEffect(() => {
    // 1. Iniciamos el efecto
    const timerId = setInterval(() => {
      console.log("Tick - Consumiendo memoria");
    }, 1000);

    // 2. Le decimos a React cómo limpiarlo
    return () => {
      clearInterval(timerId);
      console.log("Temporizador destruido 🧹");
    };
  }, []); 

  return <div>Cronómetro activo en consola</div>;
};
```

---

## 🚀 3. Nivel Senior: Resolviendo "Race Conditions" con `AbortController`

En el tema anterior hablamos del temido bug de la **Condición de Carrera** (haces clic rápido entre el Perfil A y el Perfil B, y el resultado lento del Perfil A sobreescribe la pantalla).

La forma profesional de arreglar esto en la industria actual no es solo ignorar el resultado, sino **cancelar la petición HTTP real** usando la API nativa de JavaScript `AbortController`.

```jsx
useEffect(() => {
  // 1. Creamos el controlador
  const controller = new AbortController(); 

  const cargarPerfil = async () => {
    try {
      // 2. Le pasamos la "señal" de aborto al fetch
      const respuesta = await fetch(`/api/usuarios/${userId}`, {
        signal: controller.signal 
      });
      const datos = await respuesta.json();
      setUsuario(datos);
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('Petición cancelada porque el usuario cambió rápido de ID');
      }
    }
  };

  cargarPerfil();

  // 3. La magia de la Limpieza: Si el userId cambia, 
  // cancelamos instantáneamente el fetch anterior que estuviera en curso.
  return () => {
    controller.abort(); 
  };
}, [userId]); // Vigilamos el userId
```

---

## 🤥 4. La Regla de Oro: NUNCA le mientas a React (ESLint)

Cuando empiezas a usar el Escenario C, el plugin oficial `eslint-plugin-react-hooks` (incluido en Vite) te lanzará una advertencia llamada `exhaustive-deps` si usas una variable dentro del efecto pero *olvidas* ponerla en el arreglo de dependencias.

❌ **El instinto del Junior:**
*"Esta advertencia es molesta, no quiero que el efecto se vuelva a ejecutar si cambia esta variable, así que la borraré del arreglo o apagaré el linter"*.

✅ **La realidad del Senior:**
¡Mentirle a React sobre las dependencias causa los bugs más difíciles de rastrear (los famosos *Stale Closures*, donde tu efecto usa variables viejas que ya no existen)! 

**Si React te pide una dependencia, TÚ DEBES PONERLA.** Si ponerla causa que el efecto se ejecute demasiadas veces, el problema no es el linter, es que **la arquitectura de tu efecto está mal**.
- Solución A: Mueve la variable o función DENTRO del `useEffect`.
- Solución B: Saca la variable FUERA del componente (si es una constante).
- Solución C: Usa Hooks avanzados como `useCallback` o `useMemo` (que veremos en el Módulo de Optimización).

---

## 🤯 5. La Trampa de las Referencias (Objetos en el Array)

¿Recuerdas el Tema 2 sobre la inmutabilidad de los Objetos? Ese conocimiento es vital aquí.

```jsx
const config = { modo: "oscuro" }; // Esto es un Objeto

useEffect(() => {
  aplicarTema(config);
}, [config]); // 🚩 ¡PELIGRO DE BUCLE INFINITO!
```

En JavaScript, `{}` no es igual a `{}`. Cada vez que React renderiza el componente, crea un objeto `config` **nuevo en memoria**. 
El `useEffect` mira el arreglo y dice: *"Vaya, este objeto `config` tiene una dirección de memoria distinta al del renderizado anterior. Debo ejecutar el efecto de nuevo"*.

¡Esto causa un efecto que se ejecuta infinitamente aunque el contenido del objeto sea el mismo!
**Regla:** Intenta poner solo valores primitivos (strings, números, booleanos) en el arreglo de dependencias. Si necesitas usar objetos o funciones, aprende a memorizarlos.