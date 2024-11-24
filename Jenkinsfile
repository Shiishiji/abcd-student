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
        stage('Preparation') {
            steps {
                sh 'mkdir -p results'
            }
        }

        stage('Semgrep scan') {
            steps {
                sh '(semgrep scan --config auto --json-output=${WORKSPACE}/results/semgrep-report.json) || true'
            }
        }

        stage('TruffleHog scan') {
            steps {
                sh '(trufflehog git file://. --only-verified --json > ${WORKSPACE}/results/truffle-hog-scanner.json) || true'
            }
        }

        stage('SCA scan') {
            steps {
                sh '(osv-scanner scan --lockfile package-lock.json --format json --output ${WORKSPACE}/results/sca-osv-scanner.json) || true'
            }
        }

        stage('ZAP Passive Scan') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /home/dszopinski/Projects/ABC_DevSecOps/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "ls -al /zap/wrk && zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                        || true
                '''
                sh '''
                    docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                    docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                '''
            }
            post {
                always {
                    sh '''
                        docker stop zap juice-shop
                        docker rm zap
                    '''
                }
            }
        }
    }

    post {
        always {
            sh '''
                echo "dojoPublisher(artifact: 'results/semgrep-report.json',
                        productName: 'Juice Shop',
                        scanType: 'Semgrep JSON Report',
                        engagementName: 'damian.szopinski@verestro.com')"
                '''
            sh '''
                echo "dojoPublisher(artifact: 'results/zap_xml_report.xml',
                        productName: 'Juice Shop',
                        scanType: 'ZAP Scan',
                        engagementName: 'damian.szopinski@verestro.com')"
                '''
            sh '''
                echo "dojoPublisher(artifact: 'results/sca-osv-scanner.json',
                        productName: 'Juice Shop',
                        scanType: 'OSV Scan',
                        engagementName: 'damian.szopinski@verestro.com')"
                '''
            sh '''
                echo "dojoPublisher(artifact: 'results/truffle-hog-scanner.json',
                        productName: 'Juice Shop',
                        scanType: 'Trufflehog Scan',
                        engagementName: 'damian.szopinski@verestro.com')"
                '''
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