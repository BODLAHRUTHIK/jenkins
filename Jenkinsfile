pipeline{
    agent any

    parameters{
        choice(name:"VERSION", choices: ["1.1.1", "1.2.1", "1.3.1"], description:"version to choose")
        booleanParam(name:"executeTests", defaultValue: true, description:"choose to test")
    }

    environment {
        GIT_REPO = 'https://github.com/BODLAHRUTHIK/jenkins.git'
        GIT_CREDENTIALS_ID = credentials('')
    }
    stages {
        stage ('git clone'){
            steps{
                echo 'Cloning the github repository'
                
            }
        }
            
    }
    stages {
        stage ('build'){
            steps{
                echo "Building docker image here"
                sh 'docker build -t '
               
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
