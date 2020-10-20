---
layout: post
title: Introducción a la programación
---
# Introducción a la programación

Muchas personas se asustan cuando quieren comenzar en el mundo de la programación. este miedo es normal porque no saben donde empezar.
aquí vamos a explicar un poco como funciona esto y explicar de forma muy simple la base de la programación con un simple ejemplo.

Todos conocen el código binario, las películas y series de hackers lo han hecho muy famoso con esas pantallas verdes que hacen correr gran cantidad de 1 y 0 de forma random

Bien, eso es una parte del código binario, pero porque 1 y 0? porque no todos los números? o porque no otros números?
la explicación a estas palabras es porque las computadoras actuales (y no me meto en el mundo de las computadoras cuánticas porque eso es otra cosa) son digitales, ¿que quiere decir esto? que los buses de información (veamos lo simplemente como una carretera) pueden tener dos estados, encendido y apagado.

si traducimos el código binario al estado eléctrico podemos decir que 1 es encendido y 0 es apagado.  

sabiendo esto podemos hacer un programa que encienda un led con arduino:

## Variables
Que son las variables? que función cumplen? para que las puedo utilizar?

bien, las variables son espacios en memoria ram que estas contienen información de rápido acceso en nuestro programa. dependiendo que lenguaje quieres utilizar estas pueden ser fuertemente tipadas como `C++`, `Java` o incluso el propio `C` o pueden ser dinámicas como `JavaScript`, `PHP` o `Python`

Oye oye, más despacio cerebrito, que carajos quiere decir esto?

bueno. esto quiere decir que nuestro programa va a manipular memoria para funcionar. esta memoria se manipula dentro de variables es decir, las variables son cajitas que contienen información para nosotros poder utilizarla 

que quiere decir fuerte o débilmente tipados? quiere decir que si nosotros determinamos que la variable va a contener números o de tipo `int` si nosotros intentamos guardar una cadena de caracteres el compilador no nos dirá que hay un error de tipos
luego hay distintos tipos de variables y dependiendo del lenguaje irán cambiando los nombres

sabiendo esto ya podemos decir que una variable es un espacio en memoria que guarda información, verdad? bien, como las personas no somos máquinas y a nosotros nos gusta saber las cosas por medio de nombres nosotros le podemos asignar a nuestras variables nombres para poder identificarlas mejor a lo largo de nuestro programa, ya que muchas veces este será pequeño, pero otras veces este será de gran tamaño y es mejor tener nombres significativos para poder saber que es lo que guarda nuestra variable

__*Y recuerda, no importa lo largo que sea el nombre de nuestra variable, mientras más identificativo es, mejor para la persona que lo tenga que leer*__

arduino nos exige que nosotros tengamos dos estructuras llamadas métodos los cuales contienen instrucciones y pueden devolver un valor (las llamadas funciones) o no pueden devolver nada (los llamados métodos) pero esto lo veremos más adelante, por ahora son dos estructuras que están porque sí

#### Programa

entonces lo primero que hacemos es crear una variable constante (esto se usa para cuando necesitamos utilizar 
una variable pero no queremos que se modifique durante la ejecución del programa) para luego armar el output de un led

```
const int RED_LED = 5;
```

aquí nosotros le estamos diciendo a nuestro arduino que guarde una "cajita" con el valor de 5 en la memoria ram de la placa

luego de esto, tenemos que decirle a arduino que use el pin n°5 como salida:

```
void setup()
{
    pinMode(RED_LED, OUTPUT);
}
```

antes guardamos en la variable `RED_LED` el valor de 5, entonces nosotros aquí leemos ese valor llamando a la variable 

Nuestro método `setup` ya está hecho y nos falta el método `loop` que queda algo como esto:

```
void loop()
{
    digitalWrite(RED_LED, HIGH);
}
```

el método `digitalWrite` lo que hace es cambiar el estado del pin de salida dependiendo el segundo parámetro que nosotros le pasemos, ya sea `HIGH` o `LOW`
esto es prender un led, toda la lógica de programación se resume a esto, con la ayuda de bucles, estructuras de validación y estructuras de datos se compone la programación completa y funcional.

cabe aclarar que el método loop funciona como un bucle, en cada ciclo del procesador este se ejecuta una vez
El programa completo se encuentra en el repositorio de [github](https://github.com/cristianvena18)