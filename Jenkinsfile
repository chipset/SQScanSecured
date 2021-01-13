pipeline {
    agent any
    environment {
        // Set up Environment Variables
        //CHANGE THIS SECTION FOR EACH SYSTEM 
        // OR set up a secret text credential for each of the variables.
        //ENDEVOR=" $ENDEVOR_OPTIONS" //set them as a single variable. First character is a space to ensure command doesn't attach to previous options
        ENDEVOR_OPTIONS_SECURED=credentials("EndevorOptions") //Connection string for Endevor holding only the stuff we want hidden
        ENDEVOR_OPTIONS="--comment SQScanSecured --ccid SQSCAN --os true" //standard options, shouldn't need changing 
        ENDEVOR=" $ENDEVOR_OPTIONS_SECURED $ENDEVOR_OPTIONS" //set them as a single variable. First character is a space to ensure command doesn't attach to previous options

        ZOWE_OPT_HOSTNAME=credentials("eosHost1")
        ZOWE_OPT_HOST="$ZOWE_OPT_HOSTNAME"   //Some commands require ZOWE_OPT_HOST, so it's added here.

        // z/OSMF Connection Details
        ZOWE_OPT_PORT="443"
        ZOWE_OPT_REJECT_UNAUTHORIZED=false
        
        //Defined to use SonarScanner from Global Tools.
        def scannerHome = tool 'SQScanner4.0';

    }
    stages {
        stage('Download Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'eosCreds', usernameVariable: 'ZOWE_OPT_USER', passwordVariable: 'ZOWE_OPT_PASSWORD')]) {
                    echo "C:/Users/Administrator/AppData/Roaming/npm/zowe.cmd endevor retrieve element $elementname --env $toenvironment --sn $tostageid --sys $tosystem --sub $tosubsystem --typ $totype --tf $elementname.$TOTYPE $ENDEVOR  --nosignout"
                }
            }
        }
        stage('SonarQube Scan') {
            steps {
                // Starts the SonarQube Scan
                // We want to ensure these directories exist, however if the call fails, we are ok with that.
                // Hence the returnStatus always returns true.  The BAT function only works on windows, I believe.
                // use SH for linux/mac and change the scanner function.  This could also use the scanner plugin.
                // bat returnStatus: true, script: 'mkdir cobol'
                // bat returnStatus: true, script: 'mkdir copybooks'
                // bat returnStatus: true, script: 'move /y *.COBOL ./cobol'
                // bat returnStatus: true, script: 'move /y *.cpy ./copybooks'

                // withSonarQubeEnv('VaughnsServer') {
                //     bat "$scannerHome\\bin\\sonar-scanner.bat"
                // }
                sleep(10)
                echo "Scanning"
            }
        }
        stage('Quality Gate') {
            steps {
                echo "Quality"
                // sleep(10)
                // timeout(time: 1, unit: 'HOURS') {
                //     waitForQualityGate abortPipeline: true
                // }
            }
        }
        stage('Deploying') {
            steps {
                echo "Deploying Code"
            }    
        }
        stage('Clean up') {
            steps {
                echo "Cleaning up..."
                    bat returnStatus: true, script: 'del end*.txt'
            }    
        }
    }
}
