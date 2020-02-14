<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:apikit="http://www.mulesoft.org/schema/mule/apikit"
	xmlns:db="http://www.mulesoft.org/schema/mule/db"
	xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
	xmlns:spring="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-current.xsd
http://www.mulesoft.org/schema/mule/db http://www.mulesoft.org/schema/mule/db/current/mule-db.xsd
http://www.mulesoft.org/schema/mule/apikit http://www.mulesoft.org/schema/mule/apikit/current/mule-apikit.xsd
http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd">
	<db:mysql-config name="MySQL_Configuration" host="abc" port="882" user="beens" password="abc" database="abc" doc:name="MySQL Configuration"/>


<sub-flow name="Error_logs">
        <db:insert config-ref="MySQL_Configuration" doc:name="Logging Errors ">
			<db:parameterized-query><![CDATA[INSERT INTO Table_Name(FLOW_NAME,TIMESTAMP,ERROR_MESSAGE,ERROR_DETAILS) VALUES (#[flow.name],#[server.dateTime.format('yyyy-MM-dd HH:mm:ss')],#[org.mule.util.ExceptionUtils.getRootCauseMessage(exception)],#[org.mule.util.ExceptionUtils.getFullStackTrace(exception)])]]></db:parameterized-query>

        </db:insert>
    </sub-flow>
   
     <choice-exception-strategy   name="GlobalChoiceExceptionStrategy">
 	<catch-exception-strategy when="#[exception.causeMatches('java.sql.SQLException')]" doc:name="DataBaseConnectivity-Issues">

            <logger message="#[org.mule.util.ExceptionUtils.getRootCauseMessage(exception)]" level="INFO" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>

            <set-payload value="{ &quot;message&quot;: &quot;Database Connectivity Issue! &quot; } " doc:name="Displaying Response"/>

	</catch-exception-strategy>
	
	 <catch-exception-strategy doc:name="CatchAllExceptions">
            <set-property propertyName="http.status" value="500" doc:name="Property"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>
            <set-payload value="{ &quot;Exception&quot;: &quot;#[org.mule.util.ExceptionUtils.getRootCauseMessage(exception)]&quot; }  " doc:name="Set Payload"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>
           
    </catch-exception-strategy>
</choice-exception-strategy>
  
     <choice-exception-strategy   name="APIGlobalExceptionHandling-WithResponse">
 	
	<catch-exception-strategy when="#[exception.causeMatches('org.mule.module.apikit.exception.UnsupportedMediaTypeException')]" doc:name="415-(UnsupportedMediaType)">
            <logger message="#[exception.cause.message]" level="ERROR" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>
            <set-payload value="{ &quot;message&quot;: &quot;Unsupported media type&quot; }" doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

	</catch-exception-strategy>
	<catch-exception-strategy when="#[exception.causeMatches('org.mule.module.apikit.exception.NotAcceptableException')]" doc:name="406-(NotAcceptable)">
            <logger message="#[exception.cause.message]" level="ERROR" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>
            <set-payload value="{ &quot;message&quot;: &quot;Not acceptable&quot; }" doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

	</catch-exception-strategy>
	<catch-exception-strategy when="#[exception.causeMatches('java.nio.channels.UnresolvedAddressException')]" doc:name="UnresolvedAddressException">
            <logger message="#[exception.cause.message]" level="ERROR" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>
            <set-payload value="{ &quot;message&quot;: &quot;Unresolved Exception&quot; }" doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

	</catch-exception-strategy>
	
	<catch-exception-strategy when="#[exception.causeMatches('org.springframework.security.authentication.BadCredentialsException')]" doc:name="401(Basic-Authentication)">
            <logger message="#[exception.cause.message]" level="ERROR" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>


            <set-payload value="{ &quot;message&quot;: &quot;Basic authentication failed&quot; }" doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

	</catch-exception-strategy>
	<catch-exception-strategy when="#[exception.causeMatches('org.mule.api.security.UnauthorisedException')]" doc:name="Authorization">

            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>

            <set-payload value="{ &quot;message&quot;: &quot;No Authorization Found&quot; }" doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

	</catch-exception-strategy>
	<catch-exception-strategy when="#[exception.causeMatches('org.mule.module.apikit.exception.MethodNotAllowedException')]" doc:name="405-(MethodNotAllowed)">
            <logger message="#[exception.cause.message]" level="ERROR" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>
            <set-payload value="{ &quot;message&quot;: &quot;Method2 not allowed&quot; }" doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

</catch-exception-strategy>
	<catch-exception-strategy when="#[exception.causeMatches('org.mule.module.apikit.exception.NotFoundException')]" doc:name="404(Not Found)">

            <logger message="#[exception.cause.message]" level="ERROR" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Copy_of_Json Response"/>
            <set-payload value="{ &quot;message&quot;: &quot;Resource not found&quot; } " doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

</catch-exception-strategy>

	<catch-exception-strategy when="#[exception.causeMatches('java.sql.SQLException')]" doc:name="DataBaseConnectivity-Issues">

            <logger message="@1234#[org.mule.util.ExceptionUtils.getRootCauseMessage(exception)]" level="INFO" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>

            <set-payload value="{ &quot;message&quot;: &quot;Database Connectivity Issue! &quot; } " doc:name="Displaying Response"/>

	</catch-exception-strategy>
	
	<catch-exception-strategy when="#[exception.causeMatches('org.mule.api.MessagingException')]" doc:name="MessagingException">
            <logger message="#[exception.cause.message]" level="ERROR" doc:name="Logger"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>
            <set-payload value="{ &quot;message&quot;: &quot;Bad Request&quot; }" doc:name="Displaying Response"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

	</catch-exception-strategy>
		
	 <catch-exception-strategy doc:name="ALL_Remaining">
            <set-property propertyName="http.status" value="500" doc:name="Property"/>
            <set-property propertyName="Content-Type" value="application/json" doc:name="Json Response"/>
            <set-payload value="{ &quot;message&quot;: &quot;An error occured while processing your request&quot; }  " doc:name="Set Payload"/>
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>

    </catch-exception-strategy>
</choice-exception-strategy>
  
  
   <apikit:mapping-exception-strategy name="Mutual-apiKitGlobalExceptionMapping">
        <apikit:mapping statusCode="404">
            <apikit:exception value="org.mule.module.apikit.exception.NotFoundException" />
            <set-property propertyName="Content-Type" value="application/json" doc:name="Property"/>
           
				
            <set-payload value="{ &quot;message&quot;: &quot;Resource not found&quot; }" doc:name="Set Payload" />
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>
        </apikit:mapping>
        <apikit:mapping statusCode="405">
            <apikit:exception value="org.mule.module.apikit.exception.MethodNotAllowedException" />
            <set-property propertyName="Content-Type" value="application/json" doc:name="Property" />
            <set-payload value="{ &quot;message&quot;: &quot;Method not allowed&quot; }" doc:name="Set Payload" />
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>
        </apikit:mapping>
        <apikit:mapping statusCode="415">
            <apikit:exception value="org.mule.module.apikit.exception.UnsupportedMediaTypeException" />
            <set-property propertyName="Content-Type" value="application/json" doc:name="Property" />
            <set-payload value="{ &quot;message&quot;: &quot;Unsupported media type&quot; }" doc:name="Set Payload" />
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>
        </apikit:mapping>
        <apikit:mapping statusCode="406">
            <apikit:exception value="org.mule.module.apikit.exception.NotAcceptableException" />
            <set-property propertyName="Content-Type" value="application/json" doc:name="Property" />
            <set-payload value="{ &quot;message&quot;: &quot;Not acceptable&quot; }" doc:name="Set Payload" />
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>
        </apikit:mapping>
        <apikit:mapping statusCode="400">
            <apikit:exception value="org.mule.module.apikit.exception.BadRequestException" />
            <set-property propertyName="Content-Type" value="application/json" doc:name="Property" />
            <set-payload value="{ &quot;message&quot;: &quot;Bad request&quot; }" doc:name="Set Payload" />
            <async doc:name="Async">
                <flow-ref name="Error_logs" doc:name="Inserting Logs in DB"/>
            </async>
        </apikit:mapping>
    </apikit:mapping-exception-strategy>
   
</mule>
