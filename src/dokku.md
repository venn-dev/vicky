# Dokku

Dokku is a self-hosted heroku-like service. You can `git push` to start a deployment build with heroku-like buildpacks or Docker, and it will handle
updating load balancer (nginx) configuration, zero-downtime deployments, SSL certificates via let's encrypt, etc.

## Manual deployment

In case you don't need code to be built, you can manually deploy a pre-built docker image, for example for deploying docker images built by others.

1. Create a new app with `dokku apps:create APP_NAME`
2. Tag the pre-built docker image as `dokku/APP_NAME:VERSION` with `docker image tag IMAGE dokku/APP_NAME:VERSION`. Replace VERSION with 0.0.1,0.0.2 etc.
3. Deploy the docker image using `dokku tags:deploy APP_NAME VERSION`.

### Example: Grafana deployment

In this example we deploy grafana version 7.3.7, using the /var/lib/dokku/data/storage folder for volumes as recommended by dokku.

    $ dokku apps:create grafana
    $ docker image pull grafana/grafana:7.3.7
    $ docker image tag grafana/grafana:7.3.7 dokku/grafana:7.3.7
    $ dokku storage:mount grafana /var/lib/dokku/data/storage/grafana:/var/lib/grafana
    $ sudo mkdir /var/lib/dokku/data/storage/grafana
    $ sudo chown 472:472 /var/lib/dokku/data/storage/grafana/
    $ dokku tags:deploy grafana 7.3.7
    $ dokku proxy:ports-set grafana http:80:3000
