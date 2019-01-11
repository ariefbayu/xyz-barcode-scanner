https://github.com/journeyapps/zxing-android-embedded

1. create project

2. add dependencies

3. also add multidex to deps

4. add `android:hardwareAccelerated="true"`

5. create application class

6. add layout

7. implement layout to codes (using kontlinx)

8. launch scan intent

9. catch intent result

10. handle restore instance for when screen is rotated

11. run the codes!


Lets start the new year with new tutorial. This time, I will wrote how to capture barcode / QRCode data on Android using Kotlin language. I'll assume you already have Android Studio and related dependencies (Android SDK, Emulator (of your phone's adb driver if you want to live test on device), Java, etc) installed.

Without further ado, lets create the project.

== Create New Android Project

[image-1]

Define application name, project location, and company domain. ALso don't forget to include Kotlin support as we will code in Kotlin.

[image-2]

Select what form factors and minimum SDK you want to support. Let's choose "Phone and Tablet" and API 19 (or above) for now.

[image-3] & [image-4]

Choose empty activity and name it "MainActivity'. Generate layout and enable "Backwards Compatibility (APpCompat)".

== Add Dependencies

For this tutorial, we will use Zxing's library from JourneyApps (https://github.com/journeyapps/zxing-android-embedded). Add the following lines to `build.gradle`:

implementation 'com.journeyapps:zxing-android-embedded:3.6.0'
implementation 'com.android.support:multidex:1.0.3'

Notices that I also include multidex support. This because when I run the codes, runtime log complained some missing headers that is fixed by enabling multidex support. Now, since we will be using multidex, we need to add `multiDexEnabled true` config (still inside `build.gradle`):

android {
    ...
    defaultConfig {
        ...
        multiDexEnabled true
        ...
    }
    ...
}

Multidex also require as to customize `Application`:

Add new file called `KtApplication` and fill it with the following codes:

package xyz.ariefbayu.xyzbarcodescanner
import android.support.multidex.MultiDexApplication

class KtApplication: MultiDexApplication(){
    override fun onCreate() {
        super.onCreate()
    }
}

Finally, we also have to update our `AndroidManifest.xml` to recognize the `Application`:

    <application
        ...
        android:name=".KtApplication"
        android:hardwareAccelerated="true"
        ...
    >

Notice the additional `android:hardwareAccelerated="true"`? This is required by our Zxing library.

We are done with the initial setup. Move on to the actual coding.

== Design the layout

Our layout for this one is rather simple. Add the following codes to `activity_main.xml`:

<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

    <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Scanned Value: "
            android:id="@+id/txtValueLabel"
    />
    <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="{{please scan first}}"
            android:id="@+id/txtValue"
            app:layout_constraintLeft_toRightOf="@id/txtValueLabel"
    />
    <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/btnScan"
            android:text="Scan"
            app:layout_constraintTop_toBottomOf="@id/txtValueLabel"
    />

</android.support.constraint.ConstraintLayout>

== Add codes to our layout

For this one, we will implement kotlin's view binding to handle layout.

Let's add our xml layout into import:

import kotlinx.android.synthetic.main.activity_main.*

Next, inside `onCreate`, we will attach `onClickListener` to `btnScan`:

btnScan.setOnClickListener {
    run {
        IntentIntegrator(this@MainActivity).initiateScan();
    }
}

Codes above will launch new intent where barcode scanning process is happened. To catch the scanned value, we need to add `onActivityResult`:

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {

    var result: IntentResult? = IntentIntegrator.parseActivityResult(requestCode, resultCode, data)

    if(result != null){

        if(result.contents != null){
            txtValue.text = result.contents
        } else {
            txtValue.text = "scan failed"
        }
    } else {
        super.onActivityResult(requestCode, resultCode, data)
    }
}

It's basically done. However, when you try to run it, it will sometime lost the scanned value when screen's rotated. It is an expected behavior because, by default, eerything is reloaded on screen rotate.

To overcome this problem, we need to implement `onSaveInstanceState` and `onRestoreInstanceState`

== Handle screen rotate situation

There are several ways to handle displaying data when screen is rotated. In this tutorial, I will use the `onSaveInstanceState` and `onRestoreInstanceState` method. First we will store scanned value into variable, let's call it: `scannedResult`. Add the following codes in class scope:

var scannedResult: String = ""

Next, implement `onSaveInstanceState` and `onRestoreInstanceState`:

override fun onSaveInstanceState(outState: Bundle?) {

    outState?.putString("scannedResult", scannedResult)
    super.onSaveInstanceState(outState)
}

override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
    super.onRestoreInstanceState(savedInstanceState)

    savedInstanceState?.let {
        scannedResult = it.getString("scannedResult")
        txtValue.text = scannedResult
    }
}

Finally, update our `onActivityResult` so that `scannedResult` is updated:

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {

    var result: IntentResult? = IntentIntegrator.parseActivityResult(requestCode, resultCode, data)

    if(result != null){

        if(result.contents != null){
            scannedResult = result.contents
            txtValue.text = scannedResult
        } else {
            txtValue.text = "scan failed"
        }
    } else {
        super.onActivityResult(requestCode, resultCode, data)
    }
}

Now we can finally called it done!