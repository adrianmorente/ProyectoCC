# Provisionamiento de máquina virtuales
---

En este documento se detallan los distintos pasos seguidos hasta conseguir provisionar una máquina virtual en Azure, con todo lo necesario para poder ejecutar en ella nuestro proyecto. Para ello, se seguirán los siguientes pasos:

1. Uso de *Vagrant* y *Ansible* para provisionar una máquina virtual desde local. Más información [aquí]()

2. Uso de *Ansible* para provisionar una máquina virtual en la plataforma *Azure*. Más información [aquí]()

---
## *Vagrant* y *Ansible* para provisionar una máquina virtual desde local.
### Instalación de las herramientas necesarias

#### Instalación de Vagrant
En primer lugar, vamos a trabajar con máquinas virtuales locales. Para ello, necesitamos instalar una herramienta que nos permita gestionar máquinas virtuales, de forma que podamos arrancarlas, provisionarlas y destruirlas fácilmente.

Por ello, se ha hecho uso de [Vagrant](https://www.vagrantup.com/). Se ha utilizado esta herramienta ya que, como vimos en el seminario de Ansible impartido en la asignatura, Vagrant permite configurar máquinas virtuales de una manera sencilla, además de ser muy fácil de cambiar esa configuración para trabajar con máquinas virtuales en la nube.

El primer paso por tanto, es instalar la herramienta. Para ello, hemos seguido los pasos vistos [aquí](https://howtoprogram.xyz/2016/07/23/install-vagrant-ubuntu-16-04/). Para un correcto funcionamiento de la herramienta, es esencial tener en cuenta dos aspectos:
- Necesitamos una herramienta como VirtualBox, donde podamos gestionar las máquinas virtuales que se están creando y acceder a las mismas.

- Hay que tener cuidado con la versión de Vagrant que instalamos. Si instalamos la herramienta por línea de órdenes tal y como se indica en el enlace de descarga anterior, la versión que se descarga por defecto es *Vagrant 1.8.1*. Suponiendo que queremos trabajar con *VirtualBox* (como es mi caso), es importante saber que Vagrant no trabaja con las últimas versiones de *VirtualBox*, por lo que debemos actualizar, como mínimo, a la versión 2.0.2. Para ello, se pueden seguir los pasos vistos [aquí](https://github.com/openebs/openebs/issues/32).


#### Instalación de Ansible
El primer paso es instalar ansible en la máquina con la que estemos trabajando. Para poder disponer de ansible podemos instalarla desde dos formas principales:
- Utilizar el gestor de paquetes *apt-get*, tal y como se puede ver indicado [aquí](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-16-04).
- Instalarlo mediante *pip*. En este ejercicio, vamos a seguir esta segunda forma, ya que como se vio en el seminario de Ansible de la asignatura, el instalar Ansible mediante *pip* tiene sus ventajas. Esto se debe a que te instala, de forma automática, otros modulos necesarios, como por ejemplo para trabajar con YAML (lo necesitaremos más tarde). Podemos ver cómo realizar la instalación [aquí](https://docs.ansible.com/ansible/2.7/installation_guide/intro_installation.html#latest-releases-via-pip)

En la siguiente imagen podemos ver cómo se ha llevado a cabo la instalación de ansible, versión 2.7.2.
![Instalación de Ansible](https://raw.githubusercontent.com/andreamorgar/ejerciciosCC/master/images/instalacionAnsible2.png)



### Creación de una máquina virtual con Vagrant
Una vez que tenemos Vagrant correctamente instalado, nos situamos en un directorio sobre el que trabajar. En mi caso, todo este proceso lo he realizado desde mi repositorio de ejercicios, por lo que una vez situada en la carpeta correspondiente, ejecutamos lo siguiente.
~~~
$ vagrant init
~~~
Con esta orden, estamos inicializando el directorio actual, de forma que sea un entorno *Vagrant*. Una vez ejecutada dicha orden, se crea un archivo *VagrantFile* en caso de que no exista anteriormente.
Este fichero recién creado, tenemos que modificarlo para adaptarlo a aquello que queramos hacer.

En primer lugar, **especificamos la máquina que queremos crear**. Para ello, podemos buscar [aquí](https://app.vagrantup.com/boxes/search?utf8=%E2%9C%93&sort=downloads&provider=&q=ubuntu) el nombre asociado al sistema operativo que queremos que tenga la máquina virtual que vamos a crear. Este nombre, será el que debemos asociar a "config.vm.box" en el fichero VagrantFile. En mi caso, he especificado que la máquina que quiero crear tenga como sistema operativo *Debian 9*. Se ha elegido este sistema operativo por diversas razones. Las dos más importantes son, en primer lugar, que cuenta con una versión de Python3 ya instalada, sobre la cuál podemos trabajar directamente. En segundo lugar, porque además de lo anterior, se indica que es un sistema operativo oficial, y no uno facilitado por algún usuario de la plataforma. En este punto, el contenido del fichero VagrantFile sería el que se muestra a continuación.

**Contenido del fichero *VagrantFile* hasta el momento:**
~~~
Vagrant.configure("2") do |config|
  config.vm.box = "debian/contrib-stretch64"
  config.vm.hostname = "ubuntuAndrea"

end
~~~

Por tanto, vamos a crear la máquina. Para ello ejecutamos la siguiente orden:
~~~
$ vagrant up
~~~
Una vez finalice la creación de dicha máquina, podemos abrir *VirtualBox*, y comprobar que, efectivamente se ha creado dicha máquina. Lo podemos ver en la siguiente imagen.
![Creación de la máquina virtual con Vagrant](https://raw.githubusercontent.com/andreamorgar/ejerciciosCC/master/images/hito3/mv.png)


Como podemos observar en el fichero de VagrantFile anteriormente mostrado, en este punto aún no hemos realizado ningún provisionamiento con *Ansible*.
Sin embargo, para ver si la máquina virtual que hemos creado a través de *Vagrant* está operativa, podemos hacer un simple **ping**, y de esta forma comprobarlo. En la siguiente figura, podemos ver cómo realmente funciona. En la primera orden ejecutada, podemos ver cómo estamos haciendo ping a todas las máquinas virtuales. En nuestro caso, tenemos únicamente una, por lo tanto la orden ejecutada es equivalente a hacer ping directamente de nuestra máquina. Lo hacemos también, y vemos como efectivamente obtenemos igual resultado.

![Ping a la máquina que hemos creado](https://raw.githubusercontent.com/andreamorgar/ejerciciosCC/master/images/hito3/ping.png)


Además, podemos acceder a la máquina mediante ssh, tal y como se puede ver a continuación. Como podemos observar en la imagen, hemos podido conectarnos de forma correcta mediante SSH. Además, he ejecutado algunas órdenes para poder conocer mejor el estado en el que se encuentra la máquina:
- En primer lugar, podemos comprobar que el sistema operativo de la máquina es el que queríamos, mediante la ejecución del comando **hostnamectl**. Podemos ver también, que se ha creado con el nombre que especificamos en el fichero *VagrantFile*.

- Por otra parte, podemos ver cómo las utilidades que pretendemos instalar con el provisionamiento (como **git** o **pip3**), no están. Esto nos servirá para que, cuando ejecutemos la orden asociada al provisionamiento, veamos cómo realmente hemos hecho un provisionamiento correcto.

![Acceso por ssh a la máquina que hemos creado](https://raw.githubusercontent.com/andreamorgar/ejerciciosCC/master/images/hito3/pruebaSSH.png).



---
### provision
- **Indicar el provisionamiento para dicha máquina**: para ello, le indicamos el fichero *playbook* que queremos ejecutar, el cuál contiene el provisionamiento que queremos que tenga la máquina virtual que hemos especificado anteriormente. Como vemos en el contenido del fichero VagrantFile (mostrado a continuación), ya estamos haciendo uso de Ansible para poder llevar a cabo dicha tarea.


#### Contenido del fichero *VagrantFile*:
~~~
Vagrant.configure("2") do |config|
  config.vm.box = "debian/contrib-stretch64"
  config.vm.hostname = "ubuntuAndrea"

  config.vm.provision "ansible" do |ansible|
    ansible.verbose = "v"
    ansible.playbook = "playbook.yml"
  end
end
~~~

Una vez completado el fichero *VagrantFile*, podemos crear la máquina virtual con la siguiente orden:
~~~
$ vagrant up
~~~



<!-- Instalar Vagrant
sudo apt-get update
sudo apt-get install vagrant
vagrant -- version  (sale vagrant 1.8.1)
Nos vamos a ejercicios (mi carpeta de github) desde local, y abrimos una nueva carpeta, que se va a llamar tema3.
Entramos a esa carpeta y hacemos vagrant init -->






Vamos a crear una máquina virtual por defecto, del nombre que nosotros queramos. En este caso voy a utilizar la versión LTS de ubuntu server 16.04, y vamos a escribir en VagrantFile de la misma forma que lo vimos en el seminario. Es decir, estamos creando una máquina default.

Con vagrant estamos configurando la maquina virtual que vamos a utilizar inicialmente.









<!--### Instalación de Ansible
1. Instalar ansible
pip install paramiko PyYAML jinja2 httplib2 ansible

2. Me ha dicho que dentro de mi entorno no tengo la versión más actualizda de pip, así que la actualizo. pip install --upgrade pip

3. Instalo virtualbox, pero me da problemas pq me pide que quite la seguridad de mi SO. Por eso, sigo los pasos aqui:
https://stegard.net/2016/10/virtualbox-secure-boot-ubuntu-fail/. Faltan pasos 5,6,y 7 (?))))-->






### Fichero de ansible.cfg
- Primero indicamos a false que no se haga la comprobación de claves del host, para evitar los problemas de Man in The Middle, que SSH no haga la comprobación de clave, y podamos entrar con diferentes nombres y mac.
- Estamos diciendole cual va ser el nombre de fichero de host. Tendremos un fichero *ansible.host* con el que vamos a trabajar y vamos a definir las máquinas con las que estemos.

### Fichero ansible_hosts
Los aspectos importantes de este fichero son:
- Indicar el puerto
- Indicar bien la ruta hacia la máquina y private_key


### Provisionamiento
Vamos a instalar en la máquina virtual todo aquello que necesitemos. Para ello, podemos consultar la guía oficial [aquí](https://docs.ansible.com/ansible/2.7/scenario_guides/guide_vagrant.html), concretamente el apartado Vagrant Setup. Aquí se muestra un ejemplo de cómo podemos modificar el fichero VagrantFile para provisionar una única máquina. Para ello, vamos a usar el Playbook.

#### Añadimos provisionamiento a VagrantFile

El fichero, con el añadido, sería tal y como se muestra a continuación.
~~~
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "ubuntuAndrea"

  config.vm.provision "ansible" do |ansible|
    ansible.verbose = "v"
    ansible.playbook = "playbook.yml"
  end
end
~~~
Podemos destacar dos aspectos principales:
- La sección de provisionamiento hace referencia a un playbook de ansible al que en el fichero *VagrantFile* hemos llamado *playbook.yml*.
- Vagrant ejecutará el fichero de provisionamiento  que hemos definido una vez que la máquina virtual arranca y tiene acceso a SSH.
- El hecho de tener activada la opción verbose va a provocar que se nos muestre la totalidad del comando del ansible playbook que utilicemos. Todavía no sabemos qué nos mostrará, entonces la voy a dejar de momento, y más adelante ya veremos si se quita del fichero o no.


#### Fichero playbook.yml
Para entender bien el funcionamiento de un playbook de ansible, y sobretodo, qué hace exactamente y de qué forma, podemos consultar el apartado correspondiente en la guía oficial [aquí](https://docs.ansible.com/ansible/2.7/user_guide/playbooks_intro.html). Además, se sugiere consultar este otro [enlace](https://github.com/ansible/ansible-examples), pues contiene una serie de ejemplos y buenas prácticas que se pueden llevar a cabo. De estos dos enlaces, es de donde nos basaremos para llevar a cabo este apartado.

<!-- - Lo primero: python3 [aquí](https://medium.com/@perwagnernielsen/ansible-tutorial-part-2-installing-packages-41d3ab28337d) -->
De momento, tenemos hasta este punto un único playbook genérico. Para su creación, me he inspirado en el tutorial al que se puede acceder desde [aquí](https://medium.com/@perwagnernielsen/ansible-tutorial-part-2-installing-packages-41d3ab28337d).

Hemos utilizado una máquina Debian con Python 3, que ha sido escogida debido a que de esta forma, no solo contamos con Python instalado en la máquina, sino que por defecto ya trae consigo Python 3, tal y como se puede consultar [aquí](https://linuxconfig.org/how-to-change-default-python-version-on-debian-9-stretch-linux).

En primer lugar, el archivo playbook.yml va a representar únicamente a aquellas cosas genéricas que queramos instalar en una máquina virtual. Por tanto, tendríamos que instalar dos cosas indispensables para nuestro servicio web de la práctica anterior, las cuáles obtendremos a través del gestor de paquetes **apt**:
- **Git**: nos hace falta para poder acceder a nuestro proyecto desde la máquina virtual que hemos creado mediante Vagrant.. Sin git, entre otras cosas, no podremos hacer clone de nuestro repositorio, por lo que es esencial en este caso.

- **python-pip**: para poder hacer uso de pip. Se ha comprobado experimentalmente, que para el funcionamiento correcto de pip3 desde ansible, se debe instalar pip, y posteriormente, indicar el ejecutable concreto de pip con el que queremos funcionar.

- **python3-pip**: para poder utilizar pip3 y descargar aquello que necesitemos para la versión 3 de Python. Es necesario porque voy a instalar los requerimientos para poder ejecutar mi proyecto en la máquina de esa forma. Como estamos trabajando con Python 3, queremos pip 3 concretamente.

- **python-setuptools**: necesario para poder ejecutar los requirements. Este paquete fue añadido posteriormente, ya que uno de los errores obtenidos al intentar provisionar la máquina indicaba la necesidad de disponer de este paquete.

Hasta aquí tendríamos todas las utilidades generales necesarias que deben existir en la máquina virtual de forma que podamos ejecutar nuestra aplicación.


<!-- Para ello, modificamos el fichero *playbook.yml* tal y como se muestra a continuación:
~~~
---
- hosts: all
  become: yes
  tasks:
    - name: Instala git
      apt: pkg=git state=present
~~~ -->


#### Siguiendo las buenas prácticas....

A pesar de que hay múltiples fuentes que defienden que un playbook debe ser un proceso cerrado (como por ejemplo [aquí](https://serverfault.com/questions/750856/how-to-run-multiple-playbooks-in-order-with-ansible)), esta afirmación no es compartida por el estándar de buenas prácticas de Ansible.

Si consultamos la guía de buenas prácticas de Ansible, podemos encontrar una sección llamada *Creating Reusable Playbooks*, a la cuál podemos acceder desde [aquí](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html). En ella podemos ver, que es preferible reutilizar distintos playbooks, en lugar de empezar con uno de la forma que vimos arriba. Por ello, la parte específica de lo que queremos ejecutar, la vamos a especificar en un segundo playbook, que contendrá todos aquellos específicos para poder ejecutar y desplegar nuestro proyecto:

- **git clone**: para poder descargar nuestro repositorio en la máquina virtual.

- **Dependencias específicas de la aplicación**: en este caso, aquello que sea imprescindible para el correcto funcionamiento del servicio web.


Por tanto utilizaremos un nuevo fichero, al que hemos llamado *especific_playbook.yml*,
el cuál se encargará de incorporar aquellos aspectos esenciales.

Nos quedaría por resolver cómo llevar a cabo la inclusión del playbook con el contenido específico que queremos incorporar. Para ello, hay múltiples formas, como podemos observar en la documentación oficial disponible [aquí](https://docs.ansible.com/ansible/2.4/playbooks_reuse_includes.html). La principal duda estaría en... ¿qué utilizar? ¿es preferible utilizar include  para incorporar otros playbooks al playbook principal?¿O es mejor si utilizamos import? Al final, ninguna de las opciones principales de la documentación es la solución, sino que lo preferible, en este momento, es hacer uso de *import_playbook*, ya que será la única disponible para futuras versiones de ansible. Esta información la podemos obtener si provisionamos la máquina indicando -v en la opción verbose de *VagrantFile*.

![Preferible usar import_playbook](https://raw.githubusercontent.com/andreamorgar/ejerciciosCC/master/images/razonImport.png)


Por tanto, el contenido final de nuestro playbook principal quedaría de la siguiente manera.

~~~
---
- hosts: all
  become: yes
  gather_facts: False
  tasks:
    - name: Install base packages
      apt: name={{ item }} state=present
      with_items:
        - git
        - python-pip
        - python3-pip
        - python-setuptools
      tags:
        - packages
- import_playbook: specific_playbook.yml
~~~

#### Fichero specific_playbook.yaml

Como se ha indicado anteriormente, en este playbook nos encargaremos del provisionamiento que es específico a la aplicación que queremos desplegar. Por ello, tendrá dos cosas esenciales:
- **Clonación del proyecto**: tenemos que clonar nuestro proyecto en la máquina virtual en cuestión. Esta acción podemos llevarla a cabo sin problemas, ya que como vimos anteriormente, se ha instalado *git*. Podemos especificar el nombre con el que queremos que se nos guarde el repositorio, y además debemos especificar **clone: yes** en la clonación. Los pasos seguidos se pueden ver en la documentación de Ansible [aquí](https://docs.ansible.com/ansible/2.5/modules/git_module.html)

- **Instalación de los paquetes definidos en requirements.txt**. Este paso lo llevaremos a cabo mediante **pip**. Para ello, especificamos que instale en su última versión, el contenido que tenga el fichero *requirements.txt*. Para poder llegar hasta el contenido de este fichero, podemos poner la ruta concreta. En este caso, el usuario que estamos utilizando es *vagrant*, por lo que éste es el que debe estar en la ruta hasta llegar al fichero de requirements. Por último, debemos indicar de que, de los distintos ejecutables de pip que están instalados en el sistema, coja *pip3*.


A continuación podemos ver el contenido de este playbook específico:
~~~
---
- hosts: all
  tasks:
    - name: Clone repo
      become: False
      git:
        repo: https://github.com/andreamorgar/ProyectoCC.git
        clone: yes
        dest: ProyectoCC

    - name: Install requirements
      pip:
        state: latest
        requirements: /home/vagrant/ProyectoCC/requirements.txt
        executable: pip3
~~~








### MongoDB

1. Empezar a usar pymongo
~~~
$ pip3 install pymongo
~~~
2. Hay que modificar requirements: pymongo==3.7.2

[Tutorial para usar mongo](https://datawookie.netlify.com/blog/2016/09/python-first-steps-with-mongodb/)

[Tutorial para usar mLab](https://gist.github.com/nikhilkumarsingh/a50def43d8d425b4108c2f76edc1398e)

https://gist.github.com/nikhilkumarsingh/a50def43d8d425b4108c2f76edc1398e


#### Azure
1. Nos vamos a portal
2. Creamos recurso


https://docs.microsoft.com/en-us/azure/virtual-machines/linux/nsg-quickstart


### Travis

https://docs.travis-ci.com/user/gui-and-headless-browsers/ Hacer SUDO para puertos <1024
