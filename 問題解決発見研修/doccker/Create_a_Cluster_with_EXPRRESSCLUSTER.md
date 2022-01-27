
# Create a Cluster with EXPRESSCUSTER
## Evaluation Environment
- CentOS 7 (3.10.0-1160.49.1.el7.x86_64)
- Docker(1.13.1)
- EXPRESSCUSTER X 4.3.0-2

 ```
+------------------------+
| Node #1 (node1)        |   +-------------+
| - CentOS 7             |   |             |
| - Docker               +---+ Mirror Disk |
| - EXPRESSCLUSTER X 4.3 |   |             |
+------------------------+   +------|------+
                                    | Mirroring
+------------------------+   +------V------+
| Node #2 (node2)        |   |             |
| - CentOS 7             +---+ Mirror Disk |
| - Docker               |   |             |
| - EXPRESSCLUSTER X 4.3 |   +-------------+
+------------------------+
 ```


## Prerequisite
- Install EXPRESSSCLUSTER and create a cluster. For details, refer to Installation and Configuration Guide.
   - https://docs.nec.co.jp/sites/default/files/minisite/static/09fe37c6-42ac-47c2-a2a9-93b4b24cc229/ecx_x43_linux_en/L43_IG_EN/index.html
- Add the following resources.
  - Mirror Disk Resource
  - Mount Point: /mnt/md1
  - Floating IP Resource
- Install Docker

 ```sh
 yum install docker
 ```

## MariaDB Clustering
1. Install install mysql-libs on both servers.
 ```sh
   yum install mysql-libs
 ```
2. If you have a proxy server, run the following command to pull images.
 ```sh
   export HTTP_PROXY=<your proxy server>
 ```
3. Pull MariaDB container image on both servers.
 ```sh
   docker pull mariadb
 ```

4. Start the failover group on node1.

5. Run the following command on node1 to create and run MariaDB container.
 ```sh
  # docker run --name mariadb1 -v /mnt/md1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=watch -e -p 3306:3306 -d mariadb:latest
 ```
6. Run the following command on node2 to create MariaDB container.
 ```sh
# docker create --name mariadb1 -v /mnt/md1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=watch -e -p 3306:3306 -d mariadb:latest
 ```
7. Run the following command on node1 to stop the container.

```sh
# docker stop mariadb1
```

8. Start Cluster WebUI and add Exec Resource as below.

start.sh
 ```sh
#! /bin/sh
#***************************************
#*              start.sh               *
#***************************************

#ulimit -s unlimited

docker start mariadb1

echo "EXIT"
exit 0
 ```

stop.sh 
 ```sh
#! /bin/sh
#***************************************
#*               stop.sh               *
#***************************************

#ulimit -s unlimited

docker stop mariadb1

echo "EXIT"
exit 0
 ```

9. On Cluster WebUI, add MySQL Monitor Resource as below.
   - Database Name: watch
   - IP Address: 127.0.0.1
   - Port: 3306
   - User Name: root
   - Password: password
   - Table: mysqlwatch
   - Storage Engine: InnoDB
   - Library Path: /usr/lib64/mysql/libmysqlclient.so.18

10. Click [Apply the Configuration File].
11. Check the cluster status.
 ```sh
# clpstat --long
 ========================  CLUSTER STATUS  ===========================
  Cluster : cluster
  <server>
   *node1 ............................: Online
      lankhb1                         : Normal           Kernel Mode LAN Heartbeat
      lankhb2                         : Normal           Kernel Mode LAN Heartbeat
    node2 ............................: Online
      lankhb1                         : Normal           Kernel Mode LAN Heartbeat
      lankhb2                         : Normal           Kernel Mode LAN Heartbeat
  <group>
    failover-mariadb1 ................: Online
      current                         : node1
      exec-mariadb1                   : Online
      md1                             : Online
  <monitor>
    mdnw1                             : Normal
    mdw1                              : Normal
    mysqlw1                           : Normal
 =====================================================================
 ```

## Referense
- [Create a Cluster with EXPRESSCLUSTER](https://github.com/EXPRESSCLUSTER/Podman/blob/main/CreateCluster.md)
- [How to Install Docker](https://github.com/EXPRESSCLUSTER/Docker/blob/master/HowToInstallDocker.md)