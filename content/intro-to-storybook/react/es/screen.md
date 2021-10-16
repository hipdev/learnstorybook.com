---
title: 'Construir una pantalla'
tocTitle: 'Pantallas'
description: 'Construir una pantalla con componentes'
commit: 'cec2e05'
---

Nos hemos concentrado en crear interfaces de usuario de abajo hacia arriba; comenzando por lo pequeño y añadiendo complejidad. Esto nos ha permitido desarrollar cada componente de forma aislada, determinar los datos que necesita y jugar con ellos en Storybook. Todo sin necesidad de levantar un servidor o construir pantallas!

En este capítulo continuaremos aumentando la sofisticación combinando componentes en una pantalla y desarrollando esa pantalla en Storybook.

## Componentes de contenedor anidados

Como nuestra aplicación es muy simple, la pantalla que construiremos es bastante trivial, simplemente envolviendo el componente `TaskList` (que proporciona sus propios datos a través de Redux) en alguna maqueta y sacando un campo `error` de primer nivel de redux (asumamos que pondremos ese campo si tenemos algún problema para conectarnos a nuestro servidor). Crea el archivo `InboxScreen.js` en tu directorio `components`:

```js:title=src/components/InboxScreen.js
import React from 'react';
import PropTypes from 'prop-types';

import { connect } from 'react-redux';

import TaskList from './TaskList';

export function PureInboxScreen({ error }) {
  if (error) {
    return (
      <div className="page lists-show">
        <div className="wrapper-message">
          <span className="icon-face-sad" />
          <div className="title-message">Oh no!</div>
          <div className="subtitle-message">Algo salió mal</div>
        </div>
      </div>
    );
  }
  return (
    <div className="page lists-show">
      <nav>
        <h1 className="title-page">
          <span className="title-wrapper">Taskbox</span>
        </h1>
      </nav>
      <TaskList />
    </div>
  );
}

PureInboxScreen.propTypes = {
  /** El mensaje de error */
  error: PropTypes.string,
};

PureInboxScreen.defaultProps = {
  error: null,
};

export default connect(({ error }) => ({ error }))(PureInboxScreen);
```

Sin embargo, donde las cosas se ponen interesantes es en la representación de la historia en Storybook.

Como vimos anteriormente, el componente `TaskList` es un **contenedor** que renderiza el componente de presentación `PureTaskList`. Por definición, los componentes de un contenedor no pueden simplemente hacer render de forma aislada; esperan que se les pase algún contexto o que se conecten a un servicio. Lo que esto significa es que para hacer render de un contenedor en Storybook, debemos mockearlo (es decir, proporcionar una versión ficticia) del contexto o servicio que requiere.

Al colocar la "Lista de tareas" `TaskList` en Storybook, pudimos esquivar este problema simplemente renderizando la `PureTaskList` y evadiendo el contenedor. Haremos algo similar y renderizaremos la `PureInboxScreen` en Storybook también.

Sin embargo, para la `PureInboxScreen` tenemos un problema porque aunque la `PureInboxScreen` en si misma es presentacional, su hijo, la `TaskList`, no lo es. En cierto sentido la `PureInboxScreen` ha sido contaminada por la "contenedorización". Así que cuando preparamos nuestras historias en `InboxScreen.stories.js`:

```js:title=src/components/InboxScreen.stories.js
import React from 'react';

import { PureInboxScreen } from './InboxScreen';

export default {
  component: PureInboxScreen,
  title: 'InboxScreen',
};

const Template = args => <PureInboxScreen {...args} />;

export const Default = Template.bind({});

export const Error = Template.bind({});
Error.args = {
  error: 'Something',
};
```

Vemos que aunque la historia de `error` funciona bien, tenemos un problema en la historia `default`, porque la `TaskList` no tiene una store de Redux a la que conectarse. (También encontrarás problemas similares cuando intentes probar la `PureInboxScreen` con un test unitario).

![Broken inbox](/intro-to-storybook/broken-inboxscreen.png)

Una forma de evitar este problema es nunca renderizar componentes contenedores en ninguna parte de tu aplicación excepto en el nivel más alto y en su lugar pasar todos los datos requeridos bajo la jerarquía de componentes.

Sin embargo, los desarrolladores **necesitarán** inevitablemente renderizar los contenedores más abajo en la jerarquía de componentes. Si queremos renderizar la mayor parte o la totalidad de la aplicación en Storybook (¡lo hacemos!), necesitamos una solución a este problema.

<div class="aside">
Por otro lado, la transmisión de datos a nivel jerárquico es un enfoque legítimo, especialmente cuando utilizas <a href="http://graphql.org/">GraphQL</a>. Así es como hemos construido <a href="https://www.chromatic.com">Chromatic</a> junto a más de 800+ historias.
</div>

## Suministrando contexto con decoradores

La buena noticia es que es fácil suministrar una store de Redux a la `InboxScreen` en una historia! Podemos usar una versión mockeada de la store de Redux provista en un decorador:

```diff:title=src/components/InboxScreen.stories.js
import React from 'react';
+ import { Provider } from 'react-redux';

import { PureInboxScreen } from './InboxScreen';

+ import { action } from '@storybook/addon-actions';

+ import * as TaskListStories from './TaskList.stories';

+ // Una simulación súper simple de un store de redux
+ const store = {
+   getState: () => {
+    return {
+      tasks: TaskListStories.Default.args.tasks,
+    };
+   },
+   subscribe: () => 0,
+   dispatch: action('dispatch'),
+ };

export default {
  component: PureInboxScreen,
+ decorators: [story => <Provider store={store}>{story()}</Provider>],
  title: 'InboxScreen',
};

const Template = args => <PureInboxScreen {...args} />;

export const Default = Template.bind({});

export const Error = Template.bind({});
Error.args = {
  error: 'Something',
};
```

Existen enfoques similares para proporcionar un contexto simulado para otras bibliotecas de datos, tales como [Apollo](https://www.npmjs.com/package/apollo-storybook-decorator), [Relay](https://github.com/orta/react-storybooks-relay-container) y algunas otras.

Recorrer los estados en Storybook hace que sea fácil comprobar que lo hemos hecho correctamente:

<video autoPlay muted playsInline loop >

  <source
    src="/intro-to-storybook/finished-inboxscreen-states-6-0.mp4"
    type="video/mp4"
  />
</video>

## Desarrollo basado en componentes

Empezamos desde abajo con `Task`, luego progresamos a `TaskList`, ahora estamos aquí con una interfaz de usuario de pantalla completa. Nuestra `InboxScreen` contiene un componente de contenedor anidado e incluye historias de acompañamiento.

<video autoPlay muted playsInline loop style="width:480px; height:auto; margin: 0 auto;">
  <source
    src="/intro-to-storybook/component-driven-development-optimized.mp4"
    type="video/mp4"
  />
</video>

[**El desarrollo basado en componentes**](https://www.componentdriven.org/) te permite expandir gradualmente la complejidad a medida que asciendes en la jerarquía de componentes. Entre los beneficios están un proceso de desarrollo más enfocado y una mayor cobertura de todas las posibles mutaciones de la interfaz de usuario. En resumen, la CDD te ayuda a construir interfaces de usuario de mayor calidad y complejidad.

Aún no hemos terminado, el trabajo no termina cuando se construye la interfaz de usuario. También tenemos que asegurarnos de que siga siendo duradero a lo largo del tiempo.

<div class="aside">
💡 ¡No olvides confirmar tus cambios con git!
</div>
