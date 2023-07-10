# Music-Player-Android-Studio-Project


**Build Gradle in Gradle Scripts folder**
//I have imported dexter libraries in order to get permissions from the devices after installing the application.
//Copy this line and paste it in the dependencies tag.
implementation 'com.karumi:dexter:6.2.3'



**MainActivity.java:**
package com.example.musicplayer;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.os.Environment;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.BaseAdapter;
import android.widget.ListView;
import android.widget.TextView;

import com.karumi.dexter.Dexter;
import com.karumi.dexter.PermissionToken;
import com.karumi.dexter.listener.PermissionDeniedResponse;
import com.karumi.dexter.listener.PermissionGrantedResponse;
import com.karumi.dexter.listener.PermissionRequest;
import com.karumi.dexter.listener.single.PermissionListener;

import java.io.File;
import java.lang.reflect.Array;
import java.util.ArrayList;


public class MainActivity extends AppCompatActivity {

    ListView listView;
    String[] items;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        listView = findViewById(R.id.listViewSong);
//        runtimePermission();
    }

//    public void runtimePermission(){
//        Dexter.withContext(this).withPermission(Manifest.permission.READ_EXTERNAL_STORAGE).withListener(new PermissionListener() {
//            @Override
//            public void onPermissionGranted(PermissionGrantedResponse permissionGrantedResponse) {
//                displaySongs();
//            }
//
//            @Override
//            public void onPermissionDenied(PermissionDeniedResponse permissionDeniedResponse) {
//
//            }
//
//            @Override
//            public void onPermissionRationaleShouldBeShown(PermissionRequest permissionRequest, PermissionToken permissionToken) {
//                permissionToken.continuePermissionRequest();
//            }
//        }).check();
//    }

    public ArrayList<File> findSong (File file){
        ArrayList<File> arrayList = new ArrayList<>();

        File[] files = file.listFiles();

        for (File singlefile: files){
            if (singlefile.isDirectory() && !singlefile.isHidden()){
                arrayList.addAll(findSong(singlefile));
            }
            else{
                if (singlefile.getName().endsWith(".mp3") || singlefile.getName().endsWith(".wav") ){
                    arrayList.add(singlefile);
                }
            }
        }
        return arrayList;
    }
    void displaySongs(){
        final ArrayList<File> mySongs = findSong(Environment.getExternalStorageDirectory());

        items = new String[mySongs.size()];
        for (int i=0; i< mySongs.size();i++){
            items[i] = mySongs.get(i).getName().toString().replace(".mp3","").replace(".wav","");
        }
//        ArrayAdapter<String> myAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, items);
//        listView.setAdapter(myAdapter);

        customAdapter customAdapter = new customAdapter();
        listView.setAdapter(customAdapter);

        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
                String songName = (String) listView.getItemAtPosition(i);
                startActivity(new Intent(getApplicationContext(), PlayerActivity.class)
                        .putExtra("songs",mySongs)
                        .putExtra("songname",songName)
                        .putExtra("pos",i));
            }
        });
    }

    class customAdapter extends BaseAdapter{

        @Override
        public int getCount() {
            return items.length;
        }

        @Override
        public Object getItem(int i) {
            return null;
        }

        @Override
        public long getItemId(int i) {
            return 0;
        }

        @Override
        public View getView(int i, View view, ViewGroup viewGroup) {

            View myView = getLayoutInflater().inflate(R.layout.list_item,null);
            TextView textsong = myView.findViewById(R.id.txtsongname);
            textsong.setSelected(true);
            textsong.setText(items[i]);

            return myView;
        }
    }
}



**MainActivity.xml:**
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:background="@color/black">

    <ListView
        android:id="@+id/listViewSong"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:divider="@android:color/transparent"
        android:padding="8dp"
        android:dividerHeight="10.0sp" />
</RelativeLayout>





**AndroidManifest.xml**
//Make changes after the manifest tag in order to ask the user to access the device storage
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />





