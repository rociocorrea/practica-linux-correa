# Errores encontrados y resolución

## DOCKER

### 1. Error de sintaxis en docker-compose.yml
Ejecuto el docker-compose.yml original y el primer error que veo es este:

*vagrant@linux-practice:/vagrant/contenedores$ docker-compose up
ERROR: The Compose file './docker-compose.yml' is invalid because:
'grafana', 'loki' do not match any of the regexes: '^x-'*

*You might be seeing this error because you're using the wrong Compose file vershe `version` key and place your service definitions at the root of the file to
For more on the Compose file format versions, see https://docs.docker.com/compo
vagrant@linux-practice:/vagrant/contenedores$*


Leyendo este mensaje entiendo que en la parte de declaración de  ‘grafana’ y ‘loki’ hay algo incorrecto en el formato y revisando la indentación de todos los otros, noto que no están en la misma altura, entonces los indento para corregir este primer error.


### 2. Error de volumen no declarado
Ejecuto docker-compose up nuevamente y veo este error:

*vagrant@linux-practice:/vagrant/contenedores$ docker-compose up
ERROR: Named volume "grafana-data:/var/lib/grafana:rw" is used in service "grafana" but no declaration was found in the volumes section.*

Identifica el problema en esta parte del .yml:
*volumes:
      - grafana-data:/var/lib/grafana*

y esta:
*volumes:
  grafana-storage:*

Acá el nombre es distinto entre las dos instancias, entonces cambio grafana-storage por grafana-data en volumes para que quede consistente: 

*volumes:
  grafana-data:*

### 3. Error de red no identificada
Ejecuto docker-compose up nuevamente y este es el nuevo error:

*vagrant@linux-practice:/vagrant/contenedores$ docker-compose up
ERROR: Service "redis" uses an undefined network "monitoring-network"*

Siguiendo la pista de la consigna y el ejemplo anterior, identifico que hay una declaración de networks inconsistente, en este caso en redis:

*redis:
    container_name: redis-practica
    image: redis:alpine
    ports:
      - "6379:6379"
    restart: unless-stopped
    networks:
      - monitoring-network*

cuando en las demás declaraciones se encuentra así:
*networks:
      - monitoring*

Así que basta con cambiar de monitoring-network por monitoring.

### 4. Target DOWN en Prometheus

El target de Nginx aparece como DOWN porque Prometheus intenta recolectar métricas en el puerto 9113, 
pero el contenedor nginx:alpine no expone métricas por defecto y no hay un exporter instalado. 
Esto no significa que Nginx esté caído, solo que Prometheus no puede conectarse a ese endpoint.
