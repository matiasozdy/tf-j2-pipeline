#!groovy

/**
 *Terraform pipeline *
 * @builderImage docker image:tag located in our ECR to be used as a base build container
 */

def call(String builderImage = 'terraform:latest') {

run(builderImage) {

    def result = ''

    if (!branchUtils.isMasterBranch()) {

        sh "git --no-pager diff --diff-filter=ACMT remotes/origin/refs/heads/master...HEAD --name-only | xargs -n1 dirname | sort | uniq > changes.out"
        changes = readFile('changes.out').trim()
        def tfchanges = changes.readLines()

            // Mark the code build 'plan'....
        stage('Get version') {
            sh "terraform --version"
        }


        stage('Validate and run plan') {
        sh "touch /path/terraform.pem"

            tfchanges.each { String line ->
                echo line
                //if (line.matches("(common|core|ops|modules|dev|int|stage|preview|prod).*")) {
                if (line.matches("(common|core|ops|dev|int|stage|preview|prod).*") && !line.matches('.*(\\.terraform).*')) {
                      //dir("${line}") {
                          try {
                            sh "/usr/bin/terraform get ${line}"
                              sh "/usr/bin/terraform plan -out=plan.out ${line}"
                          } catch(e) {
                              error("Plan failed on ${line}")
                          }
                        try {
                            //sh "/usr/bin/tflint --deep ${line}/*.tf"
                            sh "/usr/bin/tflint ${line}/*.tf"
                        } catch(e) {
                            error("Lint failed on ${line}")
                        }
                    //}
                } else {
                    echo "No changes on the infrastructure"
                }

            }
        }

        result = "Branch *${env.BRANCH_NAME}* looks good."
    } else {

        echo "Nothing to do in master branch"

    }

currentBuild.result = 'SUCCESS'
    return result

    }
}
