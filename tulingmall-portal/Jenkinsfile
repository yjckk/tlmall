pipeline {
    //指定任务在哪个集群节点中执行
    agent any

    //声明全局变量,方便后面使用
    environment{
        harborUser ='admin'
        harborPasswd='Harbor12345'
        harborAddress='120.77.171.208:80'
        harborRepo='repo'
    }

    // 存放所有任务的合集
    stages {
        stage('拉取Git仓库代码') {
            steps {
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '0516bcb8-285f-4c50-a674-f2bfa040bf00', url: 'http://47.106.85.182:8929/gitlab-instance-0dae4b1a/demo.git']]])
            }
        }
        stage('通过maven构建项目'){
            steps{
               sh '/var/jenkins_home/maven/bin/mvn clean package -DskipTests'
            }
        }

        stage('通过Docker制作自定义镜像') {
            steps {
              sh '''mv ./target/*.jar ./docker
docker build -t ${JOB_NAME}:latest ./docker/'''
            }
        }

        stage('将自定义镜像推送到Harbor') {
            steps {
               sh '''docker login -u ${harborUser} -p ${harborPasswd} ${harborAddress}
                        docker tag ${JOB_NAME}:latest ${harborAddress}/${harborRepo}/${JOB_NAME}:latest
                        docker push  ${harborAddress}/${harborRepo}/${JOB_NAME}:latest'''
            }
        }

        stage('将yml传到k8smaster') {
            steps {
           sshPublisher(publishers: [sshPublisherDesc(configName: 'k8s', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'pipeline.yml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
         stage('远程执行k8s-master的kubectl的命令') {
            steps {
          sh '''ssh root@120.77.85.235 kubectl apply -f /usr/local/k8s/pipeline.yml
                ssh root@120.77.85.235 kubectl rollout restart deployment pipeline -n test'''
            }
        }
    }
}
