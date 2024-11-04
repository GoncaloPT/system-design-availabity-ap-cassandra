# System design - Availability and Partition Tolerance demo

Using cassandra as a database, we will demonstrate what happens when a node goes down and how the system behaves
in terms of availability and partition tolerance.

## Instructions

Start the cluster

```bash
docker-compose up -d
```

Check the status of the cluster by connecting to it, and creating a keyspace and table:

```bash
docker exec -it cassandra-node1 cqlsh -e "CREATE KEYSPACE IF NOT EXISTS TWO_REPLICAS WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 2 };"
```

then create table and add some data

```bash
docker exec -it cassandra-node1 cqlsh -e "create table TWO_REPLICAS.Test \
( \
    name text PRIMARY KEY \
);"
docker exec -it cassandra-node1 cqlsh -e "insert into TWO_REPLICAS.Test(name) \
values ('insert-with-all-nodes-up'); "
docker exec -it cassandra-node1 cqlsh -e "select * from TWO_REPLICAS.Test";
```

simulate network partition by stopping node 2, then in node 3 insert some data:

```bash
docker stop cassandra-node2
docker exec -it cassandra-node3 -e "insert into TWO_REPLICAS.Test(name) \
values ('inserted-with-node2-down');"
docker exec -it cassandra-node3 -e "select * from TWO_REPLICAS.Test;"
```

Then back at node 1:

```bash
docker exec -it cassandra-node1 -e "insert into TWO_REPLICAS.Test(name) \
values ('inserted-with-node2-down');"

docker exec -it cassandra-node1 -e "select * from TWO_REPLICAS.Test;"
```

Take node 3 down. Then try to write on node 1:

```bash
docker down cassandra-node3
docker exec -it cassandra-node1 cqlsh -e "insert into TWO_REPLICAS.Test(name) values ('inserted-with-all-nodes-down');"
docker exec -it cassandra-node1 cqlsh -e "select * from TWO_REPLICAS.Test;"
```

all nodes up again:

```bash
docker restart cassandra-node3 && docker restart cassandra-node2
docker exec -it cassandra-node2 cqlsh -e "select * from TWO_REPLICAS.Test;"
docker exec -it cassandra-node3 cqlsh -e "select * from TWO_REPLICAS.Test;"
```

### Conclusion

Even with replication factor 2, with all nodes down the write didn't fail.
Changes are just buffered to be written after the nodes are up again.


