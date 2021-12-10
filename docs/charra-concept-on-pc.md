# How to run CHARRA concept on PC

The detailed steps of how to build and run CHARRA applications (Attester and Verifier) are
described on corresponding GitHub repository:
 - [Build and run CHARRA on PC](https://github.com/Fraunhofer-SIT/charra#build-and-run)

In general there are two common ways to reach the goal:
* Build docker container with binary packages
* Build neccessary packages from sources inside docker container

The first path is faster and more preferable.
There are two scripts there: `docker/build.sh` - to build and install docker container
and `docker/run.sh` - to run docker container with installed packages for CHARRA.

When the project is built one may run and test Attester and Verifier application:
1. List the docker networks:
```shell
$ docker network ls
NETWORK ID     NAME             DRIVER    SCOPE
....
a13b7967a1d4   `charra_default`   bridge    local
....
```
2. Configure network for docker in order to allow containers to communicate.
E.g. Edit file ./docker/run.sh and add internal virtual network:
```shell
--network=charra_default
```
to docker run command

3. Specify IP address of Attester in file src/verifier.c (char dst_host[16] in src/verifier.c)
and rebuild to project in container:
```shell
cd charra
make
```
4. Run Attester and Verifier on two different consoles:
![Attester](images/fobnail-run-attester-verifier.png)
5. Got the result.
![Verifier](images/fobnail-attestation-results.png)
