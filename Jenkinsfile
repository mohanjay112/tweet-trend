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
        stage('Build & Test') {
            steps {
                echo "---------------Build & Test started-----------------"
                // Run build with tests (skip if you want, but then no test report)
                sh 'mvn clean verify'
                echo "-------------- Build & Test completed---------------"
            }
        }

        stage('Generate Surefire Report') {
            steps {
                echo "---------Generating unit test report-----------"
                sh 'mvn surefire-report:report'
                echo "------------Unit test report generated-----------"
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
    }
}
