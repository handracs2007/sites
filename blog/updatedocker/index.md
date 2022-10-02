## Handra

### [Home](/) | Blog | [Disclaimer](/disclaimer) | [Terms and conditions](/tnc)

[<<go back](..)

### Update all docker images
Sometimes, there can be a need to pull latest images of all the docker images that we have in our installation. Unfortunately, docker does not provide a simple command for us to be able to update all the pulled docker images.

Luckily, with the power of **bash**, we will be able to accomplish this task. The command below will retrieve the images that we have in our installation and pull latest images from the docker repository.

```bash
docker images | grep -v REPOSITORY | awk '{print $1}' | xargs -L1 docker pull
```

Now, let's break down what this command actually does. I'm not going to discuss in detail on what the \| character is used for.

---
```bash
docker images
```
This command is used to list down all the images that we have in our docker installation. If you are used to docker, this should not be a surprise.

Below is the sample output of the command.
```bash
handra@nebula  ~  docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
postgres                          latest              817f2d3d51ec        2 days ago          314MB
primekey/ejbca-ce                 latest              996a15c0fb9a        5 days ago          623MB
pmlpostgres                       latest              3dc7d2347bb6        2 weeks ago         314MB
postgres                          <none>              62473370e7ee        6 weeks ago         314MB
confluentinc/ksqldb-examples      5.5.1               88f3d11247f3        3 months ago        646MB
confluentinc/cp-ksqldb-server     5.5.1               d2f03e1e91d8        3 months ago        679MB
confluentinc/cp-ksqldb-cli        5.5.1               c2768c7e4cc5        3 months ago        663MB
confluentinc/cp-kafka             5.5.1               3020f0651d7f        3 months ago        598MB
confluentinc/cp-schema-registry   5.5.1               f51e4f854dc1        3 months ago        1.19GB
confluentinc/cp-kafka-rest        5.5.1               2632bb34f956        3 months ago        1.15GB
confluentinc/cp-zookeeper         5.5.1               7149731cc563        3 months ago        598MB
cnfldemos/kafka-connect-datagen   0.3.2-5.5.0         002eda72fc69        4 months ago        1.24GB
devopsfaith/krakend               latest              c7e33ec819c0        6 months ago        145MB
```

---
```bash
grep -v REPOSITORY
```
This command is used to get all the lines output from the command ```docker images``` and filter only those lines that do not have keyword *REPOSITORY*.

Below is the sample output of the command.
```bash
handra@nebula  ~  docker images | grep -v REPOSITORY
postgres                          latest              817f2d3d51ec        2 days ago          314MB
primekey/ejbca-ce                 latest              996a15c0fb9a        5 days ago          623MB
pmlpostgres                       latest              3dc7d2347bb6        2 weeks ago         314MB
postgres                          <none>              62473370e7ee        6 weeks ago         314MB
confluentinc/ksqldb-examples      5.5.1               88f3d11247f3        3 months ago        646MB
confluentinc/cp-ksqldb-server     5.5.1               d2f03e1e91d8        3 months ago        679MB
confluentinc/cp-ksqldb-cli        5.5.1               c2768c7e4cc5        3 months ago        663MB
confluentinc/cp-kafka             5.5.1               3020f0651d7f        3 months ago        598MB
confluentinc/cp-schema-registry   5.5.1               f51e4f854dc1        3 months ago        1.19GB
confluentinc/cp-kafka-rest        5.5.1               2632bb34f956        3 months ago        1.15GB
confluentinc/cp-zookeeper         5.5.1               7149731cc563        3 months ago        598MB
cnfldemos/kafka-connect-datagen   0.3.2-5.5.0         002eda72fc69        4 months ago        1.24GB
devopsfaith/krakend               latest              c7e33ec819c0        6 months ago        145MB
```

---
```bash
awk '{print $1}'
```
This command is used to print out only the first column of an input. In this case, the input comes from the output of the command ```grep -v REPOSITORY``` above.

Below is the sample output of the command.
```bash
handra@nebula  ~  docker images | grep -v REPOSITORY | awk '{print $1}'
postgres
primekey/ejbca-ce
pmlpostgres
postgres
confluentinc/ksqldb-examples
confluentinc/cp-ksqldb-server
confluentinc/cp-ksqldb-cli
confluentinc/cp-kafka
confluentinc/cp-schema-registry
confluentinc/cp-kafka-rest
confluentinc/cp-zookeeper
cnfldemos/kafka-connect-datagen
devopsfaith/krakend
```

---
```bash
xargs -L1 docker pull
```
This command is used to execute a command, which in this case is ```docker pull``` by reading the lines one by one.

---

That's it. I hope it helps anyone in updating all of your docker images in one simple command line.

---
> Copyright &copy; 2020-2022 Handra. All Rights Reserved.