pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
        registry = 'https://techjay.jfrog.io'
        imageName = 'techjay.jfrog.io/valaxy-docker-local/ttrend'
        version = '2.1.2'
    }

    stages {

        stage('Build') {
            steps {
                echo "--------------- Build Started -----------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "--------------- Build Completed ----------------"
            }
        }

        stage('SonarQube Analysis') {
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
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
			def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            echo "Pipeline quality gate failure: ${qg.status}, continuing the pipeline."
                        }
                    }
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer(
                        url: "${registry}/artifactory",
                        credentialsId: "artifact-cred"
                    )
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "libs-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": [ "*.sha1", "*.md5" ]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }

        stage("Docker Build") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    def fullImage = "${imageName}:${version}"
                    app = docker.build(fullImage)
                    echo '<--------------- Docker Build Ended --------------->'
                }
            }
        }

        stage("Docker Publish") {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'
                    docker.withRegistry("${registry}", 'artifact-cred') {
                        app.push()
                    }
                    echo '<--------------- Docker Publish Ended --------------->'
                }
            }
        }

        stage ("Deploy"){
            steps {
                script {
                    sh './deploy.sh'
                }
            }
        }
    }
}
