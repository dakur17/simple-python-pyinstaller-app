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
        // post {
        //     always {
        //         junit 'test-reports/results.xml'
        //     }
        // }
    }

    stage('Deploy') {
        def BUILD_ID = env.BUILD_ID
        dir(BUILD_ID) {
            unstash 'compiled-results'
            sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"
        }

        try {
            archiveArtifacts "${BUILD_ID}/sources/dist/add2vals"
        } finally {
            sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
        }
    }
}
