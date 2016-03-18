        node {

            wrap([$class: 'BuildUser']) {
                def user = env.BUILD_USER_ID
            }

            env.JAVA_HOME = tool 'JDK1.7.0.21'
            def mvnHome = tool 'M3'
            def nodeJS = tool 'NodeJSv5.6.0'
            env.MAVEN_OPTS = "-Xmx1024m -XX:MaxPermSize=512m"
            env.PATH = "${mvnHome}/bin:${nodeJS}/bin:${env.JAVA_HOME}/bin:${env.PATH}"

            //checkout scm
            sh 'git rev-parse --verify HEAD > hash.txt'
            env.GIT_COMMIT = readFile('hash.txt')
            env.GIT_COMMIT = env.GIT_COMMIT.trim()

            setVersion();

            stage("Build Core")
            hipchatSend color: 'GREEN', notify: true, room: '1654572'
            BuildIt ("sharedLib1")
            BuildIt ("sharedLib2")

            stage("Build Modules")
            parallel "first-ui": {
                BuildIt ("first-ui")
            }, "second-ui": {
                BuildIt ("second-ui")
            }, "first-ws": {
                BuildIt ("first-ws")
            }, "second-ws": {
                BuildIt ("second-ws")
            }

            stage("Archive files")
            echo("================================== Archive Stage ==================================")
            step([$class: 'ArtifactArchiver', artifacts: '**/target/*.war', fingerprint: true])
            //step([$class: 'ArtifactArchiver', artifacts: 'CONFIG/*.*', fingerprint: true])
            step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])

            step([$class: 'hipchat', room: '1654572', startNotification: false, notifySuccess: true, notifyAborted: false, notifyNotBuil: false, notifyUnstable: false, notifyFailure: true, notifyBackToNormal: false])

            stage("Notification")
            step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'jvanryn@liaison-intl.com', sendToIndividuals: true])
        }

def BuildIt(module) {

    sh "echo '================================== Build Stage : ${module} ==================================';\
        cd ${module};\
        mvn --batch-mode -V -U -e clean deploy -U -DskipITs sonar:sonar -Dsonar.branch=${env.BRANCH_NAME} -Dsurefire.useFile=false"
}

def setVersion () {

    def JobCode = env.BRANCH_NAME
    def JobName = env.JOB_NAME
    def BuildURL = env.BUILD_URLD
    def BuildNum = env.BUILD_NUMBER
    def RevNum = env.GIT_COMMIT
    def WrkSpce = pwd()

    echo "Running Build for: "+JobCode
    // update version to a unique version
    def script=WrkSpce+"/Tools/PomVersionUpdater.sh "+JobCode+"-"+RevNum+"-"+BuildNum

    // sh script
}


