# Don't Overreact
Bài đầu tiên chủ yểu đọc được code, rồi tìm hidden in4 giấu trong client-side là ra được flag

Đầu tiên đọc trước file AndroidManifest.xml để biết Class nào sẽ chạy đầu tiên khi mở app
```xml
<application android:theme="@style/AppTheme" android:label="@string/app_name" 
             android:icon="@mipmap/ic_launcher" **android:name="com.awesomeproject.MainApplication" **
             android:allowBackup="false" android:roundIcon="@mipmap/ic_launcher_round" 
             android:appComponentFactory="androidx.core.app.CoreComponentFactory">
    ...
</application>
```
Ta thấy `MainApplication` là điểm bắt đầu

Đọc 1 lượt code thì tới hàm `getBundleAssetName()`

![image](https://user-images.githubusercontent.com/46492646/166906138-783ca30d-333c-496d-b0b2-0e57c95229df.png)

nó return về file `index.android.bundle` trong assets

![image](https://user-images.githubusercontent.com/46492646/166906580-ccfa1808-f68f-45ff-ac6c-04c807366391.png)

Và phần quan trọng nằm ở cuối file:

![image](https://user-images.githubusercontent.com/46492646/166906765-c661ce80-f0fe-402a-9acf-feced176b3e0.png)

Deocde base64 ra lấy flag thôi:

![image](https://user-images.githubusercontent.com/46492646/166906928-34ca6f97-2e65-4e14-b72c-4a166558c33d.png)

# Cat
Đề cho 1 file `cat.ab`, search google thì biết đây là file Android Backup, mình cần unpack nó ra để xem bên trong file có gì: https://stackoverflow.com/questions/18533567/how-to-extract-or-unpack-an-ab-file-android-backup-file

Cạy cmd trên linux: `java.exe -jar abe.jar unpack cat.ab cat.rar ""`

Sau khi chạy cmd bên trên thì unzip file `cat.rar` ra được 2 folder `app` với `shared`, tìm quanh 1 các tài nguyên để tìm flag:

![image](https://user-images.githubusercontent.com/46492646/166909219-b467325f-1456-4654-9e0e-e073e02d421c.png)

# APKey
Để làm được bài này, trước hết phải đọc Bài 4 trong series Android Pentest của anh tsu:
https://github.com/tsug0d/AndroidMobilePentest101

Đầu tiên dùng `adb install APKey.apk` để install file apk, giao diện app:
![image](https://user-images.githubusercontent.com/46492646/166911513-9dc548cf-9482-4f64-ba20-4243a3798c8e.png)

Sau đó dùng [jadx-gui](https://github.com/skylot/jadx) để reverse file apk, đoc code:
```java
public EditText f928c;
public EditText d;
// Khai báo 2 button Name & Password

try {
    if (MainActivity.this.f928c.getText().toString().equals("admin")) {
        MainActivity mainActivity = MainActivity.this;
        b bVar = mainActivity.e;
        String obj = mainActivity.d.getText().toString();
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            messageDigest.update(obj.getBytes());
            byte[] digest = messageDigest.digest();
            StringBuffer stringBuffer = new StringBuffer();
            for (byte b2 : digest) {
                stringBuffer.append(Integer.toHexString(b2 & 255));
            }
            str = stringBuffer.toString();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            str = "";
        }
        if (str.equals("a2a3d412e92d896134d9c9126d756f")) {
            Context applicationContext = MainActivity.this.getApplicationContext();
            MainActivity mainActivity2 = MainActivity.this;
            b bVar2 = mainActivity2.e;
            g gVar = mainActivity2.f;
            makeText = Toast.makeText(applicationContext, b.a(g.a()), 1);
            makeText.show();
        }
    }
    makeText = Toast.makeText(MainActivity.this.getApplicationContext(), "Wrong Credentials!", 0);
    makeText.show();
} catch (Exception e2) {
    e2.printStackTrace();
}
```
Dựa vào code ta có thể hiểu workflow code: check `Name == "admin" & md5(Password) == "a2a3d412e92d896134d9c9126d756f")`

Nếu true thì show flag còn ko thì show `"Wrong Credentials!"`

Thường mới tiếp cận thì sẽ tìm cách dịch ngược đoạn md5 tìm ra bản gốc thì sẽ rất khó, cách đơn giản nhất là sửa code trong smali

Vì sao phải sửa code smali: Vì condition check Password nằm trong code nên chúng ta chỉ có thể chỉnh sửa code smali rồi sau đó repacked lại file apk rồi install nó thì sẽ bypass được thôi.

Tìm kiếm ví trí condition trong smali

![image](https://user-images.githubusercontent.com/46492646/166912610-cef171e6-621d-4540-a8e2-14f02a4cd16d.png)

Ta có thể thấy dòng smali: `if-eqz p1, :cond_7b`, tức là nếu p1 == false thì sẽ thì chạy tới vị trí `:cond_7b` còn không thì sẽ chạy tiếp bên dưới

Công việc của mình là sửa điều kiện chỗ này cho ngược lại như trong slide anh tsu là done kèo.

Search 1 xíu để tìm câu điều kiện ngược với `if-eqz`: `if-nez`
https://stackoverflow.com/questions/40613470/copy-else-statement-to-if-statement

Giờ dùng apktool để unpack file apk:
`apk d APKey.apk`
Sau đó sửa code smali sau khi unpack:

![image](https://user-images.githubusercontent.com/46492646/166914721-9cdaba6d-a107-401e-b703-b77d6f24f5a7.png)

Sau đó repack file lại để install `apk b APKey`, file apk mới sẽ được tạo trong folder dist như ảnh bên trên.

Sign file với `keytool` và `jarsigner` như trong slide

Gỡ cài đặt file cũ rồi install lại file APK mới trong folder dist. Giờ thì nhập: `anhtradeptrai` là ra flag

![image](https://user-images.githubusercontent.com/46492646/166915287-05c2000f-7ec4-4006-ac39-05973201d8f0.png)

# Cryptohorrific
Bài này liên quan tới IOS và decrypt.
Dựa vào hint: 

![image](https://user-images.githubusercontent.com/46492646/166915680-b7bb986b-f3d2-4f9c-a1c5-038b744ca087.png)

Đầu tiên mình cài `plistutil`: https://command-not-found.com/plistutil

Sau đó đọc file `challenge.plist`: `plistutil -i ch*.plist -o challenge.xml` thì tìm được flag: 

![image](https://user-images.githubusercontent.com/46492646/166916171-b3d5c5a8-58f8-4bf3-8458-e8bbd8432300.png)

Dùng IDA để reverse file `challenge` thì ta thấy dùng hàm `CCCrypt` để encrypt flag

![image](https://user-images.githubusercontent.com/46492646/166916444-3cc1c796-0031-4dd6-945a-e42a861bfc7d.png)

Và key:

![image](https://user-images.githubusercontent.com/46492646/166916634-3944db02-160a-484e-af08-2e1f97ffbdb2.png)

Dùng tool để decrypt nó ra: https://www.devglan.com/online-tools/aes-encryption-decryption

![image](https://user-images.githubusercontent.com/46492646/166916725-3bc91baa-5200-47c2-a141-335685ea1d93.png)






