# 01 Use State

## Resumen

Este ejemplo toma como punto de partida el ejemplo _00-boiler-plate_.

Vamos a crear un componente que por un lado muestre un nombre en
un **h4**, y por otro permita editarlo utilizando un **input**.

## Paso a Paso

- Primero copiamos el ejemplo anterior y hacemos un _npm install_

```bash
npm install
```

- Vamos a crear un fichero que llamaremos _demo.tsx_. Vayamos paso a paso:
  primero nos importamos **React**. ¿Por qué si de primeras no uso nada que
  ponga _React_? porque en el momento que empezamos a poner tsx/jsx
  (es decir, esos _h1_, _input_ o _div_ que después se traducen a
  _React.createElement_) es necesario importarlo.

_demo.tsx_

```tsx
import React from "react";
```

Añadimos al fichero un componente de tipo funcion:

_demo.tsx_

```tsx
export const MyComponent: React.FC = () => {
  return <>Prueba</>;
};
```

- Usando la palabra reservada _export_ podemos exponer este fichero al exterior.
- No es estrictamente necesario tiparlo con _React.FC_ (Function Component), pero
  es buena idea, todo lo que nos atemos a _Typescript_ nos servirá para tener
  menos dolores de cabeza en el futuro.
- El componente no es más que una función que devuelve elementos de React.
  Fíjate que en este caso no hemos puesto _Props_ ya que no consume ninguna
  del exterior.

- Vamos a por la parte interesante, seguro que nuestra mente Java o Angular
  nos mueve a implementar lo siguiente (**IMPORTANTE: ESTO ESTA MAL**).

```diff
export const MyComponent : React.FC = () => {
+ let myName = 'John Doe';
  return (
    <>
-    Prueba
+    <h4>{myName}</h4>
+    <input
+      value={myName}
+      onChange={(e) => myName = e.target.value}
+     />
    </>
  )
}
```

Si llegáis a un proyecto React y os encontráis código como este, huele a que los
que lo han codificado no se tomaron el tiempo de aprender las bases de esta
librerías, veamos por qué:

- Al crear una variable, cada vez que se vuelva a repintar el componente
  _myName_ va a valer siempre _John Doe_, esto no deja de ser una función que se
  ejecuta una y otra vez en cada repintado.

- Asignando directamente en el _input_ un valor a una variable nos cargamos
  dos de los pilares de React, el flujo unidireccional, y que el seteo del
  estado sea asíncrono.

Si queréis verlo en acción sólo tenéis que añadirlo al fichero _app.tsx_.

_./src/app.tsx_

```diff
import React from "react";
+ import {MyComponent} from './demo';

export const App = () => {
-  return <h1>Hello React !!</h1>;
+  return <MyComponent/>
};
```

Vale... ¿Cómo puedo manejar esto? ¡Con los hooks de React! Tenemos
_React.useState_.

- Se inicializa con un valor por defecto.

- Te devuelve un array que contiene una especie de getter y un setter (te permite
  acceder al valor que está en useState y te permite
  hacer una petición y setearlo de forma asíncrona).

- La manera más cómoda de consumir lo que devuevel ese _useState_ es hacer
  destructuring sabiendo que el primer elemento del array siempre será
  nuestro _getter_ y el segundo nuestro \_setter.

¿Por qué narices usa un array? Aquí viene la genialidad, si hubiese devuelto
un objeto, al hacer destructuring del objeto habríamos tenido que ceñirnos
a un nombre concreto de getter y a uno de setter, esto es un rollo porque
en un componente podemos tener múltiples state, y además queremos añadirle
sentido a los nombres, ¿Por qué tener un _setState_ genérico cuando podemos
tener un _setName_ o un _setLastname_?

- Vamos a montar este componente con _Hooks_.

Primero hacemos uso del _setState_:

_./src/demo.tsx_

```diff
export const MyComponent: React.FC = () => {
-  let myName = "John Doe";
+  const [myName, setMyName] = React.useState('John Doe');
```

- Como hemos nombrado a nuestro _getter_ _myName_ nos vale
  tanto para mostrar el nombre en el _h4_, así como en el
  _input_.

