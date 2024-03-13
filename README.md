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

3. В цьому ж методі зробіть запит до таблиці, який одержує всі рядки і за допомогою наявного курсора запишіть дані в масив рядків.

```java
    private String[] retrieveData() {
        Cursor cursor = myDatabase.rawQuery("SELECT * FROM " + DATABASE_TABLE, null);
        String[] data = new String[cursor.getCount()];
        int index = 0;
        if (cursor.moveToFirst()) {
            do {
                @SuppressLint("Range") String rowData = cursor.getString(cursor.getColumnIndex("column_one"));
                data[index++] = rowData;
            } while (cursor.moveToNext());
        }
        cursor.close();
        return data;
    }


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        createDatabase();
        List<String> retrievedData = Arrays.asList(retrieveData());
    }
```

4. За допомогою додаткової розмітки і адаптера ArrayAdapter
відобразіть отримані з СУБД дані на екрані активності.

```xml
    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

```java
ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, retrievedData);
        ListView listView = findViewById(R.id.listView);
        listView.setAdapter(adapter);
```

5. Добавте обробник кліку на елемент списку, щоб при кліці
запитували з БД індекс елементу з цим рядком і відобразіть його за допомогою Toast.

```java
listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                Cursor cursor = myDatabase.rawQuery("SELECT _id FROM " + DATABASE_TABLE + " WHERE column_one = ?", new String[]{retrievedData.get(position)});
                if (cursor.moveToFirst()) {
                    @SuppressLint("Range") int dataIndex = cursor.getInt(cursor.getColumnIndex("_id"));
                    int duration = Toast.LENGTH_LONG;
                    Toast.makeText(getApplicationContext(), "Index: " + dataIndex, duration).show();
                }
                cursor.close();
            }
        });
```

6.Добавте контекстне меню до елементів списку, що містить пункт «видалити» і реалізуйте видалення. Після видалення повинні проводиться дії з п.п. 3 і 4 для зміни складу відображуваних даних.

```java
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        if (v.getId() == R.id.listView) {
            AdapterView.AdapterContextMenuInfo info = (AdapterView.AdapterContextMenuInfo) menuInfo;
            menu.setHeaderTitle("Select Action");
            menu.add(Menu.NONE, 0, 0, "Delete");
        }
    }

    @Override
    public boolean onContextItemSelected(MenuItem item) {
        AdapterView.AdapterContextMenuInfo info = (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
        if (item.getItemId() == 0) {
            String selectedItem = ((ArrayAdapter<String>) ((ListView) findViewById(R.id.listView)).getAdapter()).getItem(info.position);
            myDatabase.delete(DATABASE_TABLE, "column_one=?", new String[]{selectedItem});
            ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, Arrays.asList(retrieveData()));
            ListView listView = findViewById(R.id.listView);
            listView.setAdapter(adapter);
            return true;
        }
        return super.onContextItemSelected(item);
    }
```
