# SaaS-Virtual ve Edge-Intelligence: Tam Uygulama Planı

**Tarih:** 2026-10-17  
**Proje:** Drone Mission Platform (SaaS-Virtual & Edge-Intelligence)  
**Versiyon Hedefi:** v0.1.0  
**Geliştirici Ekip:** 2 Kişi (Kişi A: Backend & Simülasyon, Kişi B: Frontend & Dokümantasyon)

---

## 0. Hedef ve Kapsam

### Hedef
Tarayıcıda Leaflet haritası üzerinde rota çizilen, bu rotayı PX4 SITL + Gazebo simülasyonundaki İHA’ya MAVLink görevi olarak yükleyip başlatan, telemetriyi WebSocket üzerinden canlı gösteren ve CSV’ye kaydeden tam entegre bir web/edge platformu. Zaman kalırsa AprilTag ile hedef tespiti modülü eklenir.

### Kapsam ve Kararlar
| Konu | Karar | Gerekçe |
| :--- | :--- | :--- |
| **Ana Proje** | SaaS-Virtual (planlama, simülasyon, telemetri, CSV kaydı) | Üç günde bitirilebilir, portföyde net görünür. |
| **Görüntü İşleme** | AprilTag ile tespit (İsteğe bağlı) | Model eğitimi gerektirmez, deterministiktir. |
| **YOLO Eğitimi** | Yapılmıyor | Üç günde iki kişi için gerçekçi değil. |
| **Otonom İniş Komutu** | Gelecek Sürüm | Riskli, offboard kontrol zaman yetmez. |
| **Gerçek Donanım** | Yok | Pahalı, simülatör SITL odaklı. |
| **Harita** | React-Leaflet (2D) | Cesium 3D kurulum maliyeti yüksek. |
| **Veri Seti** | Yalnızca AprilTag test görselleri | Lisans/kapsam sorunlarını önler. |

---

## 1. Global Kısıtlar (Tüm Görevler İçin Geçerli)
1. Simülasyon dışında hiçbir donanım kullanılmaz.
2. PX4 ve Gazebo kodu yazılmaz, olduğu gibi Docker imajı üzerinden (`px4io/px4-sitl-gazebo-ros2`) kullanılır.
3. Tüm servisler `docker compose up` ile ayağa kalkar.
4. Backend veya frontend arasındaki her alan adı `API.md` sözleşmesi ile birebir aynıdır.
5. Her görev için birim test yazılır (TDD prensibi).
6. Her iş paketi bittiğinde bağımsız ve açıklayıcı bir commit atılır.
7. Gizli anahtar, şifre veya kişisel veri repoya girmez (`.gitignore` denetimi).

---

## 2. Sistem Mimarisi

```
+--------------------------------------------------------------------------+
|                               FRONTEND                                   |
|   React 18 + TypeScript + Vite + Zustand + Leaflet                       |
|   - Görev Planlama Haritası (Waypoint Çizimi / Düzenleme)                |
|   - Telemetri Paneli & İHA Canlı İzi                                     |
|   - Görev Kontrol Butonları (Yükle, Başlat, İptal/RTL)                   |
+-------------------+----------------------------------^-------------------+
                    | REST (HTTP)                      | WebSocket (/ws/telemetry)
                    v                                  | (5 Hz JSON Telemetri)
+------------------------------------------------------+-------------------+
|                               BACKEND                                    |
|   FastAPI + Pydantic v2 + pymavlink                                      |
|   - REST API (POST /mission, POST /mission/start, GET /health)           |
|   - Görev Doğrulayıcı & MISSION_ITEM_INT Dönüştürücü                     |
|   - MAVLink Durum Makinesi (idle -> uploaded -> flying -> completed)     |
|   - Telemetri Ayrıştırıcı & CSV Uçuş Kaydedici (flights/*.csv)           |
+-------------------+----------------------------------^-------------------+
                    | MAVLink Komutları                | MAVLink Telemetri
                    | (UDP 14540)                      | (GLOBAL_POSITION_INT, etc)
                    v                                  |
+------------------------------------------------------+-------------------+
|                        SİMÜLASYON ORTAMI                                 |
|   Docker: px4io/px4-sitl-gazebo-ros2                                     |
|   - PX4 Autopilot (v1.14+ SITL)                                          |
|   - Gazebo Harmonic Simülatörü (x500 Multirotor)                         |
|   - (Opsiyonel) AprilTag İniş Pedi & Aşağı Bakan Kamera                  |
+--------------------------------------------------------------------------+
```

