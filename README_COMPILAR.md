# Bóveda Android — cómo compilar

Proyecto Android nativo completo: UI Kotlin/Compose + núcleo Rust (probado, 18/18 tests).

## ⚠️ Léeme primero (honestidad)

El **núcleo Rust está probado**. La **app Android NO la pude compilar** donde la generé (no
hay Android SDK/NDK ahí). Está completa y ensamblada, pero en la **primera compilación
saldrán errores** (versiones, imports, algún recurso). Es normal en una app nativa: abres en
Android Studio, compilas, me pasas el error, lo arreglamos. No es "compila a la primera".

## Requisitos (instalar una vez)

1. **Android Studio** (https://developer.android.com/studio) — trae SDK. Instala el **NDK**
   desde *SDK Manager → SDK Tools → NDK (Side by side)*.
2. **Rust** (https://rustup.rs).
3. **cargo-ndk**: `cargo install cargo-ndk`
4. Targets Android:
   `rustup target add aarch64-linux-android armv7-linux-androideabi`

## Paso 1 — Probar el núcleo (rápido, sin Android)

```
cd rust
cargo test --release
```
Debe decir `18 passed`.

## Paso 2 — Compilar el núcleo a .so

Desde la carpeta `rust/`:
```
cargo ndk -t arm64-v8a -t armeabi-v7a -o ../app/src/main/jniLibs build --release
```
Esto crea `app/src/main/jniLibs/arm64-v8a/libboveda_core.so` (y armeabi-v7a).

## Paso 3 — Abrir y compilar la app

1. Abre **Android Studio** → *Open* → selecciona la carpeta `boveda_android`.
2. Deja que sincronice Gradle (descargará dependencias; la primera vez tarda).
   - Android Studio generará el **gradle wrapper** y validará versiones automáticamente.
3. Conecta tu teléfono (Depuración USB activada) o usa un emulador.
4. Pulsa **Run** (▶).

## Estructura

```
boveda_android/
├── settings.gradle.kts / build.gradle.kts / gradle.properties
├── rust/                         núcleo Rust (probado) → genera el .so
└── app/
    ├── build.gradle.kts
    └── src/main/
        ├── AndroidManifest.xml   privacidad + alias de camuflaje
        ├── kotlin/com/mv/boveda/
        │   ├── MainActivity.kt    FLAG_SECURE, auto-bloqueo, respaldo SAF
        │   ├── App.kt             todas las pantallas (bloqueo, lista, detalle, alta, ajustes)
        │   ├── RustCore.kt        enlaces al núcleo (JNI)
        │   ├── VaultRepo.kt       archivo cifrado + sesión
        │   ├── Biometric.kt       huella (BiometricPrompt + Keystore)
        │   ├── Duress.kt          coacción
        │   ├── Camouflage.kt      icono/nombre falso
        │   └── AutoLock.kt        bloqueo por inactividad/segundo plano
        └── res/                   tema, icono (XML), strings
```

## Qué hace la app (ya conectado)

- Crear/desbloquear bóveda (contraseña maestra Argon2id).
- Huella (si la activas en Ajustes).
- Lista con búsqueda; tipos: Login, Tarjeta (con marca + últimos 4), Wi-Fi.
- Detalle con copiar y 2FA en vivo.
- Ajustes: huella, camuflaje, contraseña de coacción, exportar/restaurar respaldo.
- Pantalla de privacidad (oculta en recientes) y auto-bloqueo.

## Errores probables en el primer build (y que arreglamos juntos)

- Versiones de Gradle/AGP/Kotlin/Compose que tu Android Studio prefiera distintas.
- Algún import o API de Compose que cambió de nombre.
- El `.so`: si la app abre pero crashea al inicio con "library not found", es que faltó el
  Paso 2 (cargo-ndk) o el ABI de tu dispositivo.

Compila, copia el error COMPLETO de la pestaña *Build*, y me lo mandas.
