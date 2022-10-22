# gitlab-runner on docker
```
docker volume create gitlab-runner-config
docker run -d --name gitlab-runner --restart always -v /var/run/docker.sock:/var/run/docker.sock -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest
docker run --rm -it -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest register
```
# gitlab runner register
```
root@demo:~# docker run --rm -it -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest register
Runtime platform                                    arch=amd64 os=linux pid=7 revision=0d4137b8 version=15.5.0
Running in system-mode.

Enter the GitLab instance URL (for example, https://gitlab.com/):
https://gitlab.com/
Enter the registration token:
9n79384n5mfd9m09w9f9f
Enter a description for the runner:
[0ab13b3fe236]:
Enter tags for the runner (comma-separated):

Enter optional maintenance note for the runner:

Registering runner... succeeded                     runner=GR1348941T3Bmygbo
Enter an executor: docker, parallels, ssh, docker-ssh+machine, instance, kubernetes, custom, docker-ssh, shell, virtualbox, docker+machine:
docker
Enter the default Docker image (for example, ruby:2.7):
ruby:2.7
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

# Error Builr Docker Image With Docker Runners
You must start the gitlab-runner container with
```
--privileged true
```
but that is not enough. Any runner containers that are spun up by gitlab after registering need to be privileged too. So you need to mount the gitlab-runner

```
docker exec -it runner /bin/bash
nano /etc/gitlab-runner/config.toml
```
and change privileged flag from false into true:

```
privileged = true
```
That will solve the problem!

note: you can also mount the config.toml as a volume on the container then you won't have to log into the container to change privileged to true because you can preconfigure the container before running it.

===============================================================

# gitlab-runner on linux
# Download the binary for your system
<pre>
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
</pre>

# Give it permissions to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab CI user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as service
<pre>
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start

sudo gitlab-runner register --url https://gitlab.com/ --registration-token $REGISTRATION_TOKEN
</pre>

#error no host
```
gitlab-runner stop

[[runners]]
  name = "mint.manansfarmstore.com"
  url = "https://gitlab.com/"
  token = "dlsfkj;;fdsj"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    *image = "docker:stable"*
    *privileged = true*
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    
gitlab-runner start
```

# ciyaml
```
stage-build:
  image: docker:18.09-dind
  stage: build
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2375/
    DOCKER_DRIVER: overlay2
    DOCKER_TLS_CERTDIR: ""

  script:
    - . .gitlab-deploy.sh "BUILD"
```
