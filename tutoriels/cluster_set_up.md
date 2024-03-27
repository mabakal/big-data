#### Mise en place d'un cluster hadoop
Dans ce tutoriel, je vais te montré comment mettre en place un cluster de hadoop en utilisant les virtuelles machines de AWS. On va suivre ces étapes :

- Mise en place de la machine virtuel
- Disposé les informations nécessaire ainsi que les outils nécessaire
- Définition des variables d'environnement
- Etablir la communication entre tes differentes machines
- Configuration du cluster
- Démarrage
- Vérification

1. Mise en place de la machine virtuelle
Dans ton console de AWS, tu lance une instance AWS et tu l'as personnalise. Voici les configuration nécessaire.
- Security groupe
Dans cette zone tu défini où et comment ton instance est accédé. Hadoop utilise le ssh, assure toi d'avoir sélectionner l'accès par le ````ssh```` qui à pour porte le numero ```22``` et que la machine puissent être accessible n'importe où. Une fois la machine lancée, généralement elle est automatiquement démarré, si elle n'est pas démarré il faut la lancer. Après l'avoir lancé connecte toi a cette machine en utilisant le ```ssh``` depuis ta machine ou en utilisant le ```EC2 connect```.  Dans le premier premier cas tu auras besoin d'une ```key-paire``` qui te sera fournie par AWS. Tu utilise cette clé pour se connecter depuis ta machine en écrivant cette commande.
```zsh
ssh -i key-paire.pem username@hostname # remplace key-paire, username, hostname par leur valeur
```
Si tout ce passe bien tu sera connecté á ton instance AMI. Maintenant tu passe au deuxième étapes.

1. Assurez vous de disposé les informations et logiciel suivants

- ssh
Le protocole de communicaiton utilisé par les noeuds pour communiquer est le ssh, assuré toi que le ssh est installé sur ta machine. Pour l'installer utilise cette commande

```zsh
sudo apt update
sudo apt install openssh-server
sudo systemctl status ssh # verifier si le service est lancer
sudo systemctl enable ssh # pour activer le démarrage automatique

```
- jdk
Hadoop utilise java pour s'exécuter, tu dois disposé le jdk. Pour l'installer utilise cette commande

```zsh
sudo apt update
sudo apt install default-jdk
java -version # Vérification de l'installation
```
- jps
Java process statut qui va te permettre de voir l'exécusion des noeud
```zsh
sudo apt-get install pdsh
```
- IP address des machines

Tu dois disposé des address IP de toutes tes machines et les différencier, le master avec les esclaves

- Télécharger hadoop
 Sur toutes les machines assures toi que hadoop est disponible, pour le telecharger utilise cette commande

```zsh
wget https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz 
tar -xf hadoop-3.3.6.tar.gz # Décompression
mv hadoop-3.3.6.tar.gz hadoop # renommer le fichier
```

1. Définir les variables d'environnement
Dans le repertoire ou tu te trouve, tapes ces commande pour définir les variables d'environnement
 - Pour hadoop
```zsh
echo "export HADOOP_HOME=path/to/hadoop/home" # Remplacer par le chemin vers hadoop
echo "export PATH=$HADOOP_HOME/bin:$PATH" 
```
 - Pour Java

 ```zsh
echo "export JAVA_HOME=path/to/jdk/home" # Remplacer par le chemin vers le jdk
echo "export PATH=$JAVA_HOME/bin:$PATH" 
 ```

2. Etablir la communication entre tes differentes machines

Premierement, il faut génére la clé sur chaque machine pour une connexion sécurisé. Pour le faire suivez ces étapes. Sur chaque machine entré

```zsh
ssh-keygen -t rsa -b 4096 
cat ~/.ssh/id_rsa.pub >> authorized_keys # sur la machine elle même
```
Copiez la clé publique de la machine vers les autres machines pour leurs permettre d'avoir accès a ce dernier. Si cette partie pose problème, vous povez copier manuellement la clé publique et la partager sur les autres machine

```zsh
ssh-copy-id user@hostname # remplacer user, et hostname par le nom du noeud
```

Assure toi que la communication passe entre les machines en essayant de se connecter á la machine

```zsh
ssh user@hostname
```

3. Configuration du cluster

La configuration du cluster s'effectue dans les fichiers xml. C'est dans ces fichier que tu personnalise ton cluster. Les fichiers sont les suivants. ```core-site.xml, hdfs-site.xml, yarn-site.xml et mapreduce-site.xml```. Je te presente ici la configuration minimal, tu peux l'étendre selon tes besoins.

- ```core-site.xml```
Dans ce fichier met la configuration suivant

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master-ip-address:9000</value> <! remplacle master-ip-adress par l'adress ip du la neoud maitre>
    </property>
<configuratin>
```
- ```hdfs-site.xml```

Configuration du systeme de fichier hdfs

```xml
 <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>chemin/vers/le dossier/data/de namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>chemin/vers/le dossier data/de/datanode</value>
    </property>
</configuration>
```
-```yarn-site.xml```

Configuration de gestionnaire de ressource

```xml
    <configuration>
    <!-- Emplacement du ResourceManager -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hostname</value>
    </property>
        <property>
        <name>yarn.nodemanager.hostname</name>
        <value>hostname</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>8192</value> <!-- Mémoire maximale allouée pour chaque NodeManager en Mo -->
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>4</value> <!-- Nombre maximal de cœurs de CPU alloués pour chaque NodeManager -->
    </property>
</configuration>
```
- Créer un fichiers esclaves

Pour que la machine maitre, master node sache ou se situe les machines esclave, tu crée un fichier nommé ```workers``` dans lequel tu met les adress ip des machines esclaves en fesant ceci. Note que le fichier est créé dans le dossier ```/etc```

```zsh
sudo vim workers # L'éditeur de text va s'ouvri, la tu met les adresses ip des noeuds esclaves
```

4. Excécution

Si tout est bien configuré, ainsi que les machines communiques entre elles, il faut excécuter ces commandes sur le noeud ```maitre```.

```zsh
$HADOOP_HOME/bin/hdfs namenode -format # Formatage du système de fichier
$HADOOP_HOME/bin/hdfs --daemon start namenode # démaré le namenode
$HADOOP_HOME/bin/hdfs --daemon start datanode # datanode
$HADOOP_HOME/sbin/start-dfs.sh
$HADOOP_HOME/bin/yarn --daemon start resourcemanager
$HADOOP_HOME/bin/yarn --daemon start nodemanager
$HADOOP_HOME/sbin/start-yarn.sh
```

Tout est maintenant démaré. Pour le vérifier taper cette commande sur chaque noeud

```zsh
jps
```

Si tu suit ces étapes, tu pourras mettre en place ton cluster de AWS. 
