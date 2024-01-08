pipeline {
    environment {
        registry = "777777777777777/justtestingsmth"
        registryCredential = 'dockerhub_id'
        dockerImage = "$registry:$BUILD_NUMBER"
    }

    agent {
        label 'slave1'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Cloning our Git') {
            steps {
                sh 'git clone git@github.com:macedonsky777/test_node_step_app.git'
            }
        }

        stage('Building our image') {
            steps {
                script {
                    sh "cd test_node_step_app && docker build -t $dockerImage . && cd .. && rm -rf test_node_step_app"
                }
            }
        }

        stage('Test app') {
            steps {
                script {
                    def dockerRunExitCode = sh(script: "docker run -i -p 3000:80 --rm $dockerImage test", returnStatus: true)

                    if (dockerRunExitCode == 0) {
                        echo 'Tests passed. Proceeding to push container to registry'
                    } else {
                        error 'Tests failed. Stopping the pipeline.'
                    }
                }
            }
        }

        stage('Push to Registry') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dockerhub_id', url: 'https://index.docker.io/v1/']) {
                        sh "docker push $dockerImage"
                    }
                }
            }
        }

        stage('Cleaning up') {
            steps {
                sh "docker rmi $dockerImage"
            }
        }
    }
}