---

## 3. Rol Dağılımı ve İş Paketleri (2 Kişilik Takım)

### Kişi A: Backend, MAVLink & Simülasyon Sorumlusu
- **A1. Docker Ortamı:** `docker-compose.yml`, PX4 SITL MAVLink UDP 14540 portu, backend konteyneri.
- **A2. Modeller & Config:** Pydantic `Waypoint`, `MissionRequest`, `Telemetry` modelleri.
- **A3. Doğrulayıcı (Validator):** Waypoint sayı ve sınır kontrolleri (2-20 nokta, 10-500m irtifa).
- **A4. Dönüştürücü (Converter):** Waypoint -> MAVLink `MISSION_ITEM_INT` (1e7 ölçek, Takeoff seq 0).
- **A5. MAVLink Bağlantısı & Telemetri:** UDP `mavutil` bağlantısı, telemetri ayrıştırıcı (5 Hz).
- **A6. Görev Yükleyici (Uploader):** MAVLink görev aktarım protokolü (MISSION_COUNT, REQUEST, ACK).
- **A7. Durum Makinesi & Komutlar:** Durum yönetimi (idle, uploaded, flying, completed, aborted), Arm, Start, RTL.
- **A8. CSV Telemetri Kaydedici:** `flights/<id>.csv` formatında otomatik kayıt.
- **A9. FastAPI Endpoint & WebSocket:** REST ve `/ws/telemetry` FastAPI implementasyonu.

### Kişi B: Frontend, Harita Arayüzü & Dokümantasyon Sorumlusu
- **B1. Frontend İskeleti:** Vite + React + TS, paket kurulumları (Leaflet, Zustand, Vitest).
- **B2. Zustand Store:** `missionStore.ts` (Waypoint state, telemetri buffer, trail).
- **B3. API Client & useTelemetry Hook:** REST istemcisi ve otomatik yeniden bağlanan WebSocket hook'u.
- **B4. Harita Bileşeni (MissionMap):** Harita tıklama ile waypoint ekleme, sürükleme, silme, rota çizgisi.
- **B5. Waypoint Paneli:** Nokta listesi, irtifa düzenleme, anlık validasyon hataları.
- **B6. Telemetri Paneli:** Hız, irtifa, yön göstergeleri, arm ve bağlantı durumu rozetleri.
- **B7. İHA İkonu ve Canlı İz:** Heading açısına göre dönen drone ikonu ve 200 noktalık trail çizgisi.
- **B8. Kontrol Çubuğu:** Yükle, Başlat, İptal butonları ve durum makinesi kilitlemeleri.
- **B10. Frontend Docker & Nginx:** Multi-stage build, backend ve ws proxy konfigürasyonu.
- **D. Dokümantasyon & API.md:** `API.md` sözleşmesi, mimari diyagramlar, README kullanım rehberi.

### Ortak / Opsiyonel İş Paketleri
- **İş Paketi C (Opsiyonel):** Gazebo AprilTag iniş pedi, kamera köprüsü, OpenCV AprilTag tespiti.
- **İş Paketi E (Entegrasyon):** Uçtan uca test, hata senaryoları, CI pipeline (`.github/workflows/ci.yml`).

---

## 4. Git Çalışma Akışı ve İşbirliği Kuralları
1. **Ana Dal:** `main` dalı her zaman stabil ve çalışır durumda tutulur. Doğrudan commit atılmaz.
2. **Dal İsimlendirme:**
   - `feature/a1-docker-env`
   - `feature/a4-mission-converter`
   - `feature/b4-mission-map`
   - `feature/b6-telemetry-panel`
3. **Sözleşmeye Sadakat:** `API.md` üzerinde mutabık kalınmadan veri modelleri değiştirilmez.
4. **TDD:** Testler daima implementasyon öncesinde yazılır veya test dosyasıyla birlikte PR açılır.
