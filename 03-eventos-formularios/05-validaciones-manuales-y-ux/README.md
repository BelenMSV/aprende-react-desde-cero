# 05 - Validaciones Manuales y Experiencia de Usuario (UX)

La regla de oro del desarrollo web es: **Nunca confíes en los datos del usuario**. 

Las validaciones ocurren en dos lugares:
1. **En el Backend (Seguridad):** Es obligatorio. Evita que un hacker inyecte código malicioso en la base de datos.
2. **En el Frontend (Experiencia de Usuario - UX):** Es opcional, pero vital. Evita que el usuario tenga que esperar 3 segundos para que el servidor le diga que olvidó poner su apellido.

En este tema aprenderemos a crear un sistema de validación manual robusto y a manejar el estado asíncrono para dar *feedback* visual (spinners, botones bloqueados).

---

## 🛑 1. El Estado de los Errores

Si tenemos un formulario controlado con un estado para los datos (`form`), necesitamos un estado "gemelo" para guardar los mensajes de error de esos datos (`errores`).

```jsx
const [form, setForm] = useState({ email: '', password: '' });
// Guardaremos los errores usando las mismas claves
const [errores, setErrores] = useState({}); 
```

Si el email está mal, actualizaremos el estado a: `{ email: "El correo no es válido" }`. Esto nos permitirá mostrar el texto rojo exactamente debajo del input correspondiente.

---

## 🔍 2. La Función Validadora

El patrón más limpio es separar la lógica de validación de la lógica de envío. Creamos una función que revise todo el formulario, actualice el estado de errores y devuelva `true` si todo está perfecto, o `false` si hay fallos.

```jsx
const validarFormulario = () => {
  let nuevosErrores = {}; // Empezamos asumiendo que no hay errores

  // 1. Validar campos vacíos
  if (!form.nombre.trim()) {
    nuevosErrores.nombre = "El nombre es obligatorio.";
  }

  // 2. Validar longitud
  if (form.password.length < 6) {
    nuevosErrores.password = "La contraseña debe tener al menos 6 caracteres.";
  }

  // 3. Validar con Expresiones Regulares (Regex)
  const regexEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!regexEmail.test(form.email)) {
    nuevosErrores.email = "El formato del correo es inválido.";
  }

  setErrores(nuevosErrores);

  // Si el objeto no tiene claves, significa que no hubo errores (es válido)
  return Object.keys(nuevosErrores).length === 0;
};
```

---

## ⏳ 3. UX Senior: El estado `isSubmitting`

Imagínate esto: El usuario hace clic en "Registrarse". Tu código hace una petición a la API que tarda 2 segundos. Como el usuario no ve que pase nada, hace clic 4 veces más. ¡Acabas de enviar 5 peticiones idénticas a tu servidor!

Para evitar esto, usamos un patrón fundamental de UX: **Deshabilitar el botón mientras se procesa la solicitud.**

```jsx
const [isSubmitting, setIsSubmitting] = useState(false);
```

### Uniendo las piezas: El Formulario Completo

```jsx
import { useState } from 'react';

export const FormularioRegistro = () => {
  const [form, setForm] = useState({ nombre: '', email: '', password: '' });
  const [errores, setErrores] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
    
    // UX Pro Tip: Borrar el error en tiempo real cuando el usuario empieza a corregirlo
    if (errores[name]) {
      setErrores(prev => ({ ...prev, [name]: null }));
    }
  };

  const validarFormulario = () => {
    let nuevosErrores = {};
    if (!form.nombre.trim()) nuevosErrores.nombre = "Requerido";
    if (!form.password || form.password.length < 6) nuevosErrores.password = "Mínimo 6 caracteres";
    
    setErrores(nuevosErrores);
    return Object.keys(nuevosErrores).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    // 1. Detener ejecución si la validación falla
    if (!validarFormulario()) return;

    // 2. Bloquear la Interfaz
    setIsSubmitting(true);

    try {
      // Simulamos una llamada a la API que tarda 2 segundos
      await new Promise(resolve => setTimeout(resolve, 2000));
      console.log("¡Usuario registrado con éxito!", form);
      // Limpiar formulario tras éxito
      setForm({ nombre: '', email: '', password: '' });
    } catch (error) {
      setErrores({ global: "Error en el servidor. Intenta de nuevo." });
    } finally {
      // 3. Desbloquear la Interfaz SIEMPRE (incluso si hubo error)
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="formulario">
      {errores.global && <div className="alerta-roja">{errores.global}</div>}

      <div className="grupo-input">
        <label>Nombre:</label>
        <input name="nombre" value={form.nombre} onChange={handleChange} />
        {/* Renderizado condicional del error */}
        {errores.nombre && <span className="error-texto">{errores.nombre}</span>}
      </div>

      <div className="grupo-input">
        <label>Contraseña:</label>
        <input type="password" name="password" value={form.password} onChange={handleChange} />
        {errores.password && <span className="error-texto">{errores.password}</span>}
      </div>

      {/* El botón se deshabilita y cambia su texto si isSubmitting es true */}
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Procesando...' : 'Crear Cuenta'}
      </button>
    </form>
  );
};
```

---

## 🤯 4. El dolor de cabeza de las Validaciones Manuales

Hemos escrito casi 50 líneas de código para validar dos simples campos. Imagina hacer esto para un formulario médico con 40 campos, dependencias condicionales (ej. *"Si marcó 'Embarazada', el campo 'Meses' es obligatorio"*) y validaciones cruzadas (*"La contraseña y confirmar contraseña deben ser iguales"*).

Las validaciones manuales se vuelven **inmantenibles** rápidamente y ensucian nuestros componentes con cientos de sentencias `if`.

Por eso, en la industria profesional, **nadie escribe esto a mano**. En los siguientes dos temas aprenderemos el combo mágico definitivo que usan las empresas Top: **React Hook Form + Zod**.