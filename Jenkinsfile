node {
    stage('Build') {
        docker.image('python:3.12.1-alpine3.19').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }

    stage('Deploy') {
        def buildId = env.BUILD_ID
        def volume = "${pwd()}/sources:/src"
        def image = 'cdrx/pyinstaller-linux:python2'
        dir(buildId) {
            unstash('compiled-results')
            docker.image(image).inside("-v ${volume}") {
                sh "pyinstaller -F add2vals.py"
            }
        }
        sleep time: 1, unit: "MINUTES"
        step([$class: 'ArtifactArchiver', artifacts: "${buildId}/sources/dist/add2vals"])
        docker.image(image).inside("-v ${volume}") {
            sh "rm -rf build dist"
        }
    }
}