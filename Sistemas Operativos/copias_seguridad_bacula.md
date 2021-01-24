# Sistema de copias de seguridad Bacula

Vamos a realizar copias de seguridad en nuestras máquinas de Openstack, teniendo como servidor principal Dulcinea y los clientes serán Sancho, Quijote y Freston.

Vamos a implementar un sistema de copias de seguridad para las instancias del cloud y el VPS, teniendo en cuenta las siguientes características:

- Seleccionaremos la aplicación llamada Bacula.
- Utilizaremos Dulcinea como servidor de copias de seguridad, añadiéndole un volumen y almacenando localmente las copias de seguridad que consideres adecuadas en él.
- El proceso debe realizarse de forma completamente automática.
- Seleccionaremos qué información es necesaria guardar (listado de paquetes, ficheros de configuración, documentos, datos, etc.)
- Realizaremos semanalmente una copia completa
- Realizaremos diariamente una copia incremental, diferencial o delta diferencial (decidir cual es más adecuada)
- Implementaremos una planificación del almacenamiento de copias de seguridad para una ejecución prevista de varios años, detallando qué copias completas se almacenarán de forma permanente y cuales se irán borrando.
- Crearemos un registro de las copias, indicando fecha, tipo de copia, si se realizó correctamente o no y motivo.
- Seleccionaremos un directorio de datos "críticos" que deberá almacenarse cifrado en la copia de seguridad, bien encargándote de hacer la copia manualmente o incluyendo la contraseña de cifrado en el sistema.
- Vamos a incluir en la copia los datos de las nuevas aplicaciones que se vayan instalando durante el resto del curso.
- Utilizaremos saturno u otra opción que se te facilite como equipo secundario para almacenar las copias de seguridad. Solicita acceso o la instalación de las aplicaciones que sean precisas.

## Instalación y configuración de Bacula

Vamos a proceder a instalar primero mysql.
~~~
debian@dulcinea:~$ sudo apt install mariadb-server mariadb-client
~~~

Instalamos Bacula.
~~~
debian@dulcinea:~$ sudo apt install bacula bacula-client bacula-common-mysql bacula-director-mysql bacula-server
~~~

Vamos a entrar en el fichero de configuración de Bacula y configuraremos las distintas copias de seguridad.
Fichero de configuración:
~~~
debian@dulcinea:~$ sudo nano /etc/bacula/bacula-dir.conf
~~~

~~~
Director {                           
  Name = dulcinea-dir
  DIRport = 9101                
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "bacula"         
  Messages = Daemon
  DirAddress = 10.0.1.10
}

