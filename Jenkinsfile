pipeline {
    agent  {
        label 'dind-agent'
    }
    stages {
        stage('Build image') {
            steps {
                script {
                    app = docker.build("terraform-tae/store")
                }
            }
        }
        
        stage("Push image to gcr") {
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
                    branch: 'master'

                sh "sed -i 's/store:.*\$/store:${env.BUILD_NUMBER}/g' deploy_store.yaml"
                sh "git add deploy_store.yaml"
                sh "git commit -m '[UPDATE] store ${env.BUILD_NUMBER} image versioning'"

                withCredentials([gitUsernamePassword(credentialsId: 'XOXOT')]) {
                    sh "git push -u origin master"
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

}
