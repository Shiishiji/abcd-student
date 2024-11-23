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