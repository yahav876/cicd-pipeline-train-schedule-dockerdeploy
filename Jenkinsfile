pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - \
                add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
                $(lsb_release -cs) \
                stable"
                apt-get update -y
                apt install docker -y
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("yahav/train")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push the Docker image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to prod') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to prod?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull willbla/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker stop train-schedule\""
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker rm train-schedule\""
                    } catch (err) {
                        echo: 'caught error: $err'
                    }
                        sh "sshpass -p '$USERPASS -v shh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d yahav/train:${env.BUILD_NUMBER}\""
                  }
                }
            }
        }
    }
}
