# Deprecation notice: New Version Available

This version of the SDK has been deprecated and replaced with the newly reconstructed SDK.

For the latest API features and improved integration, please visit our updated repository at https://github.com/AfterShip/tracking-sdk-java and follow the provided instructions.


# aftership-sdk-java

[![Maven Central](https://img.shields.io/maven-central/v/com.slack.api/slack-api-client.svg?label=Maven%20Central)](https://search.maven.org/search?q=g:com.aftership%20a:aftership-sdk)
[![AfterShip SDKs channel](https://aftership-sdk-slackin.herokuapp.com/badge.svg)](https://aftership-sdk-slackin.herokuapp.com/)
[![Build Status](https://api.travis-ci.org/chenjunbiao/aftership-sdk-java.svg?branch=v2)](https://travis-ci.org/github/chenjunbiao/aftership-sdk-java)

The Java SDK of `AfterShip API`, please see API documentation in: https://www.aftership.com/docs/api/4

Requirements:

- JDK 1.8 or superior.

## Installation

### Maven

```text
<dependency>
  <groupId>com.aftership</groupId>
  <artifactId>aftership-sdk</artifactId>
  <version>4.0.1</version>
</dependency>
```

### Gradle

```text
implementation "com.aftership:aftership-sdk:4.0.1"
```


## Quick Examples

### Main Steps

The following code example shows the `three main steps` to use aftership-sdk-java:

1. Create `AfterShip` Object.

```java
AftershipOption option = new AftershipOption();
option.setEndpoint("https://api.aftership.com/tracking/2023-10");

AfterShip afterShip = new AfterShip("YOUR_API_KEY", option);

// if add aes sign
AfterShip afterShip = new AfterShip("YOUR_API_KEY", AuthenticationType.AES, "YOUR_API_SECRET", option);
```

2. Get the Endpoint Interface and call the method, then return the object.

```java 
CourierList courierList = afterShip.getCourierEndpoint().listCouriers();
```

3. Handling `Data` or `AftershipException`  or `RateLimit`

```java
try {
  AftershipOption option = new AftershipOption();
  option.setEndpoint("https://api.aftership.com/tracking/2023-10");
  AfterShip afterShip = new AfterShip("YOUR_API_KEY", option);
  CourierList courierList = afterShip.getCourierEndpoint().listCouriers();
  // using data
  System.out.println(courierList);
} catch (SdkException | RequestException e) {
  // handle SdkException, RequestException
  System.out.println(e.getType());
  System.out.println(e.getMessage());
  System.out.println(e.getData());
} catch (ApiException e) {
  // handle ApiException
  if (e.isTooManyRequests()) {
    // Analyze RateLimit when TooManyRequests occur
    System.out.println(e.getRateLimit().getReset());
    System.out.println(e.getRateLimit().getLimit());
    System.out.println(e.getRateLimit().getRemaining());
    return;
  }
  System.out.println(e.getType());
  System.out.println(e.getCode());
  System.out.println(e.getMessage());
}
```
## Troubleshoot
### Received fatal alert: handshake_failure

> The SSL Protocol of `api.aftership.com` is TLSV1.2, and the Cipher is AES256-SHA.
It is a known issue due to this reason:

#### The Cipher Suite of your JVM not include `TLS_AES_256_GCM_SHA384`

You can check the CipherSuite according to this [article](https://help.mulesoft.com/s/article/How-to-list-the-Cipher-Suite-of-JVM-and-the-Cipher-used-on-a-handshake-with-endpoint)

Execute `java CipherSuite api.aftership.com:443`, under normal circumstances, it should output
```
CipherSuite used on the handshake:TLS_AES_256_GCM_SHA384
```

Otherwise, you should check the cipherSuite of your JVM.
1. Make sure `crypto.policy=unlimited` is effective in the jre/lib/securityjava.security
2. If your JDK version number is lower than 7u171 or 8u161, please follow this [oracle article](https://docs.oracle.com/cd/E60058_01/PDF/8.0.6.x/8.0.6.0.0/AG_HTML/Enabling_Unlimited_Cryptographic_Policy_for_Java.htm) to download JCE.

#### SSL protocol of your client is lower than TLSV1.2

You can check the protocol version used during the SSL negotiation by capturing transport packet

## Error Handling

There are 4 kinds of exception

- AftershipException
- SdkException
- RequestException
- ApiException

Error object of this SDK contain fields:

- `type` - **Require** - type of the error, **please handle each error by this field**

- `message` - **Optional** - detail message of the error

- `code` - **Optional** - error code for API Error

  You can find tips for Aftership's error codes in here: https://www.aftership.com/docs/tracking/quickstart/request-errors

  If it's Aftership's API Error, get code to confirm the cause of the error:

  ```java
  catch (AftershipException e){
    if(e.isApiError()){
      System.out.println(e.getCode());
    }
  }
  ```

- `data` - **Optional** - data lead to the error

  The debug Data is a `Map<String, Object>` object that can get the call parameters. 

  The following data may be available:

  ```java 
  catch (AftershipException e){
    System.out.println(e.getData());
    // or
    System.out.println(e.getData().get(DEBUG_DATA_KEY_REQUEST_CONFIG));
    System.out.println(e.getData().get(DEBUG_DATA_KEY_REQUEST_HEADERS));
    System.out.println(e.getData().get(DEBUG_DATA_KEY_REQUEST_DATA));
    System.out.println(e.getData().get(DEBUG_DATA_KEY_RESPONSE_BODY));
  }
  ```
### AftershipException

AftershipException is the base class for all exception classes and can capture it for uniform handling.

See the `Rate Limiter` section for TooManyRequests in ApiException.

```java
catch (AftershipException e){
  if(e.isApiError()){
    System.out.println(e.getCode());
    if(e.isTooManyRequests() && e instanceof ApiException){
      System.out.println(((ApiException)e).getRateLimit());
    }
  }
  System.out.println(e.getType());
  System.out.println(e.getMessage());
  System.out.println(e.getData());
}
```

### SdkException

Exception return by the SDK instance, mostly invalid param type when calling constructor or endpoint method
error.Type is one of [ErrorType](https://github.com/AfterShip/aftership-sdk-java/blob/v2/aftership-sdk/src/main/java/com/aftership/sdk/error/ErrorType.java)
**Throw** by the SDK instance

```java
try {
  AftershipOption option = new AftershipOption();
  option.setEndpoint("https://api.aftership.com/tracking/2023-10");
  AfterShip afterShip =
      new AfterShip(null, option);
} catch (SdkException e) {
  System.out.println(e.getMessage());
}

/* ConstructorError: Invalid API key; type: ConstructorError */
```

**Throw** by endpoint method

```java
try {
  AftershipOption option = new AftershipOption();
  option.setEndpoint("https://api.aftership.com/tracking/2023-10");
  AfterShip afterShip =
      new AfterShip("YOUR_API_KEY", option);
  afterShip.getTrackingEndpoint().getTracking("", null);
} catch (SdkException e) {
  System.out.println(e.getMessage());
}

/* ConstructorError: Required tracking id; type: ConstructorError */
```

### RequestException

Error return by the `request` module 
`error.Type` could be `HandlerError`, etc.  

```java
try {
  AftershipOption option = new AftershipOption();
  option.setEndpoint("https://api.aftership.com/tracking/2023-10");
  AfterShip afterShip =
      new AfterShip("YOUR_API_KEY", option);
  afterShip.getTrackingEndpoint().getTracking("abc", null);
} catch (RequestException e) {
  System.out.println(e.getMessage());
}

/* null; type: HandlerError; */
```

### ApiException

Error return by the AfterShip API  
`error.Type` should be the same as https://www.aftership.com/docs/api/4/errors

```java
try {
  AftershipOption option = new AftershipOption();
  option.setEndpoint("https://api.aftership.com/tracking/2023-10");
  AfterShip afterShip =
      new AfterShip("YOUR_API_KEY", option);
  afterShip.getTrackingEndpoint().getTracking("abc", null);
} catch (ApiException e) {
  System.out.println(e.getMessage());
}

/* The value of `id` is invalid.; type: BadRequest; code: 4015; */
```

### Rate Limiter

To understand AfterShip rate limit policy, please see `limit` session in https://www.aftership.com/docs/api/4

You can get the recent rate limit by `ApiException.getRateLimit()`. 

```java
try {
  AftershipOption option = new AftershipOption();
  option.setEndpoint("https://api.aftership.com/tracking/2023-10");
  AfterShip afterShip =
    new AfterShip("YOUR_API_KEY", option);
  afterShip.getCourierEndpoint().listCouriers();
} catch (SdkException | RequestException e) {
  System.out.println(e.getType());
} catch (ApiException e) {
  if (e.isTooManyRequests()) {
    System.out.println(e.getRateLimit().getReset());
    System.out.println(e.getRateLimit().getLimit());
    System.out.println(e.getRateLimit().getRemaining());
  }
}
// 1589869159
// 10
// 9
```

### Timeout

When creating an Aftership object, you can define the timeout time for http requests, Of course, use the default value of 20 seconds when not set. The unit is milliseconds.

```java 
AftershipOption option = new AftershipOption();
option.setEndpoint("https://api.aftership.com/tracking/2023-10");
option.setCallTimeout(10 * 1000);
option.setConnectTimeout(10 * 1000);
option.setReadTimeout(10 * 1000);
option.setWriteTimeout(10 * 1000);
AfterShip afterShip = new AfterShip(SampleUtil.getApiKey(), option);
```

### Retry
Retry policy implemented using OKHttp interceptor. Allows to customize the delay interval, maximum retry interval, and number of retries; implements an exponential backoff delay strategy.
#### 1 RetryInterceptor
```java
public class RetryInterceptor implements Interceptor {

  private long initialDelay;              // Initial retry delay in milliseconds
  private long maxDelay;                  // Maximum retry delay in milliseconds
  private int maxRetries;                 // Maximum number of retries
  private double backoffMultiplier;       // Exponential backoff multiplier. Should ideally be greater than 1.
  private double jitterFactor;            // Jitter factor for randomized delay
  private List<RetryCondition> retryConditions;  // Customizable retry condition

}
```

- `initialDelay`: The initial retry interval. Default is `500ms`.
- `maxDelay`: The maximum delay. After reaching the maximum delay, subsequent retries will use the same value. Default is `18s` (default timeout is 20s).
- `maxRetries`: The maximum number of retries. Default is `10`.
- `backoffMultiplier`: The exponential backoff multiplier. After each retry attempt, the retry interval will be multiplied by this value, resulting in exponential growth of delays. Default is `1.6`.
- `jitterFactor`: The jitter factor. Adds positive or negative jitter to calculated exponential backoff delay, helping to distribute retry requests and reduce server load. Default is `0.2`.
- `retryConditions`: Custom retry conditions allow customers to define their own retry criteria. Default is to retry on exceptions or when response status code is `≥500`.

#### 2 backoff()
Calculates the exponential backoff delay based on the number of retry attempts.

```java
public class RetryInterceptor implements Interceptor {
  /**
   * Calculates the exponential backoff delay for the specified retry attempt number, using the configured
   * backoff multiplier and maximum delay, and adds randomized jitter to the delay.
   *
   * @param retries The current retry attempt number.
   * @return The calculated delay before the next retry attempt, in milliseconds.
   */
  private long backoff(int retries) {
    // Calculate exponential backoff delay: initialDelay * (backoffMultiplier ^ retries)
    long delay = (long) (initialDelay * Math.pow(backoffMultiplier, retries));
    // Calculate jitter: jitterFactor * delay * random number in [-1, 1) range
    double jitter = jitterFactor * delay * (Math.random() - 0.5) * 2;
    // Add the exponential backoff delay and jitter to get the final delay
    long finalDelay = (long) (delay + jitter);

    // Ensure the final delay does not exceed the maximum delay
    if (finalDelay > maxDelay) {
      finalDelay = maxDelay;
    }

    return finalDelay;
  }
```

Assuming retries=10, initialDelay=500, backoffMultiplier=1.6, jitterFactor=0.2, and maxDelay=18,000; the time intervals required for each retry are as follows:

| retries | delay |
| --- | --- |
| 1 | 586 |
| 2 | 787 |
| 3 | 1342 |
| 4 | 2304 |
| 5 | 3219 |
| 6 | 4592 |
| 7 | 7293 |
| 8 | 12020 |
| 9 | 18000 |
| 10 | 18000 |

#### 3 Example
```java
public class RetrySample {
  public static void main(String[] args) {
    AftershipOption option = SampleUtil.getAftershipOption();

    RetryOption retryOption = new RetryOption();
    retryOption.setRetryDelay(500);
    retryOption.setRetryMaxDelay(18 * 1000L);
    retryOption.setRetryCount(10);
    retryOption.setRetryConditions(Arrays.asList((
      (response, exception) -> exception != null || (response != null && response.code() >= 500))));

    option.setRetryOption(retryOption);
    AfterShip afterShip = new AfterShip(SampleUtil.getApiKey(), option);
  }
}
```
- Retry interval: 500ms
- Maximum retry interval: 18s
- Number of retries: 10
- Retry conditions: exception != null || (response != null && response.code() >= 500)
- backoffMultiplier and jitterFactor are set to 1.6 and 0.2 respectively by default


### Shutdown

We recommend using only one AfterShip object to request interfaces of the API, and calling the `shutdown` method if you want to completely clean up all network resources when you shut down your system. In other cases, not running shutdown method has no effect.

```java
try {
  afterShip.getCourierEndpoint().listCouriers();
} catch (RequestException | ApiException e) {
  e.printStackTrace();
} finally {
  try {
    afterShip.shutdown();
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```

## Examples

### /couriers

- #### listCouriers [GET /couriers]

```java 
try {
  CourierList courierList = afterShip.getCourierEndpoint().listCouriers();
  System.out.println(courierList.getTotal());
  System.out.println(courierList.getCouriers().get(0).getName());
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### listAllCouriers [GET /couriers/all]

```java 
try {
  CourierList courierList = afterShip.getCourierEndpoint().listAllCouriers();
  System.out.println(courierList.getTotal());
  System.out.println(courierList.getCouriers());
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### detectCouriers [POST /couriers/detect]

```java
CourierDetectTracking tracking = new CourierDetectTracking();
tracking.setTrackingNumber("906587618687");
CourierDetectRequest courierDetectRequest = new CourierDetectRequest(tracking);

try {
  CourierDetectList courierDetectList =
    afterShip.getCourierEndpoint().detectCouriers(courierDetectRequest.getTracking());

  System.out.println(courierDetectList.getTotal());
  System.out.println(courierDetectList.getCouriers());
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

### /trackings

- #### createTracking [POST /trackings]

```java
NewTracking newTracking = new NewTracking();
// slug from listAllCouriers()
newTracking.setSlug(new String[] {"acommerce"});
newTracking.setTrackingNumber("1234567890");
newTracking.setTitle("Title Name");
newTracking.setSmses(new String[] {"+18555072509", "+18555072501"});
newTracking.setEmails(new String[] {"email@yourdomain.com", "another_email@yourdomain.com"});
newTracking.setOrderId("ID 1234");
newTracking.setOrderIdPath("http://www.aftership.com/order_id=1234");
newTracking.setCustomFields(
  new HashMap<String, String>(2) {
    {
      put("product_name", "iPhone Case");
      put("product_price", "USD19.99");
    }
  });
newTracking.setLanguage("en");
newTracking.setOrderPromisedDeliveryDate("2019-05-20");
newTracking.setDeliveryType("pickup_at_store");
newTracking.setPickupLocation("Flagship Store");
newTracking.setPickupNote(
  "Reach out to our staffs when you arrive our stores for shipment pickup");

try {
  Tracking tracking = afterShip.getTrackingEndpoint().createTracking(newTracking);
  System.out.println(tracking);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```



- #### deleteTracking [DELETE /trackings/:slug/:tracking_number]

```java 
String id = "u2qm5uu9xqpwykaqm8d5l010";
try {
  Tracking tracking = afterShip.getTrackingEndpoint().deleteTracking(id);
  System.out.println(tracking);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### getTrackings [GET /trackings]

```java 
GetTrackingsParams optionalParams = new GetTrackingsParams();
optionalParams.setFields("title,order_id");
optionalParams.setLang("china-post");
optionalParams.setLimit(10);

try {
  PagedTrackings pagedTrackings = afterShip.getTrackingEndpoint().getTrackings(optionalParams);
  System.out.println(pagedTrackings);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### getTracking [GET /trackings/:slug/:tracking_number]

  - Get by id:

    ```java
    String id = "l389dilsluk9ckaqmetr901y";
    try {
      Tracking tracking = afterShip.getTrackingEndpoint().getTracking(id, null);
      System.out.println(tracking);
    } catch (AftershipException e) {
      System.out.println(e.getMessage());
    }
    ```
    
  - Get by slug and tracking_number:
  
    ```java 
    String slug = "acommerce";
    String trackingNumber = "1234567890";
    try {
      Tracking tracking =
      afterShip
        .getTrackingEndpoint()
      .getTracking(new SlugTrackingNumber(slug, trackingNumber), null);
      System.out.println(tracking);
    } catch (AftershipException e) {
      System.out.println(e.getMessage());
    }
    ```
  
- #### updateTracking [PUT /trackings/:slug/:tracking_number]

```java
String id = "vebix4hfu3sr3kac0epve01n";
UpdateTracking updateTracking = new UpdateTracking();
updateTracking.setTitle("title123");

try {
  Tracking tracking1 = afterShip.getTrackingEndpoint().updateTracking(id, updateTracking);
  System.out.println(tracking1);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### reTrack [POST /trackings/:slug/:tracking_number/retrack]

```java
String id = "l389dilsluk9ckaqmetr901y";
try {
  Tracking tracking = afterShip.getTrackingEndpoint().reTrack(id);
  System.out.println(tracking);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### markAsCompleted [POST /trackings/:slug/:tracking_number/mark-as-completed]

```java 
String id = "5b7658cec7c33c0e007de3c5";
try {
  Tracking tracking =
    afterShip.getTrackingEndpoint().markAsCompleted(id, new CompletedStatus(ReasonKind.LOST));
  System.out.println(tracking);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

### /last_checkpoint

- #### getLastCheckpoint [GET /last_checkpoint/:slug/:tracking_number]

```java
String id = "vebix4hfu3sr3kac0epve01n";
GetCheckpointParam optionalParam = new GetCheckpointParam();
optionalParam.setFields(FieldsKind.combine(FieldsKind.TAG));
optionalParam.setLang(LangKind.CHINA_EMS);

try {
  LastCheckpoint lastCheckpoint =
    afterShip.getCheckpointEndpoint().getLastCheckpoint(id, optionalParam);
  System.out.println(lastCheckpoint.getSlug());
  System.out.println(lastCheckpoint.getTrackingNumber());
  System.out.println(lastCheckpoint.getCheckpoint());
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

### /notifications

- #### getNotification [GET /notifications/:slug/:tracking_number]

```java 
String id = "vebix4hfu3sr3kac0epve01n";
try {
  Notification notification = afterShip.getNotificationEndpoint().getNotification(id);
  System.out.println(notification);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### addNotification [POST /notifications/:slug/:tracking_number/add]

```java
String id = "vebix4hfu3sr3kac0epve01n";
Notification addNotification = new Notification();
addNotification.setSmses(new String[] {"+85261236123", "Invalid Mobile Phone Number"});

try {
  Notification notification =
    afterShip.getNotificationEndpoint().addNotification(id, addNotification);
  System.out.println(notification);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```

- #### removeNotification [POST /notifications/:slug/:tracking_number/remove]

```java 
String id = "vebix4hfu3sr3kac0epve01n";
Notification removeNotification = new Notification();
removeNotification.setEmails(new String[] {"invalid EMail @ Gmail. com"});
removeNotification.setSmses(new String[] {"+85261236123"});
try {
  Notification notification =
    afterShip.getNotificationEndpoint().removeNotification(id, removeNotification);
  System.out.println(notification);
} catch (AftershipException e) {
  System.out.println(e.getMessage());
}
```



## Notes

- #### When specifying a tracking, the `id` is equivalent to the `slug and tracking_number`.

```java 
getTracking("l389dilsluk9ckaqmetr901y", null);
```

```java 
getTracking(new SlugTrackingNumber("acommerce", "1234567890"), null);
```

## License
Copyright (c) 2015-2022 Aftership  
Licensed under the MIT license.

​	

