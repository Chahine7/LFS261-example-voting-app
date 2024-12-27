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
                        def workerImage = docker.build("11098078/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                        workerImage.push("latest")
                    }
                }
            }
        }
        stage('Triggerdeployment') {
            agent any
            environment {
                def GIT_COMMIT = "${env.GIT_COMMIT}"
            }
            steps {
                echo "${GIT_COMMIT}"
                echo "triggering deployment"
                // Passing variables to job deployment run by vote-deploy repository Jenkinsfile
                build job: 'deployment', parameters: [string(name: 'DOCKERTAG', value: GIT_COMMIT)]
            }
        }
        stage('deploy to dev') {
            agent any 
            when {
                branch 'master'
            }
            steps {
                echo 'Deploy instavote app with docker compose'
                sh 'docker compose up -d'
            }
        }
    }
    post {
        always {
            echo 'Building multibranch pipeline for worker is completed...'
        }
    }
}

