pipeline {
    agent any
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
                withSonarQubeEnv('LocalSonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 25, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy WAR to Tomcat') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'vm1-key', keyFileVariable: 'SSH_KEY_FILE', usernameVariable: 'SSH_USER'),
                    string(credentialsId: 'vault-password-id', variable: 'VAULT_PASSWORD')
                ]) {
                    sh '''
                        echo $VAULT_PASSWORD > ansible/vault/.vault_pass.txt
                        ansible-playbook \
                          -i ansible/inventories/production \
                          ansible/playbooks/deploy-tomcat.yml \
                          --private-key $SSH_KEY_FILE \
                          -u $SSH_USER \
                          --vault-password-file ansible/vault/.vault_pass.txt
                    '''
                }
            }
        }
    }
}

