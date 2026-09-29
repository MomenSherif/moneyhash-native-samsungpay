# Samsung Pay Integration Guide

## SamsungPay JS SDK

```html
<script src="https://img.mpay.samsung.com/gsmpi/sdk/samsungpay_web_sdk.js"></script>
```

## Code Sample

```js
const paymentMethods = {
  version: '2',
  serviceId: samsunPayNativeData.service_id,
  protocol: 'PROTOCOL_3DS',
  allowedBrands: samsunPayNativeData.allowed_brands,
  isCardholderNameRequired: true, // ...
};

const transactionDetail = {
  orderNumber: '<YOUR_ORDER_NUMBER>' || '<INTENT_ID>',
  merchant: {
    name: samsungPayNativeData.merchant_name,
    url: location.hostname,
    id: samsungPayNativeData.merchant_id,
    countryCode: samsungPayNativeData.country_code,
  },
  amount: {
    option: 'FORMAT_TOTAL_ESTIMATED_AMOUNT',
    currency: samsungPayNativeData.currency,
    total: samsungPayNativeData.amount,
  },
};

// Set the environment:
// "STAGE" – Testing with device
// "STAGE_WITHOUT_APK" – Testing without device (simulated)
// "PRODUCTION" – Live environment

const samsungPayClient = new SamsungPay.PaymentClient({
  environment: 'STAGE',
  // nonce: 'your-nonce', optional -  If your project has a Content-Security-Policy (CSP) applied, please ensure that you add a nonce to the CSS to maintain compliance. This can be done by updating your SDK configuration as follows:
});

// Verify Samsung Pay availability in the user’s browser/device:
samsungPayClient
  .isReadyToPay(paymentMethods)
  .then(function (response) {
    if (response.result) {
      // add a payment button
      const btn = samsungPayClient.createButton({
        onClick: onSamsungPayButtonClicked,
        buttonStyle: 'white', // "black" | "white" | "white-outline"
        type: 'buy', // "buy" | "checkout" | "pay" | "continue"
      });
      document.getElementById('samsungpay-container').appendChild(btn);
    }
  })
  .catch(function (err) {
    console.error(err);
  });

function onSamsungPayButtonClicked() {
  log('Button clicked — opening payment sheet…');
  samsungPayClient
    .loadPaymentSheet(paymentMethods, transactionDetail)
    .then(function (paymentCredential) {
      // Submit receipt, & based on response notify samsung pay sdk using paymentCredential

      // Payment status
      // The possible values are:
      // `CHARGED` = payment was charge successfully
      // `CANCELED` = payment was canceled by either user, merchant, or acquirer
      // `REJECTED` = payment was rejected by acquirer
      // `ERRED` = an error occurred during the payment process
      const paymentResult = {
        status: 'CHARGED',
      };
      samsungPayClient.notify(paymentResult);
      log('notify() sent: ' + JSON.stringify(paymentResult), 'ok');
      setStatus('Payment complete ✓', 'ok');
    })
    .catch(function (error) {
      log('Payment error: ' + JSON.stringify(error), 'err');
      setStatus('Payment failed or cancelled', 'err');

      // Report the failure to Samsung Pay so the sheet closes cleanly.
      try {
        samsungPayClient.notify({ status: 'FAILED', provider: 'MyPG' });
      } catch (e) {
        /* client may not be ready */
      }
    });
}
```
