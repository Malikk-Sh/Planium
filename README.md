# Planium

Planium — офлайн-планировщик задач и проектов для Android. Создавайте разделы под
разные сферы жизни, добавляйте задачи и подпланы, отслеживайте прогресс. Все данные
хранятся **локально на устройстве** (без интернета, рекламы и сбора данных).

Технически это лёгкая обёртка-WebView над автономным веб-интерфейсом
(`app/src/main/assets/index.html`).

## Структура проекта

```
.
├── app/
│   ├── build.gradle                  # конфигурация модуля приложения + подпись релиза
│   ├── proguard-rules.pro
│   └── src/main/
│       ├── AndroidManifest.xml        # манифест (без debuggable, без лишних разрешений)
│       ├── assets/index.html          # весь интерфейс приложения
│       ├── java/com/planium/app/MainActivity.java
│       └── res/
│           ├── mipmap-*/              # иконки приложения
│           └── values/                # strings.xml, styles.xml (тема AppTheme)
├── build.gradle                       # корневой скрипт (плагин AGP)
├── settings.gradle
├── gradle/wrapper/                    # Gradle Wrapper (8.14.3)
├── gradlew / gradlew.bat
├── keystore.properties.example        # шаблон настроек подписи
└── reference/original-apk-unpacked/   # исходный распакованный APK (для справки)
```

## Параметры сборки

| Параметр        | Значение            |
|-----------------|---------------------|
| applicationId   | `com.planium.app`   |
| minSdk          | 23 (Android 6.0)    |
| targetSdk / compileSdk | 35 (Android 15) |
| versionCode     | 1                   |
| versionName     | 1.0                 |
| AGP             | 8.7.3               |
| Gradle          | 8.14.3              |
| JDK             | 17                  |

## Сборка подписанного release-APK

1. Установите JDK 17 и Android SDK (platform-35, build-tools 35.0.0). В Android Studio
   всё ставится автоматически.

2. Настройте подпись релиза:
   ```bash
   cp keystore.properties.example keystore.properties
   ```
   Положите ваш keystore (`planium-release.jks`) в корень проекта и впишите пароли
   в `keystore.properties`.

   > Если keystore ещё нет — создайте его:
   > ```bash
   > keytool -genkeypair -v -keystore planium-release.jks -storetype JKS \
   >   -alias planium -keyalg RSA -keysize 2048 -validity 10950
   > ```
   > **Сохраните keystore и пароли навсегда** — тем же ключом подписываются все
   > будущие обновления в RuStore.

3. Соберите:
   ```bash
   ./gradlew assembleRelease
   ```
   Готовый APK: `app/build/outputs/apk/release/app-release.apk`.

   Если `keystore.properties` отсутствует, release соберётся **без подписи**
   (`app-release-unsigned.apk`) — для публикации он не подойдёт.

## Проверка перед загрузкой в RuStore

```bash
# подпись (должно быть CN=Planium, Verifies)
$ANDROID_HOME/build-tools/35.0.0/apksigner verify --print-certs app/build/outputs/apk/release/app-release.apk

# в выводе НЕ должно быть application-debuggable
$ANDROID_HOME/build-tools/35.0.0/aapt2 dump badging app/build/outputs/apk/release/app-release.apk
```

RuStore отклоняет сборку как «debug», если в манифесте есть `android:debuggable="true"`
или APK подписан стандартным debug-ключом Android. В этом проекте release-сборка не
содержит ни того, ни другого.

## Безопасность

`keystore.properties`, `*.jks`, `*.keystore`, `local.properties` и каталоги сборки
добавлены в `.gitignore` и **никогда не коммитятся**. Не храните ключи подписи в
системе контроля версий.
