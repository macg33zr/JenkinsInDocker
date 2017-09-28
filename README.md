Dockerised Jenkins
------------------

Setting up Jenkins locally using Docker. 

See reference material here on which this is based: 
https://twasink.net/2016/08/01/setting-up-a-jenkins-server-with-docker/

Backup
------
docker-compose stop
docker-compose run --rm master tar cvfz /backup/base.tgz /var/jenkins_home

Restore
-------

docker-compose stop
docker-compose --rm master bash -C "cd /var/jenkins_home && tar xvfz /backup/backup.tgz --strip 2"

Run Command on Container
------------------------

docker-compose run --rm master bash
