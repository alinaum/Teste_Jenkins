
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

String RUNSCOPE_TRIGGER 	= "https://api.runscope.com/radar/64e1751e-9e93-46ce-a9fc-156f9c7f4490/trigger";
String RUNSCOPE_TESTE     	= "https://api.runscope.com/buckets/wynpst2ckqyc/tests/c5827e2b-2866-406d-9da7-cc9dbadc3055/results/";
String RUNSCOPE_TOKEN     	= "Bearer 0c4f724e-5125-4886-a657-f26e0f9db627";


String RunScopeOk 							= "";
String RunScopeDepoisDoLoadBalanceOk 		= "";

node {
     
	
    stage("RunScope"){
        String url = RUNSCOPE_TRIGGER;
        def objRest = ChamaRest(url, RUNSCOPE_TOKEN);
        String idTrigger = objRest.data.runs.test_run_id[0].toString();
        RunScopeOk = ChamaRestTeste(idTrigger, RUNSCOPE_TESTE, RUNSCOPE_TOKEN);
        while(RunScopeOk != "pass" && RunScopeOk != "fail")
        {
          RunScopeOk = ChamaRestTeste(idTrigger, RUNSCOPE_TESTE, RUNSCOPE_TOKEN);
        }
		
		if(RunScopeOk == "pass"){
			echo "ok";
		}
		else
		{
			echo "Erro no RunScope";
			exit;
		}
    }
	
	stage("E-mail RunScope"){
		if(RunScopeOk == "pass"){
			emailext attachLog: true, body: 'Feito com sucesso.', subject: "Passou primerio runscope", to: 'aline.campos@ventron.com';
		}
		else{
			emailext attachLog: true, body: 'Falhou.', subject: "Falhou runscope", to: 'aline.campos@ventron.com';
		}
	}

	stage("RunScopeDepoisDoLoadBalance"){
		sleep time: 6, unit: 'MINUTES';
        String url = RUNSCOPE_TRIGGER;
        def objRest = ChamaRest(url, RUNSCOPE_TOKEN);
        String idTrigger = objRest.data.runs.test_run_id[0].toString();
        RunScopeDepoisDoLoadBalanceOk = ChamaRestTeste(idTrigger, RUNSCOPE_TESTE, RUNSCOPE_TOKEN);
        while(RunScopeDepoisDoLoadBalanceOk != "pass" && RunScopeDepoisDoLoadBalanceOk != "fail")
        {
          RunScopeDepoisDoLoadBalanceOk = ChamaRestTeste(idTrigger, RUNSCOPE_TESTE, RUNSCOPE_TOKEN);
        }
		
		if(RunScopeDepoisDoLoadBalanceOk == "pass"){
			echo "RunScope ok";
		}
		else
		{
			echo "Erro no RunScope";
			exit;
		}
    }
	
	stage("E-mail RunScope depois do LoadBalance Homologação"){
		if(RunScopeDepoisDoLoadBalanceOk == "pass"){
			emailext attachLog: true, body: 'Feito com sucesso.', subject: "Passou segundo runscope", to: 'aline.campos@ventron.com';
		}
		else{
			emailext attachLog: true, body: 'Falhou.', subject: "Falhou runscope", to: 'aline.campos@ventron.com';
		}
	}
	


}

 
public def ChamaRest(def url, String token){
    URL object = new  URL(url);

    HttpURLConnection connection = (HttpURLConnection) object
            .openConnection();
    // int timeOut = connection.getReadTimeout();
    connection.setReadTimeout(60 * 1000);
    connection.setConnectTimeout(60 * 1000);

    connection.setRequestProperty("Authorization", token);
    int responseCode = connection.getResponseCode();
    //String responseMsg = connection.getResponseMessage();

    if (responseCode == 200 || responseCode == 201) {
        InputStream inputStr = connection.getInputStream();
        
        String encoding = connection.getContentEncoding() == null ? "UTF-8"
                : connection.getContentEncoding();
        jsonResponse = IOUtils.toString(inputStr, encoding);
        
      def jsonSlurper = new JsonSlurper()
      def obj = jsonSlurper.parseText(jsonResponse) 
      //println(obj.data.result);
      return obj;
    }
}

public String ChamaRestTeste(def teste, String urlTeste, String token){
    URL object = new  URL(urlTeste+ teste);

    HttpURLConnection connection = (HttpURLConnection) object
            .openConnection();
    // int timeOut = connection.getReadTimeout();
    connection.setReadTimeout(60 * 1000);
    connection.setConnectTimeout(60 * 1000);

    connection.setRequestProperty("Authorization", token);
    int responseCode = connection.getResponseCode();
    //String responseMsg = connection.getResponseMessage();

    if (responseCode == 200) {
        InputStream inputStr = connection.getInputStream();
        
        String encoding = connection.getContentEncoding() == null ? "UTF-8"
                : connection.getContentEncoding();
        jsonResponse = IOUtils.toString(inputStr, encoding);
        
      def jsonSlurper = new JsonSlurper()
      def obj = jsonSlurper.parseText(jsonResponse) 
      //println(obj.data.result);
      return obj.data.result;
    }
}
