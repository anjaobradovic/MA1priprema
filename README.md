1. Unutar MainActivity postaviti Toolbar i LinearLayout sa svetlozelenom pozadinom. (1)
2. Kreirati fragment: MovieFragment. (1)
3. MovieFragment sadrži RecyclerView i služi za prikaz filma. U dnu fragmenta dodati dva dugmeta: dugme
sa nazivom “Dodaj” i dugme sa nazivom “Snimaj” koje je onemogućeno (disabled). (2)
4. Dodati meni komponentu sa stavkom Movie. (0.5)
5. Klikom na Movie, unutar MainActivity se prikazuje MovieFragment. (0.5)
6. Klikom na dugme “Dodaj” otvara se forma za dodavanje novog filma. (0.5)
7. Forma ima: naziv filma, ocenu (EditText brojčanog tipa), Checkbox ispred koga stoji labela “Odgledano”,
dugme za potvrdu i dugme za odustajanje (1.5). Klikom na dugme za odustajanje zatvoriti formu (0.5). Klikom
na potvrdu dodati film u RecyclerView (3).
8. Kreirati BroadcastReceiver. Receiver osluškuje dodavanje filma i pamti najveću ocenu do sada. Nakon
svakog dodavanja, u Toast poruci ispisati naziv filma koji ima najveću ocenu. (4)
9. Kreirati servis koji se pokreće na svaki minut (2.5) i proverava da li je dozvoljena kamera (obavezno tražiti
dozvolu) (1.5). Ako je kamera dozvoljena, omogućiti (enable) dugme “Snimaj”. (1.5)


# Kolokvijum1 — Android (Java)

Rešenje kolokvijumskog zadatka: **Toolbar + meni**, **Fragment sa RecyclerView-om**, **forma (DialogFragment) za dodavanje filma**, **BroadcastReceiver** koji pamti najveću ocenu i **Service** koji na svaki minut proverava dozvolu za kameru.

- Jezik: **Java**
- Build script: **Groovy (build.gradle)**
- Template: **Empty Views Activity** (`MainActivity`)

---

## Sadržaj