- Ahora viene la parte interesante, para poder capturar cuando
  el usuario teclea en el input nos suscribimos al evento
  _onChange_ (este es un evento estándar de HTML, más info [MDN](https://developer.mozilla.org/es/docs/Web/API/HTMLElement/change_event)).

¿Qué tenemos aquí?

- \*_e_: argumento del _eventHandler_, nos da información del evento, expone una serie de propiedades:
- **e.target**: qué DOM Element generó el evento.
- **e.target.value**: que valor tiene ese elemento (la propiedad valor
  del DOM Element que generó el evento).

En este event handler aprovechamos para recoger el valor del input y pedir
setear el estado de _myName_.

```diff
  <h4>{myName}</h4>
  <input
    value={myName}
-    onChange={(e) => (myName = e.target.value)}
+    onChange={(e) => (setMyName(e.target.value))}
  />
```

¿Qué va a provocar esto?

- Que la petición _setMyName_ actualice el estado.
- Esto va a lanzar un repintado del componente.
- Cuando se ejecute el código del componente y llegue a la línea
  de código que hace el _useState_, en vez de _John Doe_ se le
  facilitará el nombre que se almaceno con _setState_
- Al repintar el componente se usará ese nuevo valor mostrandolo
  por pantalla.

Es un cambio de mentalidad grande,... intenta repetir este ejemplo
sin ayuda y entenderlo bien, es tu primer gran paso para entender
cómo funciona esta tecnología.

> Vamos a por un bonus que pasaría si el useState de MyName está en App?

_./src/app.tsx_

```diff
import React from "react";
import {MyComponent} from './demo';

export const App = () => {
+  const [myName, setMyName] = React.useState('John Doe');

   return <MyComponent />;
```

Aquí fíjate que demo.tsx se empieza a quejar, porque ya no tiene el myName, vamos a hacerlo un component más generico.

Vamos a definir un contrato en las props de demo que tenga el nombre y el callback para establecer el nombre (en este caso
demo pasa a ser un componente tonto, sin estado, se llama también presentacional).

```diff
import React from "react";

+ interface Props {
+  value : string;
+  onChange : (value : string) => void;
+ }

- export const MyComponent: React.FC = () => {
+ export const MyComponent: React.FC<Props> = (props) => {

  return (
    <>
-      <h4>{myName}</h4>
+      <h4>{props.value}</h4>
-      <input value={myName} onChange={(e) => setMyName(e.target.value)} />
+      <input value={props.value} onChange={(e) => props.onChange(e.target.value)} />
    </>
  );
};
```

Si te fijas queda todo manchado con _props._ vamos a aplicar destructuring.

```diff
import React from "react";

interface Props {
  value: string;
  onChange: (value: string) => void;
}

export const MyComponent: React.FC<Props> = (props) => {
+ const { value, onChange } = props;

  return (
    <>
-      <h4>{props.value}</h4>
+      <h4>{value}</h4>
      <input
-        value={props.value}
+        value={value}
-        onChange={(e) => props.onChange(e.target.value)}
+        onChange={(e) => onChange(e.target.value)}

      />
    </>
  );
};
```

- Vamos ahora a actualizar _App_

```diff
import React from "react";
import { MyComponent } from "./demo";

export const App = () => {
  const [myName, setMyName] = React.useState("John Doe");

-  return <MyComponent />;
+  return <MyComponent value={myName} onChange={setMyName} />;
};
```

Esto es muy común, un componente padre le pasa un estado a un componente hijo, y el componente hijo le devuelve el nuevo estado, sobre todo cuando haces wrappers de por ejemplo un input con un formato especial.

# ¿Te apuntas a nuestro máster?

Si te ha gustado este ejemplo y tienes ganas de aprender Front End
guiado por un grupo de profesionales ¿Por qué no te apuntas a
nuestro [Máster Front End Online Lemoncode](https://lemoncode.net/master-frontend#inicio-banner)? Tenemos tanto edición de convocatoria
con clases en vivo, como edición continua con mentorización, para
que puedas ir a tu ritmo y aprender mucho.

Si lo que te gusta es el mundo del _backend_ también puedes apuntante a nuestro [Bootcamp backend Online Lemoncode](https://lemoncode.net/bootcamp-backend#bootcamp-backend/inicio).

Y si tienes ganas de meterte una zambullida en el mundo _devops_
apuntate nuestro [Bootcamp devops online Lemoncode](https://lemoncode.net/bootcamp-devops#bootcamp-devops/inicio)
