# Track 2 Step 1
## Primo passo 
Ho creato un Vagrantfile che permette di installare una VM Rocky Linux 9 e installi docker tramite ansible.

## Secondo passo
Ho configurato una rete per Docker utilizzando il modulo "docker_network".

## Terzo passo
Ho installato un jenkins master in un conatiner e ho specificato la rete da utilizzare. In seguito bisogna entrare su jenkins tramite browser dove ti richiederà di completare la configurazione, per fare ciò ho bisogno della password che è presente nel file `/var/jenkins_home/secrets/initialAdminPassword` all' interno del container, per ottenerla si può utilizzare il comando `docker exec jenkins_master cat /var/jenkins_home/secrets/initialAdminPassword` e poi completare la configurazione.  
Si può saltare questo passaggio specificando il parametro `JAVA_OPTS: "-Djenkins.install.runSetupWizard=false"` nel playbook di ansible. Questo parametro disabilita il setup guidato di jenkins cosi da rendere tutto più automatizzato.

## Quarto passo
Ho installato un jenkins slave in un container e ho specificato la rete che ho utilizzato con il master.  
Per collegare lo slave al master bisogna prima entrare su jenkins dal browser e creare un nodo slave/agent una volta fatto ciò bisogna prendere il token generato e inserirlo al posto di  `token_segreto`
``` 
- name: Start Jenkins Slave and connect to Master
      command: >
        docker exec jenkins_slave java -jar agent.jar
        -url http://192.168.10.2:8080/
        -secret token_segreto
        -name "jenkins_slave"
        -webSocket
        -workDir "/home/jenkins" &
```
una volta fatto ciò lo slave si collegherà al master pronto per essere utilizzato.

## Fase di lavorazione per rendere tutto più automatizzato e sicuro
