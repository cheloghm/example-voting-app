pipeline{
    
    agent none

    stages{
        stage('worker build'){
            agent{
                docker{
                    image 'maven:3.8.6'
                    args '-v $HOME/.m2/root/.m2'
                }
            }
            when{
                changeset "**/worker/**"
            }
            steps{
                echo 'Compiling worker app'
                dir('worker'){
                    sh 'mvn compile'
                }
            }
        }
        stage('worker test'){
            agent{
                docker{
                    image 'maven:3.8.6'
                    args '-v $HOME/.m2/root/.m2'
                }
            }
            when{
                changeset "**/worker/**"
            }
            steps{
                echo 'Running unit Tests on worker app'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker package'){
            agent{
                docker{
                    image 'maven:3.8.6'
                    args '-v $HOME/.m2/root/.m2'
                }
            }
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            steps{
                echo 'Packaging worker app'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker-docker-package'){
            agent any
            when{
                changeset "**/worker/**"
                branch 'master'
            }
            steps{
                echo 'Packaging worker app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                        def workerImage = docker.build("cheloghm/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("latest")
                    }
                }
            }
        }

        stage('vote build'){
            agent{
                docker{
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when{
                changeset "**/vote/**"
            }
            steps{
                echo 'Compiling vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote test'){
            agent{
                docker{
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when{
                changeset "**/vote/**"
            }
            steps{
                echo 'Running unit Tests on vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }
        stage('vote-docker-package'){
            agent any
            when{
                changeset "**/vote/**"
                branch 'master'
            }
            steps{
                echo 'Packaging vote app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                        def voteImage = docker.build("cheloghm/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("latest")
                    }
                }
            }
        }

        stage('result build'){
            agent{
                docker{
                    nodejs 'NodeJS 8.9.0'
                }
            }
            when{
                changeset "**/result/**"
            }
            steps{
                echo 'Compiling result app'
                dir('result'){
                    sh 'npm install'
                }
            }
        }
        stage('result test'){
            agent{
                docker{
                    nodejs 'NodeJS 8.9.0'
                }
            }
            when{
                changeset "**/result/**"
            }
            steps{
                echo 'Running unit Tests on result app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('result-docker-package'){
            agent any
            when{
                changeset "**/result/**"
                branch 'master'
            }
            steps{
                echo 'Packaging result app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                        def resultImage = docker.build("cheloghm/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        resultImage.push("latest")
                    }
                }
            }
        }
    }

    post{
        always{
            echo 'Pipeline for instavote app is complete...'
        }
        failure{
            slackSend (channel: "instavote-ci", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success{
            echo 'Pipeline for worker is complete...'
            slackSend (channel: "instavote-ci", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}