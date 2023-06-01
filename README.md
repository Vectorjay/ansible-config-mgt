# ansible-config-mgt
## We are at it again!!
#### Jenkins pipeline sytax #####
=========================================
pipeline {
  agent any

  stages {
    stage('Initial cleanup') {
      steps {
        dir("${WORKSPACE}") {
          deleteDir()
        }
      }
    }

    stage('Builds') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }

    stage('Package') {
      steps {
        script {
          sh 'echo "Packaging Stage"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Deploying Stage"'
        }
      }
    }

    stage('Clean up') {
      steps {
        cleanWs()
      }
    }
  }
}

====================================================================
pipeline {
  agent any

  environment {
    ANSIBLE_CONFIG = "${WORKSPACE}/deploy/ansible.cfg"
  }

  stages {
    stage("Initial cleanup") {
      steps {
        dir("${WORKSPACE}") {
          deleteDir()
        }
      }
    }

    stage('Checkout SCM') {
      steps {
        git branch: 'feature/jenkinspipeline-stages', url: 'https://github.com/Vectorjay/ansible-config-mgt.git' 
      }
    }

    stage('Prepare Ansible For Execution') {
      steps {
        echo "${WORKSPACE}" 
        sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
      }
    }

    stage('Run Ansible playbook') {
      steps {
        ansiblePlaybook(
          become: false,
          colorized: false,
          credentialsId: 'secon-key',
          disableHostKeyChecking: true,
          installation: 'ansible',
          inventory: 'inventory/dev',
          playbook: 'playbooks/site.yml'
        )
      }
    }

    stage('Clean Workspace after build') {
      steps {
        cleanWs(
          cleanWhenAborted: true,
          cleanWhenFailure: true,
          cleanWhenNotBuilt: true,
          cleanWhenUnstable: true,
          deleteDirs: true
        )
      }
    }
  }
}
