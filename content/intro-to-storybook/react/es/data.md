---
title: 'Introducir datos'
tocTitle: 'Datos'
description: 'Aprende como introducir datos a tus componentes UI'
commit: '167467b'
---

Hasta ahora hemos creado componentes aislados sin estado, muy útiles para Storybook, pero finalmente no son útiles hasta que les proporcionemos algunos datos en nuestra aplicación.

Este tutorial no se centra en los detalles de la construcción de una aplicación, por lo que no profundizaremos en esos detalles aquí. Pero, nos tomaremos un momento para observar un patrón común para introducir datos con componentes contenedores.

## Componentes contenedores

Nuestro componente `TaskList` como lo hemos escrito es de “presentación” (ver [artículo al respecto](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)), en el sentido que no se comunica con nada externo a su implementación. Para poder pasarle datos, necesitaremos un "contenedor".

Este ejemplo utiliza [Redux](https://redux.js.org/), la librería mas popular de React para almacenar datos, que básicamente nos permite crear un modelo simple de datos para la aplicación. De todos modos, el patrón que utilizaremos también se aplica a otras librerías de manejo de datos como [Apollo](https://www.apollographql.com/client/) y [MobX](https://mobx.js.org/).

Agrega una nueva dependencia en `package.json` con:

```bash
yarn add react-redux redux
```

Primero construiremos un simple store Redux que responde a acciones que cambian el estado de una tarea, en un archivo llamado `lib/redux.js` dentro del folder `src`, (intencionalmente lo mantendremos simple):

```js:title=src/lib/redux.js

// Una implementación simple de los store/actions/reducer de Redux.
// Una verdadera aplicación sería más compleja y se dividiría en diferentes archivos.
import { createStore } from 'redux';

// Las acciones son los "nombres" de los cambios que pueden ocurrir en el store.
export const actions = {
  ARCHIVE_TASK: 'ARCHIVE_TASK',
  PIN_TASK: 'PIN_TASK',
};

// Los creadores de acciones son la forma en que se agrupan las acciones con los datos necesarios para ejecutarlas.
export const archiveTask = id => ({ type: actions.ARCHIVE_TASK, id });
export const pinTask = id => ({ type: actions.PIN_TASK, id });

// Todos nuestros reducers simplemente cambian el estado de una sola tarea.
function taskStateReducer(taskState) {
  return (state, action) => {
    return {
      ...state,
      tasks: state.tasks.map(task =>
        task.id === action.id ? { ...task, state: taskState } : task
      ),
    };
  };
}

// El reducer describe como los contenidos del store cambian por cada acción.
export const reducer = (state, action) => {
  switch (action.type) {
    case actions.ARCHIVE_TASK:
      return taskStateReducer('TASK_ARCHIVED')(state, action);
    case actions.PIN_TASK:
      return taskStateReducer('TASK_PINNED')(state, action);
    default:
      return state;
  }
};

// El estado inicial de nuestro store cuando la app carga.
// Usualmente obtendrías esto de un servidor.
const defaultTasks = [
  { id: '1', title: 'Something', state: 'TASK_INBOX' },
  { id: '2', title: 'Something more', state: 'TASK_INBOX' },
  { id: '3', title: 'Something else', state: 'TASK_INBOX' },
  { id: '4', title: 'Something again', state: 'TASK_INBOX' },
];

// Exportamos el store de redux construido.
export default createStore(reducer, { tasks: defaultTasks });
```

Luego actualizaremos lo exportado por defecto en el componente `TaskList` para conectarlo al Store de Redux y renderizar las tareas en las que estamos interesados.

```js:title=src/components/TaskList.js

import React from 'react';
import PropTypes from 'prop-types';

import Task from './Task';
import { connect } from 'react-redux';
import { archiveTask, pinTask } from '../lib/redux';

export function PureTaskList({ loading, tasks, onPinTask, onArchiveTask }) {
  /*Implementación previa de TaskList */
}

PureTaskList.propTypes = {
  /** Comprueba si está en estado de carga */
  loading: PropTypes.bool,
  /** La lista de tareas */
  tasks: PropTypes.arrayOf(Task.propTypes.task).isRequired,
  /** Evento para cambiar la tarea a pinned(Fija) */
  onPinTask: PropTypes.func.isRequired,
  /** Evento para cambiar la tarea a archivada */
  onArchiveTask: PropTypes.func.isRequired,
};

PureTaskList.defaultProps = {
  loading: false,
};

export default connect(
  ({ tasks }) => ({
    tasks: tasks.filter((t) => t.state === 'TASK_INBOX' || t.state === 'TASK_PINNED'),
  }),
  (dispatch) => ({
    onArchiveTask: (id) => dispatch(archiveTask(id)),
    onPinTask: (id) => dispatch(pinTask(id)),
  })
)(PureTaskList);
```

Ahora que tenemos algunos datos reales poblando nuestro componente, obtenidos de Redux, podríamos haberlo conectado a `src/app.js` y renderizar el componente allí. Pero por ahora no lo haremos y vamos a continuar nuestro camino orientado por componentes.

No se preocupe, nos ocuparemos de ello en el próximo capítulo.

En esta etapa, nuestras pruebas de Storybook habrán dejado de funcionar, ya que la `TaskList` ahora es un contenedor y ya no espera ninguna de las props pasadas como parámetros, sino que se conecta a la store y establece las props en el componente `PureTaskList` que envuelve.

Sin embargo, podemos resolver este problema fácilmente renderizando `PureTaskList` --el componente de presentación, al cual acabamos de agregar la declaración `export` en el paso anterior --, en nuestras historias de Storybook:

```diff:title=src/components/TaskList.stories.js
import React from 'react';

+ import { PureTaskList } from './TaskList';
import * as TaskStories from './Task.stories';

export default {
+ component: PureTaskList,
  title: 'TaskList',
  decorators: [story => <div style={{ padding: '3rem' }}>{story()}</div>],
};

+ const Template = args => <PureTaskList {...args} />;

export const Default = Template.bind({});
Default.args = {
  // Dar forma a las historias a través de la composición de args.
  // Los datos se heredaron de la historia `Default` en task.stories.js.
  tasks: [
    { ...TaskStories.Default.args.task, id: '1', title: 'Task 1' },
    { ...TaskStories.Default.args.task, id: '2', title: 'Task 2' },
    { ...TaskStories.Default.args.task, id: '3', title: 'Task 3' },
    { ...TaskStories.Default.args.task, id: '4', title: 'Task 4' },
    { ...TaskStories.Default.args.task, id: '5', title: 'Task 5' },
    { ...TaskStories.Default.args.task, id: '6', title: 'Task 6' },
  ],
};

export const WithPinnedTasks = Template.bind({});
WithPinnedTasks.args = {
  // Dar forma a las historias a través de la composición de args.
  // Datos heredados de la historia predeterminada.
  tasks: [
    ...Default.args.tasks.slice(0, 5),
    { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
  ],
};

export const Loading = Template.bind({});
Loading.args = {
  tasks: [],
  loading: true,
};

export const Empty = Template.bind({});
Empty.args = {
  // Dar forma a las historias a través de la composición de args.
  // Datos heredados de la historia de Loading.
  ...Loading.args,
  loading: false,
};
```

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/finished-tasklist-states-6-0.mp4"
    type="video/mp4"
  />
</video>

<div class="aside">
💡 Con este cambio, sus instantáneas requerirán una actualización. Vuelva a ejecutar el comando de prueba con el indicador <code> -u </code> para actualizarlas. ¡Además, no olvides confirmar tus cambios con git!
</div>
