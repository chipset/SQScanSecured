pipeline {
    agent any
    environment {
        // Set up Environment Variables
        //CHANGE THIS SECTION FOR EACH SYSTEM 
        // OR set up a secret text credential for each of the variables.
        //ENDEVOR_HOST_INFO=credentials("EndevorHost") //Connection string for Endevor holding only the stuff we want hidden
        ENDEVOR_OPTIONS="--port 7080 --protocol http --reject-unauthorized false -i NDVRWEBS  --comment SQScanSecured --ccid SQSCAN"  //standard options, shouldn't need changing 
        //ENDEVOR=" $ENDEVOR_HOST_INFO $ENDEVOR_OPTIONS" //set them as a single variable. First character is a space to ensure command doesn't attach to previous options
        ENDEVOR=" $ENDEVOR_OPTIONS" //set them as a single variable. First character is a space to ensure command doesn't attach to previous options

        ZOWE_OPT_HOSTNAME=credentials("MSTRSVW")
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
                withCredentials([usernamePassword(credentialsId: 'mcquitty', usernameVariable: 'ZOWE_OPT_USER', passwordVariable: 'ZOWE_OPT_PASSWORD')]) {
                    // Jenkins runs as a different user.  Uncomment lines below to install plugin. Other plugins could be added here.
                    bat "npm config set @brightside:registry https://api.bintray.com/npm/ca/brightside"
                    bat "C:/Users/Administrator/AppData/Roaming/npm/zowe.cmd plugins install @brightside/endevor@lts-incremental"
                    bat "C:/Users/Administrator/AppData/Roaming/npm/zowe.cmd endevor retrieve element $elementname --env $toenvironment --sn $tostageid --sys $tosystem --sub $tosubsystem --typ $totype --tf $elementname.$TOTYPE --no-signout true $ENDEVOR"
                }
            }
        }
        stage('SonarQube Scan') {
            steps {
                // Starts the SonarQube Scan
                // We want to ensure these directories exist, however if the call fails, we are ok with that.
                // Hence the returnStatus always returns true.  The BAT function only works on windows, I believe.
                // use SH for linux/mac and change the scanner function.  This could also use the scanner plugin.
                bat returnStatus: true, script: 'mkdir cobol'
                bat returnStatus: true, script: 'mkdir copybooks'
                bat returnStatus: true, script: 'move /y *.COBOL ./cobol'
                bat returnStatus: true, script: 'move /y *.cpy ./copybooks'

                withSonarQubeEnv('VaughnsServer') {
                    bat "%SQScanner4%\\bin\\sonar-scanner.bat"
                }
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
