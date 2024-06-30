pipeline{
    agent any

    parameters{
        choice(name:"VERSION", choices: ["1.1.1", "1.2.1", "1.3.1"], description:"version to choose")
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
