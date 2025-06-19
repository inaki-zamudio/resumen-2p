# Interrupciones

* Permiten a un agente externo o interno solicitar la interrupción de la ejecución actual para atender un pedido.
* La parte solicitante ve al procesador como un recurso al que quiere tener acceso.
* El mecanismo implementado define una **identidad numérica para cada interrupción** y utiliza una tabla de descriptores donde para cada índice, o identidad, se decide:
    * Dónde se encuentra la rutina que lo atiende(dirección de memoria).
    * En qué contexto se va a ejecutar(segmento y nivel de privilegio).
    * De qué tipo de interrupción se trata.

## Qué tipos de interrupciones vamos a usar?
* **Excepciones** que van a ser generadas por el procesador cuando se cumpla una condición, por ejemplo si se quiere acceder a una dirección de memoria a través de un selector cuyo segmento tiene el bit P apagado.
* **Interrupciones**:
    * **Externas**: cuyo origen se da en un dispositivo externo, en nuestro caso van a ser el reloj y el teclado.
    * **Internas**, cuyo origen se da en una llamada a la instrucción `INT` por parte de un proceso

### Tipos de excepciones
* **Fault**: Excepción que podría corregirse para que el programa continúe su ejecución. El procesador guarda en la pila la dirección de la instrucción que produjo la falla. Algunos *faults* suman un código de error a la pila.
* **Traps**: Excepción producida al terminar la ejecución de una instrucción de trap, el procesador guarda en la pila la dirección de la instrucción a ejecutarse luego de la que causó el trap.
* **Aborts**: Excepción que no siempre puede determinar la instrucción que la causa, ni permite recuperar la ejecución de la tarea que la causó. Reporta errores severos de hardware o inconsistencias en tablas del sistema.

![Protected-Mode Exceptions and Interrupts](/img/interrupt_table.png)
![Protected-Mode Exceptions and interrupts](/img/interrupt_table_2.png)

Vamos a poder definir nuestras propias interrupciones en el rango 32 a 255.

### Cómo se implementa el mecanismo asociado a cada tipo de interrupción?

* Escribimos la rutina de atención de cada excepción/interrupción, utilizando `IRET` en lugar de `RET`.
* Definimos IDT con los descriptores correspondientes.
* Cargamos descriptor de IDT en IDTR.

## Estado de la pila
Antes de atender una excepción/interrupción el procesador pushea en la pila:
* El registro `EFLAGS`.
* El registro `CS`.
* El registro `EIP`.
* En algunas excepciones también un `error code`.
> Por eso es importante usar `IRET` en vez de ret, ya que debemos sacar más cosas de la pila.

![Uso del stack en transferencias a rutinas de interrupción o excepción](/img/stack_interrupts.png)

## Estructura de la IDT
Es muy parecida a la GDT, donde cada entrada es un descriptor de compuerta (`gate descriptor`).  
Para poder resolver las interrupciones, el procesador va a emplear la `IDT` que se encuentre referida en el registro `IDTR`.

![Relación entre IDTR e IDT](/img/idtr_idt.png)

La IDT contiene descriptores para:
* **Compuertas de tarea**: ver tareas.md
* **Compuertas de interrupción**
* **Compuertas de trampa** que son similares a las de interrupción pero manejan distinto el estado del bit de interrupción en EFLAGS y no utilizaremos.

![IDT descriptor](/img/idt_desc.png)

Nosotros vamos a usar compuertas de interrupción, así que las entradas de la IDT se verán como la **Interrupt Gate**

Atributos importantes:
* **Offset**: Dirección de memoria donde comienza la rutina de interrupción.
* **Segment Selector**: Indica qué selector utilizar al ejecutar código de la rutina.
* **P**, **DPL**: Indican si la rutina se encuentra en memoria y el nivel de privilegio.
* **Bits 8 a 12** de los bytes 4 a 7 indican el tipo de la compuerta de interrupción, el bit D indica si es de 32 o 16 bits.

![Atributo de tipo](/img/type_attr.png)

El atributo de tipo determina qué representa nuestra entrada en la IDT, sólo usamos `1110`.

![Esquema general](/img/esquema_general.png)

## Interrupciones de teclado y reloj

Las interrupciones externas son administradas por un controlador de interrupciones (**PIC**).

[**TO-DO**]

## Syscalls

- Forma que tiene el sistema de exponer funcionalidad a espacio de usuario.
- Pueden tener que ver con el manejo de procesos, filesystems, I/O, y otras cosas más.
- Hay varias formas de implementarlas, pero la que vamos a usar son las **interrupciones de software**, o sea asociando una o varias syscalls a una interrupción particular.
Una aplicación puede acceder a la syscall 76 haciendo: int 0x4c.

### Implementación de una syscall
En nuestro caso, se va a hacer con una rutina de interrupción, tal como hicimos para las excepciones o interrupciones externas:

```nasm
    global _isrXX
    ...
    _isrXX:
        pushad
        ...
        popad
        iret
```
Esta rutina atiende a la interrupción XX, con un prólogo y un epílogo muy generales. No hay llamada a `pic_finish1` ya que es interna y el PIC no tuvo intervención, y un retorno de función con `IRET`.