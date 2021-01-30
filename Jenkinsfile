def npm_registry = 'https://registry.npm.taobao.org'

def committer
def commit_message

pipeline {
  options {
    disableConcurrentBuilds()
    timeout(time: 30, unit: 'MINUTES')
  }

  agent none

  triggers {
    gitlab(triggerOnPush: true, triggerOnMergeRequest: true, branchFilterType: 'All')
  }

  environment{
    TZ='Asia/Shanghai'
    // HOME='.'
  }
  stages {
    stage('Install') {
      agent {
          docker 'node:13.8.0-buster'
      }

      steps {
          script {
             committer = sh(script: 'git log -1 --pretty=format:"%an"', returnStdout: true).trim()
             commit_message = sh(script: 'git log -1 --pretty=format:%s', returnStdout: true).trim()
          }
          sh """
            yarn config set registry ${npm_registry}
            yarn install --frozen-lockfile
          """
      }
    }
    stage('Lint') {
      agent {
          docker 'node:13.8.0-buster'
      }
      steps {
          sh """
            yarn lint
          """
      }
    }
    stage('Test') {
      agent {
          docker 'node:13.8.0-buster'
      }
      steps {
          sh """
            yarn test:ci
          """
      }
    }
    stage('Build') {
      agent {
          docker 'node:13.8.0-buster'
      }
      steps {
          sh """
            yarn build
          """
      }
    }
    stage('Deploy Prod') {
      agent {
          docker 'node:13.8.0-buster'
      }
      when { branch "release" }
      steps {
        sshagent(credentials: ['jenkins']) {
          sh """
            scp -o StrictHostKeyChecking=no -r dist/ ubuntu@192.x.x.x:~/
            ssh -o StrictHostKeyChecking=no -l ubuntu 192.x.x.x "sudo rm -rf /var/www/dist; sudo mv ~/dist /var/www"
          """
        }
      }
    }
  }
}
