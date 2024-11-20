```java
requestResponse.annotations().setNotes(requestResponse.host());

return "";
```


Give any response that discloses Telerik UI Encription keys: https://github.com/PortSwigger/BChecks/blob/main/vulnerabilities-CVEd/CVE-2017-9248_TelerikUI_Encryption_Keys_Disclosure.bcheck
```java
if (requestResponse.hasResponse() && requestResponse.response().contains("Telerik.Web.UI.WebResource.axd", false)) {
    // List of version strings
    List<String> versions = List.of(
        "2007.1423", "2007.1521", "2007.1626", "2007.2918", "2007.21010", "2007.21107",
        "2007.31218", "2007.31314", "2007.31425", "2008.1415", "2008.1515", "2008.1619",
        "2008.2723", "2008.2826", "2008.21001", "2008.31105", "2008.31125", "2008.31314",
        "2009.1311", "2009.1402", "2009.1527", "2009.2701", "2009.2826", "2009.31103",
        "2009.31208", "2009.31314", "2010.1309", "2010.1415", "2010.1519", "2010.2713",
        "2010.2826", "2010.2929", "2010.31109", "2010.31215", "2010.31317", "2011.1315",
        "2011.1413", "2011.1519", "2011.2712", "2011.2915", "2011.31115", "2011.3.1305",
        "2012.1.215", "2012.1.411", "2012.2.607", "2012.2.724", "2012.2.912", "2012.3.1016",
        "2012.3.1205", "2012.3.1308", "2013.1.220", "2013.1.403", "2013.1.417", "2013.2.611",
        "2013.2.717", "2013.3.1015", "2013.3.1114", "2013.3.1324", "2014.1.225", "2014.1.403",
        "2014.2.618", "2014.2.724", "2014.3.1024", "2015.1.204", "2015.1.225", "2015.1.401",
        "2015.2.604", "2015.2.623", "2015.2.729", "2015.2.826", "2015.3.930", "2015.3.1111",
        "2016.1.113", "2016.1.225", "2016.2.504", "2016.2.607", "2016.3.914", "2016.3.1018",
        "2016.3.1027", "2017.1.118"
    );

    return versions.stream().anyMatch(version -> requestResponse.response().contains(version, false));
}

return false;
```


Give any response with a hidden input field. https://github.com/PortSwigger/BChecks/blob/main/vulnerability-classes/injection/Detecting%20hidden%20input%20fields%20for%20XSS.bcheck

```java
if (!requestResponse.hasResponse()){
    return false;
}

String regex = "<input type=\"hidden\" id=\"[0-9A-Za-z-_]*\"";
Pattern pattern = Pattern.compile(regex);

return requestResponse.response().contains(pattern);
```

Give any responses that have a leaked AWS access key ID. https://github.com/PortSwigger/BChecks/blob/main/examples/leaked-aws-token.bcheck

```java
if (! requestResponse.hasResponse()){
    return false;
}

String regex = "AKIA[0-9A-Z]{16}";
Pattern pattern = Pattern.compile(regex);

return requestResponse.response().contains(pattern);
```

Show me items that display cross-domain referrer leakage of a sessionid
```java
String refererHeader = requestResponse.request().hasHeader("Referrer") ? requestResponse.request().headerValue("Referer") : "";
if (refererHeader.isEmpty()) {
    return false;
}

try {
    java.net.URL refererUrl = new java.net.URL(refererHeader);
    java.net.URL targetUrl = new java.net.URL(requestResponse.host());
    
    // Check if the domains are different
    if (!refererUrl.getHost().equals(targetUrl.getHost())) {
        // Domains are different, now check for session ID in the referer URL
        String query = refererUrl.getQuery();
        if (query != null && query.contains("sessionid=")) {
            return true;  // Session ID is leaked to a different domain
        }
    }
} catch (java.net.MalformedURLException e) {
    // Handle error in URL parsing
    e.printStackTrace();
}

return false;  // No leakage found
```

Show items that could have a local file path manipulation vulnerability.
```java
// Example of checking a query parameter commonly used for file operations
String queryParam = "file";
List<String> fileParams = requestResponse.request().parameters().stream()
									.filter(param -> queryParam.equals(param.name()))
									.map(param -> param.value())
									.toList();

return fileParams.stream()
		.anyMatch(fileParam -> fileParam.contains("../") 
			|| fileParam.contains("./") || fileParam.contains(":")
			|| fileParam.contains("%00") || fileParam.startsWith("/")
		);

```

Show items that transmit passwords in cleartext
```java
HttpRequest request = requestResponse.request();

return !request.httpService().secure() 
		&& request.parameters().stream()
				  .filter(p -> "password".equals(p.name()))
				  .findAny()
				  .isPresent();
```

Show any responses that could contain a java serialized object
```java
if (!requestResponse.hasResponse())
{
    return false;
}

byte[] javaSerializationPrefix = new byte[]{(byte) 0xAC, (byte) 0xED, (byte) 0x00, (byte) 0x05};
return requestResponse.response()
			.toByteArray()
			.countMatches(ByteArray.byteArray(javaSerializationPrefix)) > 0;
```