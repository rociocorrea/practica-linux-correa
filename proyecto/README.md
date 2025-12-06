# Práctica Linux - Correa Rocío Ayelén(alumna A, B, C)

Para la organización de este trabajo utilicé un tablero de Trello para identificar tareas de cada rol, aquellas en estado *In progress* y aquellas *Done*.

## Configuración de la VM con Vagrant

### Problema inicial

El `Vagrantfile` original no funcionó en mi computadora. El primer error se dio porque **fastfetch** no estaba disponible, y esto causaba errores en cascada.

#### Solución:

Instalar dependencias y compilar fastfetch manualmente:

# Instala dependencias para compilar
apt-get install -y git build-essential cmake libssl-dev

# Clona y compila Fastfetch
git clone https://github.com/LinusDierheimer/fastfetch.git /tmp/fastfetch
cd /tmp/fastfetch
mkdir build && cd build
cmake ..
make
make install

Además se agregan dependencias: build-essential, cmake y libssl-dev,  porque son necesarias para que fastfetch funcione.

# Vagrantfile final

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "linux-practice"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2

    unless File.exist?('./disk1.vdi')
      vb.customize ['createhd', '--filename', './disk1.vdi', '--size', 2048]
    end

    vb.customize [
      'storageattach', :id,
      '--storagectl', 'SCSI',
      '--port', '2',
      '--device', '0',
      '--type', 'hdd',
      '--medium', './disk1.vdi'
    ]
  end

  config.vm.network "public_network"

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y git build-essential cmake libssl-dev docker.io docker-compose lvm2

    git clone https://github.com/LinusDierheimer/fastfetch.git /tmp/fastfetch
    cd /tmp/fastfetch
    mkdir build && cd build
    cmake ..
    make
    make install

    usermod -aG docker vagrant
    systemctl enable docker
    systemctl start docker
  SHELL
end

## Ejercicio 0 - Dirección IP
/vagrant de la VM está sincronizado con mi /linux-practice/proyecto donde tengo mi Vagrantfile y pude escribir sobre el archivo .txt indicado por la consigna sin problemas.

## Ejercicio 1 - Creación de repositorio
Se inicia el repositorio y se levantan dos VMs para simular trabajo en equipo. La captura de esto se encuentra en /contenedores/capturas/creación de repositorio.png

## Ejercicio 2 - Fastfetch Colaborativo
Se ejecutó sin problemas el fastfetch con los tres roles.

## Ejercicio 3 - Gestión de permisos
Se ejecutó sin problemas la lista de comandos para luego hacer la verificación de usuarios y permisos.

### Parte colaborativa
Ejecuto la creación de usuarios con comandos indicados para cada rol, creando usuarios, agregándolos a un grupo y generando un directorio para simular colaboración.

Nota: Para que los grupos se apliquen correctamente, fue necesario cerrar sesión y volver a entrar.

## Ejercicio 4 - Operaciones con Logical Volume Manager
Pude lograr las operaciones de creación de volumenes, montado, desmontado sin problemas con los comandos propuestos en los tres roles.

## Ejercicio 5 - Gestión de Archivos y Directorios
No pude crear los directorios con *mkdir* porque se necesitaban permisos de root probablemente por ser un volumen LVM recién montado (*Permission denied* era el error), entonces los cree con sudo. Al intentar crear los txt de testeo también se me limitó el acceso por lo que cambie el propietario de la carpeta al usuario vagrant que es el usuario que estoy utilizando para los tres roles.
Ejecuté *sudo chown -R vagrant:vagrant /mnt/lvm_storage_correa* que asigna como propietario a mi usuario vagrant.

En el comando proporcionado: 
*for i in {01..10}; do touch documento_$i.txt echo "Contenido del documento $i" > documento_$i.txt done*

debí hacer modificaciones para que funcione correctamente:
*for i in {01..10}; do touch documento_$i.txt; echo "Contenido del documento $i" > documento_$i.txt; done*

Lo demás funcionó correctamente.

## Ejercicio 6 - Contenedores y Monitoreo con Docker Compose
Esta parte fue la más compleja. Me basé en ejemplos de .ymls vistos en clase y los errores que mostraba la consola para arreglar los errores.

1 - Ejecuto el docker-compose.yml original y el primer error que veo es este:

*vagrant@linux-practice:/vagrant/contenedores$ docker-compose up ERROR: The Compose file './docker-compose.yml' is invalid because:'grafana', 'loki' do not match any of the regexes: '^x-'*

*You might be seeing this error because you're using the wrong Compose file vershe `version` key and place your service definitions at the root of the file to For more on the Compose file format versions, see https://docs.docker.com/compo*

Leyendo este mensaje entiendo que en la parte de declaración de  ‘grafana’ y ‘loki’ hay algo incorrecto en el formato y revisando la indentación de todos los otros, noto que no están en la misma altura, entonces los indento para corregir este primer error.

2 - Ejecuto docker-compose up nuevamente y veo este error:

*vagrant@linux-practice:/vagrant contenedores$ docker-compose up ERROR: Named volume "grafana-data:/var/lib/grafana:rw" is used in service "grafana" but no declaration was found in the volumes section.*

Que identifica el problema en estas partes del .yml:
*volumes:*
*- grafana-data:/var/lib/grafana*

*volumes:*
*grafana-storage:*

Acá el nombre es distinto entre las dos instancias, entonces cambio grafana-storage por grafana-data en *volumes* para que quede consistente:
*volumes:*
*grafana-data:*

3 - Ejecuto docker-compose up nuevamente y este es el nuevo error:

*vagrant@linux-practice:/vagrant/contenedores$ docker-compose up ERROR: Service "redis" uses an undefined network "monitoring-network"*

Siguiendo la pista de la consigna y el ejemplo anterior, identifico que hay una declaración de networks inconsistente, en este caso en *redis*, donde está declarada *networks: - monitoring-network* pero en todas las demás declaraciones se encuentra como *networks: -monitoring*.
Así que basta con cambiar de monitoring-network por monitoring.

Con respecto al archivo prometheus.yml, comento el job erróneo. Prometheus intenta conectarse a ngnix:9113 pero no le es posible ya que no hay un exporter instalado, por lo tanto figura como target DOWN.

Capturas adicionales se encuentran en /contenedores/capturas/