**PlayerActivity.java**
//Make another empty activity
package com.example.musicplayer;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import android.animation.AnimatorSet;
import android.animation.ObjectAnimator;
import android.annotation.SuppressLint;
import android.content.Intent;
import android.graphics.PorterDuff;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Bundle;
import android.os.Handler;
import android.provider.MediaStore;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.SeekBar;
import android.widget.TextView;

import java.io.File;
import java.util.ArrayList;

public class PlayerActivity extends AppCompatActivity {

    Button btnplay , btnnext , btnprev , btnff, btnfr;
    TextView txtsname, txtsstart, txtsstop;
    SeekBar seekmusic;
    ImageView imageView;

    String sname;
    public static final String EXTRA_NAME = "song_name";
    static MediaPlayer mediaPlayer;
    int position;
    ArrayList<File> mySongs;
    Thread updateseekbar;

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        if (item.getItemId() == android.R.id.home){
            onBackPressed();
        }
        return super.onOptionsItemSelected(item);
    }


    @SuppressLint("MissingInflatedId")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_player);


        getSupportActionBar().setTitle("Now Playing");
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        getSupportActionBar().setDisplayShowHomeEnabled(true);

        btnprev = findViewById(R.id.btnprev);
        btnnext = findViewById(R.id.btnnext);
        btnplay = findViewById(R.id.playbtn);
        btnff = findViewById(R.id.btnff);
        btnfr = findViewById(R.id.btnfr);
        txtsname = findViewById(R.id.txtsn);
        txtsstart = findViewById(R.id.txtsstart);
        txtsstop = findViewById(R.id.txtsstop);
        seekmusic = findViewById(R.id.seekbar);
        imageView = findViewById(R.id.imageview);

        if (mediaPlayer != null){
            mediaPlayer.stop();
            mediaPlayer.release();
        }

        Intent i =getIntent();
        Bundle bundle = i.getExtras();

        mySongs = (ArrayList) bundle.getParcelableArrayList("songs");
        String songName = i.getStringExtra("songname");
        position = bundle.getInt("pos",0);
        txtsname.setSelected(true);
        Uri uri = Uri.parse(mySongs.get(position).toString());
        sname = mySongs.get(position).getName();
        txtsname.setText(sname);

        mediaPlayer = MediaPlayer.create(getApplicationContext(), uri);
        mediaPlayer.start();

        updateseekbar = new Thread(){
            @Override
            public void run() {
                int totalDuration = mediaPlayer.getDuration();
                int currentposition = 0;

                while(currentposition<totalDuration){
                    try {
                        sleep(500);
                        currentposition = mediaPlayer.getCurrentPosition();
                        seekmusic.setProgress(currentposition);
                    }
                    catch (InterruptedException | IllegalStateException e){
                        e.printStackTrace();
                    }
                }
            }
        };

        seekmusic.setMax(mediaPlayer.getDuration());
        updateseekbar.start();
        seekmusic.getProgressDrawable().setColorFilter(getResources().getColor(R.color.purple_200), PorterDuff.Mode.MULTIPLY);
        seekmusic.getThumb().setColorFilter(getResources().getColor(R.color.purple_200),PorterDuff.Mode.SRC_IN);

        seekmusic.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int i, boolean b) {

            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {

            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                mediaPlayer.seekTo(seekBar.getProgress());
            }
        });

        String endTime = creatTime(mediaPlayer.getDuration());
        txtsstop.setText(endTime);

        final Handler handler = new Handler();
        final int delay = 1000;

        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                String currentTime = creatTime(mediaPlayer.getCurrentPosition());
                txtsstart.setText(currentTime);
                handler.postDelayed(this,delay);
            }
        },delay);

        btnplay.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if(mediaPlayer.isPlaying()){
                    btnplay.setBackgroundResource(R.drawable.ic_play);
                    mediaPlayer.pause();
                }
                else {
                    btnplay.setBackgroundResource(R.drawable.ic_pause);
                    mediaPlayer.start();
                }
            }
        });

        //next listener
        mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
            @Override
            public void onCompletion(MediaPlayer mediaPlayer) {
                btnnext.performClick();
            }
        });

        btnnext.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                mediaPlayer.stop();
                mediaPlayer.release();
                position=((position+1)% mySongs.size());
                Uri u = Uri.parse(mySongs.get(position).toString());
                mediaPlayer = MediaPlayer.create(getApplicationContext(), u);
                sname = mySongs.get(position).getName();
                txtsname.setText(sname);
                mediaPlayer.start();
                btnplay.setBackgroundResource(R.drawable.ic_pause);
                startAnimation(imageView);
            }
        });


        btnprev.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                mediaPlayer.stop();
                mediaPlayer.release();
                position = ((position-1)<0)?(mySongs.size()-1):(position-1);

                Uri u = Uri.parse(mySongs.get(position).toString());
                mediaPlayer = MediaPlayer.create(getApplicationContext(),u);
                sname = mySongs.get(position).getName();
                txtsname.setText(sname);
                mediaPlayer.start();
                btnplay.setBackgroundResource(R.drawable.ic_pause);
                startAnimation(imageView);
            }
        });

        btnff.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (mediaPlayer.isPlaying()){
                    mediaPlayer.seekTo(mediaPlayer.getCurrentPosition()+10000);
                }
            }
        });


        btnfr.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (mediaPlayer.isPlaying()){
                    mediaPlayer.seekTo(mediaPlayer.getCurrentPosition()-10000);
                }
            }
        });
    }


    public void startAnimation(View view){
        ObjectAnimator animator = ObjectAnimator.ofFloat(imageView, "rotation" , 0f,360f);
        animator.setDuration(1000);
        AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.playTogether(animator);
        animatorSet.start();
    }


    public String creatTime(int duration){
        String time = "";
        int min = duration/1000/60;
        int sec = duration/1000%60;


        time+=min+":";

        if (sec<10){
            time+="0";
        }

        time+=sec;
        return time;
    }
}



