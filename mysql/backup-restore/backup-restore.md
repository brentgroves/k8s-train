https://dev.to/duplys/how-to-backup-mariadb-database-running-in-a-kubernetes-pod-2pd6
https://www.youtube.com/watch?v=BsKXzm6qbcM
https://www.youtube.com/watch?v=vIIRuDtG1yI

https://virtual-dba.com/blog/how-to-use-mysql-config-editor/
mysql_config_editor print --all
mysql_config_editor set --login-path=client --host=10.1.0.116 --port=31008 --user=root --password 
mysql_config_editor set --login-path=client --host=frt-ubu --port=31008 --user=root --password 
This config file is obsolete use mysql_config_editor instead.
.my.cnf
[client]
user=root
password=password

backup 1 user database
/bin/date +\%Y-\%m-\%d
export v1=$(/bin/date +\%Y-\%m-\%d)

mysqldump -u root -p -h 10.1.0.116 --port=31008 --column-statistics=0 --add-drop-table --databases mcpdw > /home/brent/backups/db/$(/bin/date +\%Y-\%m-\%d-\%R:\%S).sql.bak

mysql -u root -p -h 10.1.0.116 test --port=31008 < ~/backups/db/BACKUPNAME.sql

mysqldump -u root -p -h frt-ubu --port=31008 --column-statistics=0 --add-drop-table --databases test > /home/bgroves@BUSCHE-CNC.COM/backups/db/$(/bin/date +\%Y-\%m-\%d-\%R:\%S).sql.bak


mysqldump -u root -p -h frt-ubu --port=31008 --column-statistics=0 --add-drop-table --databases test > $(bash -c "cd ~ && pwd")/backups/db/$(/bin/date +\%Y-\%m-\%d-\%R:\%S).sql.bak

mysqldump -u root -p -h frt-ubu --port=31008 --column-statistics=0 --add-drop-table --databases test > /home/root/backups/db/$(/bin/date +\%Y-\%m-\%d-\%R:\%S).sql.bak


export user_home=$(bash -c "cd ~ && pwd")



mysql -u root -p -h frt-ubu test --port=31008

https://stackoverflow.com/questions/52423595/mysqldump-couldnt-execute-unknown-table-column-statistics-in-information-sc
mysqldump -u root -p -h frt-ubu --port=31008 --column-statistics=0 test > dump2.sql

drop database test;
create database test;

mysql -u root -p -h frt-ubu test --port=31008 < dump2.sql



https://dev.mysql.com/doc/refman/8.0/en/backup-types.html

https://www.sqlshack.com/how-to-backup-and-restore-mysql-databases-using-the-mysqldump-command/