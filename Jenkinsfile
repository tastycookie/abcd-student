pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'gh-abcd-student', url: 'https://github.com/tastycookie/abcd-student', branch: 'main'
                }
            }
        }

        stage('[ZAP] Baseline passive-scan') {
    steps {
        sh '''
            docker run --name juice-shop -d --rm \\
                -p 3000:3000 \\
                bkimminich/juice-shop
            sleep 5
        '''
        sh '''
            docker run --name zap --rm \\
                --add-host=host.docker.internal:host-gateway \\
                -v /Users/xseni/Docker/abcd-lab/passive_scan.yaml:/zap/wrk/:rw
                -t ghcr.io/zaproxy/zaproxy:stable bash -c \\
                "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \\
                || true
        '''
    }
    post {
        always {
            sh '''
                docker cp zap:/zap/wrk/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                docker stop zap juice-shop
            '''
        }
    }
}

        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }
    }
}
