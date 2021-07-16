# Practical DevSecOp Notes
![alt text](https://www.practical-devsecops.com/wp-content/uploads/2020/12/xDevSecOps-icon.png.pagespeed.ic.ccuK0pJYo1.webp "Practical DevSecOps Logo")

# WELCOME

# Table of Contents
- [How to Use This Portal](#portalUsage)
- [Linux Basic](#linuxBasic)
- [Docker](#docker)
    * [Docker Basic Command](#dockercmd)
    * [Create Docker Image](#dockerimg)
    * [Manage Data In Docker](#dockerdatamgmt)
    * [Docker Registry](#dockerreg)
    * [Docker Networking](#dockernetwork)
    * [Docker File](#dockerfile)
- [Continuous Integration And Continuous Delivery](#CICD)
    * [Create Simple CI Pipeline](#simplecicd)*
    * [Create Advanced CI CD Pipeline](#advcicd)*
    * [Conditional Pipeline with Gitlab Rules](#gitlabpipe)
    * [Continuous Deployment with Gitlab](#gitlabcd)
    * [Continuous Integration with GitHub Actions](#gaci)
    * [Continuous Deployment with GitHub Actions](#gacd)
- [Component Analysis](#CA)
    * [Sca Using Safety](#scaSafety)*
    * [Embed Safety Into CICD Pipeline](#safetyCICD)*
    * [Embed Safety Into Jenkins](#afetyCICDJenkins)
    * [Embed Safety Into Github Actions](#safetyGitHubActions)
    * [Sca Using RetireJS](#scaRetireJS)*
    * [Embed RetireJS into CICD Pipeline](#retireJSCICD)*
    * [Embed RetireJS into Github Actions](#retireJSGitHubActions)
    * [Sca Using Dependency Check](#scaDependency)
    * [Embed Dependency Check into CICD Pipeline](#dependencyCICD)
    * [Sca using Snyk](#scaSnyk)
    * [Embed Snyk into CICD Pipeline](#snykCICD)
    * [Embed Snyk into Github Actions](#snykGitHubActions)
    * [Sca Using NPM Audit](#scaNPMAudit)
    * [Sca Using AuditJS](#scaAuditJS)
    * [Sca Using Bundler Audit](#scaBundlerAudit)
    * [Sca Using Chelsea](#scaChelsea)
- [Static Analysis](#SA)
- [Dynamic Analysis](#DA)
- [Infrastructure As Code](#IaaC)
    * [IaC Using Ansible](#IaCAnsible)
    * [Ansible Adhoc Commands](#AnsibleAdHocCmd)
    * [Ansible Playbook Basics](#AnsiblePlaybook)
    * [Ansible Conditionals](#AnsibleConditional)
    * [Deploy Container Using Ansible](#AnsibleContainer)
    * [Audomated Hardening Using Ansible](#AnsibleHardening)
    * [Terraform Linter Using Tflint](#Tflint)
- [Compliance As Code](#CaaC)
- [Vulnerability Management](#VM)
- [Summary](#Summary)
- [Tips](#tips)

---
## How to Use This Portal <a name="portalUsage"></a>
---
---
## Linux Basic <a name="linuxBasic"></a>
---
---
## Docker <a name="docker"></a>
---
* Docker Basic Command<a name="dockercmd"></a>
    * Pull Image in quiet mode
        * `docker pull -q alpine`
    * Run Image
        * `docker run -d --name myubuntu ubuntu`
            * -d : run container in background and print container ID
            * `docker ps` can't find the container
            * Process not attach to the container and exits as soon as start
        * `docker run -d --name myubuntu -i ubuntu`
    * Remove Container
        * `docker rm myubuntu`
    * Run Command in a running container
        * `docker exec myubuntu whoami`
    * Container Shell
        * `docker exec -it myubuntu bash`
            * -i and -t : interactive session with a tty attached
    * List containers that exited or stopped
        * `docker ps -a`
    * Shows the running processes of a container
        * `docker top myubuntu`
* Create Docker Image<a name="dockerimg"></a>
    * Build the docker image from DOCKERFILE
        * `docker build --tag django.nv:1.0 .`
            * django.nv is the repository
            * 1.0 is the tag
    * Expose container ports to outside world
        * `docker run -v HOST_DIR:DOCKER_DIR container-name`
        * `docker run -v $(pwd):app container-name`
        * ``
        * `docker run -p8081:8080 -v $(pwd):app container-name` # expose docker port 8080 to host port 8081
            * Host Port is 8081
            * Docker Port is 8080
        * ``
    * Exercise:
        * Rename the docker image django.nv:1.0 to django.nv:1.1
            * `docker build -t django.nv:1.1 .`
        * Override Entrypoint to run a bash shell (/bin/bash)
            * `docker run -d --name mydjango -i django`
            * `docker exec --entrypoint /bin/bash -it mydjango`
        * Delete the docker image django.nv:1.0
            * `docker image rm django.nv:1.0`
* Manage Data In Docker<a name="dockerdatamgmt"></a>
    * Docker Volume
        * Manage docker volume : `docker volume --help`
        * List available volume : `docker volume ls`
        * Create a volume called demo : `docker volume create demo`
            * volume is located and create under `/var/lib/docker/volumes/demo`
        * Run container with persistent volume
            * `docker run --name ubuntu -d -v demo:/opt -it ubuntu:18.04`
                * -d : to start container in detach mode
                * -v demo:/opt : link host volume from `/var/lib/docker/volumes/demo` to container directory `/opt`
                * data created under container's `/opt` directory is listed under `/var/lib/docker/volumes/demo/_data/`
    * [Bind Mounts](https://docs.docker.com/storage/bind-mounts/) "map host directory to container's directory"
        * ```
            docker run --name ubuntu2 -d -v /opt:/opt -it ubuntu:18.04
            docker exec -it ubuntu2 bash
            ls /opt
            ```
            * /opt:/opt : from lef tto right, maps host /opt directory to container's /opt directory
    * Exercise
        * When do you use bind mounts? in development or production stage ?
            * Use `bind mounts` during development to bind source directory to built into the container. Use `volume mount` to mount it to the same location as mounted a bind mount during development
    * [tmpfs](http://docs.docker.oeynet.com/engine/admin/volumes/tmpfs/) "use host's RAM as temoporary storage"
    * Exercise
        * Create a volume called one with docker volume command
            * `docker volume create one`
        * Run an ubuntu:18.04 container with the name as one, and mount the docker volume named one at the /tmp directory of the container named one
            * `docker run --name one -d -v one:/tmp -it ubuntu:18.04`
        * Run an another ubuntu:18.04 container with the name as two, and mount the docker volume named one at the /tmp directory of the container named two
            * `docker run --name two -d -v one:/tmp -it ubuntu:18.04`
        * Create a file /tmp/one.txt in container one with any text
            * `docker exec -it one bash`
            * `echo "any text" > /tmp/one.txt`
        * Remove /tmp/one.txt from container two and create a new file named /tmp/two.txt with any text, inside container two
            * `exit`
            * `docker exec -it two bash`
            * `rm /tmp/one.txt`
            * `echo "random text on container two" > /tmp/two.txt`
            * `ls /tmp/ ; cat /tmp/two.txt`
* Docker Registry<a name="dockerreg"></a>
    * DOCKERFILE
        ```
        # FROM python base image
        FROM python:2-alpine

        # COPY startup script
        COPY . /app

        WORKDIR /app

        RUN apk add --no-cache gawk sed bash grep bc coreutils
        RUN pip install -r requirements.txt
        RUN chmod +x reset_db.sh && bash reset_db.sh

        # EXPOSE port 8000 for communication to/from server
        EXPOSE 8000

        # CMD specifies the command to execute container starts running.
        CMD ["/app/run_app_docker.sh"]

        ADD

        RUN

        ENTRYPOINT
        ```
        * FROM : initializa an OS as a base image
        * COPY : copy the files or directories from host to the container
        * WORKDIR : set working directory in the container. Similar to cd.
        * RUN : executes the OS command in the current image creation process
        * ADD : Similar to COPY but supports download (HTTP), auto extracting compressed file(s), and replacing the existing file to a specific location if needed forcefully
        * CMD : set default command that gets executed once the container starts
        * ENTRYPOINT : Run OS command as the default command when the container starts
    * Create the image `docker build -t django.nv:1.0 .`
    * Store image in the registry 
        * Deploy a registry server locally called registry `docker run -d -p 5000:5000 --restart=always --name registry registry:2`
        * Add the image name with the registry url `docker tag django.nv:1.0 localhost:5000/django.nv:1.0`
        * Push image to the docker registry `docker push localhost:5000/django.nv:1.0`
        * Confirm image on the registry `curl localhost:5000/v2/_catalog`
    * Exercise
        * Sign up for the Free Docker Hub Account
        * Login using the above account details. Use the docker login command on the DevSecOps-Box machine to login into the Docker Hub registry
        * Push/Upload django.nv:1.0 image you created in the above exercise to Docker Hub
            * Name local image in one of the three method
                * `docker built -t fdsasa/django.nv:1.0`
                * `docker tag django.nv:1.0 fdsasa/django.nv:1.0`
                * `docker commit <existing container> fdsasa/django.nv:1.0`
            * Push to DockerHub
                * `docker push fdsasa/django.nv:1.0`
        * Stop the django.nv container and remove the images to save the disk space
            * `docker stop django.nv:1.0`
            * `docker image rm fdassa/django.nv:1.0`
* Docker Networking<a name="dockernetwork"></a>
    * docker network drivers
        * bridge : Container access external network via NAT. Containers on the same bridge network can communicate with each other. Containers are isolated from other bridge network
        * host : container uses host's network directly. Port exposed by the container are exposed on the external network
        * macvlan : assign a parent network device. Each container on the macvlan network receive tis own MAC address on the network that eth0 is connected to.
        * none : disable networking
    * bridge network setup
        * Explore docker network command `docker network -h`
        * See available network `docker network ls`
        * Create a new network `docker network create mynetwork`
        * Get network details `docker inspect mynetwork`
        * Attach network to contianer `docker run -d --name ubuntu --network mynetwork -it ubuntu:18.04`
        * Test container connectivity `docker exec ubuntu apt update`
        * Remove container `docker rm -f ubuntu`
        * Remove network `docker network rm mynetwork`
    * macvlan network setup
        * Create macvlan network `docker network create --driver macvlan mymacvlan`
        * Attach macvlan to container `docker run -d --name ubuntu --network mymacvlan -it ubuntu:18.04`
        * Inspect macvlan network `docker inspect ubuntu -f "{{json .NetworkSettings.Networks }}" | jq`
        * [macvlan reference](https://docs.docker.com/network/macvlan/)
        * Question: What is the difference between bridge vs macvlan driver?
            * macvlan is faster because it skips kernel code. Host cannot communicate with the container over the macvlan interface. Number of macvlan devices dependon hardware limition of the physical NIC.
    * none network setup
        * `docker run -d --name ubuntu --network=none -it ubuntu:18.04`
        * `docker inspect ubuntu -f "{{json .NetworkSettings.Networks }}" | jq`
    * Exercise
        * Create a network with bridge driver called app and define 172.10.2.0/16 as a subnet
            * `docker network create app --subnet=172.10.2.0/16`
        * Run the containers with ubuntu:18.04 image in the background
            * `docker run -d --name ubuntu -it ubuntu:18.04`
        * Attach app network to the running container, detailed how to at this link
            * `docker network connect app ubuntu`
        * What would happen if we remove the network driver without killing the container?
            * `docker network rm app` # error response states there is an active endpoint
* Docker File<a name="dockerfile"></a>
    * Dockerfile Commands
        * FROM : initializa an OS as a base image
        * COPY : copy the files or directories from host to the container
        * WORKDIR : set working directory in the container. Similar to cd.
        * RUN : executes the OS command in the current image creation process
        * ADD : Similar to COPY but supports download (HTTP), auto extracting compressed file(s), and replacing the existing file to a specific location if needed forcefully
        * CMD : set default command that gets executed once the container starts
        * ENTRYPOINT : Run OS command as the default command when the container starts
    * SAMPLE Dockerfile
        ```
            FROM ubuntu:18.04

            RUN apt update && apt install nginx -y
        ```
        * Build Custom Docker Image
            * `docker built -t nginx-custom .`
        * Create a container using ubuntu:18.04 image
            * `docker run -d --name ubuntu -it ubuntu:18.04`
        * In order to ensure the container installs and turn on the web server we need to further modify the Dockerfile. 
    * Sample Dockerfile Update 01
        ```
        FROM ubuntu:18.04

        RUN apt update && apt install nginx -y

        CMD ["/bin/bash", "-c" , "service nginx start"]
        ```
    * Re-build the image
        * `docker build -t custom-nginx .`
        * `docker run -d -it custom-nginx`
        * `docker ps -a`
            * But docker exits after excution the CMD
            * Fix the container exit issue by adding sleep after the CMD
    * Sample Dockerfile Update 01
        ```
        FROM ubuntu:18.04

        RUN apt update && apt install nginx -y

        CMD ["/bin/bash", "-c" , "service nginx start"]
        CMD [";sleep infinity"]
        ```
        * This generates error because only the last CMD is invoked. If more than one CMD specified, only the last one is executed.
    * Sample Dockerfile Update 01
        ```
        FROM ubuntu:18.04

        RUN apt update && apt install nginx -y

        CMD ["/bin/bash", "-c" , "service nginx start; sleep infinity"]
        ```
    * Difference between ENTRYPOINT and CMD
        ```
        FROM ubuntu:18.04

        RUN apt update && apt install nginx -y

        ENTRYPOINT ["/bin/bash", "-c"]

        CMD ["service nginx start"]
        ```
        * First Run the container is not alive
            * `docker run -d --name nginx -it nginx-custom`
            * `docker ps`
        * Second Run the container is  alive
            * `docker run -d --name nginx-one -it nginx-custom`
            * `docker ps`
        * Why is the first run not alive and second run alive?
            * The first run invokes nginx service then exited. The second run only invokes bash shell but did not invoke nginx service.
---
## Continuous Integration And Continuous Delivery <a name="CICD"></a>
---
* Create Simple CI Pipeline<a name="simplecicd"></a>
    * YAML
        * Basic Syntax
        ```
        ---
        # A list of gadgets
        - Sony
        - LG
        - Apple
        - Samsung
        ```
        * Dashes are in ansible scripts but not in the Gitlab CI script because they are optional
        * Gitlab CI # Gitlab CICD use list concept to create stages
            ```
            -build
            -test
            -integration
            -prod
            ```
        * Gitlab CICD uses key: value to configure individual settings in a job
            ```
            stage: test
            image: node:alpine3.10
            script: echo "hello world"
            ```
        * Dictionary within a dictionary
            ```
            test:
                stage: test # Dictionary item stage with the value test
                script:     # Dictionary item with a list as the value
                    - echo "This is a test step"
                    - exit 1
            ```
        * Simple Gitlab CICD by combining the above concept
            ```
            # This is how a comment is added to a YAML file; please read them carefully.

            stages:         # Dictionary
            - build        # this is build stage
            - test         # this is test stage
            - integration  # this is an integration stage
            - prod         # this is prod/production stage

            build:          # this is job named build, it can be anything, job1, job2, etc.,
                stage: build  # this job belongs to the build stage. Here both job name and stage name is the same i.e., build
                script:
                    - echo "This is a build step."  # We are running an echo command, but it can be any command.

            test:
                stage: test
                script:
                    - echo "This is a test step."
                    - exit 1          # Non zero exit code, fails a job.

            integration:          # integration job under stage integration.
                stage: integration
                script:
                    - echo "This is an integration step."

            prod:
                stage: prod
                script:
                    - echo "This is a deploy step."
            ```
        * Gitlab CI Yaml file
            ```
            # This is how a comment is added to a YAML file; please read them carefully.

            stages:         # Dictionary
            - build        # this is build stage
            - test         # this is test stage
            - integration  # this is an integration stage
            - prod         # this is prod/production stage

            build:          # this is job named build, it can be anything, job1, job2, etc.,
            stage: build  # this job belongs to the build stage. Here both job name and stage name is the same, i.e., build
            script:
                - echo "This is a build step."  # We are running an echo command, but it can be any command.

            test:
            stage: test
            script:
                - echo "This is a test step."
                - exit 1          # Non zero exit code, fails a job.

            integration:          # integration job under stage integration.
            stage: integration 
            script:
                - echo "This is an integration step."

            prod:
            stage: prod
            script:
                - echo "This is a deploy step."
            ```
        * Create a simple CI/CD pipeline
            ```
            # This is how a comment is added to a YAML file; please read them carefully.

            stages:        # Dictionary
            - build        # this is build stage
            - test         # this is test stage
            - integration  # this is an integration stage
            - staging      # this is staging stage
            - prod         # this is prod/production stage

            job1:
                stage: build
                script:
                    - echo "This is a build step."

            job2:
                stage: test
                script:
                    - echo "This is a test step."


            job3:
                stage: integration
                script:
                    - echo "This is an integration step."

            job4:
                stage: staging
                script:
                    - echo "This is an staging step."

            job5:
                stage: prod
                script:
                    - echo "This is a deploy step."
                    - exit 1
            ```
* Create Advanced CI CD Pipeline<a name="advcicd"></a>
    * Fail build using exit code. `exit 1` in job 2 fails the build
      ```
        stages:   # Dictionary
            - build   # this is build stage
            - test    # this is test stage
            - integration # this is an integration stage
            - prod       # this is prod/production stage

        job1:       # this is job named build, it can be anything, job1, job2, etc.,
            stage: build    # this job belongs to the build stage. Here both job name and stage name is the same i.e., build
            script:
                - echo "This is a build step"  # We are running an echo command, but it can be any command.

        job2:
            stage: test
            script:
                - echo "This is a test step"
                #************ Non zero exit code, fails a job. ************ #
                - exit 1

        job3:        # integration job under stage integration.
            stage: integration
            script:
                - echo "This is an integration step."

        job4:
            stage: prod
            script:
                - echo "This is a deploy step."
      ```
    * Allow the job failure. Do not want to fail the builds in DevSecOps Maturity Level 1 and 2.
        ```
            job2:
                stage: test
                script:
                    - execute_script_that_will_fail
                allow_failure: true   #<--- allow the build to fail but don't mark it as such
        ```
    * Save the vulnerability scan results in a file on the CI system
        * Store the result using `artifact` tag
            ```
            someScan:
                script: ./security-tool.sh    # <-- this tool generates vulnerabilities.json as output
                artifacts:                    # <--- To save results, we use artifacts tag
                    paths:                      # <--- We then give the path/paths of the scan result files we want to store for further processing
                    - vulnerabilities.json                  #<--- The filename
                    expire_in: 1 week       # <--- To save disk space, we want to store only for 1 week
            ```
            * If we do not specify expire tag, the output file live on the CI system indefinitely
        * Update the YAML file. Artifact is at the right side of the pipeline.
            ```
            stages:   # Dictionary
            - build   # this is build stage
            - test    # this is test stage
            - integration # this is an integration stage
            - prod       # this is prod/production stage

            job1:       # this is job named build, it can be anything, job1, job2, etc.,
                stage: build    # this job belongs to the build stage. Here both job name and stage name is the same i.e., build
                script:
                    - echo "This is a build step"  # We are running an echo command, but it can be any command.
                    - echo "{\"vulnerability\":\"SQL Injection\"}" > output.json
                artifacts:      # notice a new tag artifacts
                    paths: [output.json]   # this is the path to the output.json file

            job2:
                stage: test
                script:
                    - echo "This is a test step."
                    - exit 1         # Non zero exit code, fails a job.
                allow_failure: true   #<--- allow the build to fail but don't mark it as such

            job3:        # integration job under stage integration.
                stage: integration
                script:
                    - echo "This is an integration step."

            job4:
                stage: prod
                script:
                    - echo "This is a deploy step."
            ```
        * Implement Continuous Delivery
            * Have human approval process before deployment. Use continous delivery feasture of gitlab using the `when` tag.
            * ```
              deploy_prod:
                stage: deploy
                script:
                    - echo "Deploy to prod server."
                when: manual   #<-- A human has to click a button (play button in Gitlab) for this task to execute.
              ```
            * ```
              stages:   # Dictionary
                - build
                - test
                - integration
                - prod

              job1:
                stage: build
                script:
                    - echo "This is a build step"
                    - echo "{\"vulnerability\":\"SQL Injection\"}" > output.json
                artifacts:
                    paths: [output.json]

              job2:
                stage: test
                script:
                    - echo "This is a test step."
                    - exit 1
                allow_failure: true

              job3:
                stage: integration
                script:
                    - echo "This is an integration step."

              job4:
                stage: prod
                script:
                    - echo "This is a deploy step."
                when: manual   #<-- A human has to click a button (play button in Gitlab) for this task to
              ```
    * Exercise: Fail a job and allow it to fail
        * Create four stages, namely build, test, integration, and deploy
        * Create a file using the `echo “this is an output” > output.txt` command in the integration stage and upload it using artifacts tag
        * Fail the integration job using exit code
        * Allow the integration job to fail yet move on to the next stage
        * Create a job requiring a person’s approval (a button, play button) before running this job
        ```
        ---
        stages:
        - build
        - test
        - integration
        - deploy
        job1:
          script: "echo \"This is the build step\""
          stage: build
        job2:
          script: "echo \"This is the test step\""
          stage: test
        job3:
          stage: integration
          script:
            - echo "{\"this is an output\"}" > output.json
            - exit 1
          artifacts:
            paths: [output.json]
            when: always  # generates output.json even if the job fails.
          allow_failure: true
        job4:
          script: "echo \"This is the deploy step\""
          stage: deploy
          when: manual
        ```
* Conditional Pipeline with Gitlab Rules<a name="gitlabpipe"></a>
    * Start with the simple CICD pipeline
        ```
        # This is how a comment is added to a YAML file; please read them carefully.

        stages:         # Dictionary
        - build        # this is build stage
        - test         # this is test stage
        - integration  # this is an integration stage
        - prod         # this is prod/production stage

        build:            # this is job named build, it can be anything, job1, job2, etc.,
        stage: build    # this job belongs to the build stage. Here both job name and stage name is the same, i.e., build
        script:
            - echo "This is a build step."  # We are running an echo command, but it can be any command.

        test:
        stage: test
        script:
            - echo "This is a test step."
            - exit 1     # Non zero exit code, fails a job.

        integration:            # integration job under stage integration.
        stage: integration
        script:
            - echo "This is an integration step."

        prod:
        stage: prod
        script:
            - echo "This is a deploy step."
        ```
    * Rule is used to include or exclude jobs in pipelines. In this example this job triggers when the branch name is __master__
        ```
        stages:         # Dictionary
        - build        # this is build stage
        - test         # this is test stage
        - integration  # this is an integration stage
        - prod         # this is prod/production stage

        build:              # this is job named build, it can be anything, job1, job2, etc.,
        stage: build      # this job belongs to the build stage. Here both job name and stage name is the same, i.e., build
        script:
            - echo "This is a build step."          # We are running an echo command, but it can be any command.
        rules:
            - if: '$CI_COMMIT_BRANCH == "master"'   # this job will trigger when the branch name is __master__
        ```
        * If no attributes are defined in rules clause, the defaults are `when: on_success` and `allow_failure: false`
    * This example shows that job4 will run the pipelines with Merge Request event
        ```
        stages:         # Dictionary
        - build        # this is build stage
        - test         # this is test stage
        - integration  # this is an integration stage
        - staging      # this is staging stage
        - prod         # this is prod/production stage

        job1:               # this is job named build, it can be anything, job1, job2, etc.,
        stage: build      # this job belongs to the build stage. Here both job name and stage name is the same i.e., build
        script:
            - echo "This is a build step"          # We are running an echo command, but it can be any command.

        job4:
        stage: test
        script:
            - echo "This is a test step."
        rules:
            - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
        ```
    * This example shows when the Dockerfile is changed, execute the pipelines.
        ```
        stages:         # Dictionary
        - build        # this is build stage
        - test         # this is test stage
        - integration  # this is an integration stage
        - prod         # this is prod/production stage

        build:              # this is job named build, it can be anything, job1, job2, etc.,
        stage: build      # this job belongs to the build stage. Here both job name and stage name is the same, i.e., build
        script:
            - echo "This is a build step."          # We are running an echo command, but it can be any command.
        rules:
            - changes:
            - Dockerfile
        ```
    * This example combines with the if statement. If there is merge request with change of the Dockerfile
        ```
        stages:         # Dictionary
        - build        # this is build stage
        - test         # this is test stage
        - integration  # this is an integration stage
        - prod         # this is prod/production stage

        build:              # this is job named build, it can be anything, job1, job2, etc.,
        stage: build      # this job belongs to the build stage. Here both job name and stage name is the same, i.e., build
        script:
            - echo "This is a build step."          # We are running an echo command, but it can be any command.
        rules:
            - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
            changes:
                - Dockerfile
        ```
    * This example shows to run the pipeline when the Dockerfile exists. We can use [glob patterns](https://en.wikipedia.org/wiki/Glob_(programming))
        ```
        stages:         # Dictionary
        - build        # this is build stage
        - test         # this is test stage
        - integration  # this is an integration stage
        - prod         # this is prod/production stage

        build:              # this is job named build, it can be anything, job1, job2, etc.,
        stage: build      # this job belongs to the build stage. Here both job name and stage name is the same, i.e., build
        script:
            - docker build -t $CI_REGISTRY/root/django-nv .
        rules:
            - exists:
            - Dockerfile
        ```
    * This example shows on `job4`, any changes are pushed to master branch, pipeline continues to run even if some of the job4 commands failed during execution. On `job5`, if changes are pushed for a tag with any name and needs a human approval process before the pipeline run.
        ```
        stages:         # Dictionary
        - build        # this is build stage
        - test         # this is test stage
        - integration  # this is an integration stage
        - staging      # this is staging stage
        - prod         # this is prod/production stage

        job4:
        stage: staging
        script:
            - echo "This is a deploy step to staging environment."
            - exit 1    # Non zero exit code, fails a job.
        rules:
            - if: '$CI_COMMIT_BRANCH == "master"'   # this job will trigger when you create a new tag
            allow_failure: true

        job5:
        stage: prod
        script:
            - echo "This is a deploy step to production environment."
        rules:
            - if: '$CI_COMMIT_TAG !~ "/^$/"'   # this job will trigger when you create a new tag
            when: manual
        ```
* Continuous Deployment with Gitlab<a name="gitlabcd"></a>
    * Simple CICD Pipeline
        ```
        image: docker:latest

        services:
        - docker:dind

        stages:
        - build
        - test
        - release
        - preprod
        - integration
        - prod

        build:
            stage: build
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py check

        test:
            stage: test
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py test taskManager

        integration:
            stage: integration
            script:
                - echo "This is an integration step"
                - exit 1
            allow_failure: true # Even if the job fails, continue to the next stages

        prod:
            stage: prod
            script:
                - echo "This is a deploy step."
        when: manual # Continuous Delivery. This job will not be executed automatically.
        ```
    * The above pipeline failed due to authentication denied. Add the environment variable and update the `release` job. 
        ```
        image: docker:latest

        services:
        - docker:dind

        stages:
        - build
        - test
        - release
        - preprod
        - integration
        - prod

        build:
            stage: build
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py check

        test:
            stage: test
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py test taskManager

        release:
            stage: release
            before_script:
                - echo $CI_REGISTRY_PASS | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
            script:
                - docker build -t $CI_REGISTRY/root/django-nv .   # Build the application into Docker image
                - docker push $CI_REGISTRY/root/django-nv         # Push the image into registry

        integration:
            stage: integration
            script:
                - echo "This is an integration step"
                - exit 1
            allow_failure: true # Even if the job fails, continue to the next stages

        prod:
            stage: prod
            script:
                - echo "This is a deploy step."
        when: manual # Continuous Delivery. This job will not be executed automatically.
        ```
    * Add a `release` job after `test` job to utilize container.
        ```
        image: docker:latest

        services:
        - docker:dind

        stages:
        - build
        - test
        - release
        - preprod
        - integration
        - prod

        build:
            stage: build
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py check

        test:
            stage: test
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py test taskManager

        release:
            stage: release
            script:
            - docker build -t $CI_REGISTRY/root/django-nv .   # Build the application into Docker image
            - docker push $CI_REGISTRY/root/django-nv         # Push the image into registry

        integration:
            stage: integration
            script:
                - echo "This is an integration step"
                - exit 1
            allow_failure: true # Even if the job fails, continue to the next stages

        prod:
            stage: prod
            script:
                - echo "This is a deploy step."
        when: manual # Continuous Delivery. This job will not be executed automatically.
        ```
    * Add production system to the pipeline by adding production system user name, private key, and hostname into the environment variable. Modify the `prod` job.
        ```
        image: docker:latest

        services:
        - docker:dind

        stages:
        - build
        - test
        - release
        - preprod
        - integration
        - prod

        build:
            stage: build
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py check

        test:
            stage: test
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py test taskManager

        release:
            stage: release
            script:
            - docker build -t $CI_REGISTRY/root/django-nv .   # Build the application into Docker image
            - docker push $CI_REGISTRY/root/django-nv         # Push the image into registry

        integration:
            stage: integration
            script:
                - echo "This is an integration step"
                - exit 1
            allow_failure: true # Even if the job fails, continue to the next stages

        prod:
            stage: prod
            image: kroniak/ssh-client:3.6
            environment: production
            only:
                - master
            before_script:
                - eval `ssh-agent -s`
                - mkdir -p ~/.ssh
                - chmod 700 ~/.ssh
                - echo "$PROD_SSH_PRIVKEY" > ~/.ssh/id_rsa
                - chmod 600 ~/.ssh/id_rsa
                - ssh-add ~/.ssh/id_rsa
                - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
            script:
                - echo
                - |
                    ssh root@$PROD_HOST << EOF
                    docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASS} ${CI_REGISTRY}
                    docker rm -f django.nv
                    docker pull ${CI_REGISTRY}/root/django-nv
                    docker run -d --name django.nv -p 8000:8000 ${CI_REGISTRY}/root/django-nv
                    EOF
        ```
* Continuous Integration with GitHub Actions<a name="gaci"></a>
    * Create Github account and setup Github in the terminal
        ```
            # git config --global user.email "your_email@gmail.com"
            # git config --global user.name "your_username"
        ```
    * Push to existing repository
        ```
            git remote add origin https://github.com/username/django.nv.git
            git push -u origin --all
        ```
    * Create a folder and file called .github/workflows/demo.yaml
        ```
        # Github Syntax Reference https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#about-yaml-syntax-for-workflows

        name: Django    # workflow name

        on:
        push:
            branches:   # similar to "only" in GitLab
            - master

        jobs:
        build:
            runs-on: ubuntu-latest    # similar to "image" in GitLab
            steps:
            - run: echo "This is a build step"    # similar to "script" in GitLab

        test:
            runs-on: ubuntu-latest
            steps:
            - run: echo "This is a test step"

        integration:
            runs-on: ubuntu-latest
            steps:
            - run: echo "This is an integration step"
            - run: exit 1

        prod:
            runs-on: ubuntu-latest
            steps:
            - run: echo "This is a deploy step"
        ```
        * Note: Integration has exit 1 that fails the `integration` job but `prod` job continues to run. Why is that?
            * Jobs under a stage will run independent of each other.
    * Exercise
        * Understand the demo.yaml provided in the previous step and figure out why prod job didn’t fail even though integration job failed?
            * Jobs under a stage runs independent of each other
        * Ensure jobs are run sequentially, and prevent the integration job from failing by adding syntax similar to allow_failure: true
        * Add another job called artifact, then create a file using a simple echo command and save it as an artifact

        ```
        # Github Syntax Reference
        # 1. https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#about-yaml-syntax-for-workflows
        # 2. https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idneeds
        # 3. https://docs.github.com/en/actions/guides/storing-workflow-data-as-artifacts

        name: Django    # workflow name

        on:
        push:
            branches:   # similar to "only" in GitLab
            - master

        jobs:
        build:
            runs-on: ubuntu-latest    # similar to "image" in GitLab
            steps:
            - run: echo "This is a build step"    # similar to "script" in GitLab

        test:
            runs-on: ubuntu-latest
            steps:
            - run: echo "This is a test step"

        integration:
            runs-on: ubuntu-latest
            continue-on-error: true
            steps:
            - run: echo "This is an integration step"
            - run: exit 1

        prod:
            runs-on: ubuntu-latest
            steps:
            - run: echo "This is a deploy step"

        artifact:
            runs-on: ubuntu-latest
            steps:
            - run: |
                echo "artifact job ouput" > output.txt
            - name: upload result for artifact job
                uses: actions/upload-artifact@v2
                with:
                name: result
                path: output.txt
        ```
---
## Component Analysis <a name="CA"></a>
---
* Sca Using Safety <a name="scaSafety"></a>
    * Download the source code
        * `git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp`
    * Install python Safety. 
        * `pip3 install safety`
    * Explore Safety options
        * `safety check --help`
    * Run Safety against the python requirement file
        * `safety check -r requirements.txt --json | tee safety_output.json`
            * -r : specify the input file
            * -json : output should be in the JSON format
* Embed Safety Into CICD Pipeline <a name="safetyCICD"></a>

    * Start with the simple CICD pipeline
        ```
        image: docker:latest

        services:
        - docker:dind

        stages:
            - build
            - test
            - release
            - preprod
            - integration
            - prod

        build:
            stage: build
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py check

        test:
            stage: test
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py test taskManager
        ```
    * Exercise
        * Embed OAST scanning in the test stage using the safety tool.
        * You can make use of hysnsec/safety docker image if you wish.
        * Understand the use of Docker’s -v (volume mount) flag/option.
        * Ensure you follow the DevSecOps Gospel and best practices while embedding the safety tool.
        * Rename test job name to oast.
        ```
        image: docker:latest

        services:
        - docker:dind

        stages:
            - build
            - test
            - release
            - preprod
            - integration
            - prod

        build:
            stage: build
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py check

        test:
            stage: test
            image: python:3.6
            before_script:
                - pip3 install --upgrade virtualenv
            script:
                - virtualenv env
                - source env/bin/activate
                - pip install -r requirements.txt
                - python manage.py test taskManager

        oast-frontend:
            stage: test
            image: node:alpine3.10
            script:
                - npm install
                - npm install -g retire # Install retirejs npm package.
                - retire --outputformat json --outputpath retirejs-report.json --severity high --exitwith 0
            artifacts:
                paths: [retirejs-report.json]
                when: always # What is this for? Allow other pipeline to run
                expire_in: one week
            allow_failure: true

        oast:
            stage: test
            script:
                - docker pull hysnsec/safety  # We are going to pull the hysnsec/safety image to run the safety scanner
                # third party components are stored in requirements.txt for python, so we will scan this particular file with safety.
                - docker run -v $(pwd):/src --rm hysnsec/safety check -r requirements.txt --json > oast-results.json
            artifacts:
                paths: [oast-results.json]
                when: always # What is this for? Generate output when the job fails

        integration:
            stage: integration
            script:
                - echo "This is an integration step."
                - exit 1
            allow_failure: true # Even if the job fails, continue to the next stages

        prod:
            stage: prod
            script:
                - echo "This is a deploy step."
            when: manual # Continuous Delivery
        ```
        * `-v $(pwd):/src` : mounts the current directory in the host(runner) to /src inside the container 
        * `--rm` : remove the container after the scan
* Embed Safety Into Jenkins <a name="safetyCICDJenkins"></a>
    * Default Jenkins Pipeline
    ```
        pipeline {
            agent any

            options {
                gitLabConnection('gitlab')
            }

            stages {
                stage("build") {
                    agent {
                        docker {
                            image 'python:3.6'
                            args '-u root'
                        }
                    }
                    steps {
                        sh """
                        pip3 install --user virtualenv
                        python3 -m virtualenv env
                        . env/bin/activate
                        pip3 install -r requirements.txt
                        python3 manage.py check
                        """
                    }
                }
                stage("test") {
                    agent {
                        docker {
                            image 'python:3.6'
                            args '-u root'
                        }
                    }
                    steps {
                        sh """
                        pip3 install --user virtualenv
                        python3 -m virtualenv env
                        . env/bin/activate
                        pip3 install -r requirements.txt
                        python3 manage.py test taskManager
                        """
                    }
                }
            }
            post {
                failure {
                    updateGitlabCommitStatus(name: STAGE_NAME, state: 'failed')
                }
                unstable {
                    updateGitlabCommitStatus(name: STAGE_NAME, state: 'failed')
                }
                success {
                    updateGitlabCommitStatus(name: STAGE_NAME, state: 'success')
                }
                aborted {
                    updateGitlabCommitStatus(name: STAGE_NAME, state: 'skipped')
                }
                always { 
                    deleteDir()                     // clean up workspace
                }
            }
        }
    ```
    * Embed Safety in the Jenkins Pipeline
    ```
        pipeline {
            agent any

        options {
            gitLabConnection('gitlab')
        }

        stages {
            stage("build") {
                agent { 
                    docker { 
                        image 'python:3.6'
                        args '-u root'
                    }
                }
                steps {
                    sh """
                    pip3 install --user virtualenv
                    python3 -m virtualenv env
                    . env/bin/activate
                    pip3 install -r requirements.txt
                    python3 manage.py check
                    """
                }
            }
            stage("test") {
                agent { 
                    docker { 
                        image 'python:3.6'
                        args '-u root'
                    }
                }
                steps {
                    sh """
                    pip3 install --user virtualenv
                    python3 -m virtualenv env
                    . env/bin/activate
                    pip3 install -r requirements.txt
                    python3 manage.py test taskManager
                    """
                }
            }
            stage("oast-frontend") {
                agent {
                    docker { 
                        image 'node:alpine3.10' 
                        args '-u root'
                    }
                }
                steps {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        sh """
                        npm install
                        npm install -g retire
                        retire --outputformat json --outputpath retirejs-report.json --severity high
                        """
                    }
                }
            }
            stage("oast") {
                steps {
                    sh "docker run -v \$(pwd):/src --rm hysnsec/safety check -r requirements.txt --json | tee oast-results.json"
                }
            }
            stage("integration") {
                steps {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        echo "This is an integration step."
                        sh "exit 1"
                    }
                }
            }
            stage("prod") {
                steps {
                    input "Deploy to production?"
                    echo "This is a deploy step."
                }
            }
        }
        post {
            failure {
                updateGitlabCommitStatus(name: STAGE_NAME, state: 'failed')
            }
            unstable {
                updateGitlabCommitStatus(name: STAGE_NAME, state: 'failed')
            }
            success {
                updateGitlabCommitStatus(name: STAGE_NAME, state: 'success')
            }
            aborted {
                updateGitlabCommitStatus(name: STAGE_NAME, state: 'skipped')
            }
            always { 
                deleteDir()                     // clean up workspace
            }
        }
    }
    ```
* Embed Safety Into Github Actions <a name="safetyGitHubActions"></a>
    * Create Github Action Workflow at .github/workflows/main.yaml
        ```
        name: Django                                  # workflow name

        on:
          push:
            branches:                                 # similar to "only" in GitLab
              - master

        jobs:
            build:
                runs-on: ubuntu-latest                    # similar to "image" in GitLab
                steps:
                    - uses: actions/checkout@v2

                    - name: Setup python
                        uses: actions/setup-python@v2
                        with:
                        python-version: '3.6'

                    - run: |
                        pip3 install --upgrade virtualenv
                        virtualenv env
                        source env/bin/activate
                        pip install -r requirements.txt
                        python manage.py check

            test:
                runs-on: ubuntu-latest
                needs: build
                steps:
                    - uses: actions/checkout@v2

                    - name: Setup python
                        uses: actions/setup-python@v2
                        with:
                        python-version: '3.6'

                    - run: |
                        pip3 install --upgrade virtualenv
                        virtualenv env
                        source env/bin/activate
                        pip install -r requirements.txt
                        python manage.py test taskManager

            integration:
                runs-on: ubuntu-latest
                needs: test
                steps:
                    - run: echo "This is an integration step"
                    - run: exit 1
                    continue-on-error: true

            prod:
                runs-on: ubuntu-latest
                needs: integration
                steps:
                    - run: echo "This is a deploy step."
        ```
* Sca Using RetireJS <a name="scaRetireJS"></a>
    * Download the source code
        * `git clone https://gitlab.practical-devsecops.training/pdso/django.nv webapp`
    * Install Node JS and NPM
        * `curl -sL https://deb.nodesource.com/setup_12.x | bash -`
        * `apt install nodejs -y`
    * Install RetireJS
        * `npm install -g retire`
    * Use RetireJS to find vulnerable libraries
        * `retire --outputformat json --outputpath retire_output.json`
    * Run npm install
        * `npm install`
    * Rerun RetireJS
    * Mark issue with False Positive
        * ```
        cat .retireignore.json

        [
            {
                "component": "jquery",
                "identifiers" : { "issue": "2432"},
                "justification" : "We dont call external resources with jQuery"
            },
            {
                "component": "jquery",
                "version" : "2.1.4",
                "justification" : "We dont call external resources with jQuery"
            },
            {
                "path" : "node_modules",
                "justification" : "The node modules are only used for building - client side dependencies are using bower"
            }
        ]
        ```
    * Rerun RetireJS
        * `retire --severity high --ignorefile .retireignore.json --outputformat json --outputpath retire_output.json`
* Embed RetireJS into CICD Pipeline <a name="retireJSCICD"></a>
* Embed RetireJS into Github Actions <a name="retireJSGitHubActions"></a>
* Sca Using Dependency Check <a name="scaDependency"></a>
* Embed Dependency Check into CICD Pipeline <a name="dependencyCICD"></a>
* Sca using Snyk <a name="scaSnyk"></a>
* Embed Snyk into CICD Pipeline <a name="snykCICD"></a>
* Embed Snyk into Github Actions <a name="snykGitHubActions"></a>
* Sca Using NPM Audit <a name="scaNPMAudit"></a>
* Sca Using AuditJS <a name="scaAuditJS"></a>
* Sca Using Bundler Audit <a name="scaBundlerAudit"></a>
* Sca Using Chelsea <a name="scaChelsea"></a>
---
## Static Analysis <a name="SA"></a>
---
---
## Dynamic Analysis <a name="DA"></a>
---
---
## Infrastructure As Code <a name="IaaC"></a>
---
* IaC Using Ansible to harden a production environment<a name="IaCAnsible"></a>
    * Ansible Introduction
        * Install Ansible and Ansible Lint
            * `pip3 install ansible==2.10.4 ansible-lint==4.3.7`
        * Create the inventory file
            ```
            cat > inventory.ini <<EOL

            # DevSecOps Studio Inventory
            [devsecops]
            devsecops-box-fUP1vG0E

            [prod]
            prod-fUP1vG0E

            EOL
            ```
        * Ensure SSH "Yes/No" prompt is now shown while running the ansible commands. Use ssh-keyscan to capture key signatures beforehand
            * `ssh-keyscan -t rsa prod-fUP1vG0E>> ~/.ssh/known_hosts`
            * `ssh-keyscan -t rsa devsecops-box-fUP1vG0E >> ~/.ssh/known_hosts`
            or
            * `ssh-keyscan -t rsa prod-fUP1vG0E devsecops-box-fUP1vG0E >> ~/.ssh/known_hosts`
        * Use ansible ad-hoc command to check production machine's uptime, install the ntp service, and check the bash version of all systems
            * `ansible -i inventory.ini prod -m shell -a "uptime"`
            * `ansible -i inventory.ini prod -m apt -a "name=ntp state=present"`
            * `ansible -i inventory.ini all -m command -a "bash --version"`
    * Exercise
        * Use inventory file (-i inventory.ini) and the following command to find the uptime.
        * Use the Ansible command module to find uptime.
            * `ansible -i inventory.ini all -m shell -a "uptime"`
    * Ansible Playbook
        * Create an Ansible playbook
            ```
            cat > playbook.yml <<EOL
            ---
            - name: Example playbook to install nginx
              hosts: prod
              remote_user: root
              become: yes
              gather_facts: no
              vars:
                state: present

              tasks:
              - name: ensure Nginx is at the latest version
                  apt:
                  name: nginx

            EOL
            ```
        * Run Ansible playbook against the prod machine
            `ansible-playbook -i inventory.ini playbook.yml`
            * Run the command the second time shows Nginx is at the latest version
            * Notice the change=0, why is it important? This verifies the system is setup in accordance with the defined Ansible playbook. Any changes indicates unauthorized changes.
    * Exercise
        * Create a new directory exercise-7.1 and create a file in it called playbook.yml
        * Download secfigo.terraform role using ansible-galaxy
            ```
                # Ansible Galaxy stores open source Ansible roles
            ```
            * `ansible-galaxy role --help`
            * `ansible-galaxy search terraform`
            * `ansible-galaxy install secfigo.terraform`
        * Execute the playbook to install the Terraform utility using ansible-playbook command
            * Inventory File
                ```
                [devsecops]
                devsecops-box-fUP1vG0E

                [prod]
                prod-fUP1vG0E
                ```
            * Ansible Playbook
                ```
                ---
                - name: Example playbook to install Terraform using ansible role.
                  hosts: prod
                  remote_user: root
                  become: yes

                  roles:
                    - secfigo.terraform
                ```
            * `ansible-playbook -i inventory.ini playbook.yml`
    * Ansible OS Hardening
        * [Reference](https://github.com/dev-sec/ansible-collection-hardening)
    * Exercise (Use Ansible to harden Ubuntu Server)
        * Create a new directory hardening and create a file in it called ansible-hardening.yml
            `mkdir ../hardening && cd ../hardening`
            `touch ansible-hardening.yml`
        * Download dev-sec.os-hardening role from ansible-galaxy
            `ansible-galaxy install dev-sec.os-hardening`
        * Execute the playbook to harden the Ubuntu production machine
            * Inventory File
                ```
                [devsecops]
                devsecops-box-fUP1vG0E

                [prod]
                prod-fUP1vG0E
                ```
            * Ansible Playbook
                ```
                ---
                - name: Example playbook to install Terraform using ansible role.
                  hosts: prod
                  remote_user: root
                  become: yes

                  roles:
                    - dev-sec.os-hardening
                ```
        * Optionally put this hardening job in the CI pipeline (Please refer to Exercise 6.0 for some inspiration)

* Ansible Adhoc Commands <a name="AnsibleAdHocCmd"></a>
* Ansible Playbook Basics <a name="AnsiblePlaybook"></a>
* Ansible Conditionals <a name="AnsibleConditional"></a>
* Deploy Container Using Ansible <a name="AnsibleContainer"></a>
* Audomated Hardening Using Ansible <a name="AnsibleHardening"></a>
* Terraform Linter Using Tflint <a name="Tflint"></a>
---
## Compliance As Code <a name="CaaC"></a>
---
---
## Vulnerability Management <a name="VM"></a>
---
---
## Summary <a name="Summary"></a>
---
---
## Tips <a name="tips"></a>
---
### 1. Module 01:
* allow_failure: true # allow the build to fail but don't mark it as such
* Press `F5` to refresh Windows and Linux OS pages
### 2. MOdule 02:
* create a file using cat
    * ```
        # cat > filename <<EOL

        Some text content

        EOL

        # cat filename
        Some text content

    ```