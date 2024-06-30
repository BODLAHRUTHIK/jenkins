pipeline {
    agent any

    parameters {
        choice(name: "VERSION", choices: ["1.1.1", "1.2.1", "1.3.1"], description: "Version to choose")
        booleanParam(name: "executeTests", defaultValue: true, description: "Choose to test")
    }

    tools {
        dockerTool 'docker'
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
        stage('Install Tools') {
            steps {
                sh '''
                # Define paths
                export PATH=$HOME/.local/bin:$PATH
                mkdir -p $HOME/.local/bin

                # Install AWS CLI
                if ! command -v aws &> /dev/null
                then
                    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                    unzip -y awscliv2.zip 
                    ./aws/install -i $HOME/.local/aws-cli -b $HOME/.local/bin
                fi

                # Install kubectl
                if ! command -v kubectl &> /dev/null
                then
                    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x ./kubectl
                    mv ./kubectl $HOME/.local/bin/kubectl
                fi

                # Install Helm
                if ! command -v helm &> /dev/null
                then
                    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash -s -- --version v3.5.0
                    mv /usr/local/bin/helm $HOME/.local/bin/helm
                fi
                '''
            }
        }

        stage('git clone') {
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

        stage('build') {
            steps {
                echo "Building Docker image here"
                script {
                    // Build the Docker image
                    dir("${REPO_DIR}/hello-world-app/project-flask") {
                        sh "docker build -t ${DOCKER_REPO}:${params.VERSION} ."
                    }

                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                    // Push the Docker image to Docker Hub
                    sh "docker push ${DOCKER_REPO}:${params.VERSION}"
                }
            }
        }

        stage('test') {
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

        stage('deploy') {
            steps {
                script {
                    echo "Deploying the application"

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-credentials'
                    ]]) {
                        // Configure AWS CLI and kubectl to use the EKS cluster
                        sh """
                        aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME}
                        kubectl config use-context arn:aws:eks:${AWS_REGION}:${AWS_ACCOUNT_ID}:cluster/${EKS_CLUSTER_NAME}
                        """

                        // Add Helm repo and update
                        sh """
                        helm repo add my-repo ${HELM_REPO_URL}
                        helm repo update
                        """

                        // Deploy the Helm chart
                        sh """
                        helm upgrade --install ${HELM_CHART_NAME} my-repo/${HELM_CHART_NAME} --set image.repository=${DOCKER_REPO} --set image.tag=${params.VERSION}
                        """
                    }
                }
            }
        }
    }
}
