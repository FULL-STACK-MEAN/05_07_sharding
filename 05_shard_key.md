# Shard key en sharding cluster

sh.shardCollection(<basedatos.colección>, {clave1: 1, clave2: 1, ...}) // sintaxis
// similar a la de los índices

Es una clave indexada, a nivel de colección, usada por MongoDB en los sharding cluster
para distribuir los documentos entre sus diferentes shard.

- Si la colección ya existe, para poder utilizar el método anterior, previamente
hay que crear un índice con la misma clave que pasaremos después como shard key.

- Si la colección no existe, no hace falta ningún paso previo porque shardCollection() crea la
colección y el índice de manera implícita.

Funcionalidad de la shard key:

- Permite la definición de los campos con los que se generarán los rangos de los
chunk que definen la distribución de los documentos.

- Permite realizar consultas "targeted", es decir que para localizar uno o varios documentos, si la consulta contiene los campos de la shard key puede acceder directamente al shard donde
estén los documentos sin tener que buscarlos en todos los shard ("broadcast operations").

Inconvenientes de la selección de la shard key

- Hasta MongoDB 4.4 la shard key una vez definida no se podía cambiar.

- En 4.4 la shard key se puede modificar extendiéndola desde su prefijo.

- Desde 5.0 la shard key se puede modificar sin tener que desmontar el sharding cluster.

## ¿Cómo deben ser nuestras shard key? ¿Qué objetivos debemos cumplir? 

1.- Respecto a la distribución en los shard.

- Alta cardinalidad (número de valores iguales en los documentos).
    Campos en la shard key con muchos valores distintos son los idóneos 
    porque permitirán generar muchos rangos y por tanto distrubir mejor los documentos

    En cambio baja cardinalidad concentrará los documentos en solo algunos shard (porque
    se generaran jumbo chunks), aumenta el riesgo de alta frecuencia.

    Ejemplos de baja cardinalidad a evitar
        - Género hombre mujer no contesta.

- Baja frecuencia (número de veces que se repite un valor)

    Si tenemos baja frencuencia, los documentos se repartiran uniformemente y evitaremos
    la concentración de operaciones en un solo shard y por tanto las migraciones.

    Ejemplo de alta frecuencia a evitar
        - Campo pais en una colección que tenga pocos paises en uso
            - Más de 300 valores
            - Sólo se usan dos o tres valores (España, Portugal, Francia)

- Evitar cambios monótonos (cuando hay un incremento o decremento sucesivo y constante
     en los valores del campo).

     Las operaciones de escritura se van a ir siempre a un chunk extremo lo que
     proporcionará migraciones a menudo.

     Ejemplo de cambios monótonos. Campos de fecha o que contienes marcas de tiempo.


2.- Respecto a facilitar operaciones targeted.

Intentar que él índice de la shard key sea usado por la mayoría de las consultas
de nuestra colección.

## ¿Como podemos cumplir estas condiciones?

- Usar shard keys compuestas (varios campos)

- En el caso de que para cumplir todas las condiciones, incluyendo las de
  uso del índice para operaciones targeted, la shard key resulta ser monotona se
  pueden usar índices hash (pasa el valor de uno de los campos a un hash).

## Otras consideraciones.

- Los campos de la shard key serán obligatorios.

- Respecto a los índices únicos y shard key. En las colecciones sharding, solo los
  siguientes pueden ser índices únicos:
        - El índice de la shard key
        - Un índices únicos donde la shard key sea prefijo.
        - El índice único por defecto sobre _id esté ó no el campo presente en la shard key.

 Ejemplo colección sharding runners

 shard key es { dni: 1, apellido: 1, nombre: 1 } // no va a ser índice único aunque podría serlo

¿Cuales de los siguientes podrán ser índices únicos?

{_id: 1} // Siempre con independencia de que esté o no en la shard key
{dni: 1} // Si porque está en la shard key como prefijo
{licenciaFederativa: 1} // No podría ser índice único porque no es prefijo de la shard key

- Si la colección sharding no contiene en su shard key el campo _id solo se asegura la unicidad
del campo a nivel de shard.


