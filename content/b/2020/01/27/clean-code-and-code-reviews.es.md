+++
title="Clean Code and Code Reviews"
date=2020-01-27
+++

Me crucé un artículo de [Dan Abramov](https://twitter.com/dan_abramov),
[«Goodbye, Clean Code»](https://overreacted.io/goodbye-clean-code/) (o
[«Adiós, Código Limpio»](https://overreacted.io/es/goodbye-clean-code/),
traducido al español por la comunidad), que cuenta una historia de trabajo y
reflexiona acerca de dos cosas:

- Escribir código "limpio" siguiendo alguna definición arbitraria de qué
  significa eso (y cómo puede terminar siendo algo negativo).
- La dinámica del equipo durante esa situación.

Ese segundo punto en particular resonó mucho con algunas cosas en las que me
estuve enfocando últimamente.

Resumiendo mucho el posteo (definitivamente recomiendo leerlo), lo que pasó es
que un compañero de trabajo hizo una implementación donde Dan notó mucho código
duplicado. Hizo un refactor, quitando gran parte de esa repetición, y lo mandó a
`master` sin revisarlo con su compañero.

Últimamente vengo pensando bastante en el proceso de code review y su propósito.
Quiero buscar oportunidades de mejora, y para encontrarlas necesito entender
bien cuál es el objetivo original del proceso. Me gustó esta historia porque
muestra un par de oportunidades donde hacer code reviews es particularmente
útil.

Para empezar, citando una parte del artículo con la cual estoy totalmente de
acuerdo:

> Un equipo de ingeniería sano está constantemente construyendo confianza.
> Reescribir el código de tu compañero de equipo sin una discusión previa es un
> gran golpe para su habilidad como equipo de colaborar efectivamente en una
> base de código.

Incluso si consideramos que en algunos casos excepcionales (urgencias, cambios
triviales, depende del equipo) se puede saltear un code review, lo más probable
es que un refactor no sea uno de esos casos. Esto significa que, **puramente por
seguir el proceso de code review, nunca se daría una situación donde alguien
reescriba el código de un compañero sin consultarlo.**

Otro aspecto que se menciona seguido como una ventaja de llevar adelante code
reviews es el de aprendizaje. Una de las formas de aprendizaje es relativamente
sencilla: alguien nos señala una API que no conocíamos, o una nueva forma de
encarar un problema, quizás incluso un patrón de diseño que no conocíamos. En
este caso, el post habla de cómo el hecho de reducir código duplicado no
necesariamente genera código que es objetivamente _mejor_, y cómo debería
evaluarse en el contexto de qué pros y contras tiene. Si bien es posible que
esto no surgiera en el proceso de review, si surge, sería un tipo de aprendizaje
distinto al mencionado anteriormente. Un aprendizaje donde cuestionamos lo que
sabemos y vemos desde otro ángulo algo que dábamos por relativamente seguro.
**Lo que inicialmente podía ser un comentario de rutina puede volverse una
oportunidad para reevaluar y aprender.**
