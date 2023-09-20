pipeline{
    agent any
    tools {
        maven 'maven'
    }
     environment {
        DOCKER_REGISTRY = 'sophak12'
        IMAGE_NAME = 'devops_spring'
        CONTAINER_NAME = 'my-container' // Specify the name of your container
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
                sh 'mvn package'
            }
        }
        stage ('Test'){
            steps {
                echo "Jes tae tes leng tov :)"
                sh 'mvn test'
            }

        }
        stage('Check for Existing Container') {
            steps {
                script {
                    def containerId = sh(script: "docker ps -a --filter name=${env.CONTAINER_NAME} -q", returnStdout: true).trim()
                    sh "echo containerId is ${containerId}" 
                    if (containerId) {
                        sh "docker stop ${containerId}"
                        sh "docker rm ${containerId}"
                    } else {
                        sh "echo No existing container to remove"
                    }
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    def buildNumber = currentBuild.number
                    def imageTag = "${IMAGE_NAME}:${buildNumber}"
                    sh "docker build -t ${DOCKER_REGISTRY}/${imageTag} ."

                    withCredentials([usernamePassword(credentialsId: 'dockerhub',
                            passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "echo \$PASS | docker login -u \$USER --password-stdin"
                        sh "docker push ${DOCKER_REGISTRY}/${imageTag}"
                    }
                }
            }
        }
        stage('Trigger ManifestUpdate') {
            steps {
                    build job: 'intergrate_argocd_job1', parameters: [string(name: 'BUILDTAG', value: env.BUILD_NUMBER)]
            }
        }
    }
}
// hi I rename Repository name