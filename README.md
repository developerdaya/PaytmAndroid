# PaytmAndroid
To integrate **Paytm** payment gateway into your Android app, you can use the **Paytm All-in-One SDK**. This SDK allows you to accept payments through various methods, including UPI, credit/debit cards, net banking, and wallets.

### Steps to Integrate Paytm in Android:

### 1. **Register with Paytm Developer Account**:
- You need to register as a Paytm merchant to use their payment gateway. Sign up at the [Paytm for Business](https://business.paytm.com/) portal.
- After registration, obtain your **Merchant ID** (MID) and **Merchant Key**.

### 2. **Add the Paytm SDK to Your Project**:
Add the Paytm SDK dependency to your `build.gradle` file (Module: app).

```gradle
dependencies {
    implementation 'com.paytm.appinvokesdk:paytmappinvokesdk:1.6'
    implementation 'com.paytm:pgplussdk:1.5.3'
}
```

### 3. **Permissions and ProGuard Rules**:
In your `AndroidManifest.xml`, add the following permissions:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

If you are using ProGuard or R8, add the following rules to your `proguard-rules.pro` file:

```pro
-keepattributes Signature
-keepattributes *Annotation*
-keepattributes SourceFile,LineNumberTable
-keepattributes EnclosingMethod
-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}
-keep class com.paytm.** { *; }
-dontwarn com.paytm.**
```

### 4. **Generate Checksum on the Server**:
You need a backend server to generate a **checksum** for secure transactions. Paytm provides libraries for different programming languages to generate this checksum. Here's how you can do it using Node.js as an example:

```javascript
const PaytmChecksum = require('./PaytmChecksum');

const paytmParams = {    
    MID: "YourMerchantID",
    ORDERID: "OrderID",
    TXN_AMOUNT: "TransactionAmount",
    CUST_ID: "CustomerID",
    INDUSTRY_TYPE_ID: "Retail",
    CHANNEL_ID: "WEB",
    WEBSITE: "WEBSTAGING"
};

PaytmChecksum.generateSignature(paytmParams, "YourMerchantKey").then(function(checksum){
    // Send the checksum back to your Android app
});
```

You will need to implement a similar checksum generator on your server-side and expose an API to your Android app that fetches this checksum.

### 5. **Create a Transaction Request**:
In your Android app, create a transaction request that includes the necessary parameters like **Order ID**, **Customer ID**, and **Transaction Amount**.

```kotlin
import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast
import com.paytm.pgsdk.PaytmOrder
import com.paytm.pgsdk.PaytmPaymentTransactionCallback
import com.paytm.pgsdk.TransactionManager

class MainActivity : AppCompatActivity(), PaytmPaymentTransactionCallback {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Replace with actual details
        val orderId = "ORDER12345"
        val mid = "YourMerchantID"
        val txnToken = "ChecksumTokenGeneratedFromServer"
        val txnAmount = "100.00"

        // Create the Paytm Order
        val paytmOrder = PaytmOrder(orderId, mid, txnToken, txnAmount, "WEBSTAGING")

        // Create Transaction Manager
        val txnManager = TransactionManager(paytmOrder, this)

        txnManager.setShowPaymentUrl("https://securegw-stage.paytm.in/theia/api/v1/showPaymentPage")

        // Start the transaction
        txnManager.startTransaction(this, 1001)
    }

    override fun onTransactionResponse(bundle: Bundle?) {
        Toast.makeText(this, "Payment Success: ${bundle?.getString("STATUS")}", Toast.LENGTH_LONG).show()
    }

    override fun networkNotAvailable() {
        Toast.makeText(this, "Network not available", Toast.LENGTH_LONG).show()
    }

    override fun onErrorProceed(p0: String?) {
        Toast.makeText(this, "Payment Failed: $p0", Toast.LENGTH_LONG).show()
    }

    override fun clientAuthenticationFailed(p0: String?) {
        Toast.makeText(this, "Client Authentication Failed: $p0", Toast.LENGTH_LONG).show()
    }

    override fun someUIErrorOccurred(p0: String?) {
        Toast.makeText(this, "UI Error: $p0", Toast.LENGTH_LONG).show()
    }

    override fun onTransactionCancel(p0: String?, p1: Bundle?) {
        Toast.makeText(this, "Transaction Cancelled", Toast.LENGTH_LONG).show()
    }

    override fun onBackPressedCancelTransaction() {
        Toast.makeText(this, "Transaction Cancelled", Toast.LENGTH_LONG).show()
    }

    override fun onTransactionCancel(p0: String?, p1: Bundle?) {
        Toast.makeText(this, "Transaction Cancelled", Toast.LENGTH_LONG).show()
    }
}
```

### 6. **Handle Payment Response**:
You can handle the transaction result using the callback `PaytmPaymentTransactionCallback`, which returns the status of the payment (success, failure, etc.).

### Important Methods in `PaytmPaymentTransactionCallback`:
- `onTransactionResponse(Bundle?)`: This is called when the payment is successful.
- `networkNotAvailable()`: This is called if there is a network issue.
- `onErrorProceed(String?)`: This is called if there is an error while processing.
- `onTransactionCancel(String?, Bundle?)`: This is called when the transaction is canceled.

### 7. **Test and Go Live**:
- Use the **"WEBSTAGING"** environment for testing with test credentials.
- Once you are ready for production, switch the environment to **"PRODUCTION"** and ensure your checksum generation and other details are set up correctly.

### Conclusion:
- Paytm integration in Android provides a comprehensive solution for handling payments within apps.
- By using the Paytm All-in-One SDK, you can provide various payment methods like UPI, cards, and net banking.
- Ensure that you have a secure server setup to handle the checksum generation for transactions.
