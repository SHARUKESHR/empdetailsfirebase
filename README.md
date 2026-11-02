
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: Sharukesh R
Registeration Number :212223220106
```
## MyTodoList.java
```
package com.example.mytodoapp;

public class TodoTask {
    public String id;
    public String title;
    public boolean completed;

    public TodoTask() {
        // Required by Firebase
    }

    public TodoTask(String id, String title, boolean completed) {
        this.id = id;
        this.title = title;
        this.completed = completed;
    }
}

```
## MAINACTIVITY:
```
package com.example.mytodoapp;

import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;

import com.google.firebase.database.*;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    EditText etTask;
    Button btnAdd, btnUpdate, btnDelete;
    ListView listView;

    DatabaseReference dbRef;

    ArrayList<String> items = new ArrayList<>();
    ArrayList<String> keys = new ArrayList<>();
    ArrayAdapter<String> adapter;

    String selectedKey = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etTask = findViewById(R.id.etTask);
        btnAdd = findViewById(R.id.btnAdd);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);
        listView = findViewById(R.id.listView);

        dbRef = FirebaseDatabase.getInstance()
                .getReference("Tasks");

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                items
        );
        listView.setAdapter(adapter);

        // Read tasks from Firebase
        dbRef.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(DataSnapshot snapshot) {
                items.clear();
                keys.clear();

                for (DataSnapshot d : snapshot.getChildren()) {
                    TodoTask task = d.getValue(TodoTask.class);

                    if (task != null) {
                        String status = task.completed
                                ? "Completed" : "Pending";

                        items.add(task.title + " - " + status);
                        keys.add(d.getKey());
                    }
                }

                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(DatabaseError error) {
                Toast.makeText(MainActivity.this,
                        "Read failed: " + error.getMessage(),
                        Toast.LENGTH_LONG).show();
            }
        });

        // Add a task
        btnAdd.setOnClickListener(v -> {
            String title = etTask.getText().toString().trim();

            if (title.isEmpty()) {
                etTask.setError("Enter a task");
                return;
            }

            String id = dbRef.push().getKey();

            if (id != null) {
                TodoTask task = new TodoTask(id, title, false);

                dbRef.child(id).setValue(task)
                        .addOnSuccessListener(unused -> {
                            etTask.setText("");
                            selectedKey = null;
                            Toast.makeText(this, "Task added",
                                    Toast.LENGTH_SHORT).show();
                        })
                        .addOnFailureListener(e ->
                                Toast.makeText(this,
                                        "Failed: " + e.getMessage(),
                                        Toast.LENGTH_LONG).show());
            }
        });

        // Select a task from the list
        listView.setOnItemClickListener((parent, view, position, rowId) -> {
            selectedKey = keys.get(position);

            dbRef.child(selectedKey).get()
                    .addOnSuccessListener(snapshot -> {
                        TodoTask task =
                                snapshot.getValue(TodoTask.class);

                        if (task != null) {
                            etTask.setText(task.title);
                        }
                    });
        });

        // Update a selected task
        btnUpdate.setOnClickListener(v -> {
            String title = etTask.getText().toString().trim();

            if (selectedKey == null) {
                Toast.makeText(this, "Select a task first",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            if (title.isEmpty()) {
                etTask.setError("Enter a task");
                return;
            }

            dbRef.child(selectedKey).child("title").setValue(title)
                    .addOnSuccessListener(unused -> {
                        etTask.setText("");
                        selectedKey = null;
                        Toast.makeText(this, "Task updated",
                                Toast.LENGTH_SHORT).show();
                    })
                    .addOnFailureListener(e ->
                            Toast.makeText(this,
                                    "Update failed: " + e.getMessage(),
                                    Toast.LENGTH_LONG).show());
        });

        // Delete a selected task
        btnDelete.setOnClickListener(v -> {
            if (selectedKey == null) {
                Toast.makeText(this, "Select a task first",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            dbRef.child(selectedKey).removeValue()
                    .addOnSuccessListener(unused -> {
                        etTask.setText("");
                        selectedKey = null;
                        Toast.makeText(this, "Task deleted",
                                Toast.LENGTH_SHORT).show();
                    })
                    .addOnFailureListener(e ->
                            Toast.makeText(this,
                                    "Delete failed: " + e.getMessage(),
                                    Toast.LENGTH_LONG).show());
        });
    }
}

```
## ACTIVITY_MAIN.XML:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="My To-Do List"
        android:textSize="26sp"
        android:textStyle="bold"
        android:paddingBottom="16dp" />

    <EditText
        android:id="@+id/etTask"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter your task"
        android:inputType="textCapSentences" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnAdd"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Add" />

        <Button
            android:id="@+id/btnUpdate"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Update" />

        <Button
            android:id="@+id/btnDelete"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Delete" />
    </LinearLayout>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>
```


## OUTPUT

<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/c6cc9fc7-70b6-40c7-887d-0c2604d70b1e" />

<img width="1917" height="1027" alt="image" src="https://github.com/user-attachments/assets/e812b371-a702-4884-9895-03589a8b330e" />






## RESULT
Thus a Simple Android Application create a firebase database and to display the To do List using Firebase Real Time Database in Android Studio is developed and executed successfully.
