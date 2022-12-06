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

                sh "sed -i 's/store:.*\$/store:${env.BUILD_NUMBER}/g' deployment/deploy_store.yaml"
                sh "git add deployment/deploy_store.yaml"
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


}
