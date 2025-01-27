# TextSummariser

Code for Main_Activity.java

package com.example.text_to_speech_sgm;

import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.speech.tts.TextToSpeech;
import android.text.Spannable;
import android.text.SpannableString;
import android.text.style.BackgroundColorSpan;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import androidx.appcompat.app.AppCompatActivity;
import java.util.Locale;
import android.content.SharedPreferences;
import android.content.Intent;
import android.net.Uri;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
    TextToSpeech tts;
    EditText e1;
    Button b1;
    Button loadFileButton;

    EditText manualEditText;
    boolean isTTSInitialized = false;

    private SharedPreferences sharedPreferences;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        e1 = findViewById(R.id.editTextTextMultiLine);
        b1 = findViewById(R.id.button);
        Button bInputNewText = findViewById(R.id.buttonInputNewText);
        Button historyButton = findViewById(R.id.historyButton);
        loadFileButton = findViewById(R.id.loadFileButton);

        historyButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // Create an intent to open the history activity
                Intent intent = new Intent(MainActivity.this, HistoryActivity1.class);
                startActivity(intent);
            }
        });

        // Initialize SharedPreferences
        sharedPreferences = getSharedPreferences("TextHistory", MODE_PRIVATE);

        // Set the previously used text in the EditText
        //e1.setText(getTextHistory());

        tts = new TextToSpeech(getApplicationContext(), new TextToSpeech.OnInitListener() {
            @Override
            public void onInit(int i) {
                if (i == TextToSpeech.SUCCESS) {
                    tts.setLanguage(Locale.US);
                    tts.setSpeechRate(1f);
                    isTTSInitialized = true;
                }
            }
        });
        //TextView historyTextView = findViewById(R.id.historyTextView);

        //historyTextView.setText("Text History:\n" + getTextHistory());

        manualEditText = findViewById(R.id.manualEditText);
        Button b1= findViewById(R.id.button);
        loadFileButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // Launch a file picker dialog or file explorer to let the user select a .txt file
                Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
                intent.addCategory(Intent.CATEGORY_OPENABLE);
                intent.setType("text/plain");
                startActivityForResult(intent, 1);
            }
        });

        // Your code to generate and display the summary text



        b1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (isTTSInitialized) {
                    String text = manualEditText.getText().toString();
                    tts.speak(text, TextToSpeech.QUEUE_FLUSH, null);
                    saveTextToHistory(text);
                    highlightText(text);
                }
            }
        });


        bInputNewText.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // Clear the text box
                manualEditText.setText("");
                e1.setText("");
                tts = new TextToSpeech(getApplicationContext(), new TextToSpeech.OnInitListener() {
                    @Override
                    public void onInit(int i) {
                        if (i == TextToSpeech.SUCCESS) {
                            tts.setLanguage(Locale.US);
                            tts.setSpeechRate(1f);
                            isTTSInitialized = true;
                        }
                    }
                });
                if (tts != null) {
                    tts.stop();
                }
            }
        });

        e1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (isTTSInitialized) {
                    String text = e1.getText().toString();
                    tts.speak(text, TextToSpeech.QUEUE_FLUSH, null);
                    saveTextToHistory(text); // Save the clicked text to history
                    highlightText(text);
                }
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 1 && resultCode == RESULT_OK) {
            Uri uri = data.getData();
            try {
                // Read the content of the selected .txt file
                InputStream inputStream = getContentResolver().openInputStream(uri);
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                StringBuilder text = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    text.append(line).append("\n");
                }
                reader.close();
                inputStream.close();

                // Set the loaded text to the EditText
                manualEditText.setText(text.toString());

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }


    private void saveTextToHistory(String text) {
        String existingText = sharedPreferences.getString("textHistory", "");
        String newText = existingText + "\n" + text;
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putString("textHistory", newText);
        editor.apply();
    }

    private void speakWithHighlighting(String text) {
        String[] words = text.split("\\s+");
        int startIndex = 0;

        for (String word : words) {
            SpannableString spannable = new SpannableString(text);
            int wordStartIndex = text.indexOf(word, startIndex);
            int wordEndIndex = wordStartIndex + word.length();

            // Apply the background color to highlight the word
            spannable.setSpan(new BackgroundColorSpan(0xFFFFFFFF), wordStartIndex, wordEndIndex, Spannable.SPAN_INCLUSIVE_INCLUSIVE);

            // Set the text in the EditText to the SpannableString
            e1.setText(spannable);

            // Speak the current word
            tts.speak(word, TextToSpeech.QUEUE_FLUSH, null);

            // Delay to give time for speech to complete (adjust as needed)
            int speechLengthInMillis = word.length() * 200;
            try {
                Thread.sleep(speechLengthInMillis);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            // Remove the background color after speaking
            spannable.removeSpan(spannable.getSpans(0, spannable.length(), BackgroundColorSpan.class)[0]);
            e1.setText(spannable);

            // Update the start index for the next word
            startIndex = wordEndIndex;
        }
    }

    private void highlightText(String text) {
        // Create a new SpannableString to highlight the entire text
        SpannableString spannable = new SpannableString(text);

        // Apply the background color to highlight the entire text
        spannable.setSpan(new BackgroundColorSpan(0xFFADD8E6), 0, text.length(), Spannable.SPAN_INCLUSIVE_INCLUSIVE);

        // Set the text in the EditText to the SpannableString
        e1.setText(spannable);

        // Delay to show the highlighting (adjust as needed)
        int highlightingDurationInMillis = text.length() * 200;
        new Handler(Looper.getMainLooper()).postDelayed(new Runnable() {
            @Override
            public void run() {
                // Remove the background color after highlighting
                spannable.removeSpan(spannable.getSpans(0, spannable.length(), BackgroundColorSpan.class)[0]);
                e1.setText(spannable);
            }
        }, highlightingDurationInMillis);
    }

    private String getTextHistory() {
        return sharedPreferences.getString("textHistory", "");
    }


    @Override
    public void onBackPressed() {
        super.onBackPressed();
        finishAffinity();
        System.exit(0);
    }
}

