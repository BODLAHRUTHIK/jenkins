pipeline {
    agent any

    parameters {
        choice(name: "VERSION", choices: ["1.1.1", "1.2.1", "1.3.1"], description: "Version to choose")
        booleanParam(name: "executeTests", defaultValue: true, description: "Choose to test")
    }

    tools{
        dockerTool 'docker'
    }
    environment {
        GIT_REPO = 'https://github.com/BODLAHRUTHIK/hello-world-app.git' // Correct repository URL
        GIT_CREDENTIALS_ID = 'github-credentials' // Correct GitHub credentials ID
        DOCKER_REPO = 'hruthikbodla/myprojects'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds' // Correct Docker Hub credentials ID
        GIT_BRANCH = 'main' // Specify the branch to build
        REPO_DIR = "${WORKSPACE}"
    }

    stages {
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
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
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
                echo "Deploying the application"
            }
        }
    }
}
