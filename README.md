# Progetto: Configurazione di Jenkins Master e Slave con Docker/Podman su Rocky Linux 9

## Consegna

1. **Creazione di una VM con Rocky Linux 9** tramite Vagrant.
2. **Installazione di Docker o Podman** nella VM usando Ansible.
3. **Configurazione di una rete Docker/Podman** con IP statici utilizzando Ansible.
4. **Installazione di Jenkins Master** come container Docker/Podman tramite Ansible, configurato con un IP statico.
5. **Installazione di un Jenkins Slave** come container Docker/Podman tramite Ansible, configurato per connettersi al Jenkins Master.

L'intero progetto deve essere configurato tramite strumenti di provisioning (Vagrant e Ansible), con attenzione all'automazione e alla riproducibilità.

---

## Primo passo
Ho creato un Vagrantfile che permette di installare una VM Rocky Linux 9.

## Secondo passo
Ho creato un playbook che prenda prima la repository per installare docker e in seguito lo installa.  
Di seguito le task principali per l'installazione di Docker:
```yaml
    - name: Download Docker repository
      get_url:
        url: https://download.docker.com/linux/centos/docker-ce.repo
        dest: /etc/yum.repos.d/docker-ce.repo

    - name: Install Docker
      package:
        name: docker-ce
        state: present

    - name: Start Docker service
      systemd:
        name: docker
        enabled: yes
        state: started
``` 


## Terzo passo
Ho configurato una rete per Docker utilizzando il modulo "docker_network".  
Di seguito la task:
```yaml
    - name: Create a network with custom IPAM config
      docker_network:
        name: Mario_network
        ipam_config:
          - subnet: 192.168.10.0/24
            gateway: 192.168.10.1
```

## Quarto passo
Ho installato un jenkins master in un container e ho specificato la rete da utilizzare. In seguito bisogna entrare su jenkins tramite browser dove ti richiederà di completare la configurazione, per fare ciò abbiamo bisogno della password che è presente nel file 
```bash
/var/jenkins_home/secrets/initialAdminPassword` all' interno del container, per ottenerla si può utilizzare il comando `docker exec jenkins_master cat /var/jenkins_home/secrets/initialAdminPassword` e poi completare la configurazione.
```
Si può saltare questo passaggio specificando il parametro 
```yaml
JAVA_OPTS: "-Djenkins.install.runSetupWizard=false"
```
nel playbook di ansible. Questo parametro disabilita il setup guidato di jenkins ma bisogna in seguito installare tutti i plugin necessari all'interno di jenkins.

## Quinto passo
Ho installato un jenkins slave in un container e ho specificato la rete che ho utilizzato con il master.  
Per collegare lo slave al master bisogna prima entrare su jenkins dal browser e creare un nodo slave/agent una volta fatto ciò bisogna prendere il token generato e inserirlo al posto di  `token_segreto`
```yaml
- name: Start Jenkins Slave and connect to Master
      command: >
        docker exec jenkins_slave java -jar agent.jar
        -url http://192.168.10.2:8080/
        -secret token_segreto
        -name "jenkins_slave"
        -webSocket
        -workDir "/home/jenkins"
```
una volta fatto ciò bisogna avviare il provision e lo slave si collegherà al master pronto per essere utilizzato.
# Ancora in fase di lavorazione per rendere tutto piu automatizzato e sicuro
