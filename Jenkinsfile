pipeline {
  agent any

  environment {
    MAVEN_HOST = 'jenkinadmin@13.201.133.43'
    ANSIBLE_HOST = 'jenkinadmin@13.235.71.158'
    GIT_REPO = 'https://github.com/Jayant-git-debug/CI-CD.git'
    GIT_BRANCH = 'master'
    APP_DIR = '/home/maven/app'
    JAR_NAME = 'your-app.jar'
    JAR_REMOTE_PATH = '/home/maven/app/target/your-app.jar'
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
        sh """
        ssh ${MAVEN_HOST} '
          rm -rf ${APP_DIR} &&
          git clone -b ${GIT_BRANCH} ${GIT_REPO} ${APP_DIR} &&
          cd ${APP_DIR} &&
          mvn clean package -DskipTests
        '
        """
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
        scp ${LOCAL_JAR_PATH}${JAR_NAME} ${ANSIBLE_HOST}:/home/ansible/deployments/${JAR_NAME}
        """
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        sh """
        ssh ${ANSIBLE_HOST} 'ansible-playbook /home/ansible/playbooks/deploy.yml -e jar_file=/home/ansible/deployments/${JAR_NAME}'
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
