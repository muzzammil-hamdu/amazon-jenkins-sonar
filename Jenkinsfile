pipeline {
    agent any
    environment {
        SONAR_NAME   = 'LocalSonarQube'
        VAULT_PASS_ID = 'vault-password-id'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/muzzammil-hamdu/amazon-jenkins-sonar.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_NAME}") {
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
                withCredentials([string(credentialsId: "${VAULT_PASS_ID}", variable: 'VAULT_PASSWORD')]) {
                    sh '''
                        echo $VAULT_PASSWORD > .vault_pass.txt
                        ansible-playbook -i ansible/inventories/production \
                                         ansible/playbooks/deploy-tomcat.yml \
                                         --vault-password-file .vault_pass.txt
                        rm -f .vault_pass.txt
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build + Sonar + Deploy Successful!"
        }
        failure {
            echo "❌ Pipeline Failed. Check logs."
        }
        always {
            cleanWs()
        }
    }
}

