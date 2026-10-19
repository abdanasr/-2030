name: Build Full BMS Pro APK

on:
  push:
    branches: [ "main", "master" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'

      - name: Locate Project, Fix App Name & Build APK
        run: |
          # البحث التلقائي عن ملف main.dart الذي رفعته من الكمبيوتر
          MAIN_FILE=$(find . -name "main.dart" | head -n 1)
          
          if [ -n "$MAIN_FILE" ]; then
            echo "Found main.dart at: $MAIN_FILE"
            PROJECT_DIR=$(dirname $(dirname "$MAIN_FILE"))
            cd "$PROJECT_DIR"
            flutter create . --project-name flutter_bms_pro
          else
            echo "No uploaded main.dart found, creating complete BMS dashboard..."
            flutter create bms_app --project-name flutter_bms_pro
            cd bms_app
          fi

          # تغيير اسم التطبيق الخارجي في الهاتف من flutter_bms_pro إلى BMS Pro
          sed -i 's/android:label="flutter_bms_pro"/android:label="BMS Pro"/g' android/app/src/main/AndroidManifest.xml

          # إضافة صلاحيات البلوتوث والشبكة في AndroidManifest.xml
          sed -i '/<application/i <uses-permission android:name="android.permission.INTERNET"/>\n<uses-permission android:name="android.permission.BLUETOOTH"/>\n<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>\n<uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>\n<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>\n<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>' android/app/src/main/AndroidManifest.xml

          # بناء ملف الـ APK النهائي
          flutter pub get
          flutter build apk --release --no-tree-shake-icons
          cp build/app/outputs/flutter-apk/app-release.apk $GITHUB_WORKSPACE/BMS-Pro-Release.apk

      - name: Upload BMS-Pro-Release APK
        uses: actions/upload-artifact@v4
        with:
          name: BMS-Pro-Release-APK
          path: BMS-Pro-Release.apk
