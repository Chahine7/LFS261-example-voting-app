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
    stage('Sonarqube') {
      agent any
      when{
        branch 'master'
      }
      // tools {
       // jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
     // }

      environment{
        sonarpath = tool 'SonarScanner'
      }

      steps {
            echo 'Running Sonarqube Analysis..'
            withSonarQubeEnv('sonar-instavote') {
              sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
            }
      }
    }


    stage("Quality Gate") {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
                waitForQualityGate abortPipeline: true
            }
        }
    }
stage('deploy to dev') {
agent any 
when {

branch 'master'

}
steps {
echo'Deploy instavote app with docker compose' 
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

