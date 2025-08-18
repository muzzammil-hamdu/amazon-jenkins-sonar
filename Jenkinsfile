pipeline {
    agent any
    environment {
        SONAR_NAME   = 'LocalSonarQube'
        VAULT_PASS_ID = 'vault-password-id'   // Jenkins string credential (vault password)
        SSH_KEY_ID    = 'vm1-key'             // Jenkins SSH private key credential
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/muzzammil-hamdu/amazon-jenkins-sonar.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
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
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([
                    string(credentialsId: "${VAULT_PASS_ID}", variable: 'VAULT_PASSWORD'),
                    sshUserPrivateKey(credentialsId: "${SSH_KEY_ID}", keyFileVariable: 'SSH_KEY_FILE', usernameVariable: 'SSH_USER')
                ]) {
                    sh '''
                        echo $VAULT_PASSWORD > ansible/vault/.vault_pass.txt

                        ansible-playbook \
                          -i ansible/inventories/production \
                          ansible/playbooks/deploy-tomcat.yml \
                          --vault-password-file ansible/vault/.vault_pass.txt \
                          --key-file $SSH_KEY_FILE \
                          -u $SSH_USER
                    '''
                }
            }
        }
    }
}

