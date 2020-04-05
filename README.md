# How to deploy MongoDB on Windows via Docker

First of all, we should pull mongo from Docker Hub, if version is not set it will pull the latest version. Command below will pull and install mongo.
> docker pull mongo

After installation, we have to create a network for mongo containers.

### Listing networks
> docker network ls

### Create a net network
> docker network create mongo-cluster

### Creating Replica Sets

> docker run -p 27017:27017 --name mongodb --net mongo-cluster mongo mongod --replSet rs

> docker run -p 27018:27017 --name mongodb2 --net mongo-cluster mongo mongod --replSet rs

> docker run -p 27019:27017 --name mongodb3 --net mongo-cluster mongo mongod --replSet rs

### Execute mongosrv1 replica set
> docker exec -it mongosrv1 mongo

> rs.initiate({ "_id" : "rs", "members" : [{"_id" : 0,"host" : "mongodb:27017"},{"_id" : 1,"host" : "mongodb2:27017"},{"_id" : 2,"host" : "mongodb3:27017"}]})

After intitiation you will see a command like this; > rs:PRIMARY>

Reconfig hosts with initial config parameters if it fails in the future. To reconfigure commands below you should have executed one of the replica sets already. 

> rs.reconfig({ "_id" : "rs", "members" : [{"_id" : 0,"host" : "mongodb:27017"},{"_id" : 1,"host" : "mongodb2:27017"},{"_id" : 2,"host" : "mongodb3:27017"}]})

> cfg = rs.conf();
> cfg.members[0].priority = 1;
> cfg.members[1].priority = 0.5;
> cfg.members[2].priority = 0.5;

Reconfig executation command
> rs.reconfig(cfg);

Reconfig force executation command
> rs.reconfig(cfg,{force:true})

Reconfig force execution with present parameters.
> rs.reconfig(rs.config(),{force:true})

## Error Handling

If primary and secondary replicate sets fails you will receive error message below:
``No suitable servers found (`serverSelectionTryOnce` set): [Failed to write rpc bytes. calling ismaster on 'mongodb:27017'] [Failed to write rpc bytes. calling ismaster on 'mongodb2:27017'] [Failed to write rpc bytes. calling ismaster on 'mongodb3:27017']``

If primary and secondary replicate sets are conflicted with each other you will receive error message below. In this situation you should reconfig with force, then restart in order.
``Error message something like "`No suitable servers found (`serverSelectionTryOnce` set): [Server closed connection. calling ismaster on 'mongodb:27017'] [Server closed connection. calling ismaster on 'mongodb2:27017'] [connection refused calling ismaster on 'mongodb3:27017']`"``

You will receive this error if primary 
