# 06 - El Estándar de la Industria: React Hook Form 🚀

Si has sobrevivido a los temas anteriores, te habrás dado cuenta de dos cosas:
1. Crear formularios controlados con `useState` requiere escribir muchísimo código repetitivo.
2. Las validaciones manuales se vuelven una pesadilla de sentencias `if`.

Para resolver esto, la industria adoptó **React Hook Form (RHF)**. Es una librería hiper-optimizada que abraza la filosofía de los **Componentes No Controlados** (Tema 3) para lograr un rendimiento extremo, pero te da una API tan fácil de usar como si fueran controlados.

## 📦 Instalación

Para añadirla a tu proyecto, abre la terminal y ejecuta:
```bash
npm install react-hook-form
```

---

## 🛠️ 1. El Hook `useForm`

Todo el poder de la librería reside en un solo Hook llamado `useForm`. Al invocarlo, nos devuelve varios métodos, pero los tres más importantes son:

- `register`: Conecta tus inputs HTML al estado interno de la librería.
- `handleSubmit`: Intercepta el envío del formulario, ejecuta las validaciones y solo te pasa los datos si todo está perfecto.
- `formState`: Un objeto que contiene el estado del formulario en tiempo real (`errors`, `isSubmitting`, `isDirty`, etc.).

```jsx
import { useForm } from "react-hook-form";

export const FormularioBasico = () => {
  const { register, handleSubmit } = useForm();

  // Esta función SOLO se ejecutará si el formulario pasa todas las validaciones
  const alEnviar = (datos) => {
    console.log("Datos listos para enviar a la API:", datos);
  };

  return (
    <form onSubmit={handleSubmit(alEnviar)}>
      {/* Usamos spread operator (...) para inyectar los superpoderes de RHF */}
      <input {...register("nombre")} placeholder="Nombre" />
      <input {...register("email")} placeholder="Email" />
      
      <button type="submit">Enviar</button>
    </form>
  );
};
```
**¡Míralo bien!** - No hay `useState`.
- No hay función `handleChange`.
- No hay `e.preventDefault()`. 
React Hook Form hace todo eso por ti bajo el capó usando referencias (`useRef`), por lo que **tu componente NO se re-renderiza cuando el usuario teclea**. ¡Rendimiento absoluto!

---

## 🛡️ 2. Validaciones Integradas

En lugar de escribir funciones complejas para validar, RHF usa los mismos estándares de validación de HTML5, pero potenciados con mensajes de error personalizados.

Le pasamos las reglas como un segundo parámetro a la función `register`.

```jsx
<input 
  {...register("nombre", { 
    required: "El nombre es obligatorio",
    minLength: { value: 3, message: "Mínimo 3 caracteres" } 
  })} 
/>

<input 
  {...register("edad", { 
    required: "Requerido",
    min: { value: 18, message: "Debes ser mayor de edad" } 
  })} 
/>
```

---

## 🚨 3. Mostrando los Errores

Para leer los mensajes de error que configuramos arriba, extraemos `errors` del objeto `formState`.

```jsx
import { useForm } from "react-hook-form";

export const FormularioValidado = () => {
  const { 
    register, 
    handleSubmit, 
    formState: { errors } // ⬅️ Extraemos los errores
  } = useForm();

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Email:</label>
        <input 
          {...register("email", { 
            required: "El correo es obligatorio",
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: "Formato de correo inválido"
            }
          })} 
        />
        {/* Renderizado condicional del error */}
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>

      <button type="submit">Enviar</button>
    </form>
  );
};
```

---

## ⏳ 4. UX Pro: Estados de Carga y Suciedad (`isSubmitting` / `isDirty`)

En el Tema 5 tuvimos que crear estados manuales para bloquear el botón mientras enviábamos datos. React Hook Form lo calcula automáticamente.

- `isSubmitting`: Es `true` mientras tu función asíncrona de envío se está ejecutando.
- `isDirty`: Es `true` si el usuario ha modificado al menos un campo del formulario desde su estado original. (Útil para deshabilitar el botón de "Guardar" si no ha cambiado nada).

```jsx
export const PerfilUsuario = () => {
  const { register, handleSubmit, formState: { isSubmitting, isDirty } } = useForm();

  const actualizarPerfil = async (datos) => {
    // RHF detecta esta promesa y cambia isSubmitting a true automáticamente
    await new Promise(resolve => setTimeout(resolve, 2000)); 
    console.log("Actualizado", datos);
  };

  return (
    <form onSubmit={handleSubmit(actualizarPerfil)}>
      <input {...register("biografia")} />
      
      {/* Solo se puede hacer clic si modificó algo (isDirty) Y no está cargando (!isSubmitting) */}
      <button type="submit" disabled={!isDirty || isSubmitting}>
        {isSubmitting ? "Guardando..." : "Guardar Cambios"}
      </button>
    </form>
  );
};
```

---

## ⚖️ Conclusión: ¿Por qué es el estándar?

1. **Rendimiento:** Al estar basado en `useRef`, aísla los re-renderizados solo a los elementos que muestran errores, no a todo el formulario.
2. **Menos Código:** Reduce el tamaño de tu código de formularios a la mitad.
3. **UX Impecable:** Maneja errores, focos y estados asíncronos nativamente.

Sin embargo, las reglas de validación en el `register` (ej. `pattern`, `minLength`) todavía pueden verse algo desordenadas si el formulario es gigantesco. En el siguiente y último tema, aprenderemos a separar completamente las reglas matemáticas de nuestro JSX usando **Esquemas de Validación (Zod)**.