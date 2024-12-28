pipeline {
    agent none
    stages {
        stage("build") {
            agent {
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Compiling worker app...'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage("test") {
            agent {
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Running Unit Tests on worker app...'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage("package") {
            when {
                branch 'master'
            }
            agent {
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Packaging worker app...'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage("docker-package") {
            agent any
            when {
                branch 'master'
            }
            steps {
                echo 'Packaging worker app with Docker...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker_hub_credentials') {
                        def workerImage = docker.build("11098078/vote:v${env.GIT_COMMIT}", "./vote")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                        workerImage.push("latest")
                    }
                }
            }
        }
        stage('Triggerdeployment') {
            agent any
            steps {
                script {
                    def gitCommit = env.GIT_COMMIT
                    echo "GIT_COMMIT: ${env.GIT_COMMIT}"
                    echo "Triggering deployment..."
                    build job: 'deployment', parameters: [string(name: 'DOCKERTAG', value: gitCommit)]
                }
            }
        }
    } // Closing 'stages' block
    post {
        always {
            echo 'Building multibranch pipeline for worker is completed...'
        }
    } // Closing 'post' block
} // Closing 'pipeline' block

