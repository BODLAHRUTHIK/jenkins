pipeline{
    agent any

    environment{
        server_credentials = credentials('test-credentials')
    }

    parameters{
        choices(name:"VERSION", choices: ["1.1.1", "1.2.1", "1.3.1"], defaultValue: "2.3.5", description="version to choose")
        booleanParam(name:"executeTests", defaultValue: true, description:"choose to test")
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
                when {
                    expression{
                        params.executeTests == true
                    }
                }
                echo "Hi, Here testing happens"
                sh("echo ${server_credentials_USR} ${server_credentials_PSW}")
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
