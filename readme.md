# Clickhouse Database Configuration

1. Clickhouse configuration.<br>
   2 node connection:<br>
   pc 1: /etc/clickhouse-server/config.xml<br>
   pc 2: /etc/clickhouse-server/config.xml

```xml
<listen_host>::</listen_host> <!-- enable host ip -->
<remote_server>
 <cluster1> <!-- cluster name -->
   <shard>
	<internal_replication>true</internal_replication>
	<replica>
		<host>192.168.10.129</host> <!-- pc 1 ip -->
		<port>9000</port>
	</replica>
	<replica>
		<host>192.168.10.134</host> <!-- pc 2 ip -->
		<port>9000</port>
	</replica>
   </shard>
 </cluster1>
<remote_server>
```

2. check on both pc. Cluster is created or not.

```sql
select * from system.clusters;
```

3. Configure clickhouse server: /etc/clickhouse-keeper/config.xml

```xml
<listen_host>::</listen_host>
```

4. Restart clickhouse-keeper:

```bash
systemctl restart clickhouse-keeper
```

5. Assign zookeeper : /etc/clickhouse-server/

```xml
<zookeeper>
        <node>
            <host>192.168.10.129</host> <!-- zookeeper ip -->
            <port>9181</port>
        </node>::
</zookeeper>
```

6. Check zookeeper is connected.

```sql
select * from system.zookeeper where path in ('/','/clickhouse');
```

7. Create replicated database

```sql
create database test1 on cluster cluster1;
```

8. Check on pc 2

```sql
show databases;
```

9. Create replicated table

```sql
create table student on cluster cluster1
(
	id int
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/student/{shard}','{replica}')
ORDER BY id
```
