pipeline {
    agent any
    environment {
        SONAR_NAME = 'LocalSonarQube'
        VAULT_PASS_ID = 'vault-password-id'
        SSH_KEY_ID   = 'vm1-key'   // ID of the SSH key credential you created
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/muzzammil-hamdu/amazon-jenkins-sonar.git', branch: 'main'
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
                withCredentials([
                    string(credentialsId: "${VAULT_PASS_ID}", variable: 'VAULT_PASSWORD'),
                    sshUserPrivateKey(credentialsId: "${SSH_KEY_ID}", keyFileVariable: 'SSH_KEY_FILE', usernameVariable: 'SSH_USER')
                ]) {
                    sh '''
                        # Save vault password
                        echo $VAULT_PASSWORD > ansible/vault/.vault_pass.txt

                        # Run ansible with the private key
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

