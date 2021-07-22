# Despliegue de shard en MongoDB

## Config server (cluster con los metadatos del sharding)

Necesitamos un replica set

1.- Crear directorios

configServer1
configServer2
configServer3

2.- Archivos de configuración ó comando para levantar esos tres miembros

mongod --replSet configServerGetafe --dbpath configServer1 --port 27001 --configsvr
mongod --replSet configServerGetafe --dbpath configServer2 --port 27002 --configsvr
mongod --replSet configServerGetafe --dbpath configServer3 --port 27003 --configsvr

opción en archivo de configuración:
configServer1.conf
configServer2.conf
configServer3.conf

3.- Lanzar la configuración del cluster

Nos conectamos con la shell de mongo al primario

mongo --port 27001

rs.initiate({
    _id: "configServerGetafe",
    configsvr: true,
    members: [
        {_id: 0, host: "localhost:27001"},
        {_id: 1, host: "localhost:27002"},
        {_id: 2, host: "localhost:27003"},
    ]
})

## Shards (en el ejemplo vamos a crear 2 shard)

1.- Directorios del primer shard

shard1Server1
shard1Server2
shard1Server3

2.- Levantamos cada miembro con comando ó archivo de configuración

mongod --replSet shard1ServerGetafe --dbpath shard1Server1 --port 27101 --shardsvr
mongod --replSet shard1ServerGetafe --dbpath shard1Server2 --port 27102 --shardsvr
mongod --replSet shard1ServerGetafe --dbpath shard1Server3 --port 27103 --shardsvr

configShard1Server1.conf
configShard1Server2.conf
configShard1Server3.conf

3.- Lanzamos la configuración del cluster de este primer shard

mongo --port 27101

rs.initiate({
    _id: "shard1ServerGetafe",
    members: [
        {_id: 0, host: "localhost:27101"},
        {_id: 1, host: "localhost:27102"},
        {_id: 2, host: "localhost:27103"}
    ]
})

4.- Directorios del segundo shard

shard2Server1
shard2Server2
shard2Server3

5.- Levantamos los miembros con comando o archivo de configuración

mongod --replSet shard2ServerGetafe --dbpath shard2Server1 --port 27201 --shardsvr
mongod --replSet shard2ServerGetafe --dbpath shard2Server2 --port 27202 --shardsvr
mongod --replSet shard2ServerGetafe --dbpath shard2Server3 --port 27203 --shardsvr

configShard2Server1.conf
configShard2Server2.conf
configShard2Server3.conf

6.- Lanzamos la configuración del segundo cluster shard

mongo --port 27201

rs.initiate({
    _id: "shard2ServerGetafe",
    members: [
        {_id: 0, host: "localhost:27201"},
        {_id: 1, host: "localhost:27202"},
        {_id: 2, host: "localhost:27203"}
    ]
})

## Mongos (router)

1.- Levantar mongos con comando o archivo de configuración

mongos --configdb configServerGetafe/localhost:27001,localhost:27002,localhost:27003 --port 27000

configMongos.conf

en este caso, si usamos archivo de configuración se levantará con:

mongos --config configMongos.conf

2.- Añadir los shard al sharding cluster

Nos conectamos con la shell de mongo a la instancia del mongos (router)

mongo --port 27000

Añadir cada shard

sh.addShard("shard1ServerGetafe/localhost:27101,localhost:27102,localhost:27103")
sh.addShard("shard2ServerGetafe/localhost:27201,localhost:27202,localhost:27203")

Chequeamos con 

sh.status()

## Crear base de datos con colección particionada

1.- Crear una base de datos cuyas colecciones puedan ser particionadas (desde mongos)

sh.enableSharding(<nombre-base-datos>)

sh.enableSharding('marathon')

2.- Crear en esa base de datos una colección particionada

sh.shardCollection(<basedatos.colección>, {clave1: 1, clave2: 1, ...}) // shard keys

sh.shardCollection("marathon.runners", {surname1: 1, surname2: 1})

3.- Insertamos datos en runners

let names = ['Laura','Juan','Fernando','María','Carlos','Lucía','David'];

let surnames = ['Fernández','Etxevarría','Nadal','Novo','Sánchez','López','García'];

let dniWords = ['A','B','C','D','P','X'];

let runners = [];

for (i = 0; i < 2000000; i++) {
    runners.push({
        name: names[Math.floor(Math.random() * names.length)],
        surname1: surnames[Math.floor(Math.random() * surnames.length)],
        surname2: surnames[Math.floor(Math.random() * surnames.length)],
        age: Math.floor(Math.random() * 100),
        dni: Math.floor(Math.random() * 1e8) + dniWords[Math.floor(Math.random() * dniWords.length)]
    })
}


