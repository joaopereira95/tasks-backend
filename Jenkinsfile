pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
         stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=20db424d2aee044472e3b384e2d3d492908ad3a0 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/mvn/**,**/src/test/**,**/model/**,**Application.java"
                 }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(10)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/joaopereira95/tasks-api-test.git'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {            
            steps {
                dir('frontend') {
                    git credentialsId: 'github_login', url: 'https://github.com/joaopereira95/tasks-frontend.git'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'    
                }
            }
        }
        /** stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/joaopereira95/tasks-functional-tests.git'
                    sh 'mvn test'
                }
            }
       }
       **/ 
        stage ('Deploy Prod') {
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        
        /** stage ('Health Check') {
            steps {
                sleep(10)
                dir('functional-test') {
                    sh 'mvn verify -Dskip.surefile.tests'
                }
            }
        }
        **/
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefile-reports/*.xml, APITest/target/surefire-reports/*.xml'
        }
    }
}