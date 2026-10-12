pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M3'
    }
    environment {
        DOCKERHUB_CRED = credentials('dockeCredentials')
    }
    
    stages {
        // Github로 부터 소스코드 다운로드
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/weaa0927/spring-petclinic.git',
                branch: 'main'
            }
        }
        // Maven을 이용한 Build
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }
        // Docker Image 생성 
        stage ('Docker Image Create') {
            steps {
            echo 'Docker Image Create'
            sh '''
                docker build -t spring-petclinic:${BUILD_NUMBER} .
                docker tag spring-petclinic:${BUILD_NUMBER} weaa0927/spring-petclinic:latest
                '''
            }
        }
        
        // Docker hub Image Push

        stage ('Docker Image Push') {
            steps {
                echo 'Docker Image Push'
                // Docker Login
                sh 'echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin'

                // Docker Hub Push
                sh 'docker image push weaa0927/spring-petclinic:latest'
            }
            post {
                always {
                sh '''
                docker rmi -f spring-petclinic:${BUILD_NUMBER}
                docker rmi -f weaa0927/spring-petclinic:latest
                '''
                }
        }    
        }
        
        // SSH를 이용한 Docker 컨테이너 실행
        stage('SSH Publish') {
            steps {
                echo 'SSH Publish'
                sshPublisher(publishers: [sshPublisherDesc(configName: 'target', 
                transfers: [sshTransfer(cleanRemote: false, excludes: '', 
                execCommand: '''
                docker rm -f $(docker ps -aq)
                docker rmi -f $(docker ps -q)
                docker run -itd -p 80:8080 --name=spring-petclinic weaa0927/spring-petclinicL:latest
                ''', 
                execTimeout: 120000, 
                flatten: false, 
                makeEmptyDirs: false, 
                noDefaultExcludes: false, 
                patternSeparator: '[, ]+', 
                remoteDirectory: '', 
                remoteDirectorySDF: false, 
                removePrefix: 'target', 
                sourceFiles: 'target/*.jar')], 
                usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, 
                verbose: false)])
            }
        }
    }
}
