# gitlab-runner
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permissions to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab CI user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start

sudo gitlab-runner register --url https://gitlab.com/ --registration-token $REGISTRATION_TOKEN

#error no host
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


# ciyaml
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
