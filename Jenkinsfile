pipeline{
    agent any

    environment{
        server_credentials = credentials('test-credentials')
    }
    
    stages {
        stage ('build'){
            steps{
                echo "Hi, Here building happens"
                echo "This is for testing autocommit feature"
            }
            
        }
        stage ('test'){
            steps{
                echo "Hi, Here testing happens"
            }
        }
        stage ('deploy'){
            steps{
                echo "here are my credentials ${server_credentials}"
                echo "Hi, Here deployment happens"
            }
        }
    }
}
