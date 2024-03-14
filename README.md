# ПРАКТИЧНА РОБОТА N7
**Тема роботи:** Робота з базами даних в ОС Android.

**Мета роботи:** Опанувати принципи роботи з СУБД SQLite без застосування спеціальних класів-адаптерів.

## Завдання 1 «Робота SQLite без застосування спеціальних класів-адаптерів»

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
Результат роботи:


https://github.com/katushhiaa/PR7/assets/113555695/56048dbf-a39f-43f0-abff-d6598bbd5212


## Завдання 2 «Робота SQLite з класом-адаптером»
1. Створіть новий проект NotesSample.
2. Для простоти використання всі дії із записами будуть проводитися в
рамках однієї активності, тому використовуйте максимально спрощений
інтерфейс, як показано на наступній сторінці. Для реалізації можна
скористатися такою розміткою у файлі res / layout / main.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >
    <RelativeLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content" >
        <Button
            android:id="@+id/save_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:text="@android:string/ok" />
        <EditText
            android:id="@+id/edit_text"
            android:layout_width="fill_parent"
            android:layout_toLeftOf="@id/save_button"
            android:layout_height="wrap_content"
            android:hint = "Новий запис" />

    </RelativeLayout>
    <ListView
        android:id="@+id/myListView"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```
    
3. Реалізуйте збереження записів після натискання на кнопку, а видалення через контекстне меню. Робота з СУБД повинна здійснюватися з використанням адаптера.
   
Створення внутрішього клас, що наслідує SQLiteOpenHelper, який відповідає за створення та управління базою даних SQLite для зберігання нотаток. Він містить методи для створення, оновлення, вставки та видалення даних.

```java
    static class MyDataBase extends SQLiteOpenHelper {
        private static final String DATABASE_NAME = "notes.db";
        private static final String TABLE_NAME = "notes_table";
        private static final String COL_ID = "_id";
        private static final String COL_NOTE = "note";
        private static final int DATABASE_VERSION = 1;

        MyDataBase(Context context) {
            super(context, DATABASE_NAME, null, DATABASE_VERSION);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("CREATE TABLE " + TABLE_NAME + " (" +
                    COL_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    COL_NOTE + " TEXT)");
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
            onCreate(db);
        }

        public boolean insertData(String note) {
            SQLiteDatabase db = this.getWritableDatabase();
            ContentValues contentValues = new ContentValues();
            contentValues.put(COL_NOTE, note);
            long result = db.insert(TABLE_NAME, null, contentValues);
            return result != -1;
        }

        public Cursor getAllData() {
            SQLiteDatabase db = this.getWritableDatabase();
            return db.rawQuery("SELECT * FROM " + TABLE_NAME, null);
        }

        public boolean deleteData(long id) {
            SQLiteDatabase db = this.getWritableDatabase();
            return db.delete(TABLE_NAME, COL_ID + "=" + id, null) > 0;
        }
    }
```

4. Метод onCreate():
   ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editText = findViewById(R.id.edit_text);
        saveButton = findViewById(R.id.save_button);
        listView = findViewById(R.id.myListView);
        notes = new ArrayList<>();
        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, notes);
        listView.setAdapter(adapter);

        myDB = new MyDataBase(this);

        registerForContextMenu(listView);

        displayData();
    }
   ```

5. Обробник натискання кнопки "Ок"
   ```java
   saveButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String note = editText.getText().toString().trim();
                if (!note.isEmpty()) {
                    myDB.insertData(note);
                    displayData();
                    editText.getText().clear();
                }
            }
        });
   ```

6. Метод, що викликається для створення контекстного меню для списку нотаток
   ```java
       @Override
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        menu.add(Menu.NONE, Menu.FIRST, Menu.NONE, "Видалити");
    }
   ```
7. Обробник вибору пункту контекстного меню. Він видаляє вибрану нотатку з бази даних та оновлює список нотаток
   ```java
       @Override
    public boolean onContextItemSelected(MenuItem item) {
        AdapterView.AdapterContextMenuInfo info = (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
        switch (item.getItemId()) {
            case Menu.FIRST:
                Cursor cursor = myDB.getAllData();
                if (cursor.moveToPosition(info.position)) {
                    @SuppressLint("Range") long id = cursor.getLong(cursor.getColumnIndex(MyDataBase.COL_ID));
                    myDB.deleteData(id);
                    displayData();
                }
                return true;
            default:
                return super.onContextItemSelected(item);
        }
    }
   ```
8. Метод для відображення нотаток з бази даних у списку
   ```java
       private void displayData() {
        notes.clear();
        Cursor cursor = myDB.getAllData();
        if (cursor.moveToFirst()) {
            do {
                notes.add(cursor.getString(1));
            } while (cursor.moveToNext());
        }
        adapter.notifyDataSetChanged();
    }
   ```
Результат:

https://github.com/katushhiaa/PR7/assets/113555695/e2d498d7-7d50-4171-a7e7-e74bbb8fb3b6


