Executing Selenium tests with pytest using Python.

## Contents
* [Structure setup](#structure-setup)
  + [Start Jenkin、Selenium dynamic grid、nginx(for allure) with docker compose](#start-jenkin-selenium-dynamic-grid-nginx-for-allure-with-docker-compose)
  + [Download Allure Plugin](#download-allure-plugin)
  + [Install the docker client if you want to execute the Docker command in Docker Jenkins](#install-the-docker-client-if-you-want-to-execute-the-docker-command-in-docker-jenkins)
  + [Unit test source code](#unit-test-source-code)
  + [How to use Jenkins to run test case](#how-to-use-jenkins-to-run-test-case)
    - [For Freestyle](#for-freestyle)
    - [Jenkins test case](#jenkins-test-case)
  + [Allow others to view the Allure report](#allow-others-to-view-the-allure-report)
  + [Feature](#feature)
## Structure setup
### Start Jenkin、Selenium dynamic grid、nginx(for allure) with docker compose
Running docker-compose in the background

    docker compose -f docker-compose-v3-dynamic-grid.yml up -d

Stops containers and removes containers, networks, volumes

    docker compose -f docker-compose-v3-dynamic-grid.yml down

check http://localhost:8080/ if Jenkins is started   
check http://localhost:4444/ if Selenium Grid is started

View jenkins password

    cat /var/jenkins_home/secrets/initialAdminPassword

click install suggested plugins

![Action screenshot](docs/imgs/plugins.png)

### Download Allure Plugin

From your Jenkins dashboard navigate to `"管理 Jenkins"` -> `"管理外掛程式"` and select the Available tab. Locate this plugin by searching for `allure-jenkins-plugin`.Select the option `"Download now and Install after the restart"` button. In which plugin is installed after the restart.

next step,click `"全域工具設定"` and set the `Allure Commandline` in Jenkins as shown below.

![Action screenshot](docs/imgs/allure.png)

### Install the docker client if you want to execute the Docker command in Docker Jenkins

execute the bash command in the docker Jenkins container

    docker exec -it jenkins-lts-jdk11 /bin/bash

Install the docker client package

    apt-get update && apt-get -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common && curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable" && apt-get update && apt-get -y install docker-ce

exit jenkins bash

    exit


## Unit test source code

I put the test source code and dockerfile on gitea.

![Action screenshot](docs/imgs/gitea.png)

## How to use Jenkins to run test case

Two methods, Freestyle project(easy to use) or Pipeline.

### For Freestyle

click `"新增作業"`

![](docs/imgs/create_task.png)

Enter the project name, select the Freestyle project and click `"確定"`

![](docs/imgs/jenkins_project.png)

click `"git"` and paste github repo.If you do not want to keep previous data, check `"Delete workspace before build starts"`.

![](docs/imgs/git.png)

Scroll down to find the `"Build Steps"` section, click on `"新增建置步驟"`, then select `"執行 Shell"`. In the shell, enter the command. An example is shown below.

``` bash
 df -h
```

![](docs/imgs/build_step.png)

click `"新增建置後動作"`, click `"Allure Report"` and save.

![](docs/imgs/after_build.png)

Click `"馬上建置"`, and after the execution is complete, you can click on `"Allure Report"` to view the report.

![](docs/imgs/after_test.png)

### Jenkins test case

When you have finished writing your test cases, you don't need to build a new image. The reason I keep rebuilding the image is that the test cases are still being modified, and rebuilding the image will copy the updated test cases to the Docker container.

* login test Jenkins in Build Steps
``` bash
git checkout blocka
docker build -t python3.8.16-alpine3.17:Selenium .
docker run --network="host" --rm -v /home/blocka/selenium_grid/jenkins_data/workspace/GDMS_login_test/allure-results:/test/allure-results -v /home/blocka/selenium_grid/jenkins_data/assets:/test/assets python3.8.16-alpine3.17:Selenium pytest --browser chrome -n auto login_test/ --alluredir allure-results
docker run --network="host" --rm -v /home/blocka/selenium_grid/jenkins_data/workspace/GDMS_login_test/allure-results:/test/allure-results -v /home/blocka/selenium_grid/jenkins_data/assets:/test/assets python3.8.16-alpine3.17:Selenium pytest --browser firefox -n auto login_test/ --alluredir allure-results
docker run --network="host" --rm -v /home/blocka/selenium_grid/jenkins_data/workspace/GDMS_login_test/allure-results:/test/allure-results -v /home/blocka/selenium_grid/jenkins_data/assets:/test/assets python3.8.16-alpine3.17:Selenium pytest --browser edge -n auto login_test/ --alluredir allure-results
docker rmi python3.8.16-alpine3.17:Selenium
```
* API test Jenkins in Build Steps
``` bash
git checkout API_test
docker build -t python3.8.16-alpine3.17:API .
docker run --network="host" --rm -v /home/blocka/selenium_grid/jenkins_data/workspace/GDMS_API_test/allure-results:/test/allure-results python3.8.16-alpine3.17:API
docker rmi python3.8.16-alpine3.17:API
```
* UI/UX test Jenkins in Build Steps
``` bash
git checkout UI_UX_test
docker build -t python3.8.16-alpine3.17:UI .
docker run --network="host" --rm -v /home/blocka/selenium_grid/jenkins_data/workspace/GDMS_UI_UX_test:/test python3.8.16-alpine3.17:UI python3 login.py
docker run --network="host" --rm -v /home/blocka/selenium_grid/jenkins_data/workspace/GDMS_UI_UX_test/allure-results:/test/allure-results -v /home/blocka/selenium_grid/jenkins_data/workspace/GDMS_UI_UX_test/login_cookie:/test/login_cookie -v /home/blocka/selenium_grid/jenkins_data/assets:/test/assets python3.8.16-alpine3.17:UI pytest --browser Chrome -n auto test/ --alluredir allure-results
docker rmi python3.8.16-alpine3.17:UI
```

## Allow others to view the Allure report

Enter the command `"hostname -I"` in the Ubuntu command line,and You can see it by entering the address of the first IPv4 in the browser.

![](docs/imgs/allure_report.png)

After that, click on the project name and then click on 'allure-report/' to complete the process.

## Feature
- video record
- test environment in allure environment
- View Historical Test Charts

