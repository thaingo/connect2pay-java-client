Connect2Pay Payment Page Java Client
============================

This is the Java implementation of the Connect2pay Payment Page API.

The main class to use is ```Connect2payClient``` that implements all the API calls.  
It requires request object as entries and returns response objects.  
  
Example of a payment creation with user redirection:

```java
   // Process a payment of €39.99
   PaymentRequest request = new PaymentRequest();
   request.setOrderId("1234ABCD").setCurrency("EUR").setAmount(3999);
   request.setShippingType(ShippingType.VIRTUAL);
   request.setPaymentMode(PaymentMode.SINGLE);
   // User will be redirected here when clicking on "Go back to merchant" button
   request.setCtrlRedirectURL("http://my.website/somewhere");
   // The payment page will do a callback on this URL after the payment is processed
   request.setCtrlCallbackURL("http://my.website/payment-callback");
   
   // Validate the request
   try {
     request.validate();
   } catch (BadRequestException e) {
     logger.error("Ooops, an error occurred validating the payment request: " + e.getMessage());
     // Handle the error...
   }
   
   // Instantiate the client and send the prepare request
   // Second argument is the originator ID, third one is the associated API key
   Connect2payClient c2p = new Connect2payClient("https://provided.url", "123456", "GreatP4ssw0rd");
   PaymentResponse response = null;
   try {
     response = c2p.preparePayment(request);
   } catch (Exception e) {
     logger.error("Ooops, an error occurred preparing the payment: " + e.getMessage());
     // Handle the error...
   }
   
   if (response != null && ResultCode.SUCCESS.equals(response.getCode()) {
     // Get the URL to redirect the user to
     String redirectURL = response.getCustomerRedirectURL();
     if (redirectURL != null) {
       // Redirect the user towards this URL, this will display the payment page
     }
   } else {
     // Handle the failure
   }
```

Example of the payment callback handling:

```java
   // Instantiate the client and handle the callback
   Connect2payClient c2p = new Connect2payClient("https://provided.url", "123456", "GreatP4ssw0rd");
   PaymentStatusResponse response = null;
   try {
     // Request is either the body of the received request as a string or an InputStream pointing to the received body
     response = c2p.handleCallbackStatus(request);
   } catch (Exception e) {
     logger.error("Ooops, an error occurred handling the callback: " + e.getMessage());
     // Handle the error...
   }
   
   if (response != null && ResultCode.SUCCESS.equals(response.getCode()) {
     // Check the payment status: 000 means success
     if ("000".equals(response.getErrorCode()) {
       // Handle the payment success case
       // ...
       // Access the payment mean information
       CreditCardPaymentMeanInfo pmInfo = response.getCCPaymentMeanInfo();
       logger.info("Successful payment done by " + pmInfo.getCardHolderName() + "with card " + pmInfo.getCardNumber());
     } else {
       // Handle the payment failure case
     }
     
     if (handledSuccessfully) {
       out.write(CallbackStatusResponse.getDefaultSuccessResponse().toJson()) // out is the output stream
     } else {
       out.write(CallbackStatusResponse.getDefaultFailureResponse().toJson()) // out is the output stream
     }
   } else {
     // Handle the failure
     out.write(CallbackStatusResponse.getDefaultFailureResponse().toJson()) // out is the output stream
   }
```
