# da
**DnevnikDBHelper.java**

package com.example.marksrecordapp.db;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.provider.BaseColumns;

import androidx.annotation.Nullable;

import com.example.marksrecordapp.utils.MyAppUtils;

import java.util.ArrayList;

public class DnevnikDbHelper extends SQLiteOpenHelper {

    public static final int DB_VERSION = 1;
    public static final String DB_NAME = "dnevnik.db";

    public DnevnikDbHelper(@Nullable Context context) {
        super(context, DB_NAME, null, DB_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        sqLiteDatabase.execSQL(Predmet.SQL_CREATE_TABLE_PREDMETI);
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {

    }

    public static class Predmet implements BaseColumns {



        private Predmet(){}

        public static final String TABLE_NAME = "predmeti";
        public static final String COLUMN_NAZIV = "naziv";
        public static final String COLUMN_OCENE = "ocene";

        public static String SQL_CREATE_TABLE_PREDMETI = "CREATE TABLE " +TABLE_NAME + "(" +
                _ID + " INTEGER PRIMARY KEY, " +
                COLUMN_NAZIV + " TEXT UNIQUE, " +
                COLUMN_OCENE + " TEXT);";

        public static String SQL_DELETE_TABLE_PREDMETI = "DROP TABLE IF EXISTS " + TABLE_NAME + ";";


    }

    public ArrayList<String[]> getPredmet(){
        ArrayList<String[]> izlaz = new ArrayList<>();
        SQLiteDatabase db = getReadableDatabase();
        Cursor cursor = db.query(
             Predmet.TABLE_NAME,
             new String[]{Predmet.COLUMN_NAZIV,Predmet.COLUMN_OCENE},
             null, null, null, null, null
        );
        if (cursor.moveToFirst()){
            do {
                izlaz.add(new String[]{cursor.getString(0),cursor.getString(1)});
            } while (cursor.moveToNext());
        }
        cursor.close();
        db.close();
        return izlaz;
    }

    @Nullable
    public String[] getPredmet(String naziv){
        String[] izlaz = new String[2];
        SQLiteDatabase db = getReadableDatabase();
        Cursor cursor = db.query(
                Predmet.TABLE_NAME,
                new String[]{Predmet.COLUMN_NAZIV,Predmet.COLUMN_OCENE},
                Predmet.COLUMN_NAZIV + " = ?",
                new String[]{naziv},
                null, null, null, null
        );
        if (cursor.moveToFirst()){
            izlaz[0] = cursor.getString(0);
            izlaz[1] = cursor.getString(1);
            cursor.close();
            db.close();
            return izlaz;
        } else {
            cursor.close();
            db.close();
            return null;
        }
    }

    public void addPredmet(String naziv) {
        SQLiteDatabase db = getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(Predmet.COLUMN_NAZIV,naziv);
        values.put(Predmet.COLUMN_OCENE,"");
        db.insert(Predmet.TABLE_NAME, null,values);
        db.close();
    }


    public boolean addOcena(String naziv, int ocena){
        if(ocena < 1 || ocena > 5) return false;
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor = db.query(
                Predmet.TABLE_NAME,
                new String[]{Predmet.COLUMN_OCENE},
                Predmet.COLUMN_NAZIV + " = ?",
                new String[]{naziv},
                null, null, null
        );
        String ocene_str = "";
        if(cursor.moveToFirst()){
            ocene_str = cursor.getString(0);
            int[] ocena_int = MyAppUtils.getArrayOcena(ocene_str);
            int index = MyAppUtils.findFirstIndexOfIntArray(ocena_int, 0);
            ocena_int[index] = ocena;
            ocene_str = MyAppUtils.getStringArray(ocena_int);
        } else {
            cursor.close();
            db.close();
            return false;
        }
        cursor.close();
        ContentValues values = new ContentValues();
        values.put(Predmet.COLUMN_OCENE,ocene_str);
        db.update(
                Predmet.TABLE_NAME,
                values,
                Predmet.COLUMN_NAZIV + " = ?",
                new String[]{naziv}
        );
        db.close();
        return true;
    }

    public boolean deleteOcena(String naziv, int index) {
        SQLiteDatabase db = getWritableDatabase();
        Cursor cursor = db.query(
                Predmet.TABLE_NAME,
                new String[]{Predmet.COLUMN_OCENE},
                Predmet.COLUMN_NAZIV + " = ?",
                new String[]{naziv},
                null, null, null
        );
        String ocene_str = "";
        if (cursor.moveToFirst()) {
            ocene_str = cursor.getString(0);
            int[] ocena_int = MyAppUtils.getArrayOcena(ocene_str);
            if (index >= ocena_int.length) {
                cursor.close();
                db.close();
                return false;
            }
            System.arraycopy(ocena_int, index + 1, ocena_int, index, ocena_int.length - index - 1);
            ocene_str = MyAppUtils.getStringArray(ocena_int);
        } else {
            cursor.close();
            db.close();
            return false;
        }
        cursor.close();
        ContentValues values = new ContentValues();
        values.put(Predmet.COLUMN_OCENE, ocene_str);
        db.update(
                Predmet.TABLE_NAME,
                values,
                Predmet.COLUMN_NAZIV + " = ?",
                new String[]{naziv}
        );
        db.close();
        return true;
    }

    public void deletePredmet(String naziv) {
        SQLiteDatabase db = getWritableDatabase();
        db.delete(Predmet.TABLE_NAME, Predmet.COLUMN_NAZIV + " = ?", new String[]{naziv});
        db.close();
    }

}


**MainActivity.java**

package com.example.marksrecordapp;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;

import com.example.marksrecordapp.utils.MyAppUtils;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        MyAppUtils.test(this);
    }
}



**MyAppUtils.java**

package com.example.marksrecordapp.utils;

import android.content.Context;

import androidx.annotation.NonNull;

import com.example.marksrecordapp.db.DnevnikDbHelper;

import java.util.ArrayList;

public interface MyAppUtils {

    static void test(Context context) {
        DnevnikDbHelper dbHelper = new DnevnikDbHelper(context);
        // dbHelper.addPredmet("Srpski jezik");
        // dbHelper.addOcena("Srpski jezik", 3);
        // dbHelper.addPredmet("Matematika");
        // dbHelper.addOcena("Matematika", 5);
        // dbHelper.addOcena("Matematika", 3);
        String[] predmet = dbHelper.getPredmet("Matematika");
        ArrayList<String[]> predmeti = dbHelper.getPredmet();
    }

    @NonNull
    static int[] getArrayOcena(@NonNull String ocena_str) {
        final int CODE_0 = 48;
        final int CODE_6 = 54;
        int[] izlaz = new int[20];
        int j = 0;
        for (int i = 0; i < ocena_str.length(); i++){
            int code = ocena_str.charAt(i);
            if (code > CODE_0 && code < CODE_6) izlaz[j++] = code - CODE_0;
        }
        return izlaz;
    }

    @NonNull
    static String getStringArray(@NonNull int[] ocene_int) {
        StringBuilder izlaz = new StringBuilder();
        for (int o:ocene_int){
            if (o == 0) return izlaz.toString();
            izlaz.append(" ").append(o);
        }
        return izlaz.toString();
    }

    @NonNull
    static int findFirstIndexOfIntArray(int[] array, int element){
        for (int i = 0; i < array.length; i++) {
            if (array[i] == element) return i;
        }
        return -1;
    }

}

