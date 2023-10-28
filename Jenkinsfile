pipeline {
    agent any
    stages {
        stage('SCM Checkout'){
            steps {
            git branch: 'main', credentialsId: 'jen-cred', url: 'https://github.com/st-naresh/sub-micro.git'
                sh 'ls'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                parallel (
                    'node application': {
                        script {
                            dir('cart-microservice-nodejs') {
                                def scannerHome = tool 'sonarscanner4'
                                 withSonarQubeEnv('sonar-pro') {
                                     sh "${scannerHome}/bin/sonar-scanner"
                                 }
                            }
                            dir('ui-web-app-reactjs') {
                                def scannerHome = tool 'sonarscanner4';
                                withSonarQubeEnv('sonar-pro') {
                                    sh "${scannerHome}/bin/sonar-scanner"
                                }
                            }

                        }
                    },
                    'spring boot application': {
                        script {
                            dir('offers-microservice-spring-boot') {
                                def mvn = tool 'maven3';
                                withSonarQubeEnv('sonar-pro') {
                                    sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=offers-spring-boot -Dsonar.projectName=offers-spring-boot"
                                }
                            }
                            dir('shoes-microservice-spring-boot') {
                                def mvn = tool 'maven3';
                                withSonarQubeEnv('sonar-pro') {
                                    sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=shoe-spring-boot -Dsonar.projectName=shoes-spring-boot"
                                }
                            }
                        }
                    },
                    'python app': {
                        script{
                            dir('wishlist-microservice-python') {
                                def scannerHome = tool 'sonarscanner4';
                                withSonarQubeEnv('sonar-pro') {
                                    sh """/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonarscanner4/bin/sonar-scanner \
                                    -D sonar.projectVersion=1.0-SNAPSHOT \
                                    -D sonar.sources=. \
                                    -D sonar.login=admin \
                                    -D sonar.password=arun \
                                    -D sonar.projectKey=project \
                                    -D sonar.projectName=wishlist-py \
                                    -D sonar.inclusions=index.py \
                                    -D sonar.sourceEncoding=UTF-8 \
                                    -D sonar.language=python \
                                    -D sonar.host.url=http://35.154.56.192:9000/"""
                                }
                            }
                        }
                    }
                )
            }
        }
        stage ('Build Docker Image and push'){
            steps {
                parallel (
                    'docker login': {
                        withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                            sh "docker login -u devopsbyarun -p ${dockerPassword}"
                        }
                    },
                    'ui-web-app-reactjs': {
                        dir('ui-web-app-reactjs'){
                            sh """
                            docker images
                            docker build -t devopsbyarun/ui:v10 .
                            docker push devopsbyarun/ui:v10
                            docker rmi devopsbyarun/ui:v10
                            """
                        }
                    },
                    'offers-microservice-spring-boot': {
                        dir('offers-microservice-spring-boot'){
                            sh """
                            docker build -t devopsbyarun/spring:v2 .
                            docker push devopsbyarun/spring:v2
                            docker rmi devopsbyarun/spring:v2
                            """
                        }
                    },
                    'shoes-microservice-spring-boot': {
                        dir('shoes-microservice-spring-boot'){
                            sh """
                            docker build -t devopsbyarun/spring:v4 .
                            docker push devopsbyarun/spring:v4
                            docker rmi devopsbyarun/spring:v4
                            """
                        }
                    },
                    'cart-microservice-nodejs': {
                        dir('cart-microservice-nodejs'){
                            sh """
                            docker build -t devopsbyarun/ui:v3 .
                            docker push devopsbyarun/ui:v3
                            docker rmi devopsbyarun/ui:v3
                            """
                        }
                    },
                    'wishlist-microservice-python': {
                        dir('wishlist-microservice-python'){
                            sh """
                            docker build -t devopsbyarun/python:v2 .
                            docker push devopsbyarun/python:v2
                            docker rmi devopsbyarun/python:v2
                            """
                        }
                    }
                )
            }
        }
        stage ('Deploy on k8s'){
            steps{
                parallel (
                    'deploy on k8s': {
                        script {
                            withKubeCredentials(kubectlCredentials: [[ credentialsId: 'kubernetes', namespace: 'ms' ]]) {
                                sh 'kubectl get ns' 
                                sh 'kubectl apply -f kubernetes/yamlfile'
                            }
                        }
                    }
                )
            }
        }
    }
}