# Ansible Playbook 

## Descripción

Este script sirve para aplicar linea base de seguridad, es decir, permite realizar configuraciones en el sistema operativo que brinde seguridad. Así mismo, el script se encarga de lo siguiente: 
1. Agrega una o más lineas en una ruta especifica.
2. Modifica lineas dentro de un archivo en una ruta especifica.
3. Instalar, actualizar o remover paquetes con yum o dnf.
4. Ejecutar comandos por shell. 
5. Genera reporte en .csv con el siguiente formato: 

Host | S.O | Type Action | Action Code | Path | Search Regexp | Line | Commands | Changed | Failed | Msg 
--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ...

## Preparación

El script presenta las siguientes variables:
1. **changeLine**, el cual permite modificar lineas dentro de un archivo. 
		  ...
		  vars: 
			changeLine:
			  - name: 'ID-01'
				path: '/etc/gshadow'
				toReplace:
				  - search: 'daemon::|daemon:!:'
					line: 'daemon:*:'
				  - search: 'bin::|bin:!:'
					line: 'bin:*:'

Este presenta la propiedad **name** el cual es el identificador del cambio, **path** es la ruta del archivo y **toReplace** es la propiedad que contendrá listas que automatizarán los cambios, este ultimo debe tener las propiedades: **search**, el cual se encarga de realizar la busqueda en la ruta especificada usando *expresiones regulares* y **line**, el cual será la linea que reemplazará. 

`$ cat /etc/gshadow (antes)`

		...
		daemon:::
		bin:!:::
		...

`$ cat /etc/gshadow (despues)`

		...
		daemon:*::
		bin:*:::
		...

2. **newLine**, el cual permite agregar nuevas lineas en un archivo especifico.

		  vars: 
			newLine:
				- name: 'ID-02'
				  path: '/etc/ssh/my_banner'
				  create: true
				  lines:
					- '**********************************************************'
					- '*                     ¡COMUNICADO!                       *'
					- '*           USO PARA TRABAJOS PROFESIONALES              *'
					- '**********************************************************'

Este presenta la propiedad **name** el cual es el identificador del cambio, **path** es la ruta del archivo, **create** indica si será necesario crear el archivo en caso de que no exista en la ruta especificada (*true*  > *crear archivo* | *false*  > * no crear archivo*) y **lines** es la propiedad que contendrá las lineas que serán agregadas en el archivo.

`$ cat /etc/ssh/my_banner (antes)`
`### No existe el archivo`

`$ cat /etc/ssh/my_banner (antes)`

	**********************************************************
	*                     ¡COMUNICADO!                       *
	*           USO PARA TRABAJOS PROFESIONALES              *
	**********************************************************
	

3. **packages**, el cual permite instalar, remover y actualizar paquetes

		vars:     
			packages:
				#action => absent -> remove ; present -> install; latest -> update
			  - name: 'ID-03'
				type: 'yum'
				action: present
				package: 'httpd'

			  - name: 'ID-04'
				type: 'dnf'
				action: present
				package: 'nodejs'


Este presenta la propiedad **name** el cual es el identificador del cambio, **type** el cual debe indicar que gestor de paquetes se utilizará (yum o dnf), **action** el cual es la acción de instalar*(present)*, remover*(absent)* o actualizar*(latest)* y **package** el cual debe contener el nombre del paquete.

`$ systemctl status httpd (antes)`

	Unit httpd.service could not be found.

`$ systemctl status httpd (desoues)`

	● httpd.service - The Apache HTTP Server
	   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
	   Active: inactive (dead)
		 Docs: man:httpd.service(8)

4. **exeShells**, el cual permite ejecutar comandos en shell.

		vars: 
			exeShell:
			  - name: 'ID-05'
				command: 'systemctl stop firewalld && systemctl disable firewalld'

			  - name: 'ID-06'
				command: 'systemctl enable auditd && systemctl start auditd'

Este presenta la propiedad **name** el cual es el identificador del cambio y **command** el cual debe contener el comando a ser ejecutados en la shell.

5. **secuence**, el cual permite ejecutar en secuencia comandos en shell, esto es usado en caso de que es necesario realizar un paso detrás de otro. 

		vars:    
			secuence:
			  - name: 'ID-04'
				commands:
				  - 'yum install -y package1'
				  - 'echo "parameter_01 value_01" >> /etc/x/package.conf'
				  - 'echo "parameter_02 value_02" >> /etc/x/package.conf'
				  - 'systemctl enable package1'
				  - 'systemctl start package1'

Este presenta la propiedad **name** el cual es el identificador del cambio y **commands** el cual debe contener la secuencia de comandos a ser ejecutados en la shell, tener en cuenta que en caso de ser instalación de paqueter llevar el parameto **-y** para no generar errores.

**NOTA**:  La diferencia de exeShell y secuence radica en el tipo de acción que se quiere realizar, es decir uno no tiene secuencia mientras que el otro sí. Asi mismo, esto permitirá ser identificado y diferenciado en el reporte. 

## Ejecución

Para ejecutar el playbook  usar el siguiente comando:
`$ ansible-playbook main.yml -K`

#### Reportes

Los reportes se generan en la misma ruta donde se encuentra el playbook con los nombres: 
 * file_status_lbs_report.csv
`$ cat report/file_status_lbs_report.csv`


**Esto ha sido probado con Ansible 2.10**
**¡A usarlo! @jurgennikolai**
