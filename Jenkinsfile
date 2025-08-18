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
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy WAR using Ansible') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'vm1-key', keyFileVariable: 'SSH_KEY_FILE', usernameVariable: 'SSH_USER')
                ]) {
                    sh '''
                        ansible-playbook \
                          -i ansible/inventories/production \
                          ansible/playbooks/deploy-tomcat.yml \
                          --private-key $SSH_KEY_FILE \
                          -u $SSH_USER
                    '''
                }
            }
        }
    }
}

