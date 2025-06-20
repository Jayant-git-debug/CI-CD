pipeline {
  agent { label 'master' }

  environment {
    MAVEN_HOST = 'jenkins@172.31.86.33'
    ANSIBLE_HOST = 'ansibleadmin@172.31.20.67'
    GIT_REPO = 'https://github.com/Jayant-git-debug/CI-CD.git'
    GIT_BRANCH = 'master'
    APP_DIR = '/home/maven/app'
    JAR_NAME = 'test-1.0-SNAPSHOT.jar'
    JAR_REMOTE_PATH = '/home/maven/app/target/test-1.0-SNAPSHOT.jar'
    LOCAL_JAR_PATH = 'builds/'
  }

  stages {
    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }

    stage('Trigger Maven Build on Maven Server') {
      steps {
        sshagent(['c55758a5-d427-462f-ae84-7786d7442c0c']){
         sh """
           ssh -o StrictHostKeyChecking=no ${MAVEN_HOST} '
             rm -rf ${APP_DIR} &&
             git clone -b ${GIT_BRANCH} ${GIT_REPO} ${APP_DIR} &&
             cd ${APP_DIR} &&
             mvn clean package -DskipTests
           '
        """
        }
      }
    }

    stage('Download .jar from Maven Server') {
      steps {
        sh """
        mkdir -p ${LOCAL_JAR_PATH}
        scp ${MAVEN_HOST}:${JAR_REMOTE_PATH} ${LOCAL_JAR_PATH}${JAR_NAME}
        """
      }
    }

    stage('Copy .jar to Ansible Server') {
      steps {
        sh """
        scp ${LOCAL_JAR_PATH}${JAR_NAME} ${ANSIBLE_HOST}:/home/ansibleadmin/deployments/${JAR_NAME}
        """
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        sh """
        ssh ${ANSIBLE_HOST} '~/.local/bin/ansible-playbook /home/ansible/playbooks/deploy.yml -e jar_file=/home/ansibleadmin/deployments/${JAR_NAME}'
        """
      }
    }
  }

  post {
    success {
      echo '✅ Build and deployment successful!'
    }
    failure {
      echo '❌ Failed — Check build or deployment logs.'
    }
  }
}
