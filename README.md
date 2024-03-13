# PR7

Завдання 1 «Робота SQLite без застосування спеціальних класів-
адаптерів»

1. Створіть новий проект SQLTest, головна активність якого буде розширювати клас ListActivity.

2. В методі onCreate головної активності зробіть відкриття або
створення БД, використовуючи в якості основи інформацію з пункту «Робота
з СУБД без адаптера» і методу onUpgrade допоміжного класу з попереднього
йому пункту. Після створення БД створіть таблицю з будь-яким полем (плюс
індекс), в яку додайте 3..5 унікальних записів зі строковим полем. Індекс
повинен інкрементуватись автоматично.

```java
package com.example.pr7_task1;

import android.annotation.SuppressLint;
import android.app.Activity;
import android.app.ListActivity;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.util.Log;

public class MainActivity extends Activity {
    private static final String DATABASE_NAME = "myDatabase.db";
    private static final String DATABASE_TABLE = "mainTable";
    private static final String DATABASE_CREATE = "create table "
            + DATABASE_TABLE + " ( _id integer primary key autoincrement,"
            + "column_one text not null);";

    SQLiteDatabase myDatabase;
    private void createDatabase() {
        myDatabase = openOrCreateDatabase(DATABASE_NAME, MODE_PRIVATE, null);
        Cursor cursor = myDatabase.rawQuery("SELECT * FROM sqlite_master WHERE type='table' AND name=?", new String[]{DATABASE_TABLE});

        if (cursor.getCount() == 0) {
            myDatabase.execSQL(DATABASE_CREATE);
        }
        cursor.close();
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        createDatabase();
    }
}
```

```java
    private void insertData() {
        myDatabase.execSQL("INSERT INTO " + DATABASE_TABLE + " (column_one) VALUES ('This is first value');");
        myDatabase.execSQL("INSERT INTO " + DATABASE_TABLE + " (column_one) VALUES ('This is second value');");
        myDatabase.execSQL("INSERT INTO " + DATABASE_TABLE + " (column_one) VALUES ('This is third value');");
    }

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        createDatabase();
        insertData();
    }
```
