pipeline {
    agent none
    stages {
        stage('Check yaml syntax') {
            agent { docker { image 'sdesbure/yamllint' } }
            steps {
                sh 'yamllint --version'
                sh 'yamllint \${WORKSPACE}'
            }
        }
        stage('Check markdown syntax') {
            agent { docker { image 'ruby:alpine' } }
            steps {
                sh 'apk --no-cache add git'
                sh 'gem install mdl'
                sh 'mdl --version'
                sh 'mdl --style all --warnings --git-recurse \${WORKSPACE}'
            }
        }
        stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
            }
        }
        stage('Test and deploy the application') {
            environment {
                SUDOPASS = credentials('sudopass')
                JENKINS_PRIVATE_KEY = credentials('jenkins_container_private_key')
            }
            agent { 
                docker { 
                    image 'registry.gitlab.com/robconnolly/docker-ansible:latest' 
                    args '-v /root/.ssh:/root/ssh_copy:rw'
            } }
            stages {
               stage("Deploy app in production") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh '''
                       cp -r /root/ssh_copy/jenkins_vm_key /tmp/jenkins_key.txt
                       chmod 600 /tmp/jenkins_key.txt
                       apt-get update
                       apt-get install -y sshpass
                       ansible-playbook  -i hosts.yml --vault-password-file vault.key  --extra-vars "ansible_sudo_pass=$SUDOPASS" deploy.yml
                       '''
                   }
               } 
            }
          }
      }
    }
