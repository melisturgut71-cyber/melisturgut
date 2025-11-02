#ESP32 - Joystick ile MQTT Araç Simülatörü
Bu proje, bir ESP32 geliştirme kartı ve bir analog joystick kullanarak basit bir araç simülatörü oluşturur. Simülatör, joystick hareketlerini (gaz ve fren) okur, bu hareketlere dayanarak anlık hız, motor devri (RPM) ve vites gibi sanal telemetri verileri üretir. Bu veriler, Wi-Fi üzerinden bir MQTT broker'ına JSON formatında yayınlanır.Bu kod, tezinizde bahsettiğiniz "Simülasyon Modu" için mükemmel bir temel oluşturur. 
Özellikler:
Analog joystick ile gaz ve fren kontrolü.Hıza bağlı olarak otomatik vites hesaplama.Hıza bağlı olarak basit RPM (motor devri) hesaplama.
Wi-Fi üzerinden MQTT broker'ına anlık veri gönderimi.JSON formatında yapılandırılmış veri çıkışı.
Donanım Gereksinimleri:
ESP32 Geliştirme Kartı (Örn: ESP32-WROOM-32)
Analog Joystick Modülü (KY-023 gibi, 2 eksenli)
Jumper kablolar
Yazılım ve Kütüphaneler:
Arduino IDE veya Platform IO ESP32 
Board Desteği (Arduino IDE için)
WiFi.h (ESP32 core paketi ile birlikte gelir)
PubSubClient.h (Arduino IDE'nin "Kütüphane Yöneticisi" üzerinden kurulmalıdır)
Bağlantı Şeması:
Bu kodun çalışması için joystick modülünüzü ESP32'ye bağlamanız gerekmektedir.
Kod sadece X eksenini (VRX_PIN) kullandığı için Y eksenini bağlamanıza gerek yoktur.
Joystick PiniESP32 Pini 
Açıklama GND-GNDOrtak 
Toprak+5VV IN5V 
Güç Beslemesi VRX GPIO 34X Ekseni (Gaz/Fren)(Not: GPIO 34 yalnızca bir giriş (INPUT) pinidir ve analog okuma (ADC) için idealdir.)
Kurulum ve YapılandırmaKodu:
ESP32'ye yüklemeden önce aşağıdaki değişkenleri kendi ayarlarınızla değiştirmelisiniz
Wi-Fi Ayarları:C++// --- Wi-Fi Ayarları ---
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
MQTT Ayarları:Kod, herkese açık olan broker.hivemq.com sunucusunu kullanmaktadır. 
Bu sunucuyu test için değiştirmeden kullanabilirsiniz. Verilerinizi arac/simulasyon konusunda (topic) görebilirsiniz.C++// --- MQTT Ayarları ---
const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 1883;
const char* dataTopic = "arac/simulasyon";
Çalışma Prensibi Joystick Okuma: 
Kod, loop() fonksiyonu içinde sürekli olarak GPIO 34'e bağlı joystick'in X eksenindeki analog değerini (0-4095 arası) okur.Gaz ve Fren Mantığı:
JOY_DEADZONE (2725): Joystick ileri itildiğinde, bu değerin üzerindeki okumalar "gaz" olarak algılanır. Gaz miktarı, joystick'in ne kadar ileri itildiğine bağlı olarak hızı (en fazla 200) artırır.
JOY_BRAKEZONE (1500): Joystick geri çekildiğinde, bu değerin altındaki okumalar "fren" olarak algılanır ve hızı (speedVal) kademeli olarak düşürür.Bu iki değer arasında kalan bölge (1500-2725 arası) "boş bölge" (deadzone) olup, hızda bir değişikliğe neden olmaz.
Veri Hesaplama:Vites: getGear() fonksiyonu, anlık hıza göre 1'den 6'ya kadar otomatik bir vites atar.RPM: Hıza bağlı olarak basit bir RPM değeri (maks. 8000) hesaplanır.MQTT Yayını: Sistem, her 300 milisaniyede bir hesapladığı tüm verileri dataTopic (arac/simulasyon) konusuna JSON formatında yayınlar.
Örnek MQTT ÇıktısıHerhangi bir MQTT istemcisi (MQTT Explorer, MQTT.fx vb.) ile broker.hivemq.com sunucusuna bağlanıp arac/simulasyon konusuna abone olarak aşağıdaki gibi bir veri akışı görebilirsiniz:JSON{
  "speed": 110,
  "rpm": 6600,
  "gear": 5,
  "joy_x": 3850
}
speed: Anlık hesaplanan sanal hız (0-200).rpm: Anlık hesaplanan sanal motor devri (0-8000).gear: Anlık hesaplanan sanal vites (1-6).joy_x: Joystick'in ham X ekseni değeri (hata ayıklama için).
