Paso a paso y decisiones tomadas a la hora de configurar nuestro sistema Debian:

Los dos discos que se han creados son de 10GB como se dijo en la explicación.

Necesitamos crear una tabla de particiones creando una partición sda1 y sdb1 superando el primer nivel de abstracción.
Para que el /boot siga funcionando vamos a tener que crear una abstracción ya que tenemos un RAID por software. 
A cada partición he decidido darle un tamaño de 600MB debido a que la imagen del kernel suele ocupar 75-200MB y si tenemos 3 imágenes necesitamos el triple de espacio, aunque le he dejado un poco mas de espacio extra por si acaso.
He elegido el ext4 como sistema de archivos para el RAID1 debido a que es mas moder rápido y eficiente debido a que proteje los datos en fallos repentinos.

Para el tamaño el /root de VL he decidido darle un tamaño de 7GB debido a que hay que darle la mayor parte del espacio a /root.
El tamaño de /home le he dado 1GB, y para el tamaño de swap le he dado 200MB debido a que no quiero desaprovechar RAM.

<img width="960" height="1079" alt="Particion-RAID1-Cifrado" src="https://github.com/user-attachments/assets/15788b8d-128b-4596-aaf3-29569ebf2dc2" />

A lo largo de esta practica he tenido una serie de inconvenientes con el gestor de descargas que no me ha dejado completar al principio la instalación y el montaje de las particiones. 
He logrado solucionar este inconveniente volviendo ha hacer el montaje de cero con la interfaz grafica, ya que me ha dejado montarlo con éxito.

Es por esto que los datos que aparecen en la foto anterior no concuerdan con las cifras que aparecen en la explicacion ni en la imagen de abajo. Ya que he optado por darle un poco mas de memoria a los modulos por si el error hubiera sido ese.

A continuaciòn le dejo la imagen del lsblk para ver como he realizado la practica.

<img width="957" height="1079" alt="lsblk" src="https://github.com/user-attachments/assets/38cfa9da-3b8b-42b6-bd0b-735ea6f5b085" />
