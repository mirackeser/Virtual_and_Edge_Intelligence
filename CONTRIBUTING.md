# 🤝 Katkı Sağlama ve Ekip Çalışma Kuralları (2 Kişilik Repo Rehberi)

Bu repo 2 kişilik bir ekip tarafından ortaklaşa geliştirilecektir. Çakışmaları önlemek ve temiz bir Git geçmişi sağlamak için aşağıdaki kurallar zorunludur.

---

## 🌿 Dal (Branch) Stratejisi

- `main`: Yalnızca çalışan, test edilmiş ve onaylanmış kodları barındırır. Doğrudan `main` dalına commit atılmaz.
- **Kişi A Dalları:**
  - `feat/backend-docker-env` (A1)
  - `feat/backend-models-validator` (A2, A3)
  - `feat/backend-converter-uploader` (A4, A6)
  - `feat/backend-mavlink-telemetry` (A5)
  - `feat/backend-state-recorder` (A7, A8)
  - `feat/backend-api-routes-ws` (A9)
- **Kişi B Dalları:**
  - `feat/frontend-scaffold-types` (B1)
  - `feat/frontend-store-tdd` (B2)
  - `feat/frontend-api-ws-hook` (B3)
  - `feat/frontend-leaflet-map` (B4, B7)
  - `feat/frontend-panels-control` (B5, B6, B8)
  - `feat/frontend-docker-nginx` (B10)
  - `docs/readme-diagrams` (D)

---

## 📝 Commit Mesajı Formatı (Conventional Commits)

Her commit tek bir amaca hizmet etmelidir:
- `feat(backend): add mission item converter for mavlink`
- `test(backend): add unit tests for waypoint bounds`
- `feat(frontend): implement useTelemetry auto-reconnect hook`
- `fix(mavlink): adjust UDP heartbeat timeout to 5 seconds`
- `docs(api): clarify rejected status reason payload`

---

## 🧪 Test Öncelikli Geliştirme (TDD) Kuralı

1. Önce `tests/test_*.py` veya `*.test.ts` dosyası yazılır.
2. Testin kırmızı (başarısız) olduğu doğrulanır.
3. Kodu çalıştıracak minimum implementasyon yapılır.
4. Test yeşil (başarılı) olduğunda commit atılır.

---

## 🔄 Pull Request (PR) Kontrol Listesi

PR açmadan önce:
- [ ] Backend testleri: `pytest` hatasız geçiyor mu?
- [ ] Frontend testleri: `npm run test` geçiyor mu?
- [ ] Frontend derlemesi: `npm run build` hatasız mı?
- [ ] `API.md` sözleşmesinden sapma var mı?
- [ ] `.gitignore` gereksiz dosyaları (özellikle `flights/*.csv`, `node_modules/`, `__pycache__/`) engelliyor mu?
