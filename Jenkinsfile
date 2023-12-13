node {
    try {
        stage('Build') {
            docker.image('python:3.12.1-alpine3.19').inside {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        stage('Test') {
            docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            junit 'test-reports/results.xml'
        }
        
        def userInput

        stage('Manual Approval') {
            userInput = input(
                id: 'userInput',
                message: 'Lanjutkan ke tahap Deliver?',
                parameters: [choice(name: 'OPTION', choices: ['Proceed', 'Abort'], description: 'Pilih opsi:')]
            )
        }

        if (userInput == 'Proceed') {
            stage('Deploy') {
                // Checking out the source code
                checkout scm

                // Using a Docker image with Python and necessary dependencies
                def dockerImage = 'cdrx/pyinstaller-linux:python2'
                def workspacePath = "/var/jenkins_home/workspace/submission-cicd-pipeline-raflyyogas"

                sh "docker run --rm -v ${workspacePath}/sources:/src ${dockerImage} 'pyinstaller -F /src/add2vals.py'"

                // Archiving artifacts
                archiveArtifacts artifacts: 'sources/add2vals.py', followSymlinks: false

                // Cleaning up build artifacts
                sh "docker run --rm -v ${workspacePath}/sources:/src ${dockerImage} 'rm -rf /src/build /src/dist'"

                // Adding a delay using sleep for 1 minute
                sleep time: 60, unit: 'SECONDS'
            }
        } else {
            error 'Eksekusi pipeline dihentikan.'
        }
        
    } catch (Exception e) {
        // Handle failures or exceptions here
        currentBuild.result = 'FAILURE'
        error("Pipeline failed: ${e.message}")
    }
}
