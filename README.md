package test;

import infrastructure.Constants;
import infrastructure.Entity;
import infrastructure.Entity.Fields.Field;
import infrastructure.EntityMarshallingUtils;
import infrastructure.Response;
import infrastructure.RestConnector;
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URL;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import org.junit.Before;
import org.junit.Test;
import alm.AlmConnector;

public class TestExecutionStatusUpdate {

	AlmConnector alm;
	RestConnector conn;
	String strRunID;
	String TEST_SET_ID= "3004"; String TEST_ID = "7728";
	String TC_RUN_STATUS = "Failed";
	String TC_RUN_NAME = "Automation_Run";
	String TC_RUN_DURATION = "90";

	@Before
	public void loginAlm() throws Exception
	{
		alm = new AlmConnector();
		conn = RestConnector.getInstance();
		conn.init(new HashMap<String, String>(), Constants.HOST,Constants.DOMAIN, Constants.PROJECT);
		alm.login(Constants.USERNAME,Constants.PASSWORD);
	}

	public void testUriEncoding() throws URISyntaxException,MalformedURLException {
		final String add = "";
		URL url = new URL(add);

		URI uri = new URI(url.getProtocol(), url.getHost(), url.getPath(),
				url.getQuery(), null);

		System.out.println(uri.toASCIIString());
	}
	@Test
	public void RestAPIALMdriver() throws MalformedURLException, URISyntaxException, Exception
	{
		// Update Test case status in Test Lab - Execution Grid
		//UpdateTestInstanceStatus();
		// Update Test step status in Test Runs
		//Update_RunStepStatus();
		// Update Automation Run Name
		//Update_RunName();
		// Update Run Duration
		Update_RunDuration();
				
		alm.logout();
		alm = null;
	}

	public String GetTestInstanceID() throws Exception,URISyntaxException,MalformedURLException {	

		List<String> strmagicID = null;
		String strmagicIDvalue=null;
		conn.getQCSession();
		
		/*
		 * Get the Test Instance with Test-Set ID & Configuration Test ID .
		 */
		String entitytype = "test-instance";
		String testinstanceURl = conn.buildEntityCollectionUrl(entitytype);
		String testinstanceURLquery = "query=%7Bcycle-id["+TEST_SET_ID+"];test-id["+TEST_ID+"]%7D";

		Map<String, String> requestHeaders = new HashMap<String, String>();
		requestHeaders.put("Content-Type", "application/xml");
		requestHeaders.put("Accept", "application/xml");

		Response res = conn.httpGet(testinstanceURl,testinstanceURLquery, requestHeaders);

		// xml -> class instance

		String postedEntityReturnedXml = res.toString();

		String strtoreplace = readstring();
		String strTotalResults = null;		
		String[] tokens = postedEntityReturnedXml.split("<");

		for (String t : tokens)
		{
			if (t.contains("Entities TotalResults"))
			{
				Pattern p = Pattern.compile("\"([^\"]*)\"");
				Matcher m = p.matcher(t);
				while (m.find()) {
					strTotalResults=m.group(1);
					break;
				}
			}
		}
		strtoreplace =strtoreplace.replace("1", strTotalResults);
		postedEntityReturnedXml = postedEntityReturnedXml.replace(strtoreplace, "");
		postedEntityReturnedXml = postedEntityReturnedXml.replace("</Entities>", "");

		Entity entity = EntityMarshallingUtils.marshal(Entity.class,postedEntityReturnedXml);

		/* * To print all names available in entity test instance.*/

		List<Field> fields = entity.getFields().getField();
		for (Field field : fields) 
		{
			if ((field.getName()).equals("id"))
			{
				strmagicID =field.getValue();
				strmagicIDvalue =strmagicID.get(0);
				break;
			}			
		}
		return strmagicIDvalue;
	}

	public void UpdateTestInstanceStatus() throws Exception 
	{
		String magicvalue = GetTestInstanceID();
		System.out.println("Test Instance - ID :"+magicvalue);

		// Build an URL to connect to ALM
		String entitytype = "test-instance";
		String testinstanceURl = conn.buildEntityCollectionUrl(entitytype);
		testinstanceURl += "/"+magicvalue;

		String newEntityToUpdateUrl = testinstanceURl; 
		/*String newEntityToUpdateUrl =createEntity(testinstanceURl,Constants.entityToPostXml);*/

		String updatedField = "status";
		String updatedFieldInitialUpdateValue = TC_RUN_STATUS;

		String updatedEntityXml =generateSingleFieldUpdateXml(updatedField,updatedFieldInitialUpdateValue);
		update(newEntityToUpdateUrl,updatedEntityXml).toString();
	}

