# i'mport  

:globe_with_meridians: [KO](/README.md)  

i'mport is a **payment platform** or **payment hosting service** that provides easier and faster integration with PG's payment modules in various development environments.  

i'mport service is designed to reduce unnecessary development effort for payment integration, and the service is transparent to the users of the payment service.  
This means that any existing PG's system requirements, such as security keyboard or firewall module installation, for payment processing remains the same for the user.  

i'mport service is secure and verified, complying with the authenticated payment process requirements of PG and credit card companies.  
Like shopping mall hosting services, such as Cafe24, MakeShop, and Godo Mall, i'mport service performs technical communication with PGs for payment approval on behalf of the merchant and delivers the result.  

## Key Features  

### Korean Encoding  

Since i'mport uses **UTF-8** encoding, no encoding conversion is needed.  

### JavaScript API    

To open the payment window, instead of passing data through hidden input fields via HTML form submission,  

```html
<form name="pgForm">
	<input type="hidden" name="Amt" value="1000">
	<input type="hidden" name="BuyerName" value="Gildong Hong">
	<input type="hidden" name="OrderName" value="Payment test">
</form>
```

call JavaScript function as follows:  

```javascript
IMP.request_pay({
	amount : 1000,
	buyer_name : 'Gildong Hong',
	name : 'Payment test'
}, function(response) {
	// callback to invoke after payment approval
	if ( response.success ) { // Payment successful
		console.log(response);
	} else {
		alert('Payment failed : ' + response.error_msg);
	}
})
```

### REST API  

After payment is approved, you can retrieve the payment result by calling the i'mport REST API provided using token-based authentication. Regardless of the backend programming language you are using, i'mport requires no program installations for payment integration.  


## Payment Methods  

### [General Payment](./General/README.md)  

This payment method requires entering card information and going through the Secure-Click or ISP authentication process.  


### [Subscription Payment](./Subscription/README.md)  

This payment method uses a simplified authentication process that only requires entering the customer's credit card number, password, and expiration date and date of birth, instead of using the Secure-Click or ISP authentication.  

This is provided only in special cases, such as **regular subscription type billing** that requires automatic monthly payments after the first one-time credit card registration.

## i'mport Support  

- For technical inquiries, please contact support@iamport.kr.  
- For PG contract and other general inquiries, please contact cs@iamport.kr.  