1. [Kreiranje projekta i zavisnosti](#1-kreiranje-projekta-i-zavisnosti)
2. [AndroidManifest](#2-androidmanifest)
3. [Resursi: boje, tema, meni](#3-resursi-boje-tema-meni)
4. [Layout-ovi](#4-layout-ovi)
5. [Model i adapter](#5-model-i-adapter)
6. [MovieFragment](#6-moviefragment)
7. [Forma za dodavanje filma (DialogFragment)](#7-forma-za-dodavanje-filma-dialogfragment)
8. [BroadcastReceiver — najveća ocena](#8-broadcastreceiver--najveća-ocena)
9. [Service — provera kamere na svaki minut](#9-service--provera-kamere-na-svaki-minut)
10. [MainActivity](#10-mainactivity)
11. [Testiranje](#11-testiranje)

---

## 1. Kreiranje projekta i zavisnosti

`File → New → New Project → Empty Views Activity`

| Polje | Vrednost |
|---|---|
| Name | `Kolokvijum1` |
| Package name | `com.example.kolokvijum1` |
| Language | `Java` |
| Minimum SDK | `API 24` |
| Build configuration language | `Groovy DSL (build.gradle)` |

**Struktura koju ćeš napraviti:**

```
app/src/main/java/com/example/kolokvijum1/
├── MainActivity.java
├── MovieFragment.java
├── AddMovieDialogFragment.java
├── MovieAdapter.java
├── Movie.java
├── MovieReceiver.java
└── CameraCheckService.java

app/src/main/res/
├── layout/activity_main.xml
├── layout/fragment_movie.xml
├── layout/item_movie.xml
├── layout/dialog_add_movie.xml
└── menu/main_menu.xml
```

### `app/build.gradle` (Module :app)

```groovy
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.example.kolokvijum1'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.kolokvijum1"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.recyclerview:recyclerview:1.3.2'
    implementation 'androidx.fragment:fragment:1.6.2'
}
```

→ **Sync Now**

---

## 2. AndroidManifest

`app/src/main/AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Zadatak 9: dozvola za kameru -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-feature android:name="android.hardware.camera" android:required="false" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.Kolokvijum1">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Zadatak 9: servis -->
        <service
            android:name=".CameraCheckService"
            android:enabled="true"
            android:exported="false" />

    </application>
</manifest>
```

---

## 3. Resursi: boje, tema, meni

### `res/values/colors.xml` — dodaj svetlozelenu

```xml
<color name="svetlo_zelena">#C8E6C9</color>
```

### `res/values/themes.xml` — tema mora biti **NoActionBar**

Pošto sami postavljamo `Toolbar`, tema ne sme da ima ugrađen ActionBar (inače puca sa
`This Activity already has an action bar supplied by the window decor`).

```xml
<style name="Base.Theme.Kolokvijum1" parent="Theme.Material3.DayNight.NoActionBar">
    ...
</style>
```

> Ako u tvom projektu piše `Theme.MaterialComponents.DayNight.DarkActionBar`, samo zameni
> `DarkActionBar` sa `NoActionBar`.

### `res/menu/main_menu.xml` — zadatak 4

> Desni klik na `res` → `New → Android Resource Directory` → Resource type: **menu**,
> pa u njemu `New → Menu Resource File` → naziv `main_menu`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/menu_movie"
        android:title="Movie"
        app:showAsAction="always" />
</menu>
```

---

## 4. Layout-ovi

### `res/layout/activity_main.xml` — zadatak 1

Toolbar + `LinearLayout` sa svetlozelenom pozadinom; `FrameLayout` je kontejner u koji se ubacuje fragment.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@color/svetlo_zelena">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        android:title="Kolokvijum1"
        android:titleTextColor="@android:color/white" />

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

### `res/layout/fragment_movie.xml` — zadatak 3

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="8dp">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <!-- dva dugmeta u dnu fragmenta -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnDodaj"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Dodaj" />

        <Button
            android:id="@+id/btnSnimaj"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:enabled="false"
            android:text="Snimaj" />
    </LinearLayout>
</LinearLayout>
```

### `res/layout/item_movie.xml` — jedan red u RecyclerView-u

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="12dp">

    <TextView
        android:id="@+id/tvNaziv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/tvDetalji"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="14sp" />
</LinearLayout>
```

### `res/layout/dialog_add_movie.xml` — zadatak 7 (forma)

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="20dp">

    <EditText
        android:id="@+id/etNaziv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Naziv filma"
        android:inputType="text" />

    <EditText
        android:id="@+id/etOcena"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Ocena"
        android:inputType="numberDecimal" />

    <!-- labela ISPRED checkbox-a -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Odgledano"
            android:textSize="16sp" />

        <CheckBox
            android:id="@+id/cbOdgledano"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnPotvrdi"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Potvrdi" />

        <Button
            android:id="@+id/btnOdustani"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Odustani" />
    </LinearLayout>
</LinearLayout>
```

---

## 5. Model i adapter

### `Movie.java`

```java
package com.example.kolokvijum1;

import java.io.Serializable;

public class Movie implements Serializable {

    private String naziv;
    private double ocena;
    private boolean odgledano;

    public Movie(String naziv, double ocena, boolean odgledano) {
        this.naziv = naziv;
        this.ocena = ocena;
        this.odgledano = odgledano;
    }

    public String getNaziv() { return naziv; }
    public double getOcena() { return ocena; }
    public boolean isOdgledano() { return odgledano; }
}
```

### `MovieAdapter.java`

```java
package com.example.kolokvijum1;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.List;

public class MovieAdapter extends RecyclerView.Adapter<MovieAdapter.MovieViewHolder> {

    private final List<Movie> filmovi;

    public MovieAdapter(List<Movie> filmovi) {
        this.filmovi = filmovi;
    }

    static class MovieViewHolder extends RecyclerView.ViewHolder {
        TextView tvNaziv, tvDetalji;

        MovieViewHolder(@NonNull View itemView) {
            super(itemView);
            tvNaziv = itemView.findViewById(R.id.tvNaziv);
            tvDetalji = itemView.findViewById(R.id.tvDetalji);
        }
    }

    @NonNull
    @Override
    public MovieViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_movie, parent, false);
        return new MovieViewHolder(v);
    }

    @Override
    public void onBindViewHolder(@NonNull MovieViewHolder holder, int position) {
        Movie m = filmovi.get(position);
        holder.tvNaziv.setText(m.getNaziv());
        holder.tvDetalji.setText("Ocena: " + m.getOcena()
                + (m.isOdgledano() ? " | Odgledano" : " | Nije odgledano"));
    }

    @Override
    public int getItemCount() {
        return filmovi.size();
    }
}
```

---

## 6. MovieFragment

Zadaci 3, 6, 7 (prijem rezultata forme), 8 (slanje broadcast-a) i 9 (prijem signala za dugme „Snimaj“).

### `MovieFragment.java`

```java
package com.example.kolokvijum1;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.core.content.ContextCompat;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import java.util.ArrayList;
import java.util.List;

public class MovieFragment extends Fragment {

    // kljucevi za komunikaciju sa formom
    public static final String ZAHTEV_DODAJ = "zahtev_dodaj_film";
    public static final String KEY_NAZIV = "naziv";
    public static final String KEY_OCENA = "ocena";
    public static final String KEY_ODGLEDANO = "odgledano";

    private RecyclerView recyclerView;
    private Button btnDodaj, btnSnimaj;
    private final List<Movie> filmovi = new ArrayList<>();
    private MovieAdapter adapter;

    // prima signal od servisa da je kamera dozvoljena (zadatak 9)
    private final BroadcastReceiver kameraReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            btnSnimaj.setEnabled(true);
        }
    };

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // ZADATAK 7: rezultat iz forme -> dodavanje filma u RecyclerView
        getParentFragmentManager().setFragmentResultListener(
                ZAHTEV_DODAJ, this, (requestKey, result) -> {
                    String naziv = result.getString(KEY_NAZIV);
                    double ocena = result.getDouble(KEY_OCENA);
                    boolean odgledano = result.getBoolean(KEY_ODGLEDANO);

                    Movie film = new Movie(naziv, ocena, odgledano);
                    filmovi.add(film);
                    adapter.notifyItemInserted(filmovi.size() - 1);

                    // ZADATAK 8: javi receiver-u da je film dodat
                    Intent i = new Intent(MovieReceiver.ACTION_MOVIE_ADDED);
                    i.setPackage(requireContext().getPackageName());
                    i.putExtra(KEY_NAZIV, naziv);
                    i.putExtra(KEY_OCENA, ocena);
                    requireContext().sendBroadcast(i);
                });
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_movie, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        recyclerView = view.findViewById(R.id.recyclerView);
        btnDodaj = view.findViewById(R.id.btnDodaj);
        btnSnimaj = view.findViewById(R.id.btnSnimaj);

        adapter = new MovieAdapter(filmovi);
        recyclerView.setLayoutManager(new LinearLayoutManager(requireContext()));
        recyclerView.setAdapter(adapter);

        // ZADATAK 6: otvaranje forme
        btnDodaj.setOnClickListener(v ->
                new AddMovieDialogFragment().show(getParentFragmentManager(), "add_movie"));

        btnSnimaj.setOnClickListener(v -> { /* snimanje - nije trazeno */ });
    }

    @Override
    public void onStart() {
        super.onStart();
        ContextCompat.registerReceiver(
                requireContext(),
                kameraReceiver,
                new IntentFilter(CameraCheckService.ACTION_CAMERA_ALLOWED),
                ContextCompat.RECEIVER_NOT_EXPORTED);
    }

    @Override
    public void onStop() {
        super.onStop();
        requireContext().unregisterReceiver(kameraReceiver);
    }
}
```

---

## 7. Forma za dodavanje filma (DialogFragment)

### `AddMovieDialogFragment.java`

```java
package com.example.kolokvijum1;

import android.os.Bundle;
import android.text.TextUtils;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.DialogFragment;

public class AddMovieDialogFragment extends DialogFragment {

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.dialog_add_movie, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        EditText etNaziv = view.findViewById(R.id.etNaziv);
        EditText etOcena = view.findViewById(R.id.etOcena);
        CheckBox cbOdgledano = view.findViewById(R.id.cbOdgledano);
        Button btnPotvrdi = view.findViewById(R.id.btnPotvrdi);
        Button btnOdustani = view.findViewById(R.id.btnOdustani);

        // ODUSTAJANJE -> zatvara formu
        btnOdustani.setOnClickListener(v -> dismiss());

        // POTVRDA -> salje podatke fragmentu i zatvara formu
        btnPotvrdi.setOnClickListener(v -> {
            String naziv = etNaziv.getText().toString().trim();
            String ocenaTxt = etOcena.getText().toString().trim();

            if (TextUtils.isEmpty(naziv) || TextUtils.isEmpty(ocenaTxt)) {
                Toast.makeText(requireContext(), "Popuni sva polja", Toast.LENGTH_SHORT).show();
                return;
            }

            double ocena = Double.parseDouble(ocenaTxt);

            Bundle rezultat = new Bundle();
            rezultat.putString(MovieFragment.KEY_NAZIV, naziv);
            rezultat.putDouble(MovieFragment.KEY_OCENA, ocena);
            rezultat.putBoolean(MovieFragment.KEY_ODGLEDANO, cbOdgledano.isChecked());

            getParentFragmentManager().setFragmentResult(MovieFragment.ZAHTEV_DODAJ, rezultat);
            dismiss();
        });
    }
}
```

---

## 8. BroadcastReceiver — najveća ocena

Receiver osluškuje dodavanje filma, pamti najveću ocenu i posle svakog dodavanja
prikazuje Toast sa nazivom najbolje ocenjenog filma.

> Implicitni broadcast-ovi se od API 26 **ne primaju** preko manifesta, zato se receiver
> registruje **dinamički** u `MainActivity`.

### `MovieReceiver.java`

```java
package com.example.kolokvijum1;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class MovieReceiver extends BroadcastReceiver {

    public static final String ACTION_MOVIE_ADDED = "com.example.kolokvijum1.MOVIE_ADDED";

    // pamti se najveca ocena do sada
    private double maxOcena = -1;
    private String najboljiFilm = null;

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent == null || !ACTION_MOVIE_ADDED.equals(intent.getAction())) return;

        String naziv = intent.getStringExtra(MovieFragment.KEY_NAZIV);
        double ocena = intent.getDoubleExtra(MovieFragment.KEY_OCENA, 0);

        if (ocena > maxOcena) {
            maxOcena = ocena;
            najboljiFilm = naziv;
        }

        Toast.makeText(context,
                "Najbolje ocenjen film: " + najboljiFilm + " (" + maxOcena + ")",
                Toast.LENGTH_LONG).show();
    }
}
```

---

## 9. Service — provera kamere na svaki minut

`Handler.postDelayed` se koristi jer `WorkManager` ne dozvoljava period kraći od 15 minuta.
Servis proverava dozvolu i, ako je odobrena, šalje broadcast koji `MovieFragment` hvata i
omogućava dugme „Snimaj“.

### `CameraCheckService.java`

```java
package com.example.kolokvijum1;

import android.Manifest;
import android.app.Service;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.os.Handler;
import android.os.IBinder;
import android.os.Looper;
import android.util.Log;

import androidx.annotation.Nullable;
import androidx.core.content.ContextCompat;

public class CameraCheckService extends Service {

    public static final String ACTION_CAMERA_ALLOWED = "com.example.kolokvijum1.CAMERA_ALLOWED";
    private static final long INTERVAL = 60_000; // 1 minut

    private final Handler handler = new Handler(Looper.getMainLooper());

    private final Runnable zadatak = new Runnable() {
        @Override
        public void run() {
            proveriKameru();
            handler.postDelayed(this, INTERVAL);   // ponovi za minut
        }
    };

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        handler.removeCallbacks(zadatak);
        handler.post(zadatak);       // prva provera odmah, pa svaki minut
        return START_STICKY;
    }

    private void proveriKameru() {
        boolean dozvoljena = ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                == PackageManager.PERMISSION_GRANTED;

        Log.d("CameraCheckService", "Kamera dozvoljena: " + dozvoljena);

        if (dozvoljena) {
            Intent i = new Intent(ACTION_CAMERA_ALLOWED);
            i.setPackage(getPackageName());
            sendBroadcast(i);
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        handler.removeCallbacks(zadatak);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

---

## 10. MainActivity

Zadaci 1 (Toolbar), 4 i 5 (meni → fragment), registracija receiver-a, traženje dozvole i pokretanje servisa.

### `MainActivity.java`

```java
package com.example.kolokvijum1;

import android.Manifest;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;
import androidx.core.content.ContextCompat;

public class MainActivity extends AppCompatActivity {

    private MovieReceiver movieReceiver;
    private ActivityResultLauncher<String> cameraPermissionLauncher;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // ZADATAK 1: Toolbar
        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        // ZADATAK 8: dinamicka registracija receiver-a
        movieReceiver = new MovieReceiver();
        ContextCompat.registerReceiver(
                this,
                movieReceiver,
                new IntentFilter(MovieReceiver.ACTION_MOVIE_ADDED),
                ContextCompat.RECEIVER_NOT_EXPORTED);

        // ZADATAK 9: trazenje dozvole za kameru + pokretanje servisa
        cameraPermissionLauncher = registerForActivityResult(
                new ActivityResultContracts.RequestPermission(),
                granted -> pokreniServis());

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                == PackageManager.PERMISSION_GRANTED) {
            pokreniServis();
        } else {
            cameraPermissionLauncher.launch(Manifest.permission.CAMERA);
        }
    }

    private void pokreniServis() {
        startService(new Intent(this, CameraCheckService.class));
    }

    // ZADATAK 4: meni
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main_menu, menu);
        return true;
    }

    // ZADATAK 5: klik na Movie -> prikaz MovieFragment-a
    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        if (item.getItemId() == R.id.menu_movie) {
            getSupportFragmentManager()
                    .beginTransaction()
                    .replace(R.id.container, new MovieFragment())
                    .addToBackStack(null)
                    .commit();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (movieReceiver != null) {
            unregisterReceiver(movieReceiver);
        }
        stopService(new Intent(this, CameraCheckService.class));
    }
}
```

---

## 11. Testiranje

1. Pokreni aplikaciju → vidiš **Toolbar** i svetlozelenu pozadinu, pojavi se **dijalog za dozvolu kamere**.
2. Klik na **Movie** u meniju (gore desno) → prikazuje se `MovieFragment` sa praznim RecyclerView-om, dugme **Snimaj** je sivo (disabled).
3. Klik na **Dodaj** → otvara se forma. **Odustani** je zatvara bez promene.
4. Unesi npr. `Matrix` / `8.5` / ✔ Odgledano → **Potvrdi** → film se pojavi u listi i iskoči Toast **„Najbolje ocenjen film: Matrix (8.5)“**.
5. Dodaj `Titanic` / `7` → Toast i dalje pokazuje `Matrix`; dodaj `Inception` / `9.5` → Toast pokazuje `Inception`.
6. Kad dozvola za kameru postoji, servis u roku od najviše minut pošalje broadcast i dugme **Snimaj** postaje aktivno (u `Logcat`-u filtriraj `CameraCheckService`).

> Ako želiš da brže vidiš efekat servisa na demonstraciji, privremeno smanji `INTERVAL` na
> `10_000` (10 sekundi), pa vrati na `60_000`.

---

### Mapiranje zadataka na kod

| # | Zadatak | Gde je rešeno |
|---|---|---|
| 1 | Toolbar + LinearLayout svetlozelena | `activity_main.xml`, `setSupportActionBar()` |
| 2 | `MovieFragment` | `MovieFragment.java` |
| 3 | RecyclerView + „Dodaj“ i „Snimaj“ (disabled) | `fragment_movie.xml` (`android:enabled="false"`) |
| 4 | Meni sa stavkom Movie | `res/menu/main_menu.xml`, `onCreateOptionsMenu()` |
| 5 | Klik na Movie → prikaz fragmenta | `onOptionsItemSelected()` |
| 6 | Dugme „Dodaj“ otvara formu | `btnDodaj` → `AddMovieDialogFragment.show()` |
| 7 | Forma + odustani + potvrdi → u RecyclerView | `dialog_add_movie.xml`, `AddMovieDialogFragment`, `setFragmentResultListener` |
| 8 | BroadcastReceiver pamti najveću ocenu + Toast | `MovieReceiver.java` |
| 9 | Servis na svaki minut + dozvola → enable „Snimaj“ | `CameraCheckService.java`, `MainActivity`, `kameraReceiver` |










Kreirati novi projekat pod nazivom Kolokvijum1.
Odraditi sledeće zadatke:
1. Unutar MainActivity postaviti Toolbar i ConstraintLayout sa svetloplavom pozadinom. (1.5)
2. Kreirati fragment: BookFragment. (1)
3. BookFragment sadrži RecyclerView i služi za prikaz knjige. U dnu fragmenta dodati dugme sa nazivom
“Dodaj” čija su slova plave boje. (1.5)
4. Dodati meni komponentu sa stavkom Book. (0.5)
5. Klikom na Book, unutar MainActivity se prikazuje BookFragment. (0.5)
6. Klikom na dugme otvara se forma za dodavanje nove knjige. (0.5)
7. Forma ima: naziv knjige, broj strana (EditText brojčanog tipa), Checkbox ispred koga stoji labela
“Pročitano”, dugme za potvrdu i dugme za odustajanje (1.5). Klikom na dugme za odustajanje zatvoriti formu
(0.5). Klikom na potvrdu dodati knjigu u RecyclerView (3).
8. Kreirati BroadcastReceiver. Receiver osluškuje dodavanje knjige i vodi tekući zbir broja strana samo onih
knjiga kod kojih je čekirano “Pročitano”. Nakon svakog dodavanja, taj zbir ispisati u Toast poruci. (4)
9. Kreirati servis koji se pokreće na svaki minut (2.5) i proverava da li je dozvoljena lokacija (obavezno tražiti
dozvolu) (1.5). Ako je lokacija dozvoljena, promeniti boju pozadine Toolbar-a u zelenu. (1.5)

# Kolokvijum1 — Android (Java)

Rešenje kolokvijumskog zadatka: **Toolbar + ConstraintLayout**, **BookFragment sa RecyclerView-om**, **forma za dodavanje knjige**, **BroadcastReceiver** koji vodi zbir strana pročitanih knjiga i **Service** koji na svaki minut proverava dozvolu za lokaciju.

- Jezik: **Java**
- Build script: **Groovy (build.gradle)**
- Template: **Empty Views Activity** (`MainActivity`)

---

## Sadržaj

1. [Kreiranje projekta i zavisnosti](#1-kreiranje-projekta-i-zavisnosti)
2. [AndroidManifest](#2-androidmanifest)
3. [Resursi: boje, tema, meni](#3-resursi-boje-tema-meni)
4. [Layout-ovi](#4-layout-ovi)
5. [Model i adapter](#5-model-i-adapter)
6. [BookFragment](#6-bookfragment)
7. [Forma za dodavanje knjige (DialogFragment)](#7-forma-za-dodavanje-knjige-dialogfragment)
8. [BroadcastReceiver — zbir strana](#8-broadcastreceiver--zbir-strana)
9. [Service — provera lokacije na svaki minut](#9-service--provera-lokacije-na-svaki-minut)
10. [MainActivity](#10-mainactivity)
11. [Testiranje](#11-testiranje)

---

## 1. Kreiranje projekta i zavisnosti

`File → New → New Project → Empty Views Activity`

| Polje | Vrednost |
|---|---|
| Name | `Kolokvijum1` |
| Package name | `com.example.kolokvijum1` |
| Language | `Java` |
| Minimum SDK | `API 24` |
| Build configuration language | `Groovy DSL (build.gradle)` |

**Struktura koju ćeš napraviti:**

```
app/src/main/java/com/example/kolokvijum1/
├── MainActivity.java
├── BookFragment.java
├── AddBookDialogFragment.java
├── BookAdapter.java
├── Book.java
├── BookReceiver.java
└── LocationCheckService.java

app/src/main/res/
├── layout/activity_main.xml
├── layout/fragment_book.xml
├── layout/item_book.xml
├── layout/dialog_add_book.xml
└── menu/main_menu.xml
```

### `app/build.gradle` (Module :app)

```groovy
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.example.kolokvijum1'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.kolokvijum1"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.recyclerview:recyclerview:1.3.2'
    implementation 'androidx.fragment:fragment:1.6.2'
}
```

→ **Sync Now**

---

## 2. AndroidManifest

`app/src/main/AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Zadatak 9: dozvole za lokaciju -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.Kolokvijum1">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Zadatak 9: servis -->
        <service
            android:name=".LocationCheckService"
            android:enabled="true"
            android:exported="false" />

    </application>
</manifest>
```

---

## 3. Resursi: boje, tema, meni

### `res/values/colors.xml` — dodaj boje

```xml
<color name="svetlo_plava">#BBDEFB</color>
<color name="plava">#1565C0</color>
<color name="zelena">#4CAF50</color>
```

### `res/values/themes.xml` — tema mora biti **NoActionBar**

Pošto sami postavljamo `Toolbar`, tema ne sme da ima ugrađen ActionBar (inače puca sa
`This Activity already has an action bar supplied by the window decor`).

```xml
<style name="Base.Theme.Kolokvijum1" parent="Theme.Material3.DayNight.NoActionBar">
    ...
</style>
```

> Ako u tvom projektu piše `Theme.MaterialComponents.DayNight.DarkActionBar`, samo zameni
> `DarkActionBar` sa `NoActionBar`.

### `res/menu/main_menu.xml` — zadatak 4

> Desni klik na `res` → `New → Android Resource Directory` → Resource type: **menu**,
> pa u njemu `New → Menu Resource File` → naziv `main_menu`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/menu_book"
        android:title="Book"
        app:showAsAction="always" />
</menu>
```

---

## 4. Layout-ovi

### `res/layout/activity_main.xml` — zadatak 1

`ConstraintLayout` sa svetloplavom pozadinom, u njemu Toolbar i `FrameLayout` kontejner za fragment.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/svetlo_plava">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="0dp"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        android:title="Kolokvijum1"
        android:titleTextColor="@android:color/white"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintTop_toBottomOf="@id/toolbar"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### `res/layout/fragment_book.xml` — zadatak 3

RecyclerView i, u dnu, dugme „Dodaj“ sa **plavim slovima** (`android:textColor`).

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="8dp">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <Button
        android:id="@+id/btnDodaj"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Dodaj"
        android:textColor="@color/plava" />

</LinearLayout>
```

### `res/layout/item_book.xml` — jedan red u RecyclerView-u

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="12dp">

    <TextView
        android:id="@+id/tvNaziv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/tvDetalji"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="14sp" />
</LinearLayout>
```

### `res/layout/dialog_add_book.xml` — zadatak 7 (forma)

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="20dp">

    <EditText
        android:id="@+id/etNaziv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Naziv knjige"
        android:inputType="text" />

    <EditText
        android:id="@+id/etStrane"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Broj strana"
        android:inputType="number" />

    <!-- labela ISPRED checkbox-a -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Pročitano"
            android:textSize="16sp" />

        <CheckBox
            android:id="@+id/cbProcitano"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnPotvrdi"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Potvrdi" />

        <Button
            android:id="@+id/btnOdustani"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Odustani" />
    </LinearLayout>
</LinearLayout>
```

---

## 5. Model i adapter

### `Book.java`

```java
package com.example.kolokvijum1;

import java.io.Serializable;

public class Book implements Serializable {

    private String naziv;
    private int brojStrana;
    private boolean procitano;

    public Book(String naziv, int brojStrana, boolean procitano) {
        this.naziv = naziv;
        this.brojStrana = brojStrana;
        this.procitano = procitano;
    }

    public String getNaziv() { return naziv; }
    public int getBrojStrana() { return brojStrana; }
    public boolean isProcitano() { return procitano; }
}
```

### `BookAdapter.java`

```java
package com.example.kolokvijum1;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.List;

public class BookAdapter extends RecyclerView.Adapter<BookAdapter.BookViewHolder> {

    private final List<Book> knjige;

    public BookAdapter(List<Book> knjige) {
        this.knjige = knjige;
    }

    static class BookViewHolder extends RecyclerView.ViewHolder {
        TextView tvNaziv, tvDetalji;

        BookViewHolder(@NonNull View itemView) {
            super(itemView);
            tvNaziv = itemView.findViewById(R.id.tvNaziv);
            tvDetalji = itemView.findViewById(R.id.tvDetalji);
        }
    }

    @NonNull
    @Override
    public BookViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_book, parent, false);
        return new BookViewHolder(v);
    }

    @Override
    public void onBindViewHolder(@NonNull BookViewHolder holder, int position) {
        Book k = knjige.get(position);
        holder.tvNaziv.setText(k.getNaziv());
        holder.tvDetalji.setText("Strana: " + k.getBrojStrana()
                + (k.isProcitano() ? " | Procitano" : " | Nije procitano"));
    }

    @Override
    public int getItemCount() {
        return knjige.size();
    }
}
```

---

## 6. BookFragment

Zadaci 3, 6, 7 (prijem rezultata forme) i 8 (slanje broadcast-a).

### `BookFragment.java`

```java
package com.example.kolokvijum1;

import android.content.Intent;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import java.util.ArrayList;
import java.util.List;

public class BookFragment extends Fragment {

    // kljucevi za komunikaciju sa formom
    public static final String ZAHTEV_DODAJ = "zahtev_dodaj_knjigu";
    public static final String KEY_NAZIV = "naziv";
    public static final String KEY_STRANE = "strane";
    public static final String KEY_PROCITANO = "procitano";

    private RecyclerView recyclerView;
    private Button btnDodaj;
    private final List<Book> knjige = new ArrayList<>();
    private BookAdapter adapter;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // ZADATAK 7: rezultat iz forme -> dodavanje knjige u RecyclerView
        getParentFragmentManager().setFragmentResultListener(
                ZAHTEV_DODAJ, this, (requestKey, result) -> {
                    String naziv = result.getString(KEY_NAZIV);
                    int strane = result.getInt(KEY_STRANE);
                    boolean procitano = result.getBoolean(KEY_PROCITANO);

                    Book knjiga = new Book(naziv, strane, procitano);
                    knjige.add(knjiga);
                    adapter.notifyItemInserted(knjige.size() - 1);

                    // ZADATAK 8: javi receiver-u da je knjiga dodata
                    Intent i = new Intent(BookReceiver.ACTION_BOOK_ADDED);
                    i.setPackage(requireContext().getPackageName());
                    i.putExtra(KEY_NAZIV, naziv);
                    i.putExtra(KEY_STRANE, strane);
                    i.putExtra(KEY_PROCITANO, procitano);
                    requireContext().sendBroadcast(i);
                });
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_book, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        recyclerView = view.findViewById(R.id.recyclerView);
        btnDodaj = view.findViewById(R.id.btnDodaj);

        adapter = new BookAdapter(knjige);
        recyclerView.setLayoutManager(new LinearLayoutManager(requireContext()));
        recyclerView.setAdapter(adapter);

        // ZADATAK 6: otvaranje forme
        btnDodaj.setOnClickListener(v ->
                new AddBookDialogFragment().show(getParentFragmentManager(), "add_book"));
    }
}
```

---

## 7. Forma za dodavanje knjige (DialogFragment)

### `AddBookDialogFragment.java`

```java
package com.example.kolokvijum1;

import android.os.Bundle;
import android.text.TextUtils;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.DialogFragment;

public class AddBookDialogFragment extends DialogFragment {

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.dialog_add_book, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        EditText etNaziv = view.findViewById(R.id.etNaziv);
        EditText etStrane = view.findViewById(R.id.etStrane);
        CheckBox cbProcitano = view.findViewById(R.id.cbProcitano);
        Button btnPotvrdi = view.findViewById(R.id.btnPotvrdi);
        Button btnOdustani = view.findViewById(R.id.btnOdustani);

        // ODUSTAJANJE -> zatvara formu
        btnOdustani.setOnClickListener(v -> dismiss());

        // POTVRDA -> salje podatke fragmentu i zatvara formu
        btnPotvrdi.setOnClickListener(v -> {
            String naziv = etNaziv.getText().toString().trim();
            String straneTxt = etStrane.getText().toString().trim();

            if (TextUtils.isEmpty(naziv) || TextUtils.isEmpty(straneTxt)) {
                Toast.makeText(requireContext(), "Popuni sva polja", Toast.LENGTH_SHORT).show();
                return;
            }

            int strane = Integer.parseInt(straneTxt);

            Bundle rezultat = new Bundle();
            rezultat.putString(BookFragment.KEY_NAZIV, naziv);
            rezultat.putInt(BookFragment.KEY_STRANE, strane);
            rezultat.putBoolean(BookFragment.KEY_PROCITANO, cbProcitano.isChecked());

            getParentFragmentManager().setFragmentResult(BookFragment.ZAHTEV_DODAJ, rezultat);
            dismiss();
        });
    }
}
```

---

## 8. BroadcastReceiver — zbir strana

Receiver osluškuje dodavanje knjige i vodi **tekući zbir broja strana samo pročitanih knjiga**.
Posle svakog dodavanja prikazuje taj zbir u Toast poruci.

> Implicitni broadcast-ovi se od API 26 **ne primaju** preko manifesta, zato se receiver
> registruje **dinamički** u `MainActivity`.

### `BookReceiver.java`

```java
package com.example.kolokvijum1;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class BookReceiver extends BroadcastReceiver {

    public static final String ACTION_BOOK_ADDED = "com.example.kolokvijum1.BOOK_ADDED";

    // tekuci zbir strana procitanih knjiga
    private int zbirStrana = 0;

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent == null || !ACTION_BOOK_ADDED.equals(intent.getAction())) return;

        int strane = intent.getIntExtra(BookFragment.KEY_STRANE, 0);
        boolean procitano = intent.getBooleanExtra(BookFragment.KEY_PROCITANO, false);

        // sabiraju se SAMO procitane knjige
        if (procitano) {
            zbirStrana += strane;
        }

        Toast.makeText(context,
                "Ukupno procitanih strana: " + zbirStrana,
                Toast.LENGTH_LONG).show();
    }
}
```

---

## 9. Service — provera lokacije na svaki minut

`Handler.postDelayed` se koristi jer `WorkManager` ne dozvoljava period kraći od 15 minuta.
Servis proverava dozvolu i, ako je odobrena, šalje broadcast koji `MainActivity` hvata i
menja pozadinu Toolbar-a u zelenu.

### `LocationCheckService.java`

```java
package com.example.kolokvijum1;

