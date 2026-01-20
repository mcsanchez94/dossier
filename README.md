# Guía paso a paso: Firebase Auth con Java en Android Studio

Esta guía crea desde cero un proyecto Android con **Firebase Authentication** usando **Java**, **Data Binding** y **Gradle Groovy DSL** con:

- Proyecto: `LoginFireBase`
- Package: `es.medac.loginfirebase`
- AGP: `8.13.2`
- Compile SDK: `36`
- Java: `11`

> Objetivo: una app simple con **registro**, **inicio de sesión** y **cerrar sesión** usando **correo/contraseña**.

---

## 1) Crear el proyecto en Android Studio

1. **New Project** → **Empty Views Activity**.
2. Configura:
   - **Name**: `LoginFireBase`
   - **Package name**: `es.medac.loginfirebase`
   - **Language**: `Java`
   - **Minimum SDK**: a elección (ej. API 24)
   - **Build configuration**: **Groovy DSL**
3. Finaliza y espera a que el proyecto sincronice.

---

## 2) Ajustar Gradle (AGP, Compile SDK y Java 11)

### `settings.gradle`

Verifica los repos:

```groovy
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "LoginFireBase"
include(":app")
```

### `build.gradle` (nivel proyecto)

Incluye el plugin de Google Services:

```groovy
plugins {
    id "com.android.application" version "8.13.2" apply false
    id "com.google.gms.google-services" version "4.4.2" apply false
}
```

### `app/build.gradle`

Habilita **Data Binding**, configura Java 11 y agrega Firebase Auth + BoM:

```groovy
plugins {
    id "com.android.application"
    id "com.google.gms.google-services"
}

android {
    namespace "es.medac.loginfirebase"
    compileSdk 36

    defaultConfig {
        applicationId "es.medac.loginfirebase"
        minSdk 24
        targetSdk 36
        versionCode 1
        versionName "1.0"
    }

    buildFeatures {
        dataBinding true
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_11
        targetCompatibility JavaVersion.VERSION_11
    }
}

dependencies {
    implementation platform("com.google.firebase:firebase-bom:33.4.0")
    implementation "com.google.firebase:firebase-auth"

    implementation "androidx.appcompat:appcompat:1.7.0"
    implementation "com.google.android.material:material:1.12.0"
    implementation "androidx.constraintlayout:constraintlayout:2.2.0"
}
```

> Ajusta `minSdk` si tu curso lo requiere.

---

## 3) Crear proyecto en Firebase Console

1. Ve a **https://console.firebase.google.com/**.
2. **Add project** → `LoginFireBase`.
3. Desactiva Analytics si no lo necesitas.
4. Cuando termine, entra al proyecto.

### 3.1) Registrar la app Android

1. En **Project Overview** → **Add app** → **Android**.
2. Completa:
   - **Android package name**: `es.medac.loginfirebase`
   - (Opcional) **SHA-1**: no es necesario para email/contraseña.
3. Descarga el archivo **`google-services.json`**.
4. Copia ese archivo en `app/` (ruta: `app/google-services.json`).

---

## 4) Activar el método de autenticación

1. En Firebase Console → **Authentication** → **Get started**.
2. En **Sign-in method**, habilita **Email/Password**.

---

## 5) Crear el layout con Data Binding

### `app/src/main/res/layout/activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data />

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="24dp">

        <EditText
            android:id="@+id/etEmail"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:hint="Email"
            android:inputType="textEmailAddress"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent" />

        <EditText
            android:id="@+id/etPassword"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:hint="Contraseña"
            android:inputType="textPassword"
            app:layout_constraintTop_toBottomOf="@id/etEmail"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginTop="12dp" />

        <Button
            android:id="@+id/btnRegister"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="Registrar"
            app:layout_constraintTop_toBottomOf="@id/etPassword"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginTop="16dp" />

        <Button
            android:id="@+id/btnLogin"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="Iniciar sesión"
            app:layout_constraintTop_toBottomOf="@id/btnRegister"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginTop="8dp" />

        <Button
            android:id="@+id/btnLogout"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="Cerrar sesión"
            app:layout_constraintTop_toBottomOf="@id/btnLogin"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginTop="8dp" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

---

## 6) Programar la lógica en `MainActivity`

### `app/src/main/java/es/medac/loginfirebase/MainActivity.java`

```java
package es.medac.loginfirebase;

import android.os.Bundle;
import android.text.TextUtils;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import com.google.firebase.auth.FirebaseAuth;

import es.medac.loginfirebase.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    private ActivityMainBinding binding;
    private FirebaseAuth auth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        auth = FirebaseAuth.getInstance();

        binding.btnRegister.setOnClickListener(view -> registerUser());
        binding.btnLogin.setOnClickListener(view -> loginUser());
        binding.btnLogout.setOnClickListener(view -> logoutUser());
    }

    private void registerUser() {
        String email = binding.etEmail.getText().toString().trim();
        String password = binding.etPassword.getText().toString().trim();

        if (TextUtils.isEmpty(email) || TextUtils.isEmpty(password)) {
            Toast.makeText(this, "Completa email y contraseña", Toast.LENGTH_SHORT).show();
            return;
        }

        auth.createUserWithEmailAndPassword(email, password)
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    Toast.makeText(this, "Usuario registrado", Toast.LENGTH_SHORT).show();
                } else {
                    String message = task.getException() != null
                        ? task.getException().getMessage()
                        : "Error al registrar";
                    Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
                }
            });
    }

    private void loginUser() {
        String email = binding.etEmail.getText().toString().trim();
        String password = binding.etPassword.getText().toString().trim();

        if (TextUtils.isEmpty(email) || TextUtils.isEmpty(password)) {
            Toast.makeText(this, "Completa email y contraseña", Toast.LENGTH_SHORT).show();
            return;
        }

        auth.signInWithEmailAndPassword(email, password)
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    Toast.makeText(this, "Sesión iniciada", Toast.LENGTH_SHORT).show();
                } else {
                    String message = task.getException() != null
                        ? task.getException().getMessage()
                        : "Error al iniciar sesión";
                    Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
                }
            });
    }

    private void logoutUser() {
        auth.signOut();
        Toast.makeText(this, "Sesión cerrada", Toast.LENGTH_SHORT).show();
    }
}
```

---

## 7) Probar la app

1. **Sync Now** si Android Studio lo solicita.
2. Ejecuta en emulador o dispositivo real.
3. Prueba:
   - Registrar un nuevo usuario con email/contraseña.
   - Cerrar sesión.
   - Iniciar sesión con el usuario creado.

---

## 8) (Opcional) Mostrar estado de sesión

Puedes comprobar si existe usuario autenticado al iniciar:

```java
if (auth.getCurrentUser() != null) {
    // Usuario ya autenticado
}
```

---

## 9) Errores comunes

- **`google-services.json` en ruta incorrecta** → debe ir en `app/`.
- **Email/Password no habilitado** en Firebase Console.
- **Versiones de Gradle/AGP** incompatibles con tu Android Studio.

---

## 10) Resumen rápido

1. Crear proyecto en Android Studio (Java + Groovy).
2. Configurar Gradle (AGP, Firebase BoM, Auth, Data Binding).
3. Registrar app en Firebase y copiar `google-services.json`.
4. Habilitar Email/Password.
5. UI con Data Binding + lógica en `MainActivity`.
6. Probar registro, login y logout.
