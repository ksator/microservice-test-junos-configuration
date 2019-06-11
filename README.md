![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/ksator/test_junos_configuration.svg)  ![MicroBadger Size](https://img.shields.io/microbadger/image-size/ksator/test_junos_configuration.svg) ![MicroBadger Layers](https://img.shields.io/microbadger/layers/ksator/test_junos_configuration.svg) ![Docker Pulls](https://img.shields.io/docker/pulls/ksator/test_junos_configuration.svg) 

# Description 

This microservice tests Junos configuration files on devices  
It uses Docker and Ansible.  

It: 
- loads the Junos configuration files
- does a commit check
- get the diff between running and candidate configuration 
- renders the candidate configuration
- rollbacks 0 the candidate configuration    

So no commit is done on the devices  

This microservice: 
- instanciates a container
- executes the service 
- stops the container 
- removes the container


The Docker image is [```ksator/test_junos_configuration```](https://hub.docker.com/r/ksator/test_junos_configuration)  

# Usage

## Install Docker

## Pull the Docker image
```
$ docker pull ksator/test_junos_configuration  
```
Verify
```
$ docker images ksator/test_junos_configuration
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
ksator/test_junos_configuration   latest              4f6f59eb664a        33 minutes ago      543MB
```

## Create the microservice inputs

Create this structure: 
- An `inputs` directory. With these files: 
  - An Ansible Inventory file (`hosts.ini`) with following variables:
    - `ansible_host`: IP of the device
    - `ansible_ssh_user`: Username to use for the connection
    - `ansible_ssh_pass`: Password to use for the connection
  - a directory ```configuration_files``` with Junos configuration files
    - All formats (xml, set, text, json) are supported. The format is determined by the file extension (.conf, .xml, .set, .json)
  - A YAML file (`type_to_use_when_loading_configuration.yml `) to indicate how do you want to load the configuration (replace, merge, overwrite)  
  
```
$ ls inputs
configuration_files  hosts.ini  type_to_use_when_loading_configuration.yml
```

Ansible inventory example: 
```
$ more inputs/hosts.ini
[spines]
demo-qfx10k2-14   ansible_host=172.25.90.67
demo-qfx10k2-15   ansible_host=172.25.90.68

[leaves]
demo-qfx5110-9    ansible_host=172.25.90.63
demo-qfx5110-10   ansible_host=172.25.90.64

[all:vars]
netconf_port=830
ansible_ssh_user=ansible
ansible_ssh_pass=juniper123
```

Junos configuration files example:   
```
$ ls inputs/configuration_files/
demo-qfx10k2-14.conf  demo-qfx10k2-15.conf  demo-qfx5110-10.conf  demo-qfx5110-9.conf
```
```
$ more inputs/configuration_files/demo-qfx10k2-14.conf 
system {
	services {
		extension-service {
		    request-response {
		        grpc {
		            clear-text {
		                port 32766;
		            }
		            skip-authentication;
		        }
		    }
		    notification {
		        allow-clients {
		            address 0.0.0.0/0;
		        }
		    }
		}
	}
}
```
Indicate how do you want to load the configuration 
```
$ more inputs/type_to_use_when_loading_configuration.yml 
load_type: "merge"

```
## Run the microservice

This will instanciate a container, execute the service, stop the container and remove the container.    
```
$ docker run -it --rm -v ${PWD}/inputs:/inputs -v ${PWD}/outputs:/outputs ksator/test_junos_configuration
test Junos configuration

PLAY [Deploy Junos configuration] *************************************************************************************************************************************************************************************************************************************************

TASK [include_vars] ***************************************************************************************************************************************************************************************************************************************************************
ok: [demo-qfx5110-9]
ok: [demo-qfx5110-10]
ok: [demo-qfx10k2-14]
ok: [demo-qfx10k2-15]

TASK [include_vars] ***************************************************************************************************************************************************************************************************************************************************************
ok: [demo-qfx5110-9]
ok: [demo-qfx5110-10]
ok: [demo-qfx10k2-14]
ok: [demo-qfx10k2-15]

TASK [test-configuration : Create output directory for each device] ***************************************************************************************************************************************************************************************************************
changed: [demo-qfx10k2-15]
changed: [demo-qfx5110-9]
changed: [demo-qfx5110-10]
changed: [demo-qfx10k2-14]

TASK [test-configuration : Pushing configuration, doing a commit check, getting the diff, and rollbacking] ************************************************************************************************************************************************************************
changed: [demo-qfx5110-9]
ok: [demo-qfx5110-10]
changed: [demo-qfx10k2-15]
changed: [demo-qfx10k2-14]

TASK [test-configuration : Pushing config, doing a commit check, downloading candidate config, and rollbacking] *******************************************************************************************************************************************************************
changed: [demo-qfx10k2-14]
changed: [demo-qfx10k2-15]
changed: [demo-qfx5110-10]
changed: [demo-qfx5110-9]

PLAY RECAP ************************************************************************************************************************************************************************************************************************************************************************
demo-qfx10k2-14            : ok=5    changed=3    unreachable=0    failed=0   
demo-qfx10k2-15            : ok=5    changed=3    unreachable=0    failed=0   
demo-qfx5110-10            : ok=5    changed=2    unreachable=0    failed=0   
demo-qfx5110-9             : ok=5    changed=3    unreachable=0    failed=0   

```
List the containers.  
The container doesnt exist anymore
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```


## Check the microservice output 

Here's the output generated
```
$ ls outputs/
demo-qfx10k2-14  demo-qfx10k2-15  demo-qfx5110-10  demo-qfx5110-9
```
```
$ ls outputs/demo-qfx10k2-14/
candidate-rendered-demo-qfx10k2-14.conf  configuration_diff.log
```
To get the diff, run this command: 
```
$ more outputs/demo-qfx10k2-14/configuration_diff.log 

[edit system services extension-service request-response grpc clear-text]
-       port 32768;
+       port 32766;
```
to get the rendered candidate configuration, run this command:  
```
more outputs/demo-qfx10k2-14/candidate-rendered-demo-qfx10k2-14.conf 
```
