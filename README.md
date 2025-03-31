# Computer_networking
GPRS and GSM Demonstration

#AndroidCode

Main Activity.kt file:
package com.example.cnproject

import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import android.telephony.SmsManager
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import com.example.cnproject.R
import java.io.OutputStream
import java.net.HttpURLConnection
import java.net.URL

class MainActivity : AppCompatActivity() {

    private lateinit var etPhoneNumber: EditText
    private lateinit var etMessage: EditText
    private lateinit var tvStatus: TextView

    private val SMS_PERMISSION_CODE = 100

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        etPhoneNumber = findViewById(R.id.etPhoneNumber)
        etMessage = findViewById(R.id.etMessage)
        tvStatus = findViewById(R.id.tvStatus)

        val btnSendSMS: Button = findViewById(R.id.btnSendSMS)
        val btnSendGPRS: Button = findViewById(R.id.btnSendGPRS)

        // Check SMS permission
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.SEND_SMS)
            != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.SEND_SMS), SMS_PERMISSION_CODE)
        }

        btnSendSMS.setOnClickListener { sendSMS() }
        btnSendGPRS.setOnClickListener { sendDataViaGPRS() }
    }

    private fun sendSMS() {
        val phoneNumber = etPhoneNumber.text.toString()
        val message = etMessage.text.toString()

        if (phoneNumber.isBlank() || message.isBlank()) {
            Toast.makeText(this, "Please enter phone and message", Toast.LENGTH_SHORT).show()
            return
        }

        try {
            val smsManager = SmsManager.getDefault()
            smsManager.sendTextMessage(phoneNumber, null, message, null, null)
            tvStatus.text = "SMS sent successfully"
        } catch (e: Exception) {
            tvStatus.text = "Failed to send SMS"
            e.printStackTrace()
        }
    }

    private fun sendDataViaGPRS() {
        Thread {
            try {
                val url = URL("https://gprs-drf0.onrender.com/api") // Ensure correct endpoint
                val connection = url.openConnection() as HttpURLConnection
                connection.requestMethod = "POST"
                connection.setRequestProperty("Content-Type", "application/json")
                connection.doOutput = true

                val jsonInput = "{\"message\": \"${etMessage.text}\"}"
                val outputStream: OutputStream = connection.outputStream
                outputStream.write(jsonInput.toByteArray(Charsets.UTF_8))
                outputStream.close()

                val responseCode = connection.responseCode
                val responseMessage = connection.inputStream.bufferedReader().use { it.readText() } // Read response

                runOnUiThread {
                    tvStatus.text = "GPRS Response: $responseCode\n$responseMessage"
                }

            } catch (e: Exception) {
                runOnUiThread {
                    tvStatus.text = "Error sending data via GPRS"
                }
                e.printStackTrace()
            }
        }.start()
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == SMS_PERMISSION_CODE) {
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "SMS permission granted", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this, "SMS permission denied", Toast.LENGTH_SHORT).show()
            }
        }
    }
}

activity_main.xml:
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/etPhoneNumber"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Phone Number"
        android:inputType="phone" />

    <EditText
        android:id="@+id/etMessage"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Message"
        android:inputType="text" />

    <Button
        android:id="@+id/btnSendSMS"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send SMS" />

    <Button
        android:id="@+id/btnSendGPRS"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send Data via GPRS" />

    <TextView
        android:id="@+id/tvStatus"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Status: Not Connected"
        android:textSize="16sp"
        android:paddingTop="16dp" />
</LinearLayout>


Server Code:
require('dotenv').config(); 
const express = require('express');
const cors = require('cors'); 
const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());

app.post('/api', (req, res) => {
    const { message } = req.body; 
    console.log('Message received on server:', message);
    res.status(200).json({
        status: 'success',
        receivedMessage: message,
        response: 'Message successfully received on the server!',
    });
});

app.get('/', (req, res) => {
    res.send('Welcome to the GPRS Server!'); 
});

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
}); 

