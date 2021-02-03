#ANSIBLE

Creamos entorno virtual y activamos entorno:

~~~
python3 -m venv ansible

source ansible/bin/activate
~~~

Instalamos ansible:

~~~
pip install ansible
~~~

Ficheros de configuración pueden estan guardados en diferentes entornos:

ANSIBLE_CONFIG(variable entorno)
ansible.cfg(directorio actual)
.ansible.cfg(directorio home)

Para ejecutar un modulo en ansible con el modulo ping(sirve para ver si está conectada con las otras máquinas). Se puede ejecutar a una máquina, a un conjunto de máquinas o a todas las máquinas.

~~~
ansible -m ping all
~~~

Tendremos que crear un fichero Playbooks que será con extension `.yaml`.

~~~
nano site.yaml
~~~

Dentro tendremos las plays que vamos a ejecutar. En este caso vamos a ejecutar el role commons que hace un conjunto de comandos de actualizacion del sistema.

~~~
- hosts: all
  become: true
  roles:
   - role: commons
~~~

El rol commons tiene la siguiente información:

~~~
- name: Ensure system is updated
  apt: update_cache=yes upgrade=yes
~~~

Para ejecutar el rol commons:

~~~
ansible-playbook -b site.yaml
~~~