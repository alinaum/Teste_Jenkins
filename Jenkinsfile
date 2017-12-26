#!groovy
import groovy.json.JsonOutput
import java.util.Optional
import hudson.tasks.test.AbstractTestResultAction
import hudson.model.Actionable
import hudson.tasks.junit.CaseResult

def speedUp = '--configure-on-demand --daemon --parallel'
def nebulaReleaseScope = (env.GIT_BRANCH == 'origin/master') ? '' : "-Prelease.scope=patch"
def slackNotificationChannel = "#general"
def author = ""
def message = ""
def testSummary = ""
def total = 0
def failed = 0
def skipped = 0

def isPublishingBranch = { ->
    return env.GIT_BRANCH == 'origin/master' || env.GIT_BRANCH =~ /release.+/
}

def isResultGoodForPublishing = { ->
    return currentBuild.result == null
}

def notifySlack(text, channel, attachments) {
    def slackURL = 'https://hooks.slack.com/services/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'
    def jenkinsIcon = 'https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png'

    def payload = JsonOutput.toJson([text: text,
        channel: channel,
        username: "Jenkins",
        icon_url: jenkinsIcon,
        attachments: attachments
    ])

    sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
}

def getGitAuthor = {
    def commit = sh(returnStdout: true, script: 'git rev-parse HEAD')
    author = sh(returnStdout: true, script: "git --no-pager show -s --format='%an' ${commit}").trim()
}

def getLastCommitMessage = {
    message = sh(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
}



def populateGlobalVariables = {
    getLastCommitMessage()
    getGitAuthor()
}

node {
    try {
        stage('Checkout') {
            checkout scm
        }

        stage("Restore") {
			def proc = bat (script:'''"C:\\Program Files (x86)\\nuget.exe "  restore " C:\\Program Files (x86)\\Jenkins\\jobs\\Pipeline_sintaxe\\branches\\master\\workspace\\C#\\StoreApp.sln"''', returnStatus: true)
			if (proc == 0){
				echo "Restore Nuget Packges";
			}
			else {
				emailext attachLog: true, body:" Nuget Restore falhou favor verificar", subject: "Restore", to: EMAIL_TO;
				currentBuild.result = "FAILURE";
				error 'Problema no restore do nuget packge';
			}
               populateGlobalVariables()
		}	

        stage("Build"){
			def msbuild = tool name: 'MsBuild fw4.0', type: 'hudson.plugins.msbuild.MsBuildInstallation';
			def proc = bat (script:'''"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\MSBuild.exe"   "C:\\Program Files (x86)\\Jenkins\\jobs\\Pipeline_sintaxe\\branches\\master\\workspace\\C#\\StoreApp.sln"''', returnStatus: true)
			echo "Build Execution";
			if (proc == 0){
				echo "Build execution";
				emailext attachLog: true, body:"Build e Commit feito com sussesso", subject: MAIL_SUBJECT_TESTE, to: EMAIL_TO;
			}
			else {
				emailext attachLog: true, body:"Erro no build, por favor verifique o log e reverta o commite caso necessario", subject: "Build", to: EMAIL_TO;
				currentBuild.result = "FAILURE";
				error 'Build Solution';
			}
                   populateGlobalVariables()
		}


			def buildColor = currentBuild.result == null ? "good" : "warning"
            def buildStatus = currentBuild.result == null ? "Success" : currentBuild.result
            def jobName = "${env.JOB_NAME}"

            // Strip the branch name out of the job name (ex: "Job Name/branch1" -> "Job Name")
            jobName = jobName.getAt(0..(jobName.indexOf('/') - 1))
        
            if (failed > 0) {
                buildStatus = "Failed"

                if (isPublishingBranch()) {
                    buildStatus = "MasterFailed"
                }

                buildColor = "danger"
                def failedTestsString = getFailedTests()

                notifySlack("", slackNotificationChannel, [
                    [
                        title: "${jobName}, build #${env.BUILD_NUMBER}",
                        title_link: "${env.BUILD_URL}",
                        color: "${buildColor}",
                        text: "${buildStatus}\n${author}",
                        "mrkdwn_in": ["fields"],
                        fields: [
                            [
                                title: "Branch",
                                value: "${env.GIT_BRANCH}",
                                short: true
                            ],
                            [
                                title: "Test Results",
                                value: "${testSummary}",
                                short: true
                            ],
                            [
                                title: "Last Commit",
                                value: "${message}",
                                short: false
                            ]
                        ]
                    ],
                    [
                        title: "Failed Tests",
                        color: "${buildColor}",
                        text: "${failedTestsString}",
                        "mrkdwn_in": ["text"],
                    ]
                ])          
            } else {
                notifySlack("", slackNotificationChannel, [
                    [
                        title: "${jobName}, build #${env.BUILD_NUMBER}",
                        title_link: "${env.BUILD_URL}",
                        color: "${buildColor}",
                        author_name: "${author}",
                        text: "${buildStatus}\n${author}",
                        fields: [
                            [
                                title: "Branch",
                                value: "${env.GIT_BRANCH}",
                                short: true
                            ],
                            [
                                title: "Last Commit",
                                value: "${message}",
                                short: false
                            ]
                        ]
                    ]
                ])
            }
        
			if (isPublishingBranch() && isResultGoodForPublishing()) {
				stage ('Publish') {
					sh "./gradlew ${gradleDefaultSwitches}"
				}
			}
    } catch (hudson.AbortException ae) {
        // I ignore aborted builds, but you're welcome to notify Slack here
    } catch (e) {
        def buildStatus = "Failed"

        if (isPublishingBranch()) {
            buildStatus = "MasterFailed"
        }

        notifySlack("", slackNotificationChannel, [
            [
                title: "${env.JOB_NAME}, build #${env.BUILD_NUMBER}",
                title_link: "${env.BUILD_URL}",
                color: "danger",
                author_name: "${author}",
                text: "${buildStatus}",
                fields: [
                    [
                        title: "Branch",
                        value: "${env.GIT_BRANCH}",
                        short: true
                    ],
                    [
                        title: "Last Commit",
                        value: "${message}",
                        short: false
                    ],
                    [
                        title: "Error",
                        value: "${e}",
                        short: false
                    ]
                ]
            ]
        ])

        throw e
        }

	}
 
