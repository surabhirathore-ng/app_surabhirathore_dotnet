pipeline {
    agent any
		
	environment {
		scannerHome = tool name: 'sonar_scanner_dotnet'
		registry = 'surabhirathore'
		username = 'sonar-surabhirathore'
        appName = 'NAGPDevOpsProject'
   	}	
   
	options {
        //Prepend all console output generated during stages with the time at which the line was emitted.
		timestamps()
		
		//Set a timeout period for the Pipeline run, after which Jenkins should abort the Pipeline
		timeout(time: 1, unit: 'HOURS') 
		
		//Skip checking out code from source control by default in the agent directive
		skipDefaultCheckout()
		
		buildDiscarder(logRotator(
			// number of build logs to keep
            numToKeepStr:'3',
            // history to keep in days
            daysToKeepStr: '15'
			))
    }
    
    stages {
        
    	stage ("Nuget restore") {
            steps {
		    
                //Initial message
                echo "Deployment pipeline started for  branch"
		        checkout scm
                echo "Nuget Restore step"
                bat "dotnet restore"
            }
        }
		
		stage('Start sonarqube analysis'){
            

            steps {
				  echo "Start sonarqube analysis step"
                  withSonarQubeEnv('Test_Sonar') {
                   bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll begin /k:sonar-${username} -d:sonar.cs.opencover.reportsPaths=test-project/coverage.opencover.xml -d:sonar.cs.xunit.reportsPaths='test-project/TestResults/TestFileReport.xml'"  
                  }
            }
        }

       stage('Code build') {
      steps {
        //Cleans the output of the project
        echo "Clean Previous Build"
        bat "dotnet clean"

        //Builds the project and all its dependencies
        echo "Code Build"
        bat 'dotnet build --configuration Release"'
      }
    }

    stage('Test Case Execution') {
      steps {
        echo "Execute Unit Test"
        bat "dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestFileReport.xml"
      }
    }

    stage('Stop SonarQube Analysis') {
      steps {
        echo "Stop SonarQube Analysis"
        withSonarQubeEnv("Test_Sonar") {
          bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll end"
		}

    stage ("Release artifact") {
           

            steps {
                echo "Release artifact step"
                bat "dotnet publish -c Release -o ${appName}/app/${userName}"
            }
        }    
}
}

