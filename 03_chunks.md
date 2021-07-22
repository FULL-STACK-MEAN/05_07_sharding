# Chunks en Sharding MongoDB.

Agrupación lógica de los documentos de una colección sharding con un rango para
distribuir uniformemente los documentos en los diferentes shard diseñados para minimizar el
impacto de las distribuciones.

Almacena documentos hasta un determinado tamaño que se indica en el chunkSize (por defecto es
de 64 megas). Cuando el tamaño del chunk supera ese límite se divide automaticamente (split), generando
un nuevo chunk y por tanto diviendo también el rango anterior.

Cuando se produce una diferencia entre el número de chunks de un shard y otro, se produce una migración
automática del chunk del shard de origen más próximo en rango a los rangos del shard de destino.

Esa descompensación lanza la migración dependiendo del número total de chunks generados en todos los
shard:

nº Chunks totales           Diferencia chunks/sharding que genera migración

menor 20                            2
20-79                               4
mas 80                              8

## Modificación del tamaño del chunk por defecto

Si queremos una mejor distribución de los documentos en el cluster sharding podemos reducir el tamaño
del chunk para que de esa manera aumente la posibilidad de migración y por tanto un mejor reparto.

El inconveniente es que pueden desencadenarse mas migraciones que afecten al rendimiento de los shard.

1.- Desde mongos

use config

db.settings.save({_id: "chunksize", value: 4}) // Tamaño en megas

## Procedimiento de migración

1.- Inicia el proceso, las operaciones sobre los documentos del chunk se mantienen al shard de origen.

2.- Se crea el nuevo chunk en el destino y se realiza una actualización del índice con los nuevos
documentos en el shard de destino.

3.- Escritura de los documentos en el shard de destino.

4.- Sincronización del chunk de destino con el de origen por si se producen
    actualizaciones durante el proceso.

5.- Actualiza la info del configserver sobre los chunk y shard afectados.

6.- Las operaciones se redirigen al chuck destino y el chunk origen es eliminado.

## Balancer

El balancer de los cluster sharding realiza las migraciones de manera automática de acuerdo
a los criterios anteriores. Por defecto está activado pero, por motivos de rendimiento podríamos
desactivarlo.

Disponemos de una serie de métodos para adminsitrar el balancer desde mongos:

sh.getBalancerState() // Si el balanceador está o no activado

sh.isBalancerRunning() // Si el balanceador está produciendo migraciones de chunks

sh.stopBalancer()

sh.startBalancer()

También hay un procedimiento para programar la activación del balancer en ventanas de mantenimiento.
