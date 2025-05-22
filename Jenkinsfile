pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
    }

    stages {
        stage('Build') {
            steps {
                echo "---------------Build started-----------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "-------------- Build completed---------------"
            }
        }    

        stage('SonarQube analysis') {
            environment { 
                scannerHome = tool 'valxy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('valxy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
