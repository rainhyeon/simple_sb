pipeline {
    agent any

    tools {
        maven 'my_maven' 
    }
    environment {
        GITNAME = 'rainhyoeon'            
        GITEMAIL = 'iiilee0907@naver.com' 
        GITWEBADD = 'https://github.com/rainhyeon/simple_sb.git'
        GITSSHADD = 'git@github.com:rainhyeon/dep.git'
        GITCREDENTIAL = 'git_cre'
        
        DOCKERHUB = 'iiilee0907/spring'
        DOCKERHUBCREDENTIAL = 'docker_cre'
    }

    stages {
        // github의 자격증명 및 깃 클론
        stage('Checkout Github') {
            steps {
                // 깃 클론할 곳이 main 브랜치이다
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [],
                // 인증을 GITCREDENTIAL을 통해 할 것이고, GITWEBADD 경로에서 할거다
                userRemoteConfigs: [[credentialsId: GITCREDENTIAL, url: GITWEBADD]]])

            }
            post {
                failure {
                    sh "echo clone failed"
                }
                success {
                    sh "echo clone success"
                }
            }
        }
        // maven 빌드
        stage('springboot app build') {
            steps {
                sh "mvn clean package"
            }
            post {
                failure {
                    sh "echo mvn packaging fail"
                }
                success {
                    sh "echo mvn packaging success"
                }
            }
        }

        // jar 파일을 가지고 도커 빌드드
        stage('docker image build') {
            steps {
                sh "docker build -t ${DOCKERHUB}:${currentBuild.number} ."
                sh "docker build -t ${DOCKERHUB}:latest ."
                // currentBuild.number 젠킨스가 제공하는 빌드넘버 변수
                // oolralra/fast:<빌드넘버> 와 같은 이미지가 만들어질 예정.
               
            }
            post {
                failure {
                    sh "echo image build failed"
                }
                success {
                    sh "echo image build success"
                }
            }
        }
        stage('docker image push') {
            steps {
                withDockerRegistry(credentialsId: DOCKERHUBCREDENTIAL, url: '') {
                    sh "docker push ${DOCKERHUB}:${currentBuild.number}"
                    sh "docker push ${DOCKERHUB}:latest"
                }
            }
            post {
                failure {
                    sh "docker image rm -f ${DOCKERHUB}:${currentBuild.number}"
                    sh "docker image rm -f ${DOCKERHUB}:latest"
                    sh "echo push failed"
                    // 성공하든 실패하든 로컬에 있는 도커이미지는 삭제
                }
                success {
                    sh "docker image rm -f ${DOCKERHUB}:${currentBuild.number}"
                    sh "docker image rm -f ${DOCKERHUB}:latest"
                    sh "echo push success"
                    // 성공하든 실패하든 로컬에 있는 도커이미지는 삭제
                }
            }
        }
        stage('EKS manifest file update') {
            steps {
                git credentialsId: GITCREDENTIAL, url: GITSSHADD, branch: 'main'
                sh "git config --global user.email ${GITEMAIL}"
                sh "git config --global user.name ${GITNAME}"
                sh "sed -i 's@${DOCKERHUB}:.*@${DOCKERHUB}:${currentBuild.number}@g' fast.yml"

                sh "git add ."
                sh "git branch -M main"
                sh "git commit -m 'fixed tag ${currentBuild.number}'"
                sh "git remote remove origin"
                sh "git remote add origin ${GITSSHADD}"
                sh "git push origin main"
            }
            post {
                failure {
                    sh "echo manifest update failed"
                }
                success {
                    sh "echo manifest update success"
                }
            }
        }
       

    }
}