**activity_player.xml**


<?xml version="1.0" encoding="utf-8"?>
<androidx.appcompat.widget.LinearLayoutCompat xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/background_img"
    android:orientation="vertical"
    android:weightSum="10"
    tools:context=".PlayerActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="7"
        android:gravity="center"
        android:orientation="vertical">

        <TextView
            android:id="@+id/txtsn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            android:ellipsize="marquee"
            android:marqueeRepeatLimit="marquee_forever"
            android:padding="10dp"
            android:singleLine="true"
            android:text="Song Name"
            android:textAlignment="center"
            android:textColor="#FFF"
            android:textSize="22sp"
            android:textStyle="italic" >
        </TextView>


        <ImageView
            android:id="@+id/imageview"
            android:src="@drawable/music"
            android:layout_marginBottom="8dp"
            android:layout_width="250dp"
            android:layout_height="250dp">

        </ImageView>


        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="60dp">
            <SeekBar
                android:id="@+id/seekbar"
                android:layout_centerInParent="true"
                android:layout_alignParentBottom="true"
                android:layout_margin="20dp"
                android:layout_marginBottom="40dp"
                android:layout_width="250dp"
                android:layout_height="wrap_content">

            </SeekBar>

            <TextView
                android:id="@+id/txtsstart"
                android:layout_toLeftOf="@+id/seekbar"
                android:layout_centerInParent="true"
                android:layout_alignParentLeft="false"
                android:layout_marginLeft="20dp"
                android:text="0:10"
                android:textColor="#FFF"
                android:textSize="14sp"

                android:layout_width="wrap_content"
                android:layout_height="wrap_content">

            </TextView>


            <TextView
                android:id="@+id/txtsstop"
                android:layout_toRightOf="@+id/seekbar"
                android:layout_centerInParent="true"
                android:layout_alignParentRight="false"
                android:layout_marginRight="20dp"
                android:text="4:10"
                android:textColor="#FFF"
                android:textSize="14sp"

                android:layout_width="wrap_content"
                android:layout_height="wrap_content">

            </TextView>

        </RelativeLayout>
    </LinearLayout>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="3">
        <RelativeLayout

            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <Button
                android:id="@+id/playbtn"
                android:layout_width="70dp"
                android:layout_height="70dp"
                android:layout_centerHorizontal="true"
                android:background="@drawable/ic_pause"
                app:backgroundTint="#FF362E" />


            <Button
                android:id="@+id/btnnext"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_marginLeft="2dp"
                android:layout_marginTop="15dp"
                android:layout_toRightOf="@id/playbtn"
                android:background="@drawable/ic_next"
                app:backgroundTint="#FF362E" />


            <Button
                android:id="@+id/btnprev"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_marginRight="2dp"
                android:layout_marginTop="15dp"
                android:layout_toLeftOf="@id/playbtn"
                android:background="@drawable/ic_previous"
                app:backgroundTint="#FF362E" />


            <Button
                android:id="@+id/btnff"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:layout_toRightOf="@id/btnnext"
                android:layout_marginTop="20dp"
                android:layout_marginLeft="15dp"
                android:background="@drawable/ic_fast_forward"
                app:backgroundTint="#FFFFFF" />


            <Button
                android:id="@+id/btnfr"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:layout_toLeftOf="@id/btnprev"
                android:layout_marginTop="20dp"
                android:layout_marginLeft="15dp"
                android:background="@drawable/ic_fast_rewind"
                app:backgroundTint="#FFFFFF" >
            </Button>


