/*
* @author: Juan M. Escobedo Rios <littleJmer>
* @email: <juanmanueler@gmail.com>
* @version: 1.0
* @date:
*/

def IMAGE
def SERVICE_NAME = ""
def IMAGE_NAME = ""
def TAG = ""
def NOTIFY = "juan.escobedo@g-global.com.mx"

/* true | false */
def PUSH_IMAGE = false
def CLEANUP = true

String COMMIT_HASH

node {

    try {

        notifyBuild('STARTED', NOTIFY)

        stage('Checkout and Clone Repository') {

            /* Let's make sure we have the repository 
             * cloned to our workspace */
            
            checkout scm
            
            /* Get commit hash */
            /* COMMIT_HASH = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7) */
            
        }

        stage('Composer') {
            sh "composer install"
        }

        stage('PHPunit') {
            sh "vendor/bin/phpunit"
        }

        stage('Build Image') {

            /* This builds the actual image; synonymous to
             * docker build on the command line */
            
            /* docker.withRegistry("https://registry.hub.docker.com", "dockerhub-credentials") {
                IMAGE = docker.build("${IMAGE_NAME}:${TAG}.${env.BUILD_ID}.${COMMIT_HASH}")
            } */

        }

        stage('Test Image') {

            /* Ideally, we would run a test framework against our image.
             * For this example, we're using a Volkswagen-type approach ;-) */

            /* IMAGE.inside {
                sh 'echo "Tests passed"'
            } */
            
        }

        stage('Push Images') {

            /* Finally, we'll push the image with two tags:
             * First, the commit id from GitHub
             * Second, the 'stable' tag.
             * Pushing multiple tags is cheap, as all the layers are reused. */
            
            if(PUSH_IMAGE) {
                docker.withRegistry("https://registry.hub.docker.com", "dockerhub-credentials") {
                    /* IMAGE.push("${TAG}.${env.BUILD_ID}.${COMMIT_HASH}") */
                    IMAGE.push("${TAG}")
                }
            }

        }

        stage('Update Service and Cleanup') {

            /* Update Service */
            /* sh "docker service update --force --image ${IMAGE_NAME}:${TAG}.${env.BUILD_ID}.${COMMIT_HASH} ${SERVICE_NAME}" */

            if(CLEANUP) {
                /* Remove Exited Containers */
                deleteContainerUnused()

                /* Delete all dangling (unused) images */
                deleteImagesUnused()
            }

        }


    }
    catch(e) {
        /* If there was an exception thrown, the build failed */
        currentBuild.result = "FAILED"
        throw e
    }
    finally {
        /* Success or failure, always send notifications */
        notifyBuild(currentBuild.result, NOTIFY)
    }

}

def notifyBuild(String buildStatus = 'STARTED', String notifyTo) {

    /* build status of null means successful */
    buildStatus =  buildStatus ?: 'SUCCESSFUL'

    /* Default values */
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"
    def details = """
    <p>${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
    <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>
    """

    /* Override default values based on build status */
    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESSFUL') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    /* Send notifications */

    /* slackSend (color: colorCode, message: summary) */

    /* hipchatSend (color: color, notify: true, message: summary) */

    emailext (
        subject: subject,
        body: details,
        to: notifyTo
    )

}

def deleteContainerUnused() {
    try {
        sh "docker ps -q -f status=exited | xargs docker rm"
    } catch(error){}
}

def deleteImagesUnused(){
    try {
        sh "docker image prune -a -f"
    } catch(error){}
}

