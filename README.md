<img align="right" src="https://raw.githubusercontent.com/startxfr/docker-images/master/travis/logo-small.svg?sanitize=true">

# docker-images-example-php


Example of a dynamic web application using the startx s2i builder [startx/sv-php](https://hub.docker.com/r/startx/sv-php). 
For more information on how to use this image, **[read startx php image guideline](https://github.com/startxfr/docker-images/blob/master/Services/php/README.md)**.

## Running this example in OKD (aka Openshift)

### Create a sample application

```bash
# Create a openshift project
oc new-project startx-example-php
# start a new application (build and run)
oc process -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/php/openshift-template-build.yml -p APP_NAME=myapp | oc create -f -
# Watch when resources are available
sleep 30 && oc get all
```

### Create a personalized application

- **Initialize** a project
  ```bash
  export MYAPP=myapp
  oc new-project ${MYAPP}
  ```
- **Add template** to the project service catalog
  ```bash
  oc create -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/php/openshift-template-build.yml -n startx-example-php
  ```
- **Generate** your current application definition
  ```bash
  export MYVERSION=0.1
  oc process -n startx-example-php -f startx-php-build-template \
      -p APP_NAME=v${MYVERSION} \
      -p APP_STAGE=example \
      -p BUILDER_TAG=latest \
      -p SOURCE_GIT=https://github.com/startxfr/docker-images-example-php.git \
      -p SOURCE_BRANCH=master \
      -p MEMORY_LIMIT=256Mi \
  > ./${MYAPP}.definitions.yml
  ```
- **Review** your resources definition stored in `./${MYAPP}.definitions.yml`
- **build and run** your application
  ```bash
  oc create -f ./${MYAPP}.definitions.yml -n startx-example-php
  sleep 15 && oc get all
  ```
- **Test** your application
  ```bash
  oc describe route -n startx-example-php
  curl http://<url-route>
  ```

## Running this example with source-to-image (aka s2i)

### Create a sample application

```bash
# Build the application
s2i build https://github.com/startxfr/docker-images-example-php startx/sv-php startx-php-sample
# Run the application
docker run --rm -d -p 8777:8080 startx-php-sample
# Test the sample application
curl http://localhost:8777
```

### Create a personalized application

- **Initialize** a project directory
  ```bash
  git clone https://github.com/startxfr/docker-images-example-php.git php-myapp
  cd php-myapp
  rm -rf .git
  ```
- **Develop** and create a personalized page
  ```bash
  cat << "EOF"
  <?php phpinfo(); ?>
  EOF > index.php
  ```
- **Build** your current application with startx php builder
  ```bash
  s2i build . startx/sv-php:latest startx-php-myapp
  ```
- **Run** your application and test it
  ```bash
  docker run --rm -d -p 8777:8080 startx-php-myapp
  ```
- **Test** your application
  ```bash
  curl http://localhost:8777
  ```
