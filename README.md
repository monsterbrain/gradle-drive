# gradle-drive

Lightweight gradle plugin to upload your APK to the Google Drive. Needs the ```com.android.application``` plugin applied.

## Quick Start Guide

1. Create new project on Google Developers Console (or use existing one).
1. Enable Drive API in API Manager (see [Prerequisites](#enabling-drive-api)).
1. Create Google Service Account for that project (see [Prerequisites](#google-service-account)).
1. Share target Google Drive folder with service account email or make it public (see [Drive Folder]).
1. Add the plugin to your buildscript dependencies (see [Usage](#usage)).
1. Apply the plugin (see [Usage](#usage)).
1. Apply your credentials inside the `drive` block (see [Credentials](#credentials)).

## Prerequisites

### Project at Google Developers Console

You need to have a project at https://console.developers.google.com/iam-admin/projects to be able to access Google APIs.

### Enabling Drive API

You can enable Drive API at API Manager https://console.developers.google.com/apis/dashboard. You can skip warning about credencials or click "Create credencials" button and use wizard to create required stuff. 
When you use wizard choose `Other non-UI` from dropdown menu and select `Application Data` access. For next steps see [Prerequisites](#google-service-account)

### Google Service Account

To use this plugin you have to create a service account and generate private .p12 key (.json authorization may be added in future).
Just click `create service account` button, check `Furnish a new private key` and select P12 option. 
Save generated *p12 key file* and *service account email address* we will use them later on.

### Give Drive Folder Access to the service account

Google doesn't allow you to access Drive folders for your service account directly, but there is some workaround. 
You can create folder in your account's Drive and share access to service account email. 
After that you can use that folder id setting it as a parent for service account uploads. This is secure way to access uploaded files.
You can also make your folder a folder with public access (less secure, not tested).

## Usage

Add it to your buildscript dependencies: ***(Artifactory not working yet! For temporary solution see below)***

```groovy
buildscript {

    repositories {
        // using github packages
        maven {
            name = "GithubPackages"
            url = uri("https://maven.pkg.github.com/monsterbrain/gradle-drive")
            credentials {
                username = "monsterbrain"
                password = "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"
            }
        }
    }

    dependencies {
    	// ...
        classpath 'com.github.ximik:drive:1.1'
    }
}
```

Apply it:

```groovy
apply plugin: 'com.github.ximik3.drive'
```

The plugin creates the following tasks for you:

* `uploadApkDebug` - Uploads APK file signed with temporary debug keys.
* `uploadApkRelease` - Uploads APK file signed with actual keys.

Make sure to set a valid `signingConfig` for the release build type. Otherwise, there won't be a release APK and the uploadApkRelease won't be available.

In case you are using product flavors you will get one of the above tasks for every flavor. E.g. `uploadApkQaRelease` or `uploadProductionRelease`.

## Configuration

Once you have applied this plugin to your android application project you can configure it via the ```drive``` block.

### Credentials

Drop in your service account email address and the p12 key file **(key.p12 file should be in app folder)** you generated in the API Console here.
folderId will be last part of google drive link (drive.google.com/drive/folders/**folderId**)

```groovy
drive {
    serviceAccountEmail = 'your-service-account-email'
    pk12File = file('key.p12')
    folderId = 'your-shared-folder-id'
}
```

## Advanced Topics

### Run Upload after Assemble apk
For automatically running upload task after assemble Task you can configure it like this
```
afterEvaluate {
    assembleRelease.configure {
        finalizedBy uploadApkRelease
    }
}
```

### Assemble apk before uploading

Before running above tasks you need to be sure that you have required .apk file assembled and stored under `app/build/outputs/apk` folder, otherwise task will fail with `java.io.FileNotFoundExeption`.
For example, you can run:

```bash
./gradlew assembleDebug
```

and only after successful build run:

```bash
./gradlew uploadApkDebug
```

Inspired by https://github.com/Triple-T/gradle-play-publisher .