JobDefs {
  Name = "Backup-Diario"
  Type = Backup
  Level = Incremental
  Client = dulcinea-fd
  FileSet = "Full Set"
  Schedule = "Diario"
  Pool = Daily
  Storage = vol1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

JobDefs {
  Name = "Backup-Semanal"
  Type = Backup
  Level = Incremental
  Client = dulcinea-fd
  FileSet = "Full Set"
  Schedule = "Semanal"
  Pool = Weekly
  Storage = vol1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

JobDefs {
  Name = "Backup-Mensual"
  Type = Backup
  Level = Incremental
  Client = dulcinea-fd
  FileSet = "Full Set"
  Schedule = "Mensual"
  Pool = Monthly
  Storage = vol1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

#Configuración de las tareas de cada cliente y restauración de las máquinas.
Job {
 Name = "Backup-Diario-Dulcinea"
 JobDefs = "Tarea-Diaria"
 Client = "dulcinea-fd"
 FileSet= "Copia-dulcinea"
}

Job {
 Name = "Backup-Diario-Quijote"
 JobDefs = "Tarea-Diaria"
 Client = "quijote-fd"
 FileSet= "Copia-Quijote"
}

Job {
 Name = "Backup-Diario-Sancho"
 JobDefs = "Tarea-Diaria"
 Client = "sancho-fd"
 FileSet= "Copia-Sancho"
}

Job {
 Name = "Backup-Diario-Freston"
 JobDefs = "Tarea-Diaria"
 Client = "freston-fd"
 FileSet= "Copia-Freston"
}

Job {
 Name = "Backup-Semanal-Dulcinea"
 JobDefs = "Tarea-Semanal"
 Client = "dulcinea-fd"
 FileSet= "Copia-Dulcinea"
}

Job {
 Name = "Backup-Semanal-Quijote"
 JobDefs = "Tarea-Semanal"
 Client = "quijote-fd"
 FileSet= "Copia-Quijote"
}

Job {
 Name = "Backup-Semanal-Sancho"
 JobDefs = "Tarea-Semanal"
 Client = "sancho-fd"
 FileSet= "Copia-Sancho"
}

Job {
 Name = "Backup-Semanal-Freston"
 JobDefs = "Tarea-Semanal"
 Client = "freston-fd"
 FileSet= "Copia-Freston"
}

Job {
 Name = "Backup-Mensual-Dulcinea"
 JobDefs = "Tarea-Mensual"
 Client = "dulcinea-fd"
 FileSet= "Copia-Dulcinea"
}

Job {
 Name = "Backup-Mensual-Quijote"
 JobDefs = "Tarea-Mensual"
 Client = "quijote-fd"
 FileSet= "Copia-Quijote"
}

Job {
 Name = "Backup-Mensual-Sancho"
 JobDefs = "Tarea-Mensual"
 Client = "sancho-fd"
 FileSet= "Copia-Sancho"
}

Job {
 Name = "Backup-Mensual-Freston"
 JobDefs = "Tarea-Mensual"
 Client = "freston-fd"
 FileSet= "Copia-Freston"
}

Job {
 Name = "RestoreDulcinea"
 Type = Restore
 Client=dulcinea-fd
 FileSet= "Copia-Dulcinea"
 Storage = vol1
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "RestoreQuijote"
 Type = Restore
 Client=quijote-fd
 FileSet= "Copia-Quijote"
 Storage = vol1
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "RestoreSancho"
 Type = Restore
 Client=sancho-fd
 FileSet= "Copia-Sancho"
 Storage = vol1
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "RestoreFreston"
 Type = Restore
 Client=freston-fd
 FileSet= "Copia-Freston"
 Storage = vol1
 Pool = Vol-Backup
 Messages = Standard
}

#Definición de guardado de ficheros.
FileSet {
 Name = "Copia-Dulcinea"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
    File = /bacula
 }
 Exclude {
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

FileSet {
 Name = "Copia-Quijote"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
 }
 Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

FileSet {
 Name = "Copia-Sancho"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
 }
 Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

FileSet {
 Name = "Copia-Freston"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
 }
 Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

#Configuración de ciclos de copias.
Schedule {
 Name = "Diario"
 Run = Level=Incremental Pool=Daily daily at 21:30
}

Schedule {
 Name = "Semanal"
 Run = Level=Full Pool=Weekly sat at 23:30
}

Schedule {
 Name = "Mensual"
 Run = Level=Full Pool=Monthly 1st sun at 23:00
}

#Clientes de Bacula.
Client {
 Name = dulcinea-fd
 Address = 10.0.1.10
 FDPort = 9102
 Catalog = MyCatalog
 Password = "bacula"
 File Retention = 60 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = quijote-fd
 Address = 10.0.2.7
 FDPort = 9102
 Catalog = MyCatalog
 Password = "bacula"
 File Retention = 60 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = sancho-fd
 Address = 10.0.1.12
 FDPort = 9102
 Catalog = MyCatalog
 Password = "bacula"
 File Retention = 60 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = freston-fd
 Address = 10.0.1.9
 FDPort = 9102
 Catalog = MyCatalog
 Password = "bacula"
 File Retention = 60 days
 Job Retention = 6 months
 AutoPrune = yes
}

Storage {
 Name = vol1
 Address = 10.0.1.10
 SDPort = 9103
 Password = "bacula"
 Device = FileAutochanger1
 Media Type = File
 Maximum Concurrent Jobs = 10
}

Catalog {
  Name = MyCatalog
  dbname = "bacula"; DB Address = "localhost"; DB Port= "3306"; dbuser = "bacula"; dbpassword = "dulcinea"
}

Pool {
 Name = Daily
# Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 10d
 Recycle = yes
}

Pool {
 Name = Weekly
# Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 30d
 Recycle = yes
}

Pool {
 Name = Monthly
# Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 365d
 Recycle = yes
}

Pool {
 Name = Vol-Backup
 Pool Type = Backup
 Recycle = yes
 AutoPrune = yes
 Volume Retention = 365 days
 Maximum Volume Bytes = 50G
 Maximum Volumes = 100
 Label Format = "Remoto"
}
~~~

Despues de configurar nuestro fichero en Bacula vamos a dar formato al nuevo disco que hemos asociado a la máquina de dulcinea.
~~~
debian@dulcinea:~$ sudo fdisk /dev/vdb

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xb0769437.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759):

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
debian@dulcinea:~$ sudo mkfs.ext4 /dev/vdb1
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 1310464 4k blocks and 327680 inodes
Filesystem UUID: 4578f8bc-354c-4868-b50a-93d87723f71b
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
~~~

Crearemos una carpeta para las copias de seguridad y le damos permiso.
~~~
debian@dulcinea:~$ sudo mkdir -p /bacula/copias
debian@dulcinea:~$ sudo chown bacula:bacula /bacula -R
debian@dulcinea:~$ sudo chmod 755 /bacula -R
~~~

Cogemos el identificado del disco asociado anteriormente para el fichero /etc/fstab.
~~~
debian@dulcinea:~$ sudo nano /etc/fstab
# /etc/fstab: static file system information.
UUID=4578f8bc-354c-4868-b50a-93d87723f71b       /bacula/copias  ext4    defaults        0       0
~~~

Una vez guardado los cambios para que los cambios hagan efecto ejecutaremos el siguiente comando.
~~~
debian@dulcinea:~$ sudo mount -a
debian@dulcinea:~$ lsblk -l
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda  254:0    0  10G  0 disk
vda1 254:1    0  10G  0 part /
vdb  254:16   0   5G  0 disk
vdb1 254:17   0   5G  0 part /bacula/copias
~~~

Entramos en la configuración del fichero bacula-sd.
~~~
Storage {
 Name = dulcinea-sd
 SDPort = 9103
 WorkingDirectory = "/var/lib/bacula"
 Pid Directory = "/run/bacula"
 Maximum Concurrent Jobs = 20
 SDAddress = 10.0.1.10
}

Director {
 Name = dulcinea-dir
 Password = "bacula"
}

Director {
 Name = dulcinea-mon
 Password = "bacula"
 Monitor = yes
}

Autochanger {
 Name = FileAutochanger1
 Device = DispositivoCopia
 Changer Command = ""
 Changer Device = /dev/null
}

Device {
 Name = DispositivoCopia
 Media Type = File
 Archive Device = /bacula/copias
 LabelMedia = yes;
 Random Access = Yes;
 AutomaticMount = yes;
 RemovableMedia = no;
 AlwaysOpen = no;
 Maximum Concurrent Jobs = 5
}

Messages {
  Name = Standard
  director = dulcinea-dir = all
}
~~~

Editamos la consola mediante el fichero bconsole.conf.
~~~
Director {
  Name = dulcinea-dir
  DIRport = 9101
  address = 10.0.1.10
  Password = "bacula"
}
~~~

Ahora vamos a instalar los clientes en las distintas máquinas, el paquete se llama bacula-client.
Una vez instalado entraremos en el fichero de configuración de clientes y tendremos que configurarlo de la siguiente forma cada uno de los clientes.
- Máquina Sancho
~~~
Director {
  Name = dulcinea-dir
  Password = "bacula"
}

Director {
  Name = dulcinea-mon
  Password = "bacula"
  Monitor = yes
}

FileDaemon {                          # this is me
  Name = sancho-fd
  FDport = 9102                  # where we listen for the director
  WorkingDirectory = /var/lib/bacula
  Pid Directory = /run/bacula
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib/bacula
  FDAddress = 10.0.1.12
}

Messages {
  Name = Standard
  director = dulcinea-dir = all, !skipped, !restored
}
~~~
- Máquina Freston
~~~
Director {
  Name = dulcinea-dir
  Password = "bacula"
}

Director {
  Name = dulcinea-mon
  Password = "bacula"
  Monitor = yes
}

FileDaemon {                          # this is me
  Name = freston-fd
  FDport = 9102                  # where we listen for the director
  WorkingDirectory = /var/lib/bacula
  Pid Directory = /run/bacula
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib/bacula
  FDAddress = 10.0.1.9
}

Messages {
  Name = Standard
  director = dulcinea-dir = all, !skipped, !restored
}
~~~
- Máquina Quijote
~~~
Director {
  Name = dulcinea-dir
  Password = "bacula"
}

Director {
  Name = dulcinea-mon
  Password = "bacula"
  Monitor = yes
}

FileDaemon {                          # this is me
  Name = quijote-fd
  FDport = 9102                  # where we listen for the director
  WorkingDirectory = /var/spool/bacula
  Pid Directory = /var/run
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib64/bacula
}

Messages {
  Name = Standard
  director = dulcinea-dir = all, !skipped, !restored
}
~~~

Una vez tengamos los clientes configurados reiniciamos bacula client en cada cliente con el siguiente comando.
~~~
sudo systemctl restart bacula-fd.service
~~~

Reiniciamos también el servidor dulcinea.
~~~
debian@dulcinea:~$ sudo systemctl restart bacula-sd.service
debian@dulcinea:~$ sudo systemctl restart bacula-director.service
~~~

Hacemos prueba de funcionamiento a los clientes desde el servidor de Bacula.
~~~
debian@dulcinea:~$ sudo bconsole
Connecting to Director 10.0.1.10:9101
1000 OK: 103 dulcinea-dir Version: 9.4.2 (04 February 2019)
Enter a period to cancel a command.
*run
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"
A job name must be specified.
The defined Job resources are:
     1: Backup-Diario-Dulcinea
     2: Backup-Diario-Quijote
     3: Backup-Diario-Sancho
     4: Backup-Diario-Freston
     5: Backup-Semanal-Dulcinea
     6: Backup-Semanal-Quijote
     7: Backup-Semanal-Sancho
     8: Backup-Semanal-Freston
     9: Backup-Mensual-Dulcinea
    10: Backup-Mensual-Quijote
    11: Backup-Mensual-Sancho
    12: Backup-Mensual-Freston
    13: RestoreDulcinea
    14: RestoreQuijote
    15: RestoreSancho
    16: RestoreFreston
4
Run Backup job
JobName:  Backup-Diario-Freston
Level:    Incremental
Client:   freston-fd
FileSet:  Copia-Freston
Pool:     Daily (From Job resource)
Storage:  vol1 (From Job resource)
When:     2021-01-24 12:43:08
Priority: 10
OK to run? (yes/mod/no): yes
Job queued. JobId=10
You have messages.
*status client
The defined Client resources are:
     1: dulcinea-fd
     2: quijote-fd
     3: sancho-fd
     4: freston-fd
Select Client (File daemon) resource (1-4): 3
Connecting to Client sancho-fd at 10.0.1.12:9102

sancho-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu ubuntu 20.04
Daemon started 24-Jan-21 12:31. Jobs: run=0 running=0.
 Heap: heap=110,592 smbytes=22,002 max_bytes=22,019 bufs=68 max_bufs=68
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so

Running Jobs:
Director connected at: 24-Jan-21 12:31
No Jobs running.
====

Terminated Jobs:
====
*status client
The defined Client resources are:
     1: dulcinea-fd
     2: quijote-fd
     3: sancho-fd
     4: freston-fd
Select Client (File daemon) resource (1-4): 4
Connecting to Client freston-fd at 10.0.1.9:9102

freston-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian 10.5
Daemon started 24-Jan-21 12:32. Jobs: run=0 running=0.
 Heap: heap=114,688 smbytes=22,004 max_bytes=22,021 bufs=68 max_bufs=68
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so

Running Jobs:
Director connected at: 24-Jan-21 12:32
No Jobs running.
====

Terminated Jobs:
====
*status client
The defined Client resources are:
     1: dulcinea-fd
     2: quijote-fd
     3: sancho-fd
     4: freston-fd
Select Client (File daemon) resource (1-4): 2
Connecting to Client quijote-fd at 10.0.2.7:9102

quijote-fd Version: 9.0.6 (20 November 2017) x86_64-redhat-linux-gnu redhat (Core)
Daemon started 24-Jan-21 12:33. Jobs: run=0 running=0.
 Heap: heap=102,400 smbytes=21,976 max_bytes=21,993 bufs=68 max_bufs=68
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so

Running Jobs:
Director connected at: 24-Jan-21 12:33
No Jobs running.
====

Terminated Jobs:
====
*
~~~

Pasamos a crear las copias de las maquinas, para ello crearemos las copias mediantes los label.
~~~
debian@dulcinea:~$ sudo bconsole
Connecting to Director 10.0.1.10:9101
1000 OK: 103 dulcinea-dir Version: 9.4.2 (04 February 2019)
Enter a period to cancel a command.
*label
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"
Automatically selected Storage: vol1
Enter new Volume name: copiadiaria
Defined Pools:
     1: Daily
     2: Default
     3: File
     4: Monthly
     5: Scratch
     6: Vol-Backup
     7: Weekly
Select the Pool (1-7): 1
Connecting to Storage daemon vol1 at 10.0.1.10:9103 ...
Sending label command for Volume "copiadiaria" Slot 0 ...
3000 OK label. VolBytes=223 VolABytes=0 VolType=1 Volume="copiadiaria" Device="DispositivoCopia" (/bacula/copias)
Catalog record for Volume "copiadiaria", Slot 0  successfully created.
Requesting to mount FileAutochanger1 ...
3906 File device ""DispositivoCopia" (/bacula/copias)" is always mounted.
*label
Automatically selected Storage: vol1
Enter new Volume name: copiasemanal
Defined Pools:
     1: Daily
     2: Default
     3: File
     4: Monthly
     5: Scratch
     6: Vol-Backup
     7: Weekly
Select the Pool (1-7): 7
Connecting to Storage daemon vol1 at 10.0.1.10:9103 ...
Sending label command for Volume "copiasemanal" Slot 0 ...
3000 OK label. VolBytes=225 VolABytes=0 VolType=1 Volume="copiasemanal" Device="DispositivoCopia" (/bacula/copias)
Catalog record for Volume "copiasemanal", Slot 0  successfully created.
Requesting to mount FileAutochanger1 ...
3906 File device ""DispositivoCopia" (/bacula/copias)" is always mounted.
*label
Automatically selected Storage: vol1
Enter new Volume name: copiamensual
Defined Pools:
     1: Daily
     2: Default
     3: File
     4: Monthly
     5: Scratch
     6: Vol-Backup
     7: Weekly
Select the Pool (1-7): 4
Connecting to Storage daemon vol1 at 10.0.1.10:9103 ...
Sending label command for Volume "copiamensual" Slot 0 ...
3000 OK label. VolBytes=226 VolABytes=0 VolType=1 Volume="copiamensual" Device="DispositivoCopia" (/bacula/copias)
Catalog record for Volume "copiamensual", Slot 0  successfully created.
Requesting to mount FileAutochanger1 ...
3906 File device ""DispositivoCopia" (/bacula/copias)" is always mounted.
~~~

Una vez creado los label vamos a ver que las copias se han realizado correctamente en cada máquina.
- Máquina Dulcinea
~~~
*status client
The defined Client resources are:
     1: dulcinea-fd
     2: quijote-fd
     3: sancho-fd
     4: freston-fd
Select Client (File daemon) resource (1-4): 1
Connecting to Client dulcinea-fd at 10.0.1.10:9102

dulcinea-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian 10.5
Daemon started 24-Jan-21 09:56. Jobs: run=4 running=0.
 Heap: heap=4,096 smbytes=431,548 max_bytes=701,952 bufs=93 max_bufs=153
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so

Running Jobs:
Director connected at: 24-Jan-21 13:18
No Jobs running.
====

Terminated Jobs:
 JobId  Level    Files      Bytes   Status   Finished        Name
===================================================================
    11  Full      3,537    24.67 M  OK       24-Jan-21 13:13 Backup-Diario-Dulcinea
    15  Full      3,549    29.47 M  OK       24-Jan-21 13:15 Backup-Semanal-Dulcinea
    19  Full      3,541    33.29 M  OK       24-Jan-21 13:16 Backup-Mensual-Dulcinea
====
~~~
- Máquina Quijote
~~~
*status client
The defined Client resources are:
     1: dulcinea-fd
     2: quijote-fd
     3: sancho-fd
     4: freston-fd
Select Client (File daemon) resource (1-4): 2
Connecting to Client quijote-fd at 10.0.2.7:9102

quijote-fd Version: 9.0.6 (20 November 2017) x86_64-redhat-linux-gnu redhat (Core)
Daemon started 24-Jan-21 12:33. Jobs: run=3 running=0.
 Heap: heap=24,576 smbytes=428,757 max_bytes=1,260,164 bufs=88 max_bufs=242
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so

Running Jobs:
Director connected at: 24-Jan-21 13:18
No Jobs running.
====

Terminated Jobs:
 JobId  Level      Files    Bytes   Status   Finished        Name
===================================================================
    12  Full       7,110    149.4 M  OK       24-Jan-21 13:13 Backup-Diario-Quijote
    16  Full       7,110    149.4 M  OK       24-Jan-21 13:15 Backup-Semanal-Quijote
    20  Full       7,110    149.4 M  OK       24-Jan-21 13:16 Backup-Mensual-Quijote
====
~~~
- Máquina Sancho
~~~
*status client
The defined Client resources are:
     1: dulcinea-fd
     2: quijote-fd
     3: sancho-fd
     4: freston-fd
Select Client (File daemon) resource (1-4): 3
Connecting to Client sancho-fd at 10.0.1.12:9102

sancho-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu ubuntu 20.04
Daemon started 24-Jan-21 12:39. Jobs: run=3 running=0.
 Heap: heap=18,446,744,073,709,543,424 smbytes=433,229 max_bytes=708,603 bufs=98 max_bufs=170
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so

Running Jobs:
Director connected at: 24-Jan-21 13:19
No Jobs running.
====

Terminated Jobs:
 JobId  Level    Files      Bytes   Status   Finished        Name
===================================================================
    13  Full      5,252    528.7 M  OK       24-Jan-21 13:14 Backup-Diario-Sancho
    17  Full      5,252    528.7 M  OK       24-Jan-21 13:15 Backup-Semanal-Sancho
    21  Full      5,252    528.7 M  OK       24-Jan-21 13:16 Backup-Mensual-Sancho
====
~~~
- Máquina Freston
~~~
*status client
The defined Client resources are:
     1: dulcinea-fd
     2: quijote-fd
     3: sancho-fd
     4: freston-fd
Select Client (File daemon) resource (1-4): 4
Connecting to Client freston-fd at 10.0.1.9:9102

freston-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian 10.5
Daemon started 24-Jan-21 12:32. Jobs: run=4 running=0.
 Heap: heap=114,688 smbytes=442,688 max_bytes=709,072 bufs=99 max_bufs=154
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Plugin: bpipe-fd.so

Running Jobs:
Director connected at: 24-Jan-21 13:19
No Jobs running.
====

Terminated Jobs:
 JobId  Level    Files      Bytes   Status   Finished        Name
===================================================================
    14  Full      3,237    22.99 M  OK       24-Jan-21 13:13 Backup-Diario-Freston
    18  Full      3,237    22.99 M  OK       24-Jan-21 13:15 Backup-Semanal-Freston
    22  Full      3,237    22.99 M  OK       24-Jan-21 13:16 Backup-Mensual-Freston
====
~~~
