# replication
1 Create VM from template
master 35.224.222.200
slave 199.223.234.158
2. Install soft for master and slave
sudo apt update
sudo apt install postgresql postgresql-contrib pg-activity
(for each)
3. Create db for master, add data
sudo su postgres
psql
create database homeworkdb;
exit, exit
git clone https://github.com/pthom/northwind_psql.git
sudo su postgres
psql -d homeworkdb -f /home/elena.razgulayeva/northwind_psql/northwind.sql
4. Configure master
echo "host replication all 199.223.234.158/32 trust" >> /etc/postgresql/12/main/pg_hba.conf
psql -c "ALTER SYSTEM SET listen_addresses TO '*'";
exit
sudo systemctl restart postgresql.service
4. Configure slave
sudo su postgres
psql -c "ALTER SYSTEM SET listen_addresses TO '*'";
echo "host all all 199.223.234.158/32 trust" >> /etc/postgresql/12/main/pg_hba.conf
exit
sudo systemctl restart postgresql.service

sudo su postgres
mkdir /var/lib/postgresql/12/orig
mv /var/lib/postgresql/12/main/* /var/lib/postgresql/12/orig/

pg_basebackup -h 35.224.222.200 -D /var/lib/postgresql/12/main/ -P -v -R -X stream -C -S pgstandby3
4. Check
on master
sudo su postgres
psql -c "\x" -c "SELECT slot_name, slot_type, active_pid FROM pg_replication_slots;"

Expanded display is on.
-[ RECORD 3 ]----------
slot_name  | pgstandby3
slot_type  | physical
active_pid | 17343

on slave
sudo su postgres
psql -c "\x" -c "SELECT pid, status, slot_name, sender_host, conninfo FROM pg_stat_wal_receiver;"
Expanded display is on.
-[ RECORD 1 ]-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
pid         | 5284
status      | streaming
slot_name   | pgstandby3
sender_host | 35.224.222.200
conninfo    | user=postgres passfile=/var/lib/postgresql/.pgpass dbname=replication host=35.224.222.200 port=5432 fallback_application_name=12/main sslmode=prefer sslcompression=0 gssencmode=prefer krbsrvname=postgres target_session_attrs=any
