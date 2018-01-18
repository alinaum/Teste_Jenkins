#!/usr/bin/env groovy
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
int returOrder = 0;
int returUnit = 0;
int returBack = 0;
int returDll = 0;
int returUpload = 0;
String stageName = "";

node {
    
    stage('Checkout') {
        checkout scm
		stageName = "Checkout"
    }
	
    stage("MSBUILD"){
		stageName = "MSBUILD"
		bat (script:'"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\MSBuild.exe" "C:\\Program Files (x86)\\Jenkins\\jobs\\Release-Branch\\workspace\\MedgrupoAPI.sln" /property:Configuration=Release')	
			
	}
	stage("Test") {
		stageName = "Testes"
		returOrder = bat (script:'"C:\\Program Files (x86)\\Microsoft Visual Studio 11.0\\Common7\\IDE\\CommonExtensions\\Microsoft\\TestWindow\\vstest.console.exe" "C:\\Program Files (x86)\\Jenkins\\jobs\\Release-Branch\\workspace\\Medgrupo.RestfulService.Tests\\Ordered\\TrocaDevice.orderedtest"', returnStatus: true)						
		if(returOrder != 0){
			currentBuild.result = "FAILED"
			notifyBuild(currentBuild.result, "Order Teste")
			error 'Order teste error'
		}
		else{
			returUnit = bat (script:'"C:\\Program Files (x86)\\Microsoft Visual Studio 11.0\\Common7\\IDE\\CommonExtensions\\Microsoft\\TestWindow\\vstest.console.exe" "C:\\Program Files (x86)\\Jenkins\\jobs\\Release-Branch\\workspace\\Medgrupo.RestfulService.Tests\\bin\\Release\\Medgrupo.RestfulService.Tests.dll"', returnStatus: true)
			if(returUnit != 0){
				currentBuild.result = "FAILED"
				notifyBuild(currentBuild.result, "Test Unitario")
				error 'Erro teste unitario'
			}
			else{
				echo "Test unitario ok"
			}
		}
		
		
	}	
	
	stage("Backup"){       
		returBack = bat (script: '"powershell" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\backup_script.ps1"', returnStatus: true )
		stageName = "Backup"
		if(returBack != 0){
			currentBuild.result = "FAILED"
			notifyBuild(currentBuild.result, "Backup")
			error 'Backup Error'
		}
		else{
			echo "Backup"
		}

	}
	stage("DLL") {
		String DLLS = "E:\\ScriptsJenkins\\Scripts\\git_scripts\\DLL_separator.ps1";
		returDll = bat (script: 'powershell "'+DLLS+'"', returnStatus: true)
		stageName = "Preparar Publish"
		if(returDll != 0){
			currentBuild.result = "FAILED"
			notifyBuild(currentBuild.result, "Preparar Publish")
			error 'Preparar Publish Error'
		}
		else{
			echo "Preparar Publish"
		}
	}	
	stage("UploadFTP"){       
		returUpload = bat (script: '"powershell" "E:\\ScriptsJenkins\\Scripts\\git_scripts\\UPLOAD_FTP.ps1"', returnStatus: true)
		stageName = "UploadFTP"
		if(returUpload != 0){
			currentBuild.result = "FAILED"
			notifyBuild(currentBuild.result, "UploadFTP")
			error 'UploadFTP Error'
		}
		else{
			echo "UploadFTP"
		}
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
				error "Rollback feito"
			}else{
				echo "Erro no Rollback";
				currentBuild.result = "FAILED"
				notifyBuild(currentBuild.result, "Rollback" )
				error 'Rollback Error'
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
				error 'Rollback Depois Do LoadBalance Feito'
				
			}else{
				currentBuild.result = "FAILED"
				notifyBuild(currentBuild.result, "Rollback Depois Do LoadBalance")
				error 'Rollback Depois Do LoadBalance Error'
			}
		}
		else{
			echo "Sem RollbackDepoisDoLoadBalance";
		}
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
  <p><img src="https://itisatechiesworld.files.wordpress.com/2015/01/cool-jenkins2x3.png?w=200"></p>
  </br>
 """
 
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