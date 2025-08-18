pipeline {
    agent any
    environment {
        SONAR_NAME = 'LocalSonarQube'
        VAULT_PASS_ID = 'vault-password-id'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/muzzammil-hamdu/amazon-jenkins-sonar.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("$SONAR_NAME") {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                withCredentials([string(credentialsId: "$VAULT_PASS_ID", variable: 'VAULT_PASSWORD')]) {
                    sh '''
                        echo $VAULT_PASSWORD > ansible/vault/.vault_pass.txt
                        ansible-playbook -i ansible/inventories/production ansible/playbooks/deploy-tomcat.yml --vault-password-file ansible/vault/.vault_pass.txt
                    '''
                }
            }
        }
    }
}

