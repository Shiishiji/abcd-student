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
                    git credentialsId: 'github-token', url: 'https://github.com/Shiishiji/abcd-student', branch: 'main'
                }
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }
        stage('ZAP Passive Scan') {
            steps {
                sh 'mkdir results'
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v ./.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communictyScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" 
                        || true
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
                }
            }
        }
    }

//    Example - publish report to DefectDojo
//    stage('SCA scan') {
//        steps {
//            sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json'
//        }
//    }
//    post {
//        always {
//            defectDojoPublisher(artifact: 'results/sca-osv-scanner.json',
//                    productName: 'Juice Shop',
//                    scanType: 'OSV Scan',
//                    engagementName: 'damian.szopinski@verestro.com')
//        }
//
//    -------------------------------------------------------------
//      Possible scanType values reference
//    -------------------------------------------------------------
//      Tool	                Format      ScanType:
//      Zed Attack Proxy (ZAP)	XML	        'ZAP Scan'
//      OSV-Scanner	            JSON	    'OSV Scan'
//      TruffleHog	            JSON	    'Trufflehog Scan'
//      Semgrep                 JSON	    'Semgrep JSON Report'
//    -------------------------------------------------------------
//    }

}