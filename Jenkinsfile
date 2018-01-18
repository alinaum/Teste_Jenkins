
import groovy.json.JsonOutput
import java.util.Optional
import hudson.tasks.test.AbstractTestResultAction
import hudson.model.Actionable
import hudson.tasks.junit.CaseResult

import groovy.json.JsonSlurper;
import org.apache.commons.io.IOUtils;
import groovyx.net.http.*;
import groovyx.net.http.ContentType.*;
import groovyx.net.http.Method.*;
import net.sf.json.*;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.BufferedInputStream;

import java.io.BufferedInputStream;
import java.io.DataInputStream;
import java.io.BufferedInputStream.*;

import java.io.File;
import java.io.FileInputStream;
import java.io.*;

int RunScopeOk = 0;
int RunScopeDepoisDoLoadBalanceOk = 0;
int Test1 = 0;
int Test2 = 0;
String stageName = "";
String RegexTest = "";
node {
    try {

        stage('Checkout') {
            checkout scm
			stageName = "Checkout"
        }
		
        stage("MSBUILD"){
			bat (script:'"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\MSBuild.exe" "C:\\Program Files (x86)\\Jenkins\\jobs\\Test_free_git\\workspace\\MedgrupoAPI.sln" /property:Configuration=Release')
			echo "Build Execution";
			stageName = "MSBUILD"
		}
		stage("Test") {
			stageName = "Test Unitario"
			Test1 = bat (script:'"C:\\Program Files (x86)\\Microsoft Visual Studio 11.0\\Common7\\IDE\\CommonExtensions\\Microsoft\\TestWindow\\vstest.console.exe" "C:\\Program Files (x86)\\Jenkins\\jobs\\Test_free_git\\workspace\\Medgrupo.RestfulService.Tests\\Ordered\\TrocaDevice.orderedtest"')			
			
			if(Test1 == 0){
				Test2 = bat (script:'"C:\\Program Files (x86)\\Microsoft Visual Studio 11.0\\Common7\\IDE\\CommonExtensions\\Microsoft\\TestWindow\\vstest.console.exe" "C:\\Program Files (x86)\\Jenkins\\jobs\\Test_free_git\\workspace\\Medgrupo.RestfulService.Tests\\bin\\Release\\Medgrupo.RestfulService.Tests.dll"', returnStatus: true)
			}
			else{
				currentBuild.result = "FAILED"
				RegexTest = ${BUILD_LOG_REGEX, regex="^Failed*",showTruncatedLines = false, maxMatches = 1,  linesAfter= 2}
				notifyBuild(currentBuild.result, stageName)	
				exit ;
			}
			if(Test1 == 0 & Test2 == 0){
				echo " Test unitario ok"
			}
			else{
				currentBuild.result = "FAILED"
				RegexTest = ${BUILD_LOG_REGEX, regex="^Failed*",showTruncatedLines = false, maxMatches = 1,  linesAfter= 2}
				notifyBuild(currentBuild.result, stageName)				
			}
			
		}	
		
		stage("Backup"){       
			bat (script: '"powershell" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\backup_script.ps1"' )
			stageName = "Backup"
		}
		stage("DLL") {
			String DLLS = "E:\\ScriptsJenkins\\Scripts\\git_scripts\\DLL_separator.ps1";
			bat (script: 'powershell "'+DLLS+'"')
			stageName = "Preparar Publish"
		}	
		stage("UploadFTP"){       
			bat (script: '"powershell" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\UPLOAD_FTP.ps1"')
			stageName = "UploadFTP"
		}
		
		stage("RunScopeAntes"){

			RunScopeOk = bat (script: '"C:\\Python27\\python.exe" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\runscope_python.py"', returnStatus: true)
			if(RunScopeOk == 0){
				notifyBuild(currentBuild.result, "RunScope Antes")
			}
			else{
				currentBuild.result = "FAILED"
				notifyBuild(currentBuild.result, "RunScope Antes")
			}
		}
		
		stage("Rollback"){
			if (RunScopeOk != 0){
				echo "Rollback";
				def powerSRollback1 = bat (script: '"powershell" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\rollback.ps1"', returnStatus: true)
				if(powerSRollback1 == 0){
					echo "ok";
					notifyBuild(currentBuild.result, "Rollback" )
				}else{
					echo "Erro no Rollback";
					currentBuild.result = "FAILED"
					notifyBuild(currentBuild.result, "Rollback" )
					exit;
				}
			}
			else{
				echo "Sem Rollback";
			}
		}
	
		stage("RunScopeDepoisDoLoadBalance"){
			sleep time: 6, unit: 'MINUTES';
			RunScopeDepoisDoLoadBalanceOk = bat (script: '"C:\\Python27\\python.exe" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\runscope_python.py"', returnStatus: true)
			if(RunScopeDepoisDoLoadBalanceOk == 0){
				notifyBuild(currentBuild.result, "RunScope Depois Do LoadBalance")
			}
			else{
				currentBuild.result = "FAILED"
				notifyBuild(currentBuild.result, "RunScope Depois Do LoadBalance")
			}
		}
		
		stage("RollbackDepoisDoLoadBalance"){
			if (RunScopeDepoisDoLoadBalanceOk != 0){
				echo "RollbackDepoisDoLoadBalance";
				def powerSRollback2 = bat (script: '"powershell" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\rollback.ps1"', returnStatus: true)
				if(powerSRollback2 == 0){
					echo "RollbackDepoisDoLoadBalance ok";
					notifyBuild(currentBuild.result, "Rollback Depois Do LoadBalance")
				}else{
					currentBuild.result = "FAILED"
					notifyBuild(currentBuild.result, "Rollback Depois Do LoadBalance")
				}
			}
			else{
				echo "Sem RollbackDepoisDoLoadBalance";
			}
		}

  } catch (e) {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
	notifyBuild(currentBuild.result,stageName )
    throw e
  } 
}
 
def notifyBuild(String buildStatus = 'STARTED', String stageName) {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'
 
  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: ${stageName} "
  def summary = "${subject}: Job rodou usando a (${env.BRANCH_NAME})"
  def details = """<p>${stageName}: ${buildStatus}</p>
  <p>Branch: Branch utilizada para o publish ${env.BRANCH_NAME}</p>
  <p>Por favor verifique o log para mais informacoes</p>
  <hr>
  <p>${RegexTest}<p>
  </br>
 <p><img src="https://itisatechiesworld.files.wordpress.com/2015/01/cool-jenkins2x3.png?w=200"></p>"""
 
  // Override default values based on build status
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
 
  // Send notifications
  slackSend (color: colorCode, message: summary)
    //email
  emailext (
	  mimeType: 'text/html',
	  attachLog: true,
      subject: subject,
      body: details,
      to: "aline.campos@ventron.com.br"
    ) 
}