Dockerised Jenkins
------------------

Setting up Jenkins locally using Docker.

This is experimental for local dev use and should have more secure setup for production setups. 

See reference material here on which this is based: 
- https://twasink.net/2016/08/01/setting-up-a-jenkins-server-with-docker/

Containers
----------

- data: For jenkins_home
- master: Jenkins master
- slave: Jenkins slave
- nginx: Proxy for Jenkins to run port 80 etc
- socat: Remapping unix docker socket to TCP port for Jenkins to access the host/docker

Slave container really only needs to be built and not run up as Jenkins runs it up on demand.


Run it up and get initial Jenkins admin password
------------------------------------------------
- docker-compose build
- docker-compose up -d data master nginx socat
- docker-compose exec master cat /var/jenkins_home/secrets/initialAdminPassword
- http://localhost => Go do Jenkins config

Configure the Docker Slave
--------------------------
- Create Credentials for slave user / password (set in Slave Dockerfile, but consider SSH keys)
- Jenkins configure: http://localhost/configure
- Go to Cloud section (bottom of config)
- Add New Cloud : Docker
- Name: SomeName
- Docker URL: tcp://<DOCKER_HOST_IP>:2375 (reason for socat container, unix socket doesn't work)
- Test Connection => 'Version = 17.06.2-ce, API Version = 1.30' (the Docker version)
- Add Docker template
- Docker template / Docker Image: jenkinsindocker_slave (name of docker slave image)
- Docker template / Instance capacity : 2
- Docker template / Set SSH Creds for Docker SSH Launcher
- Docker template / Host Key Verification Strategy : non-verifying (should be more secure for real system)
- Set a slave label if required
- Configure master for 0 executors
- A job should spin up a slave (make a dummy job and try it with the label or 'agent any' in pipeline)

Backup the base config (so it can be restored when updating the Jenkins Docker Image)
-------------------------------------------------------------------------------------
- docker-compose stop
- docker-compose run --rm master tar cvfz /backup/base.tgz /var/jenkins_home

Restore
-------

- docker-compose stop
- docker-compose run --rm master bash -C "cd /var/jenkins_home && tar xvfz /backup/backup.tgz --strip 2"

Run Command on Container
------------------------

- docker-compose run --rm master bash

New Jenkins Version and Restore Jenkins Home / Config
-----------------------------------------------------
- docker-compose build --pull master
- docker-compose run --rm master bash -C "cd /var/jenkins_home && tar xvfz /backup/backup.tgz --strip 2"
