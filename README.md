# garage-sample

## レイアウトの設定

ノード1台の場合でも、レイアウトを設定する必要がある。

- https://garagehq.deuxfleurs.fr/documentation/quick-start/#creating-a-cluster-layout


```
docker exec -ti garage-sample_devcontainer-garage-1 /garage status
```

```
docker exec -ti garage-sample_devcontainer-garage-1 /garage layout assign -z dc1 -c 1G <ID>
docker exec -ti garage-sample_devcontainer-garage-1 /garage layout apply --version 1
```

```
$ docker exec -ti garage-sample_devcontainer-garage-1 /garage status
2025-11-30T23:23:37.325200Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
2025-11-30T23:23:37.369621Z  INFO garage_net::netapp: Connection established to 120ff271f8be138f
==== HEALTHY NODES ====
ID                Hostname      Address         Tags  Zone  Capacity          DataAvail  Version
120ff271f8be138f  e24a94a24c69  127.0.0.1:3901              NO ROLE ASSIGNED             v2.1.0
$ docker exec -ti garage-sample_devcontainer-garage-1 /garage layout assign -z dc1 -c 1G 120ff271f8be138f
2025-11-30T23:25:34.588856Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
2025-11-30T23:25:34.637746Z  INFO garage_net::netapp: Connection established to 120ff271f8be138f
Role changes are staged but not yet committed.
Use `garage layout show` to view staged role changes,
and `garage layout apply` to enact staged changes.
$ docker exec -ti garage-sample_devcontainer-garage-1 /garage layout apply --version 1
2025-11-30T23:25:56.338499Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
2025-11-30T23:25:56.381795Z  INFO garage_net::netapp: Connection established to 120ff271f8be138f
==== COMPUTATION OF A NEW PARTITION ASSIGNATION ====

Partitions are replicated 1 times on at least 1 distinct zones.

Optimal partition size:                     3.9 MB
Usable capacity / total cluster capacity:   1000.0 MB / 1000.0 MB (100.0 %)
Effective capacity (replication factor 1):  1000.0 MB

dc1                 Tags  Partitions        Capacity   Usable capacity
  120ff271f8be138f  []    256 (256 new)     1000.0 MB  1000.0 MB (100.0%)
  TOTAL                   256 (256 unique)  1000.0 MB  1000.0 MB (100.0%)


New cluster layout with updated role assignment has been applied in cluster.
Data will now be moved around between nodes accordingly.
```

## バケットの作成

- https://garagehq.deuxfleurs.fr/documentation/quick-start/#create-a-bucket

```
docker exec -ti garage-sample_devcontainer-garage-1 /garage bucket create nextcloud-bucket
```

```
$ docker exec -ti garage-sample_devcontainer-garage-1 /garage bucket create nextcloud-bucket
2025-11-30T23:45:42.803170Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
2025-11-30T23:45:42.845730Z  INFO garage_net::netapp: Connection established to 120ff271f8be138f
==== BUCKET INFORMATION ====
Bucket:          58743f032daba2e269b082ca686f71464bb3f1f7883ac6decfb6ccf2ddef7ec6
Created:         2025-11-30 23:45:42.850 +00:00

Size:            0 B (0 B)
Objects:         0

Website access:  false

Global alias:    nextcloud-bucket

==== KEYS FOR THIS BUCKET ====
Permissions  Access key    Local aliases
```

## APIキーの作成

- https://garagehq.deuxfleurs.fr/documentation/quick-start/#create-an-api-key

```
docker exec -ti garage-sample_devcontainer-garage-1 /garage key create nextcloud-app-key
```

```
$ docker exec -ti garage-sample_devcontainer-garage-1 /garage key create nextcloud-app-key
2025-11-30T23:46:50.304647Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
2025-11-30T23:46:50.353567Z  INFO garage_net::netapp: Connection established to 120ff271f8be138f
==== ACCESS KEY INFORMATION ====
Key ID:              GK1ec093e651e2bd46eee1391c
Key name:            nextcloud-app-key
Secret key:          965e6ff28d61877f4c2d6c3f0d562212a6eff8aa17f4e81df8a7d652be2d39fb
Created:             2025-11-30 23:46:50.354 +00:00
Validity:            valid
Expiration:          never

Can create buckets:  false

==== BUCKETS FOR THIS KEY ====
Permissions  ID  Global aliases  Local aliases
```

## バケットに対するアクセス許可

- https://garagehq.deuxfleurs.fr/documentation/quick-start/#allow-a-key-to-access-a-bucket

```
docker exec -ti garage-sample_devcontainer-garage-1 /garage bucket allow --read --write --owner nextcloud-bucket --key nextcloud-app-key
```

## awscliでの確認

- https://garagehq.deuxfleurs.fr/documentation/quick-start/#example-usage-of-awscli

```
export AWS_ACCESS_KEY_ID=GK1ec093e651e2bd46eee1391c      # put your Key ID here
export AWS_SECRET_ACCESS_KEY=965e6ff28d61877f4c2d6c3f0d562212a6eff8aa17f4e81df8a7d652be2d39fb  # put your Secret key here
export AWS_DEFAULT_REGION='garage'
export AWS_ENDPOINT_URL='http://garage:3900'

# list buckets
aws s3 ls

# list objects of a bucket
aws s3 ls s3://nextcloud-bucket

# copy from your filesystem to garage
aws s3 cp /proc/cpuinfo s3://nextcloud-bucket/cpuinfo.txt

# copy from garage to your filesystem
aws s3 cp s3://nextcloud-bucket/cpuinfo.txt /tmp/cpuinfo.txt
```

```
ubuntu@e63e225165f6:/workspaces/garage-sample$ export AWS_ACCESS_KEY_ID=GK1ec093e651e2bd46eee1391c      # put your Key ID here
export AWS_SECRET_ACCESS_KEY=965e6ff28d61877f4c2d6c3f0d562212a6eff8aa17f4e81df8a7d652be2d39fb  # put your Secret key here
export AWS_DEFAULT_REGION='garage'
export AWS_ENDPOINT_URL='http://garage:3900'
ubuntu@e63e225165f6:/workspaces/garage-sample$ aws s3 ls
2025-11-30 23:45:42 nextcloud-bucket
ubuntu@e63e225165f6:/workspaces/garage-sample$ aws s3 ls s3://nextcloud-bucket
ubuntu@e63e225165f6:/workspaces/garage-sample$ aws s3 cp /proc/cpuinfo s3://nextcloud-bucket/cpuinfo.txt
upload: ../../proc/cpuinfo to s3://nextcloud-bucket/cpuinfo.txt
ubuntu@e63e225165f6:/workspaces/garage-sample$ aws s3 cp s3://nextcloud-bucket/cpuinfo.txt /tmp/cpuinfo.txt
download: s3://nextcloud-bucket/cpuinfo.txt to ../../tmp/cpuinfo.txt
```

## garage-webuiでの確認

- http://localhost:3909/

