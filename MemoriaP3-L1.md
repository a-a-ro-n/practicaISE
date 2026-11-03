# Ejercicio 2: Monitorización y automatización del RAID1
El objetivo de este ejercicio es automatizar la comprobación del estado de los discos RAID1 que tengo configurados en mi máquina Debian reusada de la `P0-L1`

Para ello, he utilizado Python para crear un pequeño script que consulta la información del archivo /proc/mdstat y comprueba si alguno de 
los discos presenta algún fallo

Posteriormente, he utilizado systemd para automatizar la ejecución de este script mediante un servicio y un temporizador. Así, el sistema 
puede comprobar periódicamente el estado de los RAID sin necesidad de hacerlo manualmente

2. Comprobación inicial del almacenamiento

Antes de empezar, he comprobado cómo están distribuidos los discos y las particiones mediante el comando lsblk

<img width="720" height="372" alt="Captura de pantalla 2026-11-03 214733" src="https://github.com/user-attachments/assets/f92a28a4-81c9-4bd3-ad4f-03956a5def22" />

En mi instalación de Debian tengo configuradas dos matrices RAID1:

- /dev/md0: formada por las particiones /dev/sda1 y /dev/sdb1, y montada en /boot
- /dev/md1: formada por las particiones /dev/sda2 y /dev/sdb2. Sobre esta matriz tengo configurado el cifrado y, dentro de él, LVM,
donde se encuentran los volúmenes lógicos de la raíz (/), /home y swap

A continuación, he consultado el archivo /proc/mdstat. Este archivo proporciona información sobre el estado de las matrices RAID del 
sistema. En mi caso, las dos matrices aparecen activas y con el indicador `[2/2] [UU]`.
Esto significa que los dos dispositivos de cada matriz están funcionando correctamente. Si alguno de ellos presenta un fallo, el 
indicador puede mostrar un guion bajo, por ejemplo `[_U]`.

Para automatizar la comprobación del estado de los RAID, he creado un script en Python guardado en /usr/local/bin/monitor_raid.py
El funcionamiento del script consiste en abrir /proc/mdstat, recorrer su contenido y buscar mediante expresiones regulares los 
indicadores que representan el estado de los discos.

Por ejemplo, utiliza la expresión regular:    error_pattern = re.compile(r'\[[U_]{2,}]')

Esta expresión permite localizar indicadores entre corchetes que contienen letras `U` o guiones bajos. Cuando encuentra uno, comprueba 
si la línea contiene "_". Si es así, muestra un mensaje indicando que se ha detectado un error en el RAID.
Si no encuentra ningún indicador que cumpla esa condición, muestra que todos los RAID están correctos.

Una vez creado el script, el siguiente paso ha sido automatizar su ejecución utilizando systemd, creando dos archivos dentro del 
directorio /etc/systemd/system/: uno para definir el servicio que ejecuta el script y otro para establecer cada cuánto tiempo debe ejecutarse.

El archivo /etc/systemd/system/check-raid.service define el servicio encargado de ejecutar el script de Python.
El tipo oneshot no necesita que el programa permanezca ejecutándose continuamente. Simplemente quiero que se inicie, realice la comprobación 
del RAID y finalice.
La opción ExecStart indica la ubicación del script que debe ejecutar el servicio.

El archivo /etc/systemd/system/check-raid.timer permite programar las ejecuciones del servicio.
He configurado el temporizador para que la primera comprobación se realice un minuto después del arranque del sistema y, posteriormente, 
se repita cada dos minutos.
Para ello he utilizado las opciones OnBootSec y OnUnitActiveSec qeu de esta forma, el temporizador se encarga de activar automáticamente 
el servicio sin que tenga que ejecutarlo manualmente cada vez.

<img width="1075" height="717" alt="Captura de pantalla 2026-11-03 220337" src="https://github.com/user-attachments/assets/ee6351aa-bebe-4232-9021-78c5a2c83243" />


Una vez creados los dos archivos, he actualizado la configuración de systemd y he habilitado el temporizador:
```
systemctl daemon-reload
systemctl enable --now check-raid.timer
```


Una vez configurado el servicio y el temporizador, he realizado varias pruebas para comprobar que el script se ejecuta correctamente y 
que sus resultados quedan registrados en el sistema.
Para ejecutar manualmente el servicio he utilizado systemctl start check-raid.service


Después, para consultar los resultados de la ejecución, he utilizado: journalctl -u check-raid.service -n 10 permitiendo consultar los 
últimos mensajes generados por el servicio, por lo que resulta útil para comprobar las ejecuciones y detectar posibles problemas.

<img width="1125" height="408" alt="Captura de pantalla 2026-11-03 221014" src="https://github.com/user-attachments/assets/dde340a6-e734-4454-941a-c9ba793ff8ba" />

El mensaje de Error: Esto indica que encontró un indicador que interpretó como un posible fallo en una de las matrices RAID.



Posteriormente, volví a ejecutar el servicio y consulté de nuevo los registros.
En esta ocasión, el resultado fue:

<img width="1068" height="219" alt="Captura de pantalla 2026-11-03 221205" src="https://github.com/user-attachments/assets/3b8af49e-fbe5-49c5-bf1c-92595e300f54" />

Esto indica que el script no encontró ningún indicador que identificase como fallo y terminó su ejecución correctamente.


Finalmente, he comprobado que las ejecuciones quedan registradas mediante `journalctl`, lo que permite consultar los resultados sin tener que ejecutar el programa directamente.

De esta manera, he conseguido automatizar la supervisión básica de mis matrices RAID1 y disponer de información sobre su estado de forma periódica.
