# Para inciar desde cero - Instalar

<https://documentacion.siu.edu.ar/wiki/SIU-Guarani/version3.19.0/instalacion_desde_cero/instalacion/gestion/linux>

## SVN

svn co https://colab.siu.edu.ar/svn/guarani3/nodos/enerc/gestion/versiones/3.19.0.1/ gestion_31901

## Construir la imágen

```bash
./build73.sh
docker compose up -d
```

## Adentro del contenedor

```bash
docker exec -ti gestion_incaa_php73 bash
```

### 1. composer install

### 2. Instalar el framework SIU-Toba

```bash
cd bin
export TOBA_INSTANCIA=desarrollo
export TOBA_INSTALACION_DIR=/app/instalacion
./toba instalacion instalar
```

Completar:

* Nombre del Alias: guarani3
* Nro desarrollador: 1227
* Se trata de una instalacion de producción?: n
* Nombre de la instalación: guarani3
* Ubicación: db
* Puerto: 5432
* Usuario: postgres
* Clave: postgres
* Base de datos: guarani3
* Nombre del schema a usar: desarrollo
* Toba - Clave (usuario "toba"): toba

### 3. Darle permisos a las siguientes carpetas de manera recursiva para que el usuario con el que se ejecuta Apache pueda escribir.

Hacerlo desde la compu, afuera del contenedor!!!!

```bash
cd gestion_31901
sudo chown -R $(whoami):www-data www temp instalacion vendor/siu-toba/framework/www vendor/siu-toba/framework/temp
chmod 775 -R www temp instalacion vendor/siu-toba/framework/www vendor/siu-toba/framework/temp
```

### 4. Crear un link simbólico a toba.conf. 

Desde adentro del contenedor!!! (gestion_incaa_php73)

```bash
ln -s /app/instalacion/toba.conf /etc/apache2/sites-available/gestion.conf
a2ensite gestion.conf
service apache2 reload
```

### 5. Agregar los parámetros de localización de fop al final del archivo de inicialización de la instalación Toba. 

Hacerlo desde la compu, afuera del contenedor!!!!

Modificar el archivo:

* gestion_31901/instalacion/instalacion.ini

```bash
[xslfo]
fop=/app/php/3ros/fop/fop
```

### 6. Cargar el proyecto. Desde adentro del contenedor!!! (gestion_incaa_php73)

```bash
bin/guarani cargar -d /app
    # Desea agregar el alias de apache al archivo toba.conf?: Si
service apache2 reload
```

#### 7. Configurar API REST ¿?¿?¿?  >>>> esto no hice nada, lo pasé por alto

#### 8. Crear la bd

```bash
bin/guarani  instalar
```

### 9. Levantar la BD del INCAA dentro del contenedor de postgres

```bash
docker compose exec -T db dropdb -U postgres guarani3
docker compose exec -T db createdb -U postgres guarani3
docker compose exec -T db psql -U postgres guarani3 < guarani3.20260218.sql
```

### 10. Si se quieren resetear las claves (123456):

```sql
UPDATE negocio.mdp_personas
    SET clave = '$2a$10$bPEtb1gPLMduHUgJsvq.V.D9c0uognsVFc2e39Ry3Gpbfgu9Znpf6'
WHERE true;
```

### 11. Activar esquema de auditoría Desde adentro del contenedor!!! (gestion_incaa_php73)

Realizar backup de los datos de auditoria no anda,  porque no encuentra pg_dump.

```bash
bin/guarani crear_auditoria -f guarani
```

### 12. Volver a hacer el punto 3. 

En algún momento algo se pierde... Darle permisos a las siguientes carpetas de manera recursiva para que el usuario con el que se ejecuta Apache pueda escribir. Hacerlo desde la compu, afuera del contenedor!!!!

```bash
cd gestion_31901
sudo chown -R $(whoami):www-data www temp instalacion vendor/siu-toba/framework/www vendor/siu-toba/framework/temp
chmod 775 -R www temp instalacion vendor/siu-toba/framework/www vendor/siu-toba/framework/temp
```

### 13. URL

<http://localhost:1040/guarani/3.19/>

### 14. Para levantar el PHP-Java Bridge, y que anden los reportes Jasper Reports, habría que hacer algo de esto. Pero no logré hacerlo andar.

```bash
java -jar /app/vendor/siu-toba/jasper/JavaBridge/WEB-INF/lib/JavaBridge.jar SERVLET_LOCAL:8081
```
