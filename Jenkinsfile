node {
    stage('Build') {
        def pythonImage = 'python:3.12.1-alpine3.19'
        checkout scm

        docker.image(pythonImage).inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }

    stage('Test') {
        def pytestImage = 'qnib/pytest'

        docker.image(pytestImage).inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }

        junit 'test-reports/results.xml'
    }

    stage('Deploy') {
        def volume = "${pwd()}/sources:/src"
        def pyinstallerImage = 'cdrx/pyinstaller-linux:python2'

        docker.image(pyinstallerImage).inside("-v ${volume}") {
            dir(env.BUILD_ID) {
                unstash 'compiled-results'
                sh "pyinstaller -F add2vals.py"
            }
        }

        archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", allowEmptyArchive: true

        docker.image(pyinstallerImage).inside("-v ${volume}") {
            sh "rm -rf build dist"
        }
    }
}
