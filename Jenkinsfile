pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9002 -Dsonar.login=e0e13b3f3934cae795d4b28213996d20beada150 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=\"**/.mvn/**, **/src/test/**, **/model/**, **/Application.java\""
                }  
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(20)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin2', path: '', url: 'http://localhost:8080/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test') {
                    git 'https://github.com/OmTsTM/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git 'https://github.com/OmTsTM/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin2', path: '', url: 'http://localhost:8080/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git 'https://github.com/omtstm/tasks-functional-tests'
                    bat 'mvn test'
                }
            }
        }
    }
}