	public String GetRunID() throws Exception,URISyntaxException,MalformedURLException {	
		// To get the Latest Run ID

		List<String> strmagicRunID = null;
		String strmagicRunIDvalue=null;
		conn.getQCSession();
		/*
		 * Get the Test Instance with Test-Set Id & Configuration Test ID .
		 */
		String entitytype = "run";
		String testinstanceURl = conn.buildEntityCollectionUrl(entitytype);
		String testinstanceURLquery = "query=%7Bcycle-id["+TEST_SET_ID+"];test-id["+TEST_ID+"]%7D&fields=id,test-id,state,name,status,execution-date,duration,execution-time,cycle-id&page-size=1&order-by=%7Bexecution-date[DESC];execution-time[DESC]%7D";

		Map<String, String> requestHeaders = new HashMap<String, String>();
		requestHeaders.put("Content-Type", "application/xml");
		requestHeaders.put("Accept", "application/xml");

		Response res = conn.httpGet(testinstanceURl,testinstanceURLquery, requestHeaders);

		// xml -> class instance

		String postedEntityReturnedXml = res.toString();

		String strTotalResults = null;		
		String[] tokens = postedEntityReturnedXml.split("<");

		for (String t : tokens)
		{
			if (t.contains("Entities TotalResults"))
			{
				Pattern p = Pattern.compile("\"([^\"]*)\"");
				Matcher m = p.matcher(t);
				while (m.find()) {
					strTotalResults=m.group(1);
					break;
				}
			}
		}
		String strtoreplace = readstring();
		strtoreplace =strtoreplace.replace("1", strTotalResults);

		postedEntityReturnedXml = postedEntityReturnedXml.replace(strtoreplace, "");
		postedEntityReturnedXml = postedEntityReturnedXml.replace("</Entities>", "");

		Entity entity = EntityMarshallingUtils.marshal(Entity.class,postedEntityReturnedXml);

		/* * To print all names available in entity test instance.*/

		List<Field> fields = entity.getFields().getField();
		for (Field field : fields) 
		{
			if ((field.getName()).equals("id"))
			{
				strmagicRunID =field.getValue();
				strmagicRunIDvalue =strmagicRunID.get(0);
				break;
			}			
		}

		return strmagicRunIDvalue;
	}

	public void Update_RunStepStatus() throws Exception,URISyntaxException,MalformedURLException {	

		String  strRunStepID=null;
		conn.getQCSession();

		/*
		 * Get the Test Instance with Test-Set Id & Configuration Test ID .
		 */
		strRunID=GetRunID();
		System.out.println("Run-ID :"+strRunID);

		// Build an URL to connect to ALM
		String entitytype = "run";
		String testinstanceURl = conn.buildEntityCollectionUrl(entitytype);
		testinstanceURl += "/"+strRunID+"/"+"run-steps"+"/";

		Map<String, String> requestHeaders = new HashMap<String, String>();
		requestHeaders.put("Content-Type", "application/xml");
		requestHeaders.put("Accept", "application/xml");

		Response res = conn.httpGet(testinstanceURl,null, requestHeaders);

		// xml -> class instance

		String postedEntityReturnedXml = res.toString();
		String[] tokens = postedEntityReturnedXml.split("<");

		boolean flag=false;

		for (String i : tokens)
		{
			String valueBeforeTheExpectedValue = readstring_remove();
			if(i.equals(valueBeforeTheExpectedValue)){
				flag=true;
				continue;
			} 
			if (flag)
			{ 
				int output = extractInt(i);
				strRunStepID = Integer.toString(output);
				flag=false;
				if(!strRunStepID.equals("0")){
					System.out.println(strRunStepID);
					String newEntityToUpdateUrl = testinstanceURl+strRunStepID;

					String updatedField_runstatus = "status";
					String updatedFieldInitialUpdateValue_runstatus = TC_RUN_STATUS;

					String updatedEntityXml_RunStatus =generateSingleFieldUpdateXml_runstatus(updatedField_runstatus,updatedFieldInitialUpdateValue_runstatus);
					update(newEntityToUpdateUrl,updatedEntityXml_RunStatus).toString();

				}
			}
		}
	}

	public void Update_RunName() throws Exception,URISyntaxException,MalformedURLException {	

		conn.getQCSession();

		// Build an URL to connect to ALM
		String entitytype = "run";
		String testinstanceURl = conn.buildEntityCollectionUrl(entitytype);
		testinstanceURl += "/"+strRunID;

		Map<String, String> requestHeaders = new HashMap<String, String>();
		requestHeaders.put("Content-Type", "application/xml");
		requestHeaders.put("Accept", "application/xml");

		String updatedField_runName = "name";
		String updatedFieldInitialUpdateValue_runName = TC_RUN_NAME;

		String updatedEntityXml_RunName =generateSingleFieldUpdateXml_runName(updatedField_runName,updatedFieldInitialUpdateValue_runName);
		update(testinstanceURl,updatedEntityXml_RunName).toString();
	}
	
