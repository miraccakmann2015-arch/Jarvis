# JARVIS AI — Android (Kotlin + Jetpack Compose)

Tony Stark'ın JARVIS'inden esinlenen, Türkçe konuşan Android yapay zekâ asistanı.
Groq (beyin) • Fish Audio (ses) • Picovoice Porcupine (offline "Jarvis" uyandırma) • Room (hafıza).

## Kurulum
1. **Android Studio Ladybug (2024.2) veya üstü** ile `JarvisAI` klasörünü açın (File > Open). JDK 17 gerekir (Studio ile gelir).
2. Gradle senkronizasyonunu bekleyin (Studio gerekirse Gradle 8.9'u indirir). Gradle wrapper jar dosyası pakete dahil değildir; Studio oluşturur ya da `gradle wrapper` komutunu çalıştırın.
3. Gerçek bir telefon bağlayın (Android 8.0+ / API 26+) ve **Run** deyin. Emülatörde mikrofon ve konuşma tanıma sınırlıdır.
4. Uygulamada **Ayarlar** ekranından anahtarları girin:
   - **Groq API anahtarı:** https://console.groq.com/keys — model adı varsayılan `llama-3.3-70b-versatile` (değiştirilebilir).
   - **Fish Audio:** https://fish.audio → API anahtarı ve istediğiniz ses modelinin ID'si (`reference_id`). Türkçe ses için bir Türkçe model seçin. Adres varsayılan `https://api.fish.audio/v1/tts`.
   - **Picovoice AccessKey:** https://console.picovoice.ai (ücretsiz). Yalnızca uyandırma kelimesi için gerekir.
5. **Groq testi** ve **Ses testi** düğmeleriyle bağlantıyı doğrulayın.

## Kullanım
- **Mikrofon butonu:** bas → konuş → metne çevrilir → Groq → yanıt seslendirilir. Tekrar basmak durdurur.
- **Uyandırma (kulak simgesi):** foreground service başlar, kalıcı bildirimde "Mikrofon aktif" görünür. "Jarvis" deyince "Buradayım efendim." der ve sizi dinler. Bildirimdeki **Dinlemeyi kapat** veya aynı düğme ile kapatılır.
- **Telefon komutları (örnekler):** "Feneri aç/kapat", "Pil durumu", "Saat kaç", "Bugün tarih ne", "Sesi %50 yap / sesi artır", "Saat 7.30'a alarm kur", "10 dakika sonra su içmeyi hatırlat", "YouTube'u aç", "Wi-Fi ayarlarını aç", "Python nedir internette araştır".
- **Hafıza:** "Hatırla: kahveyi sade içerim" veya "Benim adım Ali" bilgi olarak kaydedilir; Hafıza ekranından görüntülenir/silinir. Sohbet geçmişi Room'da tutulur, ilgili eski parçalar bağlama eklenir.

## Mimari (`app/src/main/java/com/jarvis/ai/`)
| Paket | İçerik |
|---|---|
| `ai/` | `GroqClient`, `WebSearch` (Vikipedi), `JarvisEngine` (dinle→düşün→konuş akışı) |
| `audio/` | `SpeechInput` (SpeechRecognizer tr-TR), `Speaker` (Fish Audio + TextToSpeech yedeği) |
| `wakeword/` | `WakeWordService` (mikrofon tipi foreground service, Porcupine "JARVIS") |
| `memory/` | Room: mesajlar + bilgiler |
| `commands/` | `CommandRouter`, `ReminderReceiver` |
| `settings/` | `Settings` |
| `security/` | `SecureStore` (Android Keystore AES-256-GCM) |
| `ui/` | Compose HUD, hologram çekirdek, ayarlar, hafıza |

## Güvenlik
Anahtarlar kaynak kodda yok; Keystore ile şifreli saklanır, loglanmaz. Tüm ağ trafiği HTTPS (cleartext kapalı). Mikrofon kullanımı ekranda ve bildirimde görünür. Telefon işlemleri yalnızca siz komut verince yapılır; ayar değişiklikleri için ilgili sistem ekranı açılır.

## Bilinen sınırlar
- Konuşma tanıma Google servisini kullanır; cihaza göre internet isteyebilir.
- Android 10+ arka planda başka uygulama/alarm ekranı açmayı kısıtlar: uygulama ekranda değilken wake-word ile verilen "alarm kur / uygulama aç" komutları sistem tarafından engellenebilir. Fener, ses, pil, saat, sohbet ve hatırlatıcı arka planda çalışır.
- Porcupine'in yerleşik "JARVIS" kelimesi İngilizce telaffuz modelidir; hassasiyet `WakeWordService`'te (0.7) ayarlanır.
- Üretici pil optimizasyonları (Xiaomi, Samsung vb.) servisi kapatabilir; uygulamayı pil kısıtlamasından muaf tutun.
