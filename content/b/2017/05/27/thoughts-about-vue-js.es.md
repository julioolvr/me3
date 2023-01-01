+++
title="Thoughts about Vue.js"
date=2017-05-27
+++

Hace poco fui a un workshop de Vue.js que dio
[@ianaya89](https://twitter.com/ianaya89), organizado por la gente de
[@workshopsjs](https://twitter.com/workshopsjs). En este post voy a tratar de
recopilar lo que saqué del mismo.

Antes que nada, los ejercicios del workshop están muy buenos y el repo que creó
Ignacio está muy bien organizado y es muy simple de seguir - así que si alguno
está interesado en probar un poco Vue, sugiero revisarlo en
[github.com/ianaya89/workshop-vuejs/](https://github.com/ianaya89/workshop-vuejs/).

Lo primero que se me vino a la cabeza después de ver un poco de código es que
**se parece mucho a Angular 1**. Creo que haber trabajado con Angular un par de
años me simplificó un poco entender algunos conceptos que maneja Vue, como su
forma de hacer el binding dentro de templates y el uso de directivas.

Ahora, si bien se nota la inspiración en Angular 1, Vue es más reciente y creo
que eso se nota en su foco por los componentes. En los últimos años la comunidad
se estuvo moviendo hacia arquitecturas más orientadas a componentes, lo que
pareciera traer una serie de beneficios a la hora de organizarse y razonar
acerca de UIs complejas. Si bien en Angular 1 era técnicamente posible armar UIs
de esta manera, el hecho de que las directivas hicieran las veces de componentes
y de herramientas de manipulación del DOM hacía que no fuera evidente
organizarse de esa manera. Tanto Angular 1.5+ como las nuevas versiones se
esfuerzan por hacer esta separación entre directivas y componentes, y Vue hace
exactamente lo mismo. Creo que esto hace más obvia la forma de organizar la app,
lo cual hace que todos los devs del equipo puedan converger en una misma manera
de pensar la UI.

Algo a lo que me vine acostumbrando con React y me gusta es el hecho de **tener
el código JavaScript, la vista y potencialmente los estilos de un componente en
un mismo archivo**. Vue logra esto usando archivos `.vue`. Parece ser posible
incluso correr cada parte (template, estilos) por un preprocesador, para usar
por ejemplo Pug, SASS y JSX, aunque es algo que no llegué a probar.

**La arquitectura de Vue como librería en sí está organizada en capas**. El core
incluye toda la parte de rendering y el sistema de componentes, y el resto está
construído por encima de eso. Aunque gran parte de ese resto tiene soporte
oficial, al no estar incluído en el framework por default logra que **la
librería solamente sea tan compleja como es necesario para cada caso**. Si
necesitamos routing o manejo de estado podemos agregarlo, pero si necesitamos
algo mucho más simple entonces el core es más que suficiente. **Ahorramos no
solo en bytes, si no es la carga conceptual que tenemos que tener en mente a la
hora de usar la librería**.

Hay una CLI que el workshop te indica usar para crear el proyecto que se llama
[`vue-cli`](https://github.com/vuejs/vue-cli). Fue útil para generar el proyecto
base que usa Webpack para transpilar los archivos `.vue`. No la usé mucho más,
pero sé que tiene varios «templates» que aplican a diferentes herramientas
(Webpack vs Browserify) pero también a diferentes complejidades. Siguiendo lo
que comentaba de las capas, se puede crear un proyecto simple que solo incluya
el core, o uno que traiga más capas encima y con una estructura de directorios
más armada.

En conclusión, Vue parece ser un Angular 1 repensado unos años después. Aunque
las últimas versiones de Angular cumplen con esto, siguen teniendo una API
gigante y usarlas sin TypeScript no es lo más sencillo del mundo. En este
sentido, **Vue parecería ser una alternativa más simple, mejorando Angular 1
pero manteniendo una API más reducida y fácil de aprender**. No solamente eso,
sino que al incluir un core reducido permite que el camino a futuro no esté
bloqueado para incluir mejores prácticas o ideas que surjan de otras
comunidades. Por ejemplo, [`vuex`](https://github.com/vuejs/vuex) es una
librería para manejo de estado inspirada en Flux. Así como esa, debería ser
posible incluir otros patrones mientras no tengan que ver con ese core de
rendering y componentes.

Aunque no sé si pasaría de React a Vue, no me molestaría trabajar con la
librería y definitivamente voy a estar prestando atención a ver como evoluciona
tanto como librería como comunidad.
