# 07 - Validación por Esquemas con Zod 💎

A medida que las aplicaciones crecen, las reglas de validación se vuelven complejas. Mezclar reglas matemáticas o expresiones regulares dentro de nuestro JSX (como vimos en el Tema 6) hace que los componentes sean difíciles de leer y testear.

El estándar actual de la industria es separar las validaciones en **Esquemas**. Un esquema es un objeto que define exactamente qué forma deben tener tus datos. 

Hoy en día, la librería reina para esto es **Zod**. Es diminuta, increíblemente poderosa y se lleva perfectamente con TypeScript (y JavaScript puro).

## 📦 Instalación del Combo Definitivo

Para conectar Zod con React Hook Form, necesitamos instalar Zod y un "puente" llamado `@hookform/resolvers`.

```bash
npm install zod @hookform/resolvers
```

---

## 📐 1. Creando nuestro primer Esquema Zod

Zod nos permite definir la estructura de nuestros datos como si estuviéramos armando piezas de Lego, encadenando métodos para añadir reglas. 

Lo mejor de todo es que este esquema vive **fuera** de tu componente React. Puedes exportarlo y reutilizarlo en cualquier parte de tu aplicación.

```javascript
import { z } from "zod";

// Definimos el "contrato" que nuestros datos deben cumplir
export const esquemaRegistro = z.object({
  nombre: z.string().min(3, { message: "El nombre debe tener al menos 3 letras" }),
  email: z.string().email({ message: "Por favor, introduce un correo válido" }),
  edad: z.coerce.number().min(18, { message: "Debes ser mayor de edad" }),
});
```
*(Nota Pro: Zod lee todo lo que viene de un formulario como texto (String). Usamos `z.coerce.number()` para que intente convertir ese texto a número automáticamente antes de validarlo).*

---

## 🔗 2. Conectando Zod con React Hook Form

Ahora viene la magia. Vamos a decirle a `useForm` que no haga las validaciones por sí mismo, sino que le pase los datos al esquema de Zod. Si Zod dice que están mal, RHF nos mostrará los errores automáticamente.

Usamos la propiedad `resolver` y le pasamos el `zodResolver`.

```jsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { esquemaRegistro } from "./esquemas"; // Importamos el esquema de arriba

export const FormularioZod = () => {
  const { 
    register, 
    handleSubmit, 
    formState: { errors } 
  } = useForm({
    resolver: zodResolver(esquemaRegistro) // 🔗 ¡Conexión establecida!
  });

  const onSubmit = (data) => console.log("Datos perfectos:", data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* MIRA QUÉ LIMPIO QUEDA EL HTML */}
      <div>
        <input {...register("nombre")} placeholder="Nombre" />
        {errors.nombre && <span className="error">{errors.nombre.message}</span>}
      </div>

      <div>
        <input {...register("email")} placeholder="Email" />
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>

      <div>
        <input type="number" {...register("edad")} placeholder="Edad" />
        {errors.edad && <span className="error">{errors.edad.message}</span>}
      </div>

      <button type="submit">Enviar</button>
    </form>
  );
};
```
**Resultado:** Un HTML totalmente limpio. Solo tiene el `register("nombre")` y React Hook Form se encarga de hablar con Zod en segundo plano.

---

## 🤯 3. Nivel Arquitecto: Validaciones Cruzadas (`refine`)

Este es un caso clásico de prueba técnica Senior: *"Valida que la Contraseña y Confirmar Contraseña sean idénticas"*.

Hacer esto a mano es un infierno. En Zod, usamos el método `.refine()`, que nos permite evaluar todo el objeto en su conjunto.

```javascript
import { z } from "zod";

export const esquemaPassword = z.object({
  email: z.string().email(),
  password: z.string().min(6, "Mínimo 6 caracteres"),
  confirmarPassword: z.string()
}).refine(
  (datos) => datos.password === datos.confirmarPassword, 
  {
    message: "Las contraseñas no coinciden",
    path: ["confirmarPassword"] // Le dice a RHF dónde colocar el error visual
  }
);
```
Así de fácil. Zod comparará ambos campos y, si no coinciden, inyectará el error directamente en `errors.confirmarPassword` dentro de tu componente React.

---

## 🏆 Resumen del Módulo 3: El Flujo Profesional

Acabas de dominar el ciclo de vida de los datos del usuario. Si vas a trabajar en una empresa moderna, este será tu flujo diario:

1. **Diseñar el Contrato:** Escribes un esquema con `Zod` definiendo cómo deben ser los datos.
2. **Conectar la Memoria:** Usas `React Hook Form` + `zodResolver` para delegar el rendimiento (Componentes No Controlados).
3. **Pintar la Interfaz:** Escribes tu JSX limpio, usando `{...register("campo")}` y mostrando los errores de `formState.errors`.

Con esto, pasas de escribir 100 líneas de código espagueti con re-renderizados constantes, a 20 líneas de código robusto, escalable y con rendimiento nativo.

---
### 🎉 ¡Felicidades! Has completado el Módulo 3.
Tu conocimiento sobre React acaba de subir a la categoría de Profesional. Ya estás listo para el siguiente gran desafío: conectar tus aplicaciones con el mundo exterior en el **Módulo 4: Asincronía y APIs**.