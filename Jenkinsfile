pipeline {
    agent any
    stages {
        stage('Tests') {
            steps {
                bat 'npm i npx'
                bat 'npm i newman-reporter-junitfull'
                bat 'npx newman run W33Collection.json -e W33Env.json --reporter-junit-export results.xml --reporters cli,junit' 
                junit '**/results.xml'
                archiveArtifacts artifacts: '**/results.xml'
               // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'result.html', reportName: 'Newman API Test', reportTitles: ''])
            }
        }
    }
    
}




