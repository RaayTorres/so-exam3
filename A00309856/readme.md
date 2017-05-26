
### Parcial 3 sistemas operativos

### Estudiantes: 
**Ray Torres - A00309856**

**URL de github: //www.github.com/RaayTorres/so-exam3**

### Procedimiento

* ** Configuración del cliente **

Para configurar el cliente se realizaron los siguientes pasos:

primero se creo el archivo de configuración de sensu:
``` 
echo '[sensu]
name=sensu
baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
gpgcheck=0
enabled=1' | sudo tee /etc/yum.repos.d/sensu.repo

```

posteriormente se instalo sensu y su plugin mediante los siguientes comandos:

```
yum install sensu -y
sensu-install -p sensu-plugin
```

se inicia el servicio del sensu del cliente:

```
service sensu-client start
```
por ultimo se instalo el servicio httpd por medio del comando:

```
yum install httpd -y
```

y se inicio el servicio httpd por medio del comando:

```
systemctl start httpd.service
```


* ** Configuración rabbit MQ del cliente**

Vamos al archivo de configuracion de sensu ubicado en la siguiente direccion:
```
cd /etc/sensu/conf.d
```
Dentro  se crean dos archivos json (cliente.json) y (rabbitmq.json)

```
{
  "client": {
    "name": "A00317220",
    "address": "192.168.57.4",
    "subscriptions": ["webservers"]
  }
}


{
  "rabbitmq": {
    "host": "192.168.57.3",
    "port": 5672,
    "vhost": "/sensu",
    "user": "sensu",
    "password": "password",
    "heartbeat": 10,
    "prefetch": 50
  }
}

```

Finalmente se instalaron algunos plugins necesarios para el correcto funcionamiento del daemon:

```
vim /etc/sensu/plugins/check-apache.rb
```
Con la siguiente informacion:
```
#!/usr/bin/env ruby

procs = `ps aux`
running = false
procs.each_line do |proc|
  running = true if proc.include?('httpd')
end
if running
  puts 'OK - Apache daemon is running'
  exit 0
else
  puts 'WARNING - Apache daemon is NOT running'
  exit 1
end

```

* ** Configuración del servidor**

La instalacion del servivor es igual a la del cliente hasta la instalacion del sensu:

```
echo '[sensu]
name=sensu
baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
gpgcheck=0
enabled=1' | sudo tee /etc/yum.repos.d/sensu.repo
```

```
yum install sensu -y
sensu-install -p sensu-plugin
```
Adcionalmente se instalaron plugins y servicios como:

```
sensu-install -p sensu-plugins-slack
su -c 'rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm'
```

Se instalo erlang

```
yum install erlang -y
```

Se instalo redis

```
yum install redis -y
```
Iniciar el servicio redis
```
service redis start
```

Se instalo socat
```
yum install socat -y
```

* ** Configuración rabbit MQ del servidor**

```
su -c 'rpm -Uvh http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.9/rabbitmq-server-3.6.9-1.el7.noarch.rpm'
service rabbitmq-server start
rabbitmqctl add_vhost /sensu
rabbitmqctl add_user sensu password
rabbitmqctl set_permissions -p /sensu sensu ".*" ".*" ".*"
rabbitmq-plugins enable rabbitmq_management
chown -R rabbitmq:rabbitmq /var/lib/rabbitmq

```

 se configura un usuario para probar el funcionamiento de rabbitmq

```
rabbitmqctl add_user test test
rabbitmqctl set_user_tags test administrator
rabbitmqctl set_permissions -p / test ".*" ".*" ".*"

```

* se instala uchiwa por medio del comando

```
yum install uchiwa -y
```

se realiza la configuracion de los puertos a usar:

```
firewall-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --zone=public --add-port=15672/tcp --permanent
firewall-cmd --zone=public --add-port=3000/tcp --permanent
firewall-cmd --reload
```
y se reinician los servicios
```
service sensu-server restart
service sensu-api restart
service uchiwa restart

```

