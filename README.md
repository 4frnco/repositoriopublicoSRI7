P7-DNS_Linux

### 1. Preparación del entorno  
- Creamos un archivo llamado `docker-compose.yml` para definir los servicios, las configuraciones, y los archivos de zonas DNS.  


---

### 2. Configuración del archivo `docker-compose.yml`  

#### Contenido del archivo:  
```
services:
  bind9:
    container_name: serverfranco
    image: internetsystemsconsortium/bind9:9.18
    platform: linux/amd64
    ports:
      - 53:53/tcp
      - 53:53/udp
    networks:
      franco_subnet:
        ipv4_address: 192.168.1.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
    restart: always
  cliente:
    container_name: cliente_franco
    image: alpine
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 192.168.1.1
    networks:
      franco_subnet:
        ipv4_address: 192.168.1.2

networks:
  franco_subnet:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.0.0/16
          ip_range: 192.168.1.0/24
          gateway: 192.168.1.254
```  

#### Descripción:  
1. **Servidor (`serverfranco`):**
   - Utilizamos la imagen `bind9:9.18`.
   - Exponemos los puertos `53:53` en TCP y UDP.
   - Configuramos una red personalizada con la IP `192.168.1.1`.
   - Asociamos volúmenes locales para las configuraciones y zonas.  

2. **Cliente (`cliente_franco`):**
   - Usamos una imagen `alpine`.
   - Activamos la terminal con `tty` y `stdin_open`.
   - Configuramos el cliente para usar el DNS `192.168.1.1` y asignamos la IP fija `192.168.1.2`.  

3. **Red (`franco_subnet`):**
   - Definimos una red tipo `bridge`.
   - Establecemos un rango de IPs asignables entre `192.168.1.0/24` y la puerta de enlace `192.168.1.254`.  

---

### 3. Configuración de archivos  

#### **1. Archivo `named.conf.local`**  
```
zone "asirfranco.int" {
    type master;
    file "/etc/bind/db.asirfranco.int";
};
```  

#### **2. Archivo `named.conf.options`**  
```
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    dnssec-validation no;
    forwarders {
        8.8.8.8;
        1.1.1.1;
    };
    listen-on { any; };
    listen-on-v6 { any; };
};
```  

#### **3. Archivo `named.conf`**  
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```  

---


