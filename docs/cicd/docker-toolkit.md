 # Simple Docker engine role
 This role simply provides a latest Docker engine, docker image archiving and Nexus private registry Integration.
 
 ### Archive Docker images
 In order to install our CICD Stack offline we need few docker image binaries(i.e Juseppe, Gitlab),
 The role downloads and archives those images according to the provided var `docker_archives` list.
 The archived images are placed in `{{ work_dir }}/lib/prereqs/` with a suffix of 'EXAMPLE-docker.tar'
 
 The process clearly needs to be done with an internet connection.
 
 Run `ansible-playbook playbooks/docker-daemon.yml -i inventory/hosts.ci` 
 
 ### Nexus as a Docker private Registry
 
 #### Daemon.json
 This file configures Docker to work with Nexus as a Docker private registry (Proxy and Hosted).
 should be located at `/etc/docker/daemon.json`
 ```bash
 {
  "insecure-registries": [
    "{{ nexus_hostname }}:{{ nexus_repos_docker_group[0].http_port }}"
  ],
  "disable-legacy-registry": true
}
 ```
 ***Note:*** The example above is pre-deploy daemon.json
 
 
 ### Push images to Nexus Hosted Docker registry
To push images:
* Login
* Tag the local image to our docker repository
* Push

> EXAMPLE
```bash
docker login -u NEXUS_USERNAME -p NEXUS_PASSWORD 10.0.0.1:9082

docker tag lanwen/juseppe:1.0.0 10.0.0.1:9082/lanwen/juseppe:1.0.0

docker push 10.0.0.1:9082/lanwen/juseppe:1.0.0
```

### Pull images through Nexus Proxy Docker registry
***Note:*** Internet connection is necessary.
* Login (See the above example)
* Pull with nexus url tag

> EXAMPLE
```bash
docker login -u NEXUS_USERNAME -p NEXUS_PASSWORD 10.0.0.1:9082

docker pull 10.0.0.1:9082/exmaple/image:latest

docker images # List images

docker rmi 10.0.0.1:9082/exmaple/image:latest # To delete the local image copy.
```

### Archive images to tar
To move local docker image from host to host we need to archive the image into a binary.
to do so run the following command.

```bash
docker images 

docker save -o ubuntu-binary.tar lanwen/juseppe:1.0.0 
```