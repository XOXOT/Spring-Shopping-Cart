pipeline {
    agent  {
        label 'dind-agent'
    }
    stages {
        stage('Start') {
            agent any
            steps {
                slackSend (channel: '#test', color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }


        stage('Build image') {
            steps {
                script {
                    app = docker.build("terraform-tae/store")
                }
            }
        }
        
        stage("Push image to AR") {
            steps {
                script {
                    docker.withRegistry('https://asia.gcr.io', 'gcr:terraform-tae') {
                        app.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('K8S Manifest Update') {

            steps {

                git credentialsId: 'XOXOT',
                    url: 'https://github.com/XOXOT/argoCD_yaml.git',
                    branch: 'main'

                sh "sed -i 's/store:.*\$/store:${env.BUILD_NUMBER}/g' ./deployment/deploy_store.yaml"
                sh "git add deploy_store.yaml"
                sh "git commit -m '[UPDATE] store ${env.BUILD_NUMBER} image versioning'"

                withCredentials([gitUsernamePassword(credentialsId: 'XOXOT')]) {
                    sh "git push -u origin main"
                }
            }
            post {
                    failure {
                    echo 'Update failure !'
                    }
                    success {
                    echo 'Update success !'
                    }
            }
        }


    }             
    post {
        success {
            slackSend (channel: '#test', color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (channel: '#test', color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }

}
