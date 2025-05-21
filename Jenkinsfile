pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('clone code') {
            steps {
                git 'https://github.com/mohanjay112/tweet-trend.git'
            }
        }
    }
}
