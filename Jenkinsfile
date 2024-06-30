pipeline {
    agent any

    parameters {
        choice(name: "VERSION", choices: ["1.1.1", "1.2.1", "1.3.1"], description: "Version to choose")
        booleanParam(name: "executeTests", defaultValue: true, description: "Choose to test")
    }

    environment {
        GIT_REPO = 'https://github.com/BODLAHRUTHIK/hello-world-app.git'
        GIT_CREDENTIALS_ID = 'github-credentials'
        DOCKER_REPO = 'hruthikbodla/myprojects'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        GIT_BRANCH = 'main'
        REPO_DIR = "${WORKSPACE}"
        AWS_REGION = 'ap-south-1'
        EKS_CLUSTER_NAME = 'eks-cluster'
        HELM_REPO_URL = 'your-helm-repo-url'
        HELM_CHART_NAME = 'https://github.com/BODLAHRUTHIK/my-flask-helm.git'
        AWS_ACCOUNT_ID = '874789631010' // Ensure you have this value
    }

    stages {
        stage('Download and Install AWS CLI') {
            steps {
                sh 'curl -o awscliv2.zip https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip'
                sh 'unzip -qo awscliv2.zip'
                sh './aws/install -i ~/aws-cli -b ~/bin'
            }
        }

        stage('Verify AWS CLI Installation') {
            steps {
                sh '~/bin/aws --version'
            }
        }
        stage('Install Tools') {
            steps {
                script {


                    // Install kubectl
                    if (!commandExists('kubectl')) {
                        sh '''
                        curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        mv kubectl $HOME/.local/bin/kubectl
                        '''
                    }

                    // Install Helm
                    if (!commandExists('helm')) {
                        sh '''
                        curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
                        mv /usr/local/bin/helm $HOME/.local/bin/helm
                        '''
                    }
                }
            }
        }

        stage('Git Clone') {
            steps {
                echo 'Cloning the GitHub repository'
                checkout([$class: 'GitSCM',
                          branches: [[name: "*/${env.GIT_BRANCH}"]],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: env.GIT_REPO, credentialsId: env.GIT_CREDENTIALS_ID]]
                ])
            }
        }

        stage('Build') {
            steps {
                echo "Building Docker image here"
                script {
                    dir("${REPO_DIR}/hello-world-app/project-flask") {
                        sh "docker build -t ${DOCKER_REPO}:${params.VERSION} ."
                    }

                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                    sh "docker push ${DOCKER_REPO}:${params.VERSION}"
                }
            }
        }

        stage('Test') {
            when {
                expression {
                    params.executeTests == true
                }
            }
            steps {
                echo "Running tests"
                echo "Selected version: ${params.VERSION}"
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying the application to EKS cluster ${EKS_CLUSTER_NAME}"
                script {
                    withAWS(region: AWS_REGION, credentials: 'aws-creds') {
                        sh "helm repo add my-helm-repo ${HELM_CHART_REPO}"
                        sh "helm upgrade --install my-app my-helm-repo/my-chart --namespace my-namespace --set image.tag=${params.VERSION}"
                    }
                }
            }
        }
    }
}

def commandExists(String command) {
    return sh(script: "command -v $command", returnStatus: true) == 0
}
