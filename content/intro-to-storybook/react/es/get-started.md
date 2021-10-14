---
title: 'Tutorial de Storybook para React'
tocTitle: 'Empezando'
description: 'Configurar React Storybook en tu entorno de desarrollo'
commit: '6fdf7e3'
---

Storybook se ejecuta junto con tu aplicación en modo desarrollo. Te ayuda a crear componentes de UI aislados de la lógica y el contexto de tu aplicación. Esta edición de Aprende Storybook es para React; existe otras ediciones para [React Native](/intro-to-storybook/react-native/es/get-started), [Vue](/intro-to-storybook/vue/es/get-started), [Angular](/intro-to-storybook/angular/es/get-started) y [Svelte](/intro-to-storybook/svelte/es/get-started).

![Storybook and your app](/intro-to-storybook/storybook-relationship.jpg)

## Configurando React Storybook

Necesitaremos seguir algunos pasos para configurar el proceso de build de nuestro entorno. Para iniciar, vamos a usar [degit](https://github.com/Rich-Harris/degit) para configurar nuestro sistema de build. Usando este paquete usted puede descargar "Plantillas" (aplicaciones parcialmente construidas con alguna configuración predeterminada) para ayudarlo a acelerar su flujo de desarrollo.

Ejecutemos los siguientes comandos:

```bash
# Clonar la plantilla
npx degit chromaui/intro-storybook-react-template taskbox

cd taskbox

# Instalar las dependencias
yarn
```

<div class="aside">
💡 Esta plantilla contiene los estilos, recursos y configuraciones esenciales necesarios para esta versión del tutorial.
</div>

Podemos comprobar rápidamente que los distintos entornos de nuestra aplicación funcionan correctamente:

```bash
# Corre el test de prueba (Jest) en una terminal:
yarn test --watchAll

# Inicia el explorador de componentes en el puerto 6006:
yarn storybook

# Ejecuta el frontend de la aplicación en el puerto 3000:
yarn start
```

<div class="aside"> 
💡 Observe el indicador <code>--watchAll</code> en el comando test, incluir este indicador garantiza que se ejecuten todas las pruebas. A medida que avanza en este tutorial, se le presentarán diferentes escenarios de prueba. Tal vez pueda considerar ajustar sus scripts en el  <code>package.json</code> en consecuencia.
</div>

Nuestras tres modalidades del frontend de la aplicación son: test automatizado (Jest), desarrollo de componentes (Storybook) y la propia aplicación.

![3 modalidades](/intro-to-storybook/app-three-modalities.png)

Dependiendo de en qué parte de la aplicación estés trabajando, es posible que quieras ejecutar uno o más de estos simultáneamente. Dado que nuestro objetivo actual es crear un único componente de UI, seguiremos ejecutando Storybook.

## Confirmar cambios

En esta etapa, es seguro agregar nuestros archivos a un repositorio local. Ejecute los siguientes comandos para inicializar un repositorio local, agregue y confirme los cambios que hemos realizado hasta ahora.

```shell
$ git init
```

Seguido por:

```shell
$ git add .
```

Luego:

```shell
$ git commit -m "primer commit"
```

Y finalmente:

```shell
$ git branch -M main
```

¡Comencemos a construir nuestro primer componente!
