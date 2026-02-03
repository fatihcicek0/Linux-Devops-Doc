# Zabbix ve Grafana Kurulumu deneme emri
## İçindekiler
- ### Zabbix 
   - [İndirme](#indirme)
   - [Konfigürasyon](#konfigürasyon)
   - [Timescaledb](#timescaledb)

- [Grafana](#grafana)



## indirme
#### - Öncelikle Zabbix repository indiriyoruz 

```bash
  wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu20.04_all.deb
 
```
```bash
 dpkg -i zabbix-release_6.0-4+ubuntu20.04_all.deb
```
```bash
 apt update
```
#### - Zabbix server, frontend, agent  kuruyoruz

```bash
 apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```



## Konfigürasyon

### Postgresql
- Postgresql için zabbix adında database ve user oluşturuyoruz

````bash
   sudo -u postgres createuser --pwprompt zabbix
````

````bash
   sudo -u postgres createdb -O zabbix zabbix
````
- ilk şemayı ve verileri içe aktarıyoruz. Postgresql de zabbix kullanıcısı oluştururken yeni oluşturduğunuz şifreyi girmeniz istenecektir.

````bash
   zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
````
- Zabbix sunucusu için `/etc/zabbix/zabbix_server.conf` dizinindeki dosyaya veritabanı şifresini yazıyoruz

````bash
   DBPassword=password
````


### Nginx
`/etc/zabbix/nginx.conf` dizinindeki dosya ile nginx konfigürasyonunu yapıyoruz örneğin ben zabbix domainini kullanıyorum diyelim

````bash
 listen 8080;
 server_name zabbix;
````

#### Ve şimdi Zabbixi başlatalım
 ````bash
 systemctl restart zabbix-server zabbix-agent nginx php7.4-fpm
````
````bash
 systemctl enable zabbix-server zabbix-agent nginx php7.4-fpm
````


## Timescaledb
### Ubuntu için;

- 1 
````bash
 apt install gnupg postgresql-common apt-transport-https lsb-release wget
````
- 2
````bash
 /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
````
- 3
````bash
 curl -s https://packagecloud.io/install/repositories/timescale/timescaledb/script.deb.sh | sudo bash
````
- 4
````bash
 apt-get update
````
- 5  (istediğiniz versionu bu şekilde belirtebiliyorsunuz) 
````bash
 sudo apt-get install timescaledb-2-postgresql-13=2.5.2~ubuntu20.04
````
- 6  Gerekli konfigürasyonları yapmak için zabbix-serveri durduruyoruz
````bash
 systemctl status zabbix-server
 systemctl stop zabbix-server
````
- 7 Bu Komuttan sonra tüm adımlara yes diyoruz  
````bash
  sudo timescaledb-tune
````

### Postgresql konfigurasyonu

- 8   
````bash
apt-get update

apt-get install postgresql-client

````
- 9  
````bash
systemctl restart postgresql

````

- 10  
````bash

sudo -u zabbix psql
````

- 11  
````bash

\c zabbix
````

- 12  
````bash
CREATE EXTENSION IF NOT EXISTS timescaledb;
````
### * ``eğer istediğimiz bir sürümü kuruyorsak psql zabbix kullanıcısında bu şekilde kurmalıyız;``

````bash
   CREATE EXTENSION IF NOT EXISTS timescaledb WITH VERSION '2.5.2' CASCADE;
````

- 13  
````bash
cat /usr/share/zabbix-sql-scripts/postgresql/timescaledb.sql | sudo -u zabbix psql zabbix
````
- 14 Son olarak zabbixi yeniden başlatıyoruz  
````bash
systemctl start zabbix-server

````
# Grafana

- 1
````bash
sudo apt-get install -y adduser libfontconfig1 musl
```` 
- 2
````bash
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.4.1_amd64.deb
```` 
- 3
````bash
sudo dpkg -i grafana-enterprise_10.4.1_amd64.deb
```` 
- Çalıştırmak için
````bash
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable grafana-server
 sudo /bin/systemctl start grafana-server
```` 
`Grafana  default olarak 3000 portunda çalışmaktadır.`

