# 多种Exporter监控示例

- consul-exporter
- mysqld-exporter
- nginx-exporter
- tomcat

### mysqld-exporter使用说明

需要在mysqld上运行如下命令，为mysqld-exporter创建具有采集监控数据权限的用户账号

  mysql> CREATE USER 'exporter'@'mysqld-exporter' IDENTIFIED BY 'exporter';
  mysql> GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'mysqld-exporter';
  mysql> GRANT SELECT ON performance_schema.* TO 'exporter'@'mysqld-exporter';


