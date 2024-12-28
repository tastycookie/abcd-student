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
        stage('OSV-Scanner') {
            steps {
                sh 'echo $WORKSPACE'
                sh 'mkdir -p ${WORKSPACE}/results'
                sh 'touch ${WORKSPACE}/results/test.txt'
                sh  '''
                    osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json || echo "OSV-Scanner failed but continuing pipeline."
                '''
            }
        }

        stage('Secret-Scanner') {
            steps {
                sh  '''
                    trufflehog git file://. --only-verified --json > ${WORKSPACE}/results/trufflehog-results.json

                '''
            }
        }

        stage('SAST-Scanner') {
            steps {
                sh  '''
                    semgrep --config auto --json --output ${WORKSPACE}/results/semgrep-results.json || echo "Semgrep scan completed with warnings."

                '''
            }
        }
        
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap --add-host=host.docker.internal:host-gateway -v /Users/xseni/Docker/devsecops/abcd-lab/zap:/zap/wrk/:rw -t ghcr.io/zaproxy/zaproxy:stable bash -c  "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml"  || true
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap
                    '''
                    archiveArtifacts artifacts: 'results/*.json, results/*.html, results/*.xml', allowEmptyArchive: true
                    defectDojoPublisher(artifact: 'results/zap_xml_report.xml', 
                        productName: 'Juice Shop', 
                        scanType: 'ZAP Scan', 
                        engagementName: 'sec@shinsec.pl')

                    defectDojoPublisher(artifact: 'results/sca-osv-scanner.json', 
                        productName: 'Juice Shop', 
                        scanType: 'OSV Scan', 
                        engagementName: 'sec@shinsec.pl')

                       defectDojoPublisher(artifact: 'results/trufflehog-results.json', 
                        productName: 'Juice Shop', 
                        scanType: 'Trufflehog Scan', 
                        engagementName: 'sec@shinsec.pl')

                       defectDojoPublisher(artifact: 'results/semgrep-results.json', 
                        productName: 'Juice Shop', 
                        scanType: 'Semgrep JSON Repor', 
                        engagementName: 'sec@shinsec.pl')

                }
            }
        }

    }
}
