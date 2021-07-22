# Primario en los sharding cluster

El shard primary de los sharding cluster (NO CONFUNDIR CON EL PRIMARIO DE LOS REPLICA SET) será
el que tenga todas las colecciones no particionadas. 

El shard primary se establece a nivel de base de datos, por tanto una misma base de datos tendra
un determinado shard como primary y allí estarán sus colecciones no particionadas. En un mismo sharding cluster
distintas bases de datos pueden tener distintos primary shard (en cada uno de ellos estarán las colecciones
no particionadas de cada base de datos).

¿Como selecciona o determina MongoDB el shard primary de cada base de datos?

Cuando se crea la base de datos selecciona el shard que tenga menos cantidad de datos.

¿Como sabemos cual es el primary shard de una base de datos?

En la base de datos config del configServer

Desde mongos

use config
db.databases.find()

{ "_id" : "marathon", "primary" : "shard2ServerGetafe", "partitioned" : true, "version" : { "uuid" : UUID("c3505539-7bed-48c4-9c56-929f7db0a35a"), "lastMod" : 1 } }