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
