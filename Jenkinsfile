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
                sh 'yarn test:unit'
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

        stage('Pushing Artifact') {
            agent {
                docker { 
                    image 'mrabbah/mc:1.1'
                    reuseNode true
                } 
            }
            steps {
                sh 'mc --config-dir /tmp/.mc alias set minio $MINIO_HOST $MINIO_CREDS_USR $MINIO_CREDS_PSW'
                sh 'mc --config-dir /tmp/.mc cp ./corteza-server-corredor-${BRANCH_NAME}.tar.gz minio/corteza-artifacts' 
                sh 'rm -f ./corteza-server-corredor-${BRANCH_NAME}.tar.gz'              
            }
        }

        stage('Build Docker image') {
            
            steps {
                sh 'docker build -t mrabbah/corteza-server-corredor:${BRANCH_NAME} . '
            }
        }
        
        stage('Push Docker image') {
            
            steps {
                echo 'Pushing docker image'
                script {
                    sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'    
                    sh 'docker push mrabbah/corteza-server-corredor:${BRANCH_NAME}'           
                }
                
            }
        }
        stage('Deploy Dev') {
            when {
                branch "r*x"
            }
            steps {
                script {
                    sh 'sed -i "s/TAG_NAME/${BRANCH_NAME}/g" ./k8s/deployment-dev.yml'
                    sh 'curl -LO "https://dl.k8s.io/release/v1.24.0/bin/linux/amd64/kubectl"' 
                    sh 'chmod u+x ./kubectl'  
                    withKubeConfig([credentialsId: 'k8s-token', serverUrl: 'https://rancher.rabbahsoft.ma/k8s/clusters/c-m-6mdv2kbw']) {
                        sh './kubectl apply -f k8s/deployment-dev.yml'
                    }           
                }
                
            }
        }    
    }
    post {
        always {
            sh 'docker logout'
        }
    }        
}
