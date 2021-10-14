---
title: 'Construye un componente simple'
tocTitle: 'Componente Simple'
description: 'Construye un componente simple en aislamiento'
commit: '8ce7e80'
---

Construiremos nuestra UI siguiendo la metodología (CDD) [Component-Driven Development](https://www.componentdriven.org/). Es un proceso que construye UIs de “abajo hacia arriba”, empezando con los componentes y terminando con las vistas. CDD te ayudará a escalar la cantidad de complejidad con la que te enfrentas a medida que construyes la UI.

## Task - Tarea

![Task component in three states](/intro-to-storybook/task-states-learnstorybook.png)

`Task` (o Tarea) es el componente principal en nuestra app. Cada tarea se muestra de forma ligeramente diferente según el estado en el que se encuentre. Mostramos un checkbox marcado (o no marcado), información sobre la tarea y un botón “pin” que nos permite mover la tarea hacia arriba o abajo en la lista de tareas. Poniendo esto en conjunto, necesitaremos estas propiedades -props- :

- `title` – un string que describe la tarea
- `state` - ¿en qué lista se encuentra la tarea actualmente y está marcado el checkbox?

A medida que comencemos a construir `Task`, primero escribiremos nuestros tests para los estados que corresponden a los distintos tipos de tareas descritas anteriormente. Luego, utilizamos Storybook para construir el componente de forma aislada usando datos de prueba. Vamos a “testear visualmente” la apariencia del componente a medida que cambiemos cada estado.

## Ajustes iniciales

Primero, vamos a crear el componente Task y el archivo de historias de storybook que lo acompaña: `src/components/Task.js` y `src/components/Task.stories.js`.

Comenzaremos con una implementación básica de `Task`, simplemente teniendo en cuenta los atributos que sabemos que necesitaremos y las dos acciones que puedes realizar con una tarea (para moverla entre las listas):

```js:title=src/components/Task.js
import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className="list-item">
      <input type="text" value={title} readOnly={true} />
    </div>
  );
}
```

Arriba, renderizamos directamente `Task` basándonos en la estructura HTML existente de la app Todos.

A continuación creamos los tres estados de prueba de Task dentro del archivo de historia:

```js:title=src/components/Task.stories.js
import React from 'react';

import Task from './Task';

export default {
  component: Task,
  title: 'Task',
};

const Template = args => <Task {...args} />;

export const Default = Template.bind({});
Default.args = {
  task: {
    id: '1',
    title: 'Test Task',
    state: 'TASK_INBOX',
    updatedAt: new Date(2021, 0, 1, 9, 0),
  },
};

export const Pinned = Template.bind({});
Pinned.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_PINNED',
  },
};

export const Archived = Template.bind({});
Archived.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_ARCHIVED',
  },
};
```

Existen dos niveles básicos de organización en Storybook. El componente y sus historias hijas. Piensa en cada historia como una permutación posible del componente. Puedes tener tantas historias por componente como se necesite.

- **Componente**
  - Historia
  - Historia
  - Historia

Para decirle a Storybook sobre el componente que estamos documentando, creamos una exportación `default`(por defecto), que contiene:

- `component` - el componente en sí,
- `title` - cómo hacer referencia al componente en la barra lateral de la aplicación Storybook,

Para definir nuestras historias, exportamos una función para cada uno de nuestros estados de prueba para generar una historia. La historia es una función que retorna un elemento renderizado (es decir, un componente con un conjunto de props) en un estado dado, exactamente como un [Componente funcional](https://reactjs.org/docs/components-and-props.html#function-and-class-components).

Como tenemos múltiples permutaciones de nuestro componente, es conveniente asignarlo a una variable "Template" (Plantilla). La introducción de este patrón en sus historias reducirá la cantidad de código que necesita escribir y mantener.

<div class="aside">
💡 <code> Template.bind({})</code> es un método <a href = "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind" > estandar de JavaScript </a> para hacer una copia de una función. Usamos esta técnica para permitir que cada historia exportada establezca sus propias propiedades, pero usamos la misma implementación.
</div>

Los argumentos o [`args`](https://storybook.js.org/docs/react/writing-stories/args) para abreviar, nos permiten editar en vivo nuestros componentes con el complemento de controles sin reiniciar Storybook. Una vez que un valor de [`args`](https://storybook.js.org/docs/react/writing-stories/args) cambia, también cambia el componente.

Al crear una historia usamos un argumento de una tarea o `task` base para construir la forma de la tarea que espera el componente. Por lo general, esto se modela a partir de cómo se ven los datos reales. Nuevamente, `export`(exportar) de esta forma nos permitirá reutilizarla en historias posteriores, como veremos.

<div class="aside">
Las <a href="https://storybook.js.org/docs/react/essentials/actions"><b>Acciones</b></a> ayudan a verificar las interacciones cuando creamos componentes UI en aislamiento. A menudo no tendrás acceso a las funciones y el estado que tienes en el contexto de la aplicación. Utiliza <code>action()</code> para agregarlas.
</div>

## Configuración

Tendremos que hacer un par de cambios en los archivos de configuración de Storybook para que no solo note nuestras historias creadas recientemente y nos permita usar el archivo CSS de la aplicación (ubicado en `src / index.css`).

Comience cambiando su archivo de configuración de Storybook (`.storybook / main.js`) a lo siguiente:

```diff:title=.storybook/main.js
module.exports = {
- stories: [
-   '../src/**/*.stories.mdx',
-   '../src/**/*.stories.@(js|jsx|ts|tsx)'
- ],
+ stories: ['../src/components/**/*.stories.js'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/preset-create-react-app',
  ],
};
```

Después de completar el cambio anterior, dentro de la carpeta `.storybook`, cambie su` preview.js` a lo siguiente:

```diff:title=.storybook/preview.js
+ import '../src/index.css';

//👇 Configura Storybook para registrar las acciones (onArchiveTask y onPinTask) en la UI(interfaz de usuario).
export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
};
```

Los [`parameters`](https://storybook.js.org/docs/react/writing-stories/parameters) se utilizan normalmente para controlar el comportamiento de las funciones y complementos de Storybook. En nuestro caso, los usaremos para configurar cómo se manejan las `acciones` (devoluciones de llamada simuladas).

`actions` nos permite crear callbacks (devoluciones de llamada) que aparecen en el panel de **actions** de la UI de Storybook cuando se les hace clic. Así que, cuando creamos un botón con un pin, podremos determinar en la UI de prueba si el clic de un botón es exitoso.

Una vez que hayamos hecho esto, reiniciando el servidor de Storybook debería producir casos de prueba para los tres estados de Task:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/inprogress-task-states-6-0.mp4"
    type="video/mp4"
  />
</video>

## Construyendo los estados

Ahora tenemos configurado Storybook, los estilos importados y los casos de prueba construidos; podemos comenzar rápidamente el trabajo de implementar el HTML del componente para que coincida con el diseño.

El componente todavía es básico. Primero escribiremos el código que se aproxima al diseño sin entrar en demasiados detalles:

```js:title=src/components/Task.js
import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className={`list-item ${state}`}>
      <label className="checkbox">
        <input
          type="checkbox"
          defaultChecked={state === 'TASK_ARCHIVED'}
          disabled={true}
          name="checked"
        />
        <span className="checkbox-custom" onClick={() => onArchiveTask(id)} />
      </label>
      <div className="title">
        <input type="text" value={title} readOnly={true} placeholder="Input title" />
      </div>

      <div className="actions" onClick={event => event.stopPropagation()}>
        {state !== 'TASK_ARCHIVED' && (
          // eslint-disable-next-line jsx-a11y/anchor-is-valid
          <a onClick={() => onPinTask(id)}>
            <span className={`icon-star`} />
          </a>
        )}
      </div>
    </div>
  );
}
```

El maquetado adicional de arriba, combinado con el CSS que hemos importado antes, produce la siguiente UI:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/finished-task-states-6-0.mp4"
    type="video/mp4"
  />
</video>

## Especificar los requerimientos de datos

Es una buena práctica en React utilizar `propTypes` para especificar la forma de los datos que espera recibir un componente. No sólo se auto documenta, sino que también ayuda a detectar problemas rápidamente.

```diff:title=src/components/Task.js
import React from 'react';
+ import PropTypes from 'prop-types';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  // ...
}

+ Task.propTypes = {
+  /** Composition of the task */
+  task: PropTypes.shape({
+    /** Id of the task */
+    id: PropTypes.string.isRequired,
+    /** Title of the task */
+    title: PropTypes.string.isRequired,
+    /** Current state of the task */
+    state: PropTypes.string.isRequired,
+  }),
+  /** Event to change the task to archived */
+  onArchiveTask: PropTypes.func,
+  /** Event to change the task to pinned */
+  onPinTask: PropTypes.func,
+ };
```

Ahora aparecerá una advertencia en modo desarrollo si el componente Task se utiliza incorrectamente.

<div class="aside">
Una forma alternativa de lograr el mismo propósito es utilizando un sistema de tipos de JavaScript como TypeScript, para crear un tipo para las propiedades del componente.
</div>

## Componente construido!

Ahora hemos construido con éxito un componente sin necesidad de un servidor o sin ejecutar toda la aplicación frontend. El siguiente paso es construir los componentes restantes de la Taskbox, uno por uno de manera similar.

Como puedes ver, comenzar a construir componentes de forma aislada es fácil y rápido. Podemos esperar producir una UI de mayor calidad con menos errores y más pulida porque es posible profundizar y probar todos los estados posibles.

## Pruebas automatizadas

Storybook nos dio una excelente manera de probar visualmente nuestra aplicación durante su construcción. Las 'historias' ayudarán a asegurar que no rompamos nuestra Task visualmente, a medida que continuamos desarrollando la aplicación. Sin embargo, en esta etapa, es un proceso completamente manual y alguien tiene que hacer el esfuerzo de hacer clic en cada estado de prueba y asegurarse de que se visualice bien y sin errores ni advertencias. ¿No podemos hacer eso automáticamente?

### Pruebas de instantáneas

La prueba de instantáneas se refiere a la práctica de registrar la salida "correcta" de un componente para una entrada dada y luego en el futuro marcar el componente siempre que la salida cambie. Esto complementa a Storybook, porque es una manera rápida de ver la nueva versión de un componente y verificar los cambios.

<div class="aside">
💡 Asegúrese de que sus componentes rendericen datos que no cambien, de modo que sus pruebas de instantáneas no fallen cada vez. Tenga cuidado con cosas como fechas o valores generados aleatoriamente.
</div>

Con [Storyshots addon](https://github.com/storybooks/storybook/tree/master/addons/storyshots) se crea una prueba de instantánea para cada una de las historias. Usalo agregando una dependencia en modo desarrollo en el paquete:

```bash
yarn add -D @storybook/addon-storyshots react-test-renderer
```

Luego crea un archivo `src/storybook.test.js` con el siguiente contenido:

```js:title=src/storybook.test.js
import initStoryshots from '@storybook/addon-storyshots';
initStoryshots();
```

Una vez hecho lo anterior, podemos ejecutar `yarn test` y veremos el siguiente resultado:

![Task test runner](/intro-to-storybook/task-testrunner.png)

Ahora tenemos una prueba de instantánea para cada una de las historias de `Task`. Si cambiamos la implementación de `Task`, se nos pedirá que verifiquemos los cambios.

<div class="aside">
💡 ¡No olvides confirmar tus cambios con git!
</div>