import android.Manifest;
import android.app.Service;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.os.Handler;
import android.os.IBinder;
import android.os.Looper;
import android.util.Log;

import androidx.annotation.Nullable;
import androidx.core.content.ContextCompat;

public class LocationCheckService extends Service {

    public static final String ACTION_LOCATION_ALLOWED = "com.example.kolokvijum1.LOCATION_ALLOWED";
    private static final long INTERVAL = 60_000; // 1 minut

    private final Handler handler = new Handler(Looper.getMainLooper());

    private final Runnable zadatak = new Runnable() {
        @Override
        public void run() {
            proveriLokaciju();
            handler.postDelayed(this, INTERVAL);   // ponovi za minut
        }
    };

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        handler.removeCallbacks(zadatak);
        handler.post(zadatak);       // prva provera odmah, pa svaki minut
        return START_STICKY;
    }

    private void proveriLokaciju() {
        boolean dozvoljena = ContextCompat.checkSelfPermission(
                this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED;

        Log.d("LocationCheckService", "Lokacija dozvoljena: " + dozvoljena);

        if (dozvoljena) {
            Intent i = new Intent(ACTION_LOCATION_ALLOWED);
            i.setPackage(getPackageName());
            sendBroadcast(i);
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        handler.removeCallbacks(zadatak);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

---

## 10. MainActivity

Zadaci 1 (Toolbar), 4 i 5 (meni → fragment), registracija receiver-a, traženje dozvole,
pokretanje servisa i promena boje Toolbar-a.

### `MainActivity.java`

```java
package com.example.kolokvijum1;

import android.Manifest;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;
import androidx.core.content.ContextCompat;

public class MainActivity extends AppCompatActivity {

    private Toolbar toolbar;
    private BookReceiver bookReceiver;
    private ActivityResultLauncher<String> locationPermissionLauncher;

    // ZADATAK 9: signal od servisa -> Toolbar postaje zelen
    private final BroadcastReceiver lokacijaReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            toolbar.setBackgroundColor(ContextCompat.getColor(MainActivity.this, R.color.zelena));
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // ZADATAK 1: Toolbar
        toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        // ZADATAK 8: dinamicka registracija receiver-a za dodavanje knjige
        bookReceiver = new BookReceiver();
        ContextCompat.registerReceiver(
                this,
                bookReceiver,
                new IntentFilter(BookReceiver.ACTION_BOOK_ADDED),
                ContextCompat.RECEIVER_NOT_EXPORTED);

        // ZADATAK 9: receiver za signal servisa
        ContextCompat.registerReceiver(
                this,
                lokacijaReceiver,
                new IntentFilter(LocationCheckService.ACTION_LOCATION_ALLOWED),
                ContextCompat.RECEIVER_NOT_EXPORTED);

        // ZADATAK 9: trazenje dozvole za lokaciju + pokretanje servisa
        locationPermissionLauncher = registerForActivityResult(
                new ActivityResultContracts.RequestPermission(),
                granted -> pokreniServis());

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)
                == PackageManager.PERMISSION_GRANTED) {
            pokreniServis();
        } else {
            locationPermissionLauncher.launch(Manifest.permission.ACCESS_FINE_LOCATION);
        }
    }

    private void pokreniServis() {
        startService(new Intent(this, LocationCheckService.class));
    }

    // ZADATAK 4: meni
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main_menu, menu);
        return true;
    }

    // ZADATAK 5: klik na Book -> prikaz BookFragment-a
    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        if (item.getItemId() == R.id.menu_book) {
            getSupportFragmentManager()
                    .beginTransaction()
                    .replace(R.id.container, new BookFragment())
                    .addToBackStack(null)
                    .commit();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (bookReceiver != null) unregisterReceiver(bookReceiver);
        unregisterReceiver(lokacijaReceiver);
        stopService(new Intent(this, LocationCheckService.class));
    }
}
```

---

## 11. Testiranje

1. Pokreni aplikaciju → vidiš **Toolbar** i svetloplavu pozadinu, pojavi se **dijalog za dozvolu lokacije**.
2. Klik na **Book** u meniju (gore desno) → prikazuje se `BookFragment` sa praznim RecyclerView-om i dugmetom **Dodaj** plavim slovima.
3. Klik na **Dodaj** → otvara se forma. **Odustani** je zatvara bez promene.
4. Unesi `Na Drini cuprija` / `350` / ✔ Pročitano → **Potvrdi** → knjiga se pojavi u listi i iskoči Toast **„Ukupno procitanih strana: 350“**.
5. Dodaj `Prokleta avlija` / `120` **bez** čekiranja → Toast i dalje pokazuje `350` (nepročitane se ne sabiraju).
6. Dodaj `Zlocin i kazna` / `600` / ✔ → Toast pokazuje **950**.
7. Kad je dozvola za lokaciju data, servis u roku od najviše minut pošalje broadcast i **Toolbar postane zelen** (u `Logcat`-u filtriraj `LocationCheckService`).

> Ako želiš da brže vidiš efekat servisa na demonstraciji, privremeno smanji `INTERVAL` na
> `10_000` (10 sekundi), pa vrati na `60_000`.

---

### Mapiranje zadataka na kod

| # | Zadatak | Gde je rešeno |
|---|---|---|
| 1 | Toolbar + ConstraintLayout svetloplava | `activity_main.xml`, `setSupportActionBar()` |
| 2 | `BookFragment` | `BookFragment.java` |
| 3 | RecyclerView + dugme „Dodaj“ plavim slovima | `fragment_book.xml` (`android:textColor="@color/plava"`) |
| 4 | Meni sa stavkom Book | `res/menu/main_menu.xml`, `onCreateOptionsMenu()` |
| 5 | Klik na Book → prikaz fragmenta | `onOptionsItemSelected()` |
| 6 | Dugme otvara formu | `btnDodaj` → `AddBookDialogFragment.show()` |
| 7 | Forma + odustani + potvrdi → u RecyclerView | `dialog_add_book.xml`, `AddBookDialogFragment`, `setFragmentResultListener` |
| 8 | BroadcastReceiver — zbir strana pročitanih | `BookReceiver.java` |
| 9 | Servis na svaki minut + dozvola → zelen Toolbar | `LocationCheckService.java`, `lokacijaReceiver` u `MainActivity` |
