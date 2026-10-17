# Drone Mission Platform - API Sözleşmesi (v0.1.0)

> **DİKKAT:** Bu sözleşme Kişi A (Backend) ve Kişi B (Frontend) arasındaki bağlayıcı protokoldür. Sözleşmede değişiklik yapılmadan kod tarafında alan isimleri veya tipleri değiştirilemez.

---

## 1. REST Endpoint'leri

### 1.1 Sistem Sağlık Kontrolü
- **URL:** `GET /health`
- **Açıklama:** PX4 SITL bağlantısı ve sistem durumunu döner.
- **Yanıt (200 OK):**
```json
{
  "px4": true,
  "camera": false,
  "version": "0.1.0"
}
```

---

### 1.2 Görev Yükleme (Mission Upload)
- **URL:** `POST /mission`
- **Açıklama:** Arayüzde çizilen rota noktalarını doğrular, MAVLink formatına dönüştürüp araca yükler.
- **İstek Gövdesi (Content-Type: application/json):**
```json
{
  "waypoints": [
    { "lat": 40.7812, "lon": 29.4102, "alt": 50.0 },
    { "lat": 40.7820, "lon": 29.4110, "alt": 60.0 }
  ]
}
```

- **Başarılı Yanıt (200 OK):**
```json
{
  "mission_id": "m_20261017_001",
  "status": "accepted",
  "reason": null
}
```

- **Başarısız / Doğrulama Hatası (422 Unprocessable Entity):**
```json
{
  "mission_id": null,
  "status": "rejected",
  "reason": "waypoint sayısı 2 ile 20 arasında olmalı"
}
```

---

### 1.3 Görevi Başlatma (Mission Start)
- **URL:** `POST /mission/start`
- **Açıklama:** Aracı ARM eder ve `AUTO.MISSION` uçuş moduna geçirir.
- **Yanıt (200 OK):**
```json
{
  "status": "started",
  "reason": null
}
```
- **Yanıt Hata Durumu (400 Bad Request):**
```json
{
  "status": "failed",
  "reason": "Araç ARM edilemedi veya görev yüklenmemiş"
}
```

---

### 1.4 Görevi İptal Etme (Mission Abort / RTL)
- **URL:** `POST /mission/abort`
- **Açıklama:** Uçuşu derhal durdurur ve aracı RTL (Return To Launch) moduna geçirir.
- **Yanıt (200 OK):**
```json
{
  "status": "aborted"
}
```

---

### 1.5 Görev Durumu Sorgulama
- **URL:** `GET /mission/status`
- **Açıklama:** Mevcut görev icra durumunu döner.
- **Yanıt (200 OK):**
```json
{
  "state": "flying",
  "current_wp": 2,
  "total_wp": 5
}
```
- **Geçerli `state` Değerleri:**
  - `idle`: Görev yüklenmedi, araç beklemede.
  - `uploaded`: Görev başarıyla yüklendi, başlatmaya hazır.
  - `flying`: Görev icra ediliyor, İHA uçuşta.
  - `completed`: Tüm waypoint'ler tamamlandı.
  - `aborted`: Görev iptal edildi (RTL devrede).
  - `failed`: Görev veya bağlantı hatası oluştu.

---

### 1.6 AprilTag İniş Durumu (Opsiyonel Modül)
- **URL:** `GET /landing/status`
- **Açıklama:** Aşağı bakan kamera üzerinden hedef AprilTag durumu.
- **Yanıt (Hedef Tespit Edildi):**
```json
{
  "tag_detected": true,
  "offset_x": 12.4,
  "offset_y": -3.1,
  "distance": 1.82
}
```
- **Yanıt (Hedef Yok):**
```json
{
  "tag_detected": false,
  "offset_x": null,
  "offset_y": null,
  "distance": null
}
```

---

## 2. WebSocket Telemetri Akışı

- **URL:** `ws://<host>:<port>/ws/telemetry`
- **Frekans:** 5 Hz (Her 200 ms'de bir)
- **Mesaj Formatı (Telemetri):**
```json
{
  "type": "telemetry",
  "t": 1729170000.123,
  "lat": 40.7812000,
  "lon": 29.4102000,
  "alt": 52.3,
  "heading": 137.0,
  "groundspeed": 8.4,
  "armed": true,
  "mode": "MISSION",
  "tag_detected": false,
  "offset_x": null,
  "offset_y": null
}
```

- **Hata Mesajı Formatı (Bağlantı koptuğunda / veri yokken):**
```json
{
  "type": "error",
  "message": "PX4 bağlantısı koptu"
}
```

---

## 3. Doğrulama (Validation) Kuralları

Her iki taraf (frontend form denetimi ve backend Pydantic doğrulaması) bu kurallara uymalıdır:

| Parametre | Minimum | Maksimum | Açıklama |
| :--- | :--- | :--- | :--- |
| **Waypoint Sayısı** | 2 | 20 | İlk nokta kalkış noktası referansıdır. |
| **Enlem (`lat`)** | -90.0 | +90.0 | WGS84 formatında ondalık derece. |
| **Boylam (`lon`)** | -180.0 | +180.0 | WGS84 formatında ondalık derece. |
| **İrtifa (`alt`)** | 10.0 metre | 500.0 metre | Kalkış noktasına göre bağıl irtifa (AGL). |
