# python_jenkins_docker


## 整合 pytest，jenkins 和 docker 构建自动化测试。

### 准备工作
[https://github.com/pluckhuang/python_jenkins_docker](https://github.com/pluckhuang/python_jenkins_docker)
文件目录结构：
```
- .
├── Dockerfile
├── JenkinsDockerfile
├── pytest.ini
├── requirements.txt
├── src
│   ├── __init__.py
│   └── calculator.py
└── test_addition.py
```

### Dockerfile
- 执行 pytest 的执行环境，这里用的是 python 3.6-slim 镜像。
### JenkinsDockerfile
- jenkins 执行镜像
### src是源文件目录，test_*是测试文件

### 整体执行流程

- 本地构建 jenkins 镜像：docker build -t jenkins-docker-image -f JenkinsDockerfile .
- 建立本地容器执行环境：docker run -d -p 8080:8080 --name jenkins-docker-container -v /var/run/docker.sock:/var/run/docker.sock ~/jenkins_home:/var/jenkins_home jenkins-docker-image
- check admin pwd: docker exec -it jenkins-docker-container cat var/jenkins_home/secrets/initialAdminPassword
- jenkins下配置 github project、source code manger-git
- 添加构建步骤 -> 执行 shell：
    ```shell
    IMAGE_NAME="test-image"
    CONTAINER_NAME="test-container"
    echo "Check current working directory"
    pwd
    echo "Build docker image and run container"
    docker build -t $IMAGE_NAME .
    docker run -d --name $CONTAINER_NAME $IMAGE_NAME
    echo "Copy result.xml into Jenkins container"
    rm -rf reports; mkdir reports
    docker cp $CONTAINER_NAME:/python-test-calculator/reports/result.xml reports/
    echo "Cleanup"
    docker stop $CONTAINER_NAME
    docker rm $CONTAINER_NAME
    docker rmi $IMAGE_NAME
    ```
- 添加 Post-Build Actions, publish junit test result report:
    ```
    reports/result.xml
    ```
    
- 构建完成
   
