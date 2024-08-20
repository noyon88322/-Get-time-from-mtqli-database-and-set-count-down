## Get time from mtqli database and set count down

 java

   ```bash


package com.example.timer;

import android.app.TimePickerDialog;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.os.CountDownTimer;
import android.os.Handler;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;

import com.android.volley.Request;
import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.JsonArrayRequest;
import com.android.volley.toolbox.JsonObjectRequest;
import com.android.volley.toolbox.Volley;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.HashMap;
import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    private TextView countdownTextView;
    private long endTimeMillis;
    String time;
    Handler handler;
    Runnable runnable;
    String url;
    RequestQueue requestQueue;
    Button add;


    HashMap<String,String>hashMap;
    ArrayList<HashMap<String,String>>arrayList = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
         handler =new Handler();

        countdownTextView = findViewById(R.id.countdownTextView);
        add = findViewById(R.id.add);

        requestQueue = Volley.newRequestQueue(this);


         url = "http://192.168.137.1/admin/timerget/conn.php"; // Replace with your URL


        add.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                fetchData();
            }
        });













    }
    private void startCountdown(String timeInput) {
        SimpleDateFormat sdf = new SimpleDateFormat("hh:mm a", Locale.getDefault()); // AM/PM ফরম্যাটের জন্য
        Calendar now = Calendar.getInstance();
        Calendar eventTime = Calendar.getInstance();

        try {
            eventTime.setTime(sdf.parse(timeInput));
            eventTime.set(Calendar.YEAR, now.get(Calendar.YEAR));
            eventTime.set(Calendar.MONTH, now.get(Calendar.MONTH));
            eventTime.set(Calendar.DAY_OF_MONTH, now.get(Calendar.DAY_OF_MONTH));

            if (eventTime.before(now)) {
                // যদি eventTime এখন সময়ের আগে থাকে, তাহলে পরের দিনের জন্য সেট করুন
                eventTime.add(Calendar.DAY_OF_MONTH, 1);
            }

            final long eventTimeMillis = eventTime.getTimeInMillis();


            runnable = new Runnable() {
                @Override
                public void run() {
                    long currentTimeMillis = System.currentTimeMillis();
                    long remainingTimeMillis = eventTimeMillis - currentTimeMillis;

                    if (remainingTimeMillis > 0) {
                        long hours = (remainingTimeMillis / (1000 * 60 * 60)) % 24;
                        long minutes = (remainingTimeMillis / (1000 * 60)) % 60;
                        long seconds = (remainingTimeMillis / 1000) % 60;

                        // যদি ঘন্টা বা মিনিট 0 হয় তাহলে সেগুলি শো করবেন না
                        String timeString = "";
                        if (hours > 0) {
                            timeString += String.format("%02d:", hours);
                        }
                        if (minutes > 0 || hours > 0) { // যদি minutes > 0 হয় বা hours > 0 হয়
                            timeString += String.format("%02d:", minutes);
                        }
                        timeString += String.format("%02d", seconds);

                        countdownTextView.setText(timeString);
                        handler.postDelayed(this, 1000);
                    } else {
                        countdownTextView.setText("Time's up!");
                    }
                }
            };

            handler.post(runnable);
        } catch (ParseException e) {
            e.printStackTrace();
            countdownTextView.setText("Invalid time format.");
          }
    }


    private void fetchData() {
        RequestQueue queue = Volley.newRequestQueue(this);

        JsonArrayRequest jsonArrayRequest = new JsonArrayRequest(
                Request.Method.GET,
                url,
                null,
                new Response.Listener<JSONArray>() {
                    @Override
                    public void onResponse(JSONArray response) {
                        try {
                            StringBuilder times = new StringBuilder();
                            for (int i = 0; i < response.length(); i++) {
                                JSONObject obj = response.getJSONObject(i);
                                String time = obj.getString("time");
                                times.append(time).append("\n");
                            }

                                startCountdown(times.toString());

                            //startCountdown(time);
//                            countdownTextView.setText(times.toString());
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                    }
                },
                new Response.ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError error) {
//                        countdownTextView.setText("Error fetching data!");
                        error.printStackTrace();
                    }
                }
        );

        queue.add(jsonArrayRequest);
   }



}




   ```



   
 Php

   ```bash


<?php
header('Content-Type: application/json; charset=utf-8');
$servername = "localhost";
$username = "root";
$password = '';
$db = "time_column";


$conn = mysqli_connect($servername, $username, $password, $db);




   $qry="SELECT * FROM time_test";

   $result = mysqli_query($conn,$qry);

   $count =mysqli_num_rows($result);

   $data=array();
  foreach( $result as $item){
    $time = $item['time'];

    $userInfo['time']=$time;

    array_push($data,$userInfo);

  }
  echo json_encode($data);


?>




   ```

