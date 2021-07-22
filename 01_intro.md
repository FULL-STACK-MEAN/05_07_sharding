# Sharding en MongoDB

- Arquitectura distribuida de los clusters de servidores de base de datos MongoDB en la
que los datos de una colección se reparten entre los diferentes shards (partición) para escalar
horizontalmente nuestros sistemas.

- Componentes en el sharding MongoDB

    - Shards (clusters replica set con un set de datos de la colección)
    - Config server (cluster con los metadatos de los shards)
    - Mongos (router) servidor que enruta las operaciones a cada shard.

- Chunk. Mecanismo automatizado de migración de datos de un shard a otro para balancear la
 colección en todos ellos.