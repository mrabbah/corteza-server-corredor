pipeline {
    agent any
   
    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        BRANCH_NAME = "${GIT_BRANCH.split('/').size() > 1 ? GIT_BRANCH.split('/')[1..-1].join('/') : GIT_BRANCH}"
        MINIO_CREDS = credentials('minio-credentials')
        MINIO_HOST = "https://minio.rabbahsoft.ma:9900"
    }
    stages {
        stage('Test') {
            agent {
                docker {
                  image 'node:16.16.0'
                  reuseNode true
                }
            }
            steps {
                //sh 'git reset --hard  && git clean -fdx --exclude="/node_modules/"'
                sh 'yarn install'
                sh 'yarn lint'
                // sh 'yarn test:unit'
            }
        }
        stage('Package') {
            steps {
                sh 'mkdir -p .tmp/corteza-server-corredor'
                sh 'cp -r README.* LICENSE CONTRIBUTING.md DCO .env.example package.json .eslintrc.js .mocharc.js tsconfig.json yarn.lock src node_modules .tmp/corteza-server-corredor/'
	            sh 'tar -C .tmp -czf corteza-server-corredor-${BRANCH_NAME}.tar.gz corteza-server-corredor'
                sh 'ls -l && pwd && ls -l .tmp/'
            }
        }
    }        
}