Code for Main_Activity.xml

<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="0dp"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/LinearLayout01"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:background="@drawable/greenbg"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0" />

    <EditText
        android:id="@+id/editTextTextMultiLine"
        android:layout_width="250dp"
        android:layout_height="80dp"
        android:layout_marginBottom="480dp"
        android:contentDescription="Enter your text here"
        android:ems="10"
        android:gravity="start|bottom"
        android:inputType="textMultiLine"
        android:lines="10"
        android:textColor="@color/black"
        app:layout_constraintBottom_toBottomOf="parent"

        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.496"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="1.0" />

    <Button
        android:id="@+id/button"
        android:layout_width="122dp"
        android:layout_height="45dp"
        android:backgroundTint="#0a6680"
        android:elevation="20dp"
        android:text="Convert"
        android:textColor="@color/white"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="@+id/LinearLayout01"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.572" />

    <ImageView
        android:id="@+id/imageView2"
        android:layout_width="180dp"
        android:layout_height="90dp"
        android:layout_marginTop="-6dp"
        android:layout_marginLeft="-10dp"

        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.0"
        app:srcCompat="@drawable/speakscribesgm" />

    <Button
        android:id="@+id/buttonInputNewText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:backgroundTint="#0a6680"
        android:text="Input New Text"
        android:textColor="@color/white"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.178"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@+id/LinearLayout01"
        app:layout_constraintVertical_bias="0.656" />

    <Button
        android:id="@+id/loadFileButton"
        android:layout_width="122dp"
        android:layout_height="45dp"
        android:text="Load File"
        android:textColor="@color/white"
        android:backgroundTint="#0a6680"
        android:textSize="14sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.837"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@+id/LinearLayout01"
        app:layout_constraintVertical_bias="0.655" />

    <EditText
        android:id="@+id/manualEditText"
        android:layout_width="268dp"
        android:layout_height="59dp"
        android:layout_margin="16dp"
        android:hint="Enter text to convert"
        android:inputType="textMultiLine"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.496"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.485" />

    <Button
        android:id="@+id/historyButton"
        android:layout_width="110dp"
        android:layout_height="30dp"
        android:layout_margin="10dp"
        android:layout_marginBottom="-5dp"
        android:text="View History"
        android:textSize="-500dp"
        android:textColor="#0a6680"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="1.0"
        android:backgroundTint="@color/white"/>
</androidx.constraintlayout.widget.ConstraintLayout>

Code for History.java

package com.example.text_to_speech_sgm;

import android.content.SharedPreferences;
import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;

public class HistoryActivity1 extends AppCompatActivity{
    private SharedPreferences sharedPreferences;
    private TextView historyTextView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_history1);

        sharedPreferences = getSharedPreferences("TextHistory", MODE_PRIVATE);
        historyTextView = findViewById(R.id.historyTextView);

        historyTextView.setText("Text History:\n" + getTextHistory());

        Button clearHistoryButton = findViewById(R.id.clearHistoryButton);
        clearHistoryButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                clearHistory();
            }
        });
    }
    private void clearHistory() {
        // Clear the text history in shared preferences
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.remove("textHistory"); // Remove the history key from shared preferences
        editor.apply();

        // Update the displayed history
        historyTextView.setText("Text History:\n");
    }

    private String getTextHistory() {
        return sharedPreferences.getString("textHistory", "");
    }

}


Code for History.xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp">

    <RelativeLayout
        android:id="@+id/relativeLayout2"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ScrollView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:padding="4.5dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical">

                <TextView
                    android:id="@+id/historyTextView"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Text History:"

                    android:textSize="16sp"
                    android:textStyle="bold"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="@+id/LinearLayout01"
                    app:layout_constraintHorizontal_bias="0.176"
                    app:layout_constraintStart_toStartOf="@+id/LinearLayout01"
                    app:layout_constraintTop_toTopOf="parent"
                    app:layout_constraintVertical_bias="0.844" />
            </LinearLayout>
        </ScrollView>
    </RelativeLayout>

    <Button
        android:id="@+id/clearHistoryButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginLeft="242dp"
        android:layout_marginBottom="670dp"
        android:backgroundTint="#0a6680"
        android:text="Clear History"
        android:textColor="@color/white"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="@+id/relativeLayout2"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@+id/relativeLayout2" />

</androidx.constraintlayout.widget.ConstraintLayout>




Code for Splashscreen

package com.example.text_to_speech_sgm;

import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;

public class splashscreen extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.splashscreen);

        Runnable r = new Runnable() {
            @Override
            public void run() {
                startActivity(new Intent(splashscreen.this, MainActivity.class));
            }
        };

        Handler h = new Handler();
// The Runnable will be executed after the given delay time
        h.postDelayed(r, 3000);
        }
    
}

Code for splashscreen.xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".splashscreen"
    android:background="@drawable/splashwhitebluefinal">


  </androidx.constraintlayout.widget.ConstraintLayout>



