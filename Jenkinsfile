pipeline{
    agent any

    parameters{
        choice(name:"VERSION", choices: ["1.1.1", "1.2.1", "1.3.1"], description:"version to choose")
        booleanParam(name:"executeTests", defaultValue: true, description:"choose to test")
    }

    environment {
        GIT_REPO = 'https://github.com/BODLAHRUTHIK/jenkins.git'
        GIT_CREDENTIALS_ID = credentials('github-credentials')
        DOCKER_REPO = 'hruthikbodla/myprojects'
        DOCKER_CREDENTIALS_ID = credentials('dockerhub-creds')
    }

    stages {
        stage ('git clone'){
            steps{
                echo 'Cloning the github repository'
                git credentialsId: env.GIT_CREDENTIALS_ID, URL: env.GIT_REPO
            }
        }
        stage ('build'){
            steps{
                echo "Building docker image here"
                sh 'docker build -t ${DOCKER_REPO}:${params.VERSION} .'
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]){
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
                sh 'docker push ${DOCKER_REPO}:${params.VERSION}'
            }
            
        }
        stage ('test'){
            when {
                    expression{
                        params.executeTests == true
                    }
                }
            steps{
                
                echo "Hi, Here testing happens"
                echo "${params.VERSION}"

            }
        }
        stage ('deploy'){
            steps{
                echo "Hi, Here deployment happens"
            }
        }
    }
}
