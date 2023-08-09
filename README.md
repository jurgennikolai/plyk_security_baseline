# Ansible Playbook 

## Description

This script is used to apply a security baseline, i.e., it allows you to configure the operating system to provide security. Also, the script is in charge of the following: 
1. Adds one or more lines in a specific path.
2. Modifies lines within a file in a specific path.
3. Install, update or remove packages with yum or dnf.
4. Execute shell commands. 
5. Generate a .csv report with the following format: 

Host | S.O | Type Action | Action Code | Path | Search Regexp | Line | Commands | Changed | Failed | Msg 
--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ...


## Preparation

The script presents the following variables:
1. **changeLine**, which allows you to change lines within a file. 

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
					

This one presents the property **name** which is the identifier of the change, **path** is the path of the file and **toReplace** is the property that will contain lists that will automate the changes, this last one must have the properties **search**, which is in charge of searching in the specified path using *regular expressions* and **line**, which will be the line that will replace: **search**, which is in charge of performing the search in the specified path using *regular expressions* and **line**, which will be the line to replace. 

`$ cat /etc/gshadow (before)`

		...
		daemon:::
		bin:!:::
		...

`$ cat /etc/gshadow (after)`

		...
		daemon:*::
		bin:*:::
		...

2. **newLine**, which allows you to add new lines in a specific file.

		  vars: 
			newLine:
				- name: 'ID-02'
				  path: '/etc/ssh/my_banner'
				  create: true
				  lines:
					- '**********************************************************'
					- '*                                ¡COMUNICADO!                            *'
					- '*           USO PARA TRABAJOS PROFESIONALES            *'
					- '**********************************************************'

It presents the **name** property which is the change identifier, **path** is the file path, **create** indicates if it will be necessary to create the file in case it does not exist in the specified path (*true* > *create file* | *false* > *do not create file*) and **lines** is the property that will contain the lines that will be added in the file.

`$ cat /etc/ssh/my_banner (before)`
`### File does not exist.`

`$ cat /etc/ssh/my_banner (after)`
	**********************************************************
	*                                ¡COMUNICADO!                            *
	*           USO PARA TRABAJOS PROFESIONALES            *
	**********************************************************
	

3. **Packages**, which allows you to install, remove and update packages.

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

It has the **name** property which is the change identifier, **type** which must indicate which package manager will be used (yum or dnf), **action** which is the action to install*(present)*, remove*(absent)* or update*(latest)* and **package** which must contain the package name.

`$ systemctl status httpd (before)`

	Unit httpd.service could not be found.

`$ systemctl status httpd (after)`

	● httpd.service - The Apache HTTP Server
	   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
	   Active: inactive (dead)
		 Docs: man:httpd.service(8)

4. **exeShells**, which allows you to execute shell commands.

		vars: 
			exeShell:
			  - name: 'ID-05'
				command: 'systemctl stop firewalld && systemctl disable firewalld'

			  - name: 'ID-06'
				command: 'systemctl enable auditd && systemctl start auditd'

It presents the **name** property which is the change identifier and **command** which must contain the command to be executed in the shell.

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

This presents the property **name** which is the identifier of the change and **commands** which must contain the sequence of commands to be executed in the shell, take into account that in case of being a package installation, take the parameter **-y** in order not to generate errors.

**NOTE**: The difference between exeShell and secuence lies in the type of action to be performed, i.e. one does not have a sequence while the other does. Likewise, this will allow to be identified and differentiated in the report. 

## Execution

To run the playbook use the following command:

`$ ansible-playbook playbook.prd.yml -K`

#### Reports

The reports are generated in the same path where the playbook with the names is located: 

 * file_status_lbs_report.csv
`$ cat file_status_lbs_report.csv`

## Note

The **playbook.dev.yml** and **playbook.prd.yml** files differ in that the first *(dev.)* is for development, i.e., that the logic that generates the report is understandable and can be easily modified. While the second *(prd.)* is for executing and getting the report right.

**This has been tested with Ansible 2.10**.
**To use it! @jurgennikolai**