** pruebas de  funcionamiento  rabbitmq y uchiwa**

* Prueba de funcionamiento de Rabbitmq

* Prueba 1

![GitHub Logo0](Imagenes/Rabbit.png)

* Prueba 2

![GitHub Logo0](Imagenes/Rabbit2.png)

* Prueba de funcionamiento de uchiwa

* Prueba 1
![GitHub Logo0](Imagenes/uchiwa.png)

* Prueba 2
![GitHub Logo0](Imagenes/uchiwa2.png)


A continuación se muestra la generación de una alerta por parte de uchiwa cuanto se detecta la caida de un servicio (ver el gif hasta el final duración aproximada 60 segundos)

![GitHub Logo0](Imagenes/parcial3.gif)




## falta punto 5 y 6


**A continuación se muestra los pasos para la configuración y la instalación del stack de ELK**

se instalo el openjdk de Java

```
yum install java-1.8.0-openjdk.x86_64
```
* Elasticsearch

se descarga e instala la llave publica de elasticsearch

```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

```

Para poder instalar Elasticsearch se debe crear un archivo de configuracion de Elasticsearch  que contenga el repositorio  con la siguiente informacion

```
vi /etc/yum.repos.d/elasticsearch.repo
```

```
[elasticsearch-5.x]
name=Elasticsearch repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

se instala elasticsearch por medio del comando

```
yum install elasticsearch -y
```

* Logstash

Para poder instalar logstash se debe crear un archivo de configuracion de logstash  que contenga el repositorio  con la siguiente informacion

```
vi /etc/yum.repos.d/logstash.repo
```

```
[logstash-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md

```

se instala logstash

```
yum install logstash -y
```

* KIBANA

Para poder instalar kibana se debe crear un archivo de configuracion de kibana  que contenga el repositorio  con la siguiente informacion

```
vi /etc/yum.repos.d/kibana.repo
```
```
[kibana-5.x]
name=Kibana repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

se instala kibana

```
yum install kibana -y
```

* Instalación de filebeat 

Para poder instalar filebeat se debe crear un archivo de configuracion de filebeat  que contenga el repositorio  con la siguiente informacion

```
vi /etc/yum.repos.d/filebeat.repo
```

```
[elastic-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

se instala filebeat mediante el siguiente comando:

```
sudo yum install filebeat -y
```

* Una vez hecha toda la instalación de elasticsearch, se hizo una prueba de funcionamiento con el ejemplo 0 del gist.github.com/d4n13lbc/be1ad5039dff1c058b335482488d4965


## Ejemplo 0

Para la realización de este ejemplo deben realizarse las siguientes configuraciones:

Primero  se configura el siguiente archivo

```
vi /etc/elasticsearch/elasticsearch.yml
```

Aqui esta parte del archivo debe quedar igual, con la unica diferencia que lo que va a cambiar es la dirección ip

```
# Set the bind address to a specific IP (IPv4 or IPv6):
network.host: 192.168.57.5
# Set a custom port for HTTP:
http.port: 9200
```
Una vez hecha la configuración deben abrirse los puertos

```
firewall-cmd --zone=public --add-port=9200/tcp --permanent
firewall-cmd --reload
```

Luego realizamos la siguiente configuración para kibana

```
vi /etc/kibana/kibana.yml
```

Aqui también lo unico que cambiamos es la dirección ip

```
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 5601
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "192.168.57.5"
# The URL of the Elasticsearch instance to use for all your queries.
elasticsearch.url: "http://192.168.57.5:9200
```

Luego de haber configurado el archivo se abren los puertos

```
firewall-cmd --zone=public --add-port=5601/tcp --permanent
firewall-cmd --reload
```

Finalmente iniciamos los servicios

```
service kibana start
service elasticsearch start
```

* Pruebas de funcionamiento

## Kibana

![GitHub Logo0](Imagenes/kibana.png)


## Elasticsearch

![GitHub Logo0](Imagenes/ejemplo0.png)
