# Tareas

Las computadoras ejecutan varios programas en simultáneo a la vista de los usuarios. Por ejemplo, mientras navegamos por la web podemos utilizar una aplicación para escuchar música. 

Sin embargo, como vimos, cada procesador ejecuta un programa por vez. ¿Cómo se logra esto el sistema operativo?

Para comprenderlo, vamos a usar una serie de estructuras y funciones de Intel que nos permiten definir tareas para el procesador.

A su vez, el sistema operativo va a implementar un módulo de software que se va a encargar de decidir que tarea ejecutar en cada tic del reloj: <u>scheduler</u>.

Una **tarea** es una unidad de trabajo que el procesador puede despachar, ejecutar, y suspender. Puede ser usada para ejecutar un programa.

Dos o más tareas distintas pueden tener un mismo código de programa, sin embargo, sus contexto de ejecución y datos asociados pueden ser distintos. Podemos pensarlo como distintas instancias del mismo programa.