<!--            <com.gauravk.audiovisualizer.visualizer.BarVisualizer-->
<!--                xmlns:custom="http://schemas.android.com/apk/res-auto"-->
<!--                android:id="@+id/blast"-->
<!--                android:layout_width="match_parent"-->
<!--                android:layout_height="70dp"-->
<!--                android:layout_alignParentBottom="true"-->
<!--                custom:avDensity="0.5"-->
<!--                custom:avType="outline"-->
<!--                custom:avWidth="4dp"-->
<!--                custom:avColor="#FF362E"-->
<!--                custom:avSpeed="normal"/>-->


        </RelativeLayout>
    </LinearLayout>

</androidx.appcompat.widget.LinearLayoutCompat>





**list_item.xml**
//make an xml in layout folder of res in android project
//making this xml file in order to design the list items

<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginStart="8dp"
    android:layout_marginTop="8dp"
    android:layout_marginEnd="8dp">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="34dp"
        android:background="@drawable/list_bg"
        android:padding="8dp">

        <ImageView
            android:id="@+id/imgsong"
            android:layout_width="37dp"
            android:layout_height="26dp"
            android:layout_alignParentStart="true"
            android:layout_alignParentTop="true"
            android:layout_marginStart="0dp"
            android:layout_marginTop="0dp"
            android:background="@drawable/list_bg"
            android:src="@drawable/ic_music"></ImageView>

        <TextView
            android:id="@+id/txtsongname"
            android:layout_width="wrap_content"
            android:layout_height="40dp"
            android:layout_alignParentEnd="true"
            android:layout_marginStart="5dp"
            android:layout_marginEnd="5dp"
            android:layout_toEndOf="@id/imgsong"
            android:ellipsize="marquee"
            android:marqueeRepeatLimit="marquee_forever"
            android:padding="6dp"
            android:scrollHorizontally="true"
            android:singleLine="true"
            android:text="Song Name"
            android:textColor="@color/white"
            android:textSize="15sp">

        </TextView>
    </RelativeLayout>



</androidx.cardview.widget.CardView>



**list_bg.xml**
//make another xml file in order to make a background for the list


<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="@color/black"></solid>
    <stroke android:width="1dp"
        android:color="#FF362E"></stroke>


</shape>
