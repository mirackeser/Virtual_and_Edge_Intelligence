# 🚁 Drone Mission Platform (SaaS-Virtual & Edge-Intelligence)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18.x-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![Leaflet](https://img.shields.io/badge/Leaflet-1.9+-199900?logo=leaflet&logoColor=white)](https://leafletjs.com/)
[![PX4 SITL](https://img.shields.io/badge/PX4-v1.14+-005696?logo=drone&logoColor=white)](https://px4.io/)

Web tarayıcısı üzerinde interaktif haritada rota çizilen, oluşturulan görev rotasını **PX4 SITL + Gazebo Harmonic** simülasyon ortamındaki otonom İHA'ya **MAVLink** protokolü ile aktaran, uçuşu başlatan, canlı telemetriyi **WebSocket** üzerinden haritada takip eden ve tüm telemetri akışını CSV dosyasına kaydeden tam entegre uçuş yönetim platformu.

---

## 🏛️ Sistem Mimarisi

```
+-------------------------------------------------------------------------------+
|                                FRONTEND                                       |
|    React 18 + TypeScript + Vite + Zustand + React-Leaflet                     |
|    - 2D Harita Arayüzü: Waypoint ekleme, sürükleme, irtifa belirleme          |
|    - Telemetri Paneli: İrtifa, yer hızı, heading, mod, arm durumu             |
|    - Canlı Drone Görselleştirme: Açıya göre dönen ikon ve son 200 nokta iz    |
|    - Kontrol Çubuğu: Gönder (POST /mission), Başlat, Acil İptal (RTL)         |
+---------------------+-----------------------------------^---------------------+
                      | REST API (HTTP 8000)              | WebSocket (ws:// 8000)
                      v                                   | (/ws/telemetry @ 5 Hz)
+---------------------------------------------------------+---------------------+
|                                BACKEND                                        |
|    Python 3.11+ / FastAPI / pymavlink / Pydantic v2                           |
|    - REST Routes: /health, /mission, /mission/start, /mission/abort           |
|    - Waypoint Validator: 2-20 nokta, 10-500m irtifa, WGS84 sınır kontrolleri  |
|    - MAVLink Converter: Lat/Lon 1e7 tam sayı, Takeoff (22), Waypoint (16)     |
|    - MAVLink Uploader: MISSION_COUNT, MISSION_REQUEST_INT, MISSION_ACK        |
|    - Durum Makinesi: idle -> uploaded -> flying -> completed / aborted       |
|    - Flight Recorder: flights/YYYYMMDD_HHMMSS.csv kayıt motoru                |
+---------------------+-----------------------------------^---------------------+
                      | MAVLink Komutları                 | MAVLink Telemetri
                      | (UDP 14540)                       | (GLOBAL_POSITION_INT, etc)
                      v                                   |
+---------------------------------------------------------+---------------------+
|                           SİMÜLASYON ORTAMI                                   |
|    Docker: px4io/px4-sitl-gazebo-ros2 (HEADLESS)                              |
|    - PX4 Autopilot v1.14+ SITL                                                |
|    - Gazebo Harmonic (x500 Multicopter Modeli)                                |
|    - [Opsiyonel Modül]: AprilTag İniş Pedi & Sanal Kamera                     |
+-------------------------------------------------------------------------------+
```

---

## 👥 2 Kişilik Takım ve Rol Dağılımı

| Kişi | Rol | Sorumlu Olduğu Alanlar |
| :--- | :--- | :--- |
| **Kişi A** | Backend & MAVLink Mühendisi | `docker-compose.yml`, FastAPI backend, pymavlink UDP bağlantısı, MAVLink görev yükleme protokolü, telemetri ayrıştırıcı, durum makinesi, CSV logger |
| **Kişi B** | Frontend & UI/UX Mühendisi | React/TypeScript, Leaflet haritası, Zustand state yönetimi, WebSocket canlı telemetri hook'u, Nginx reverse proxy, dokümantasyon |

---

## 🚀 Hızlı Başlangıç (Tek Komutla Çalıştırma)

### Gereksinimler
- Docker & Docker Compose
- Minimum 8 GB RAM (Gazebo SITL için)
- Web tarayıcısı (Chrome, Firefox, Safari)

### Çalıştırma
```bash
# Depoyu klonlayın
git clone https://github.com/KULLANICI_ADI/drone-mission-platform.git
cd drone-mission-platform

# Tüm servisleri ayağa kaldırın
docker compose up --build
```

> **Önemli Not:** Gazebo simülatörünün ilk defa ayağa kalkması ve dünya modellerini indirmesi sistem özelliklerinize bağlı olarak 2-4 dakika sürebilir. Backend servisi simülatörün sağlıklı olmasını (`service_healthy`) otomatik olarak bekler.

### Arayüz ve Servis Adresleri
- **Web Arayüzü:** [http://localhost:8080](http://localhost:8080)
- **FastAPI Swagger Dokümantasyonu:** [http://localhost:8000/docs](http://localhost:8000/docs)
- **Sağlık Kontrolü:** [http://localhost:8000/health](http://localhost:8000/health)

---

## 📁 Proje Dosya Yapısı

```
drone-mission-platform/
├── docker-compose.yml          # 3 servis: px4-sim, backend, frontend
├── API.md                      # Backend ve Frontend bağlayıcı arayüz sözleşmesi
├── ROADMAP.md                  # 2 kişilik detaylı görev ve bağımlılık haritası
├── CONTRIBUTING.md             # Dal (branch), commit ve PR kuralları
├── README.md                   # Proje genel rehberi
├── .gitignore                  # flights/, node_modules/, __pycache__/ koruması
├── .github/
│   └── workflows/
│       └── ci.yml              # Test ve derleme otomasyonu
├── backend/
│   ├── Dockerfile
│   ├── pyproject.toml
│   ├── app/
│   │   ├── main.py             # FastAPI uygulaması & router'lar
│   │   ├── config.py           # Ortam değişkenleri (PX4_CONNECTION, FLIGHTS_DIR)
│   │   ├── models.py           # Pydantic modelleri (Waypoint, Telemetry, Mission)
│   │   ├── mavlink/
│   │   │   ├── connection.py   # UDP soket & heartbeat yönetimi
│   │   │   └── telemetry.py    # MAVLink telemetri ayrıştırma
│   │   ├── mission/
│   │   │   ├── validator.py    # Waypoint doğrulama (2-20 nokta, 10-500m)
│   │   │   ├── converter.py    # Waypoint -> MISSION_ITEM_INT (1e7 ölçek)
│   │   │   ├── uploader.py     # MAVLink görev aktarım protokolü
│   │   │   └── state.py        # Uçuş durum makinesi (idle, flying, aborted)
│   │   ├── recorder.py         # CSV telemetri kayıt mekanizması
│   │   ├── landing.py          # AprilTag hedef durumu (opsiyonel)
│   │   └── api/
│   │       ├── routes.py       # REST API endpoint'leri
│   │       └── ws.py           # WebSocket telemetri yayını (5 Hz)
│   └── tests/
│       ├── test_validator.py
│       ├── test_converter.py
│       ├── test_telemetry.py
│       ├── test_state.py
│       ├── test_recorder.py
│       ├── test_routes.py
│       └── test_ws.py
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf              # SPA yönlendirmesi & /api/, /ws/ proxy ayarları
│   ├── package.json
│   ├── vite.config.ts
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── types/index.ts      # API.md ile birebir TypeScript tipleri
│   │   ├── api/client.ts       # Axios/Fetch REST istemcisi
│   │   ├── store/missionStore.ts # Zustand global durum deposu
│   │   ├── hooks/useTelemetry.ts # Otomatik yeniden bağlanan WS hook'u
│   │   └── components/
│   │       ├── MissionMap.tsx    # Leaflet harita & çizim katmanı
│   │       ├── WaypointPanel.tsx # Nokta düzenleme ve validasyon paneli
│   │       ├── TelemetryPanel.tsx# Anlık uçuş verileri paneli
│   │       ├── ControlBar.tsx    # Görev yürütme & acil durum barı
│   │       └── LandingPanel.tsx  # Hedef tespit paneli (opsiyonel)
│   └── src/store/missionStore.test.ts
├── simulation/                 # Opsiyonel: AprilTag ve Gazebo kamera eklentileri
└── flights/                    # Üretilen uçuş CSV kayıtları (.gitignore'da)
```

---

## 🎯 Mimari Kararlar ve Gerekçeleri

1. **FastAPI & pymavlink Seçimi:** Python ekosisteminde MAVLink haberleşmesi standarttır (`pymavlink`). FastAPI'nin asenkron yapısı, hem REST isteklerini hem de 5 Hz WebSocket telemetri akışını tek bir süreçte bloklanmadan sunar.
2. **React-Leaflet 2D Tercihi:** Cesium 3D yüksek GPU/token maliyeti ve karmaşık koordinat dönüşümleri gerektirir. 3 günlük maratonda 2D Leaflet, rota çizimi ve canlı takip için hafif, kararlı ve hızlıdır.
3. **YOLO Eğitimi Neden Kapsam Dışı?** 3 günlük süre zarfında sıfırdan model eğitmek yerine otonom uçuş, MAVLink protokolü ve telemetri mimarisinin hatasız çalışması önceliklendirilmiştir.
4. **AprilTag Tercihi:** Hedef tespitinde derin öğrenme yerine matematiksel olarak kesin, eğitim gerektirmeyen deterministik AprilTag (tag36h11) mimarisi tercih edilmiştir.

---

## 🔮 Gelecek Sürüm Notları (Roadmap v0.2.0+)
- **VisDrone Veri Seti Entegrasyonu:** Tianjin Üniversitesi'nin İHA nesne tespiti veri seti ile arama-kurtarma tespiti.
- **Offboard Otonom Hassas İniş:** AprilTag merkez ofsetleri kullanılarak araca 2 Hz üzerinde velocity/position setpoint gönderilip otonom iniş icrası.
- **Cesium Ion 3D Yükseltisi:** Görev rotalarının 3D arazi ve bina modelleri üzerinde görselleştirilmesi.
- **Çoklu İHA (Swarm) Desteği:** Aynı harita üzerinde birden fazla SYS_ID'ye sahip İHA'nın koordineli kontrolü.
