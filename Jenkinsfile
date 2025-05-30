pipeline {
    agent any
    tools{
        jdk 'JDK21'
        maven 'Maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '0675d4ca-a79f-4e58-8047-10aa4109801a', poll: false, url: 'https://github.com/Saijayaram27/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
         stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Ekart '''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'd1a54ebc-430d-47ca-a844-688b3411c101', toolName: 'Docker') {

                        sh "docker build -t \"ekart:$BUILD_NUMBER\" -f docker/Dockerfile ."
                        sh "docker tag \"ekart:$BUILD_NUMBER\" \"saijayaram27/ekart:$BUILD_NUMBER\""
                        sh "docker push \"saijayaram27/ekart:$BUILD_NUMBER\""

                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'd1a54ebc-430d-47ca-a844-688b3411c101', toolName: 'Docker') {
                        sh "docker run -d --name shop-shop -p 8070:8070 \"saijayaram27/ekart:$BUILD_NUMBER\""
                    }
                }
            }
        }
    }
}
