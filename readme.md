# Utiliser Hadoop et MapReduce avec Docker

![Apache Hadoop](https://img.shields.io/badge/Apache%20Hadoop-66CCFF?style=for-the-badge&logo=apachehadoop&logoColor=black)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white)

Ce guide vous explique comment utiliser Hadoop et MapReduce en utilisant Docker. Les étapes sont les suivantes :

1. Installer Docker
2. Récupérer l'image Docker pour Hadoop
3. Lancer les containers Docker
4. Importer tous nos fichiers dans notre contexte Hadoop
5. Utiliser Hadoop
6. Lancer notre MapReduce avec Hadoop
7. Récupérer le résultat de notre MapReduce

## Étape 1 : Lancer son conteneur Hadoop

Ouvrez votre terminal et tapez la commande suivante :

`git clone https://github.com/big-data-europe/docker-hadoop`

Ensuite, rendez-vous dans le chemin du dossier que vous venez de cloner :

`cd docker-hadoop`

Avant de poursuivre, n'oubliez pas de lancer Docker Desktop.

Une fois Docker Desktop lancé, retournez dans votre terminal et exécutez la commande suivante :

`docker-compose up -d`

Cette commande va lancer les containers Docker pour Hadoop et créer un réseau pour les containers.

ATTENTION, n'oubliez pas de bien ajouter l'option `-d` sinon l'exécution prendra beaucoup de temps.

Une fois l'exécution terminée, vérifiez que les images Docker de Hadoop sont bien présentes en tapant la commande
suivante :

``docker container ls``

Maintenant que Hadoop est installé, vous pouvez y accéder en tapant la commande suivante dans votre terminal :

`docker exec -it namenode /bin/bash`

Nous allons appeler ce terminal avec Hadoop qui tourne `*terminal 1`. Laissez-le de côté sans le fermer pour le moment.

Nous allons maintenant créer un projet Java et écrire notre code MapReduce en Java.

Pour plus de simplicité dans ce tutoriel, j'utilise un projet Maven lancé sur IntelliJ.

L'avantage est qu'il a déjà Maven en tant que plugin, ce qui facilite grandement les choses.

## Étape 2 : Créer un projet Maven avec IntelliJ

Allons sur IntelliJ et créons un nouveau projet Maven comme ceci :

![img.png](https://media.discordapp.net/attachments/951037583876558882/1102586498781548564/image.png?width=856&height=655)

# Instructions pour créer un projet Maven et l'utiliser avec Hadoop

Une fois le projet Maven créé, allons dans le `pom.xml` de notre projet et ajoutons 3 dépendances :

````xml

<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-examples</artifactId>
        <version>3.3.5</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-common</artifactId>
        <version>3.3.5</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.3.5</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
````

> **Attention** : bien utiliser les dernières versions disponibles en vérifiant sur le site https://mvnrepository.com/

Ensuite, si ce n'est pas déjà présent dans votre `pom.xml`, créons nos balises :

```xml

<build>
    <plugins>
    </plugins>
</build>
````

C'est dedans qu'on va mettre tous les plugins des dépendances qu'on a ajoutées plus haut :

````xml

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.4.1</version>
            <configuration>
                <transformers>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
                    </transformer>
                </transformers>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
````

> **Attention** : ICI AUSSI À VÉRIFIER LES BONNES VERSIONS DE VOS PLUGINS : https://mvnrepository.com/


Pour exécuter des commandes Maven sur IntelliJ, il est recommandé d'utiliser l'interface intégrée à l'IDE. En effet, le
projet Maven par défaut d'IntelliJ utilise son propre Maven, ce qui peut causer des problèmes si vous n'avez pas Maven
installé sur votre machine en dehors d'IntelliJ.

Pour accéder aux commandes Maven, il suffit de cliquer sur l'onglet Maven situé sur le côté droit de votre IDE, puis de
sélectionner l'option LifeCycle.

![img_1.png](https://media.discordapp.net/attachments/951037583876558882/1102588322855006318/image.png?width=427&height=655)

On va maintenant cliquer sur ``install`` pour installer toutes nos dépendances et plugins écrits plus haut. Une fois
fini, on peut commencer à coder !

## Étape 3 : Génération du .jar

Créez une nouvelle classe WordCount dans votre projet. Gardez le nom, c'est important.
Une fois la classe créée, je vous invite à copier-coller ce code qui est le code du MapReduce qu'on utilisera.
Le code ci-dessous est déjà modifié pour pouvoir traiter et séparer les lettres contenant des caractères non
alphanumériques et les compter comme mots à part entière :

````java
package org.example;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class WordCount {

    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            String temp = value.toString().replaceAll("([^\\p{Alnum}])", " $1 ");
            StringTokenizer itr = new StringTokenizer(temp);
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage: wordcount <in> <out>");
            System.exit(2);
        }
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
````

Noté ici le package :

````java
package org.example;
````

Le nom du package est très important, gardez-le en tête pour plus tard.

Maintenant, on peut lancer notre code avec le bouton package de l'interface Maven de IntelliJ ou mvn package pour ceux
qui ont Maven sur leur machine.

Une fois l'exécution finie, un nouveau dossier va se créer dans notre projet ``target``. C'est dans le fichier target
qu'on trouvera notre fichier .jar qu'on utilisera pour Hadoop.

![img_2.png](https://media.discordapp.net/attachments/951037583876558882/1102590795204272158/image.png?width=291&height=655)

## Étape 4 : Importer tous nos fichiers dans notre contexte Hadoop

Maintenant qu'on a notre fichier .jar créé, si on parle de MapReduce, il nous faut un fichier texte à traiter !

- Choisissez le fichier texte de votre choix et ouvrez un nouveau terminal (on l'appellera terminal 2). N'oubliez pas
  qu'on a toujours notre terminal 1 d'ouvert.

- On va maintenant taper cette ligne de commande pour pouvoir ajouter notre fichier .jar et notre fichier à traiter

Ajout du fichier text à traiter :

- ``docker cp <Chemin du fichier text a traité> namenode:/tmp``

Ajout du fichier jar créé précédemment :

- ``docker cp <Chemin du fichier jar crée précédement> namenode:/tmp``

Maintenant qu'on a nos 2 fichiers présents dans Hadoop, on peut retourner sur notre terminal 1 et continuer la suite :

## Étape 5 : Utiliser Hadoop

De retour sur notre terminal 1, on peut lancer la commande ``ls`` pour vérifier que notre fichier jar et notre fichier
text sont bien présents.

Maintenant, pour pouvoir utiliser Hadoop, il faut encore déplacer le fichier text traité mais cette fois-ci dans le
système de traitement de fichier de Hadoop : ``hdfs``.

Donc, pour ce faire, on doit d'abord créer un nouveau répertoire /user/root dans la HDFS :

``hdfs dfs -mkdir -p /user/root``

Et ensuite y déposer notre fichier text :

``hdfs dfs -put <file_name> /user/root``

On peut vérifier que cela a bien été ajouté en exécutant la commande :

``hdfs dfs -cat <file_name>``

Le fichier déposé ici est le fichier d'entrée qu'on utilisera pour notre MapReduce.

## Étape 6 : Lancer notre MapReduce avec Hadoop

Maintenant que tout est prêt, on peut lancer le MapReduce.

Maintenant que tout est prêt, on peut lancer le MapReduce. N'oubliez pas de faire attention à votre nom de classe et de
package, car ils seront utilisés dans la commande suivante :

````shell
hadoop jar <jar_file_name> <package_name>.<class_name> user/root/<input> <output>
````

Dans l'option ``input``, on spécifie l'emplacement du fichier texte à traiter, en l'occurrence celui déposé dans la HDFS
précédemment. Dans l'option ``output``, on donne un nom quelconque pour le fichier de sortie, qui sera créé
automatiquement
dans le répertoire ``user/root`` que nous avons créé auparavant.

Reprenons l'exemple de projet que j'ai utilisé dans le screenshot précédent :

```shell 
hadoop jar big_data-1.0-SNAPSHOT.jar org.example.WordCount /user/root/rousseauline-all res.txt
```

Ensuite, pour visualiser le résultat, utilisez la commande suivante :

```shell 
hdfs dfs -cat /user/root/<file_name>/part-r-00000
```

Le fichier de resultat s'appelera toujour : `part-r-00000`
et on obtient bien le resultat de notre map reduce

![img_3.png](https://media.discordapp.net/attachments/951037583876558882/1102597331238531122/image.png?width=516&height=655)

## Étape 7 : Récupérer les résultats du MapReduce

Maintenant que nous avons notre fichier de résultat dans HDFS, nous pouvons le récupérer et le copier sur notre machine
hôte. Tout d'abord, dans le terminal 1, nous allons copier le fichier de la HDFS vers le contexte local Hadoop :

- ``hdfs dfs -copyToLocal /user/root/<file_name>/part-r-00000 .``

Ensuite, dans le terminal 2, nous allons récupérer le fichier sur notre machine :

- ``docker cp namenode:/tmp/part-r-00000 <path>``

Félicitations ! Nous avons terminé ! Vous pouvez maintenant fermer les terminaux et travailler sur votre fichier de
résultat à votre guise.

## Simplifier l'exécution de MapReduce Hadoop avec un script bash

Cependant, vous avez peut-être remarqué qu'il y a plusieurs étapes redondantes lorsque vous exécutez plusieurs fois le
code. Il serait donc judicieux de créer un script bash pour automatiser certaines étapes !

````sh
#!/bin/bash

if [ "$#" -ne 3 ]; then
    echo "Usage: $0 <jar_file_name> <<package_name>.<class_name>> <input> <output>"
    exit 1
fi

jar_file_name=$1
package_class_name=$2
input=$3
output=$4

echo "Uploading file $input to HDFS..."
hdfs dfs -put "$input" /user/root/

echo "Running Hadoop job..."
hadoop jar "$jar_file_name" "$package_class_name" "$input" "$output"

echo "Retrieving output from HDFS..."
hdfs dfs -cat /user/root/$output/part-r-00000

echo "Copying output to local file system..."
hdfs dfs -copyToLocal /user/root/$output/part-r-00000 .

echo "Output saved to file part-r-00000"
````

Voici un exemple d'utilisation :

````shell
./script.sh big_data-1.0-SNAPSHOT.jar org.example.WordCount /user/root/rousseauline-all res.txt
````

# Conclusion

Dans ce tutoriel, nous avons appris comment exécuter un MapReduce Hadoop sur un cluster Hadoop en utilisant Docker. Nous
avons également vu comment travailler avec HDFS et comment importer des fichiers dans le contexte Hadoop pour le
traitement. J'espère que ce tutoriel vous a été utile et que vous avez appris quelque chose de nouveau !
