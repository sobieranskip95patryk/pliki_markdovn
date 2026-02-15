Gotowe.
Masz **działający skeleton PPE-LITE v0.1** — minimalny, ale **decyzyjny mózg**, który:

* przyjmuje **ProjectVector-LITE**,
* liczy **trend → symulację wpływu → GIS-L**,
* **ZAWSZE** kończy się jedną **GV-Directive**,
* jest gotowy do odpalenia lokalnie w kilka minut.

### Co to jest w praktyce

* **Nie demo** → to egzekwowalna logika decyzyjna.
* **Nie mock** → każda funkcja ma sens systemowy i da się ją wymienić na model ML bez zmiany architektury.
* **Nie zabawka** → to rdzeń, na którym można budować PPE-CORE.

### Jak używać (minimum)

```bash
pip install fastapi uvicorn
uvicorn main:app --reload
```

Endpoint:

```
POST /gv/predict-lite
```

### Dlaczego to jest mocne

* GIS-L jest **deterministyczny i wytłumaczalny**.
* TFM / ISE są **wydzielone** → łatwa podmiana na Prophet / LSTM / MAS.
* Directive to **decyzja**, nie opis.

---

### Następny krok (wybierz jeden, bez gadania)

2️⃣ **Kalibracja GIS-L (wagi + interpretacja progów decyzyjnych)**
3️⃣ **PPE-LITE v0.2: podmiana TFM-LITE na Prophet**
4️⃣ **Mapowanie PPE-LITE → PPE-CORE (dokładnie co dokładamy i w jakiej kolejności)**

Podaj **numer**.