	public void Update_RunDuration() throws Exception,URISyntaxException,MalformedURLException {	

		conn.getQCSession();
		
		String strval = "";

		String[] strrun = strval.split(";");
		for (String i:strrun)
		{
		// Build an URL to connect to ALM
		String entitytype = "run";
		String testinstanceURl = conn.buildEntityCollectionUrl(entitytype);
		testinstanceURl += "/"+i;

		Map<String, String> requestHeaders = new HashMap<String, String>();
		requestHeaders.put("Content-Type", "application/xml");
		requestHeaders.put("Accept", "application/xml");

		String updatedField_duration = "duration";
		int rand=getRandomNumber(60,180);
		String updatedFieldInitialUpdateValue_duration = Integer.toString(rand);

		String updatedEntityXml_RunName =generateSingleFieldUpdateXml_runName(updatedField_duration,updatedFieldInitialUpdateValue_duration);
		update(testinstanceURl,updatedEntityXml_RunName).toString();
		}
		
	}
	
	public int getRandomNumber(int min, int max) {
	    return (int) Math.floor(Math.random() * (max - min + 1)) + min;
	}

	public static int extractInt(String str) {
		Matcher matcher = Pattern.compile("\\d+").matcher(str);

		if (!matcher.find())
			throw new NumberFormatException("For input string [" + str + "]");

		return Integer.parseInt(matcher.group());
	}

	public String createEntity(String collectionUrl, String postedEntityXml)
			throws Exception {

		Map<String, String> requestHeaders = new HashMap<String, String>();
		requestHeaders.put("Content-Type", "application/xml");
		requestHeaders.put("Accept", "application/xml");

		// As can be seen in the implementation below, creating an entity
		//is simply posting its xml into the correct collection.
		Response response = conn.httpPost(collectionUrl,
				postedEntityXml.getBytes(), requestHeaders);

		Exception failure = response.getFailure();
		if (failure != null) {
			throw failure;
		}

		/*
	         Note that we also get the xml of the newly created entity.
	         at the same time we get the url where it was created in a
	         location response header.
		 */
		String entityUrl =response.getResponseHeaders().get("Location").iterator().next();

		return entityUrl;
	}
	/**
	 * @param field
	 *            the field name to update
	 * @param value
	 *            the new value to use
	 * @return an xml that can be used to update an entity's single
	 *          given field to given value
	 */
	private static String generateSingleFieldUpdateXml(String field,String value) {
		return "<Entity Type=\"test-instance\"><Fields>"
				+ Constants.generateFieldXml(field, value)
				+ "</Fields></Entity>";
	}

	private static String generateSingleFieldUpdateXml_runstatus(String field,String value) {
		return "<Entity Type=\"run-step\"><Fields>"
				+ Constants.generateFieldXml(field, value)
				+ "</Fields></Entity>";
	}

	private static String generateSingleFieldUpdateXml_runName(String field,String value) {
		return "<Entity Type=\"run\"><Fields>"
				+ Constants.generateFieldXml(field, value)
				+ "</Fields></Entity>";
	}

	private Response update(String entityUrl, String updatedEntityXml)throws Exception 
	{
		Map<String, String> requestHeaders = new HashMap<String, String>();
		requestHeaders.put("Content-Type", "application/xml");
		requestHeaders.put("Accept", "application/xml");

		Response putResponse =conn.httpPut(entityUrl, updatedEntityXml.getBytes(), requestHeaders);

		if (putResponse.getStatusCode() != HttpURLConnection.HTTP_OK) {
			throw new Exception(putResponse.toString());
		}
		return putResponse;
	}

	public String readstring() throws IOException
	{
		BufferedReader br = null;
		FileReader fr = null;
		String filepath =System.getProperty("user.dir");

		String FILENAME = filepath+"//StringinXml.txt";
		String sCurrentLine = null;

		try {
			fr = new FileReader(FILENAME);
			br = new BufferedReader(fr);

			sCurrentLine = br.readLine();

		} catch (FileNotFoundException e) 
		{
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return sCurrentLine;
	}
	public String readstring_remove() throws IOException

	{
		BufferedReader br = null;
		FileReader fr = null;
		String filepath =System.getProperty("user.dir");

		String FILENAME = filepath+"//RemoveString.txt";
		String sCurrentLine = null;

		try {
			fr = new FileReader(FILENAME);
			br = new BufferedReader(fr);

			sCurrentLine = br.readLine();

		} catch (FileNotFoundException e) 
		{
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return sCurrentLine;
	}
}
