# A repülőjáratok heti trendjeinek elemzése az EU-ban

# 1. Projekt célja
A cél a heti repülési adatok elemzése és vizualizációja az EU légiforgalmáról. Az elemzés célja:

- Megérteni a repülőjáratok heti eloszlását.
- Azonosítani a csúcsidőszakokat és visszaeséseket.
- Elemezni a 2019-es referenciaadatokhoz képesti helyreállítás mértékét.
- A légitársaságok számára hasznos döntéshozatali információk biztosítása.

# 2. Használati esetek
- Légitársaságok: Optimális ütemezés és erőforrás-allokáció.
- Iparági elemzők: Trendek és helyreállítási mutatók azonosítása.
- Utasok: Tájékoztatás a repülési időszakokról és kapacitások optimalizálása.

# 3. Adatok forrása
- Fájl neve: States.xlsx
- Forrás: Eurocontrol napi forgalom variációs adatai 2024 (https://www.eurocontrol.int/Economics/DailyTrafficVariation-States.html)
- Adatok tartalma: Heti bontású légiforgalmi adatok, 2019-es referenciaértékek, százalékos változások és 7 napos mozgóátlagok.

# 4. Használt eszközök
- Fejlesztői környezet: Google Colab (https://colab.research.google.com/)
- Programozási nyelv: Python

**Könyvtárak:**
- pandas: Adatfeldolgozáshoz
- matplotlib.pyplot: Grafikonok készítéséhez
```python
import pandas as pd  # Adatkezeléshez
import matplotlib.pyplot as plt  # Vizualizációhoz
import numpy as np # Vizualizációhoz
```

# 5. Módszertan
## 1. Adatok előkészítése és tisztítása:
- Az **States.xlsx** fájl importálása és előfeldolgozása.
```python
# Fájl elérési útja
file_path = '/content/States.xlsx'

# Excel betöltése Pandas-szal
data = pd.read_excel(file_path)

# Adatok megjelenítése
data.head()  # Az első 5 sor megjelenítése
```
## 2. Adatok áttekintése
```python
# Az oszlopok nevei
print(data.columns)

# Adatok összegzése
data.info()

# Hiányzó adatok ellenőrzése
print(data.isnull().sum())
```
## 3. Adatok tisztítása (ha szükséges)
- Az előkészítés során eltávolítottuk az üres sorokat és oszlopokat, amelyek nem tartalmaztak hasznos adatokat.
```python
# Üres sorok vagy oszlopok eltávolítása (ha vannak)
data.dropna(axis=0, inplace=True)  # Üres sorok
data.dropna(axis=1, inplace=True)  # Üres oszlopok

# Ellenőrzés után:
data.head()
```
## 4. Heti bontás és aggregáció:
- A napi bontású adatokat heti szintű adatokra összesítettük, a "Day" oszlopot felhasználva.
- A járatok heti trendjeinek elemzése.
- 7 napos mozgóátlag számítása.
- Összehasonlítás 2019-es adatokkal
```python
# Dátum konvertálása datetime formátumra
data['Day'] = pd.to_datetime(data['Day'])

# Heti bontás és numerikus oszlopok összesítése
weekly_data = data.groupby(data['Day'].dt.isocalendar().week).sum(numeric_only=True)

# 7 napos mozgóátlag kiszámítása
weekly_data['7-day Moving Average'] = weekly_data['Flights'].rolling(window=7).mean()

# Százalékos eltérés a 2019-es adatokhoz képest
weekly_data['% vs 2019'] = ((weekly_data['Flights'] - weekly_data['Flights 2019 (Reference)']) / weekly_data['Flights 2019 (Reference)']) * 100

# Gyors ellenőrzés
weekly_data.head()
```

# 6. Eredmények és vizualizáció
## 1. Heti adatok grafikus ábrázolása
**A heti járatok száma és a 7 napos mozgóátlag**
```python
# Heti trendek ábrázolása a mozgóátlaggal
plt.figure(figsize=(12, 6))
plt.plot(weekly_data.index, weekly_data['Flights'], label='Flights (Weekly)', marker='o')
plt.plot(weekly_data.index, weekly_data['7-day Moving Average'], label='7-day Moving Average', linestyle='--')
plt.title('Heti repülőjáratok száma és mozgóátlaga')
plt.xlabel('Hét')
plt.ylabel('Járatok száma')
plt.legend()
plt.grid()
plt.show()
```
![Heti_repulojaratok_szama_es_mozgoatlag](https://github.com/user-attachments/assets/78266ca0-aef2-49ff-a7bd-140929b8ac7a)

A görbe egy évszakos mintázatot mutat:
- Az év elején (1-10. hét körül) alacsony értékek vannak, majd a járatszámok meredeken növekednek a 20. hétig.
- A csúcsérték a 25-30. hét között jelentkezik, ahol a járatok száma 600 ezer körüli.
- Ezután az év végére egy fokozatos, majd meredek csökkenés következik be.

**Százalékos eltérés a 2019-es referenciaévhez képest**
```python
# Százalékos eltérés ábrázolása
plt.figure(figsize=(12, 6))
plt.bar(weekly_data.index, weekly_data['% vs 2019'], color='orange', alpha=0.7)
plt.axhline(0, color='red', linestyle='--', linewidth=1)
plt.title('Százalékos eltérés a 2019-es évhez képest')
plt.xlabel('Hét')
plt.ylabel('Eltérés (%)')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```
![szazalekos_elteres_2019_2024](https://github.com/user-attachments/assets/fdfef78b-5109-42f0-aabe-60c4b47c7d59)

- Az eltérések a legtöbb héten -2% és -6% között mozognak, azonban az év elején és végén nagyobb eltérések is láthatók:
- Az 1-5. hét körül -10% alatti értékek vannak.
- Az év végi hetekben szintén -8% körüli visszaesés tapasztalható.

**NumPy-alapú vizualizáció: Százalékos eltérések eloszlása**
```python
# Százalékos eltérések NumPy használatával
percent_differences = weekly_data['% vs 2019'].to_numpy()

# Histogram ábrázolása
plt.figure(figsize=(10, 6))
plt.hist(percent_differences, bins=20, color='skyblue', edgecolor='black', alpha=0.7)
plt.axvline(np.mean(percent_differences), color='red', linestyle='--', label=f'Átlag: {np.mean(percent_differences):.2f}%')
plt.title('Százalékos eltérések eloszlása (2019-hez képest)')
plt.xlabel('Eltérés (%)')
plt.ylabel('Hetek száma')
plt.legend()
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```
![Elteresek_eloszlasa](https://github.com/user-attachments/assets/1c170aec-340b-4d7d-9117-a2dbd9881d08)

Az ábrán egy hisztogram látható, amely a százalékos eltérések eloszlását mutatja:
- A vörös szaggatott vonal jelzi az átlagos eltérést, amely -3.86% körül van.
- A hisztogramon egyértelműen látszik, hogy minden hét negatív eltérést mutat, tehát 2024-ben eddig nem volt olyan hét, amikor a járatok száma meghaladta volna a 2019-es értékeket.
- A legnagyobb negatív eltérések a -14% körül mozognak, míg a legkisebb eltérések közelítenek a 0%-hoz.


# 7. Neurális háló a repülőjáratok trendjeinek előrejelzésére

## 1. Adatok előkészítése:
**Az adatok normalizálása (pl. min-max scaling)**
- A repülőjáratokkal kapcsolatos adatok jellemzően különböző skálán mozognak, például a járatszámok, az időbeli mozgóátlagok stb. A neurális hálók jobban teljesítenek, ha az inputadatok normalizálva vannak. Az egyik leggyakoribb normalizálási technika a min-max scaling, amely 0 és 1 közé skálázza az adatokat.

**Az adatok szekvenciákra bontása**
- A neurális háló trendek előrejelzéséhez múltbeli adatokat használ fel. Például a múlt 4 heti adatokból megpróbáljuk megjósolni az 5. heti járatszámokat. Ezért az adatokat szekvenciákra kell bontanunk egy sliding window módszerrel.

Először normalizáljuk az adatokat, majd elkészítjük a szekvenciákat egy példában, amely az 'Flights' oszlopot használja. Így készítjük elő a neurális háló bemenetét:
```python
from sklearn.preprocessing import MinMaxScaler

# Csak a releváns oszlop kiválasztása (pl. 'Flights')
flight_data = data[['Flights']]

# 1. Normalizálás (Min-Max Scaling)
scaler = MinMaxScaler(feature_range=(0, 1))
normalized_data = scaler.fit_transform(flight_data)

# 2. Szekvenciák létrehozása
def create_sequences(data, seq_length):
    x, y = [], []
    for i in range(len(data) - seq_length):
        x.append(data[i:i + seq_length])
        y.append(data[i + seq_length])
    return np.array(x), np.array(y)

# Szekvencia hossza (pl. 4 hét)
sequence_length = 4
x, y = create_sequences(normalized_data, sequence_length)

# Kimenetek ellenőrzése
print(f"X shape: {x.shape}")
print(f"Y shape: {y.shape}")

# Egy példa az X-ből és Y-ból
print(f"Sample X: {x[0]}")
print(f"Sample Y: {y[0]}")
```
**output:**
```
X shape: (14696, 4, 1)
Y shape: (14696, 1)
Sample X: [[0.0041445 ]
 [0.00490059]
 [0.00484458]
 [0.00481658]]
Sample Y: [0.00490059]
```

### 1. X shape: (14696, 4, 1):
- Összesen 14,696 szekvencia készült.
- Minden szekvencia 4 időpontból áll (4 hét).
- Minden időpont egyetlen jellemzőt tartalmaz (a Flights értékét).

### 2. Y shape: (14696, 1):
- Minden szekvenciához tartozik egy előrejelzendő célérték.

### 3. Példa szekvencia (X):
- Ez az adott 4 hetes szekvencia normált értékekkel.
- Az első szekvencia alapján a Y érték a következő hét normált értéke.

## 2. Neurális hálózat modell felépítése:
**Neurális háló modell: Egyszerű LSTM**
- LSTM rétegeket használunk, mivel időalapú adatokat dolgozunk fel. A modell egy bemeneti, egy rejtett (LSTM), és egy kimeneti rétegből fog állni.
```python
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import LSTM, Dense

# Modell létrehozása
model = Sequential([
    LSTM(50, activation='relu', input_shape=(sequence_length, 1)),
    Dense(1)  # Kimeneti réteg: egyetlen érték becslése
])

# Modell fordítása
model.compile(optimizer='adam', loss='mean_squared_error')

# Modell összegzése
model.summary()

# Adatok szétosztása tanítási és validációs halmazra
split_index = int(0.8 * len(x))  # 80% tanítás, 20% validáció
x_train, x_val = x[:split_index], x[split_index:]
y_train, y_val = y[:split_index], y[split_index:]

# Modell tanítása
history = model.fit(
    x_train, y_train,
    validation_data=(x_val, y_val),
    epochs=20,  # Iterációk száma
    batch_size=32,  # Minta darabszám iterációnként
    verbose=1
)
```
### 1. Modell definiálása:
- Az LSTM réteg megtanulja a trendeket és mintázatokat a szekvenciákból.
- A kimeneti réteg (Dense) egyetlen számot becsül, ami az előrejelzett érték.

### 2. Tanítási adatok megosztása:
- Az adatok 80%-át tanítjuk, a maradék 20%-ot a validációra használjuk.

### 3. Tanítás:
- Az epochs=20 iterációs lépés során a modell többször frissíti a súlyokat.
- A batch_size=32 biztosítja, hogy egyszerre 32 minta frissítse a súlyokat.

**Output:**
```
Model: "sequential"
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┓
┃ Layer (type)                         ┃ Output Shape                ┃         Param # ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━┩
│ lstm (LSTM)                          │ (None, 50)                  │          10,400 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ dense (Dense)                        │ (None, 1)                   │              51 │
└──────────────────────────────────────┴─────────────────────────────┴─────────────────┘
 Total params: 10,451 (40.82 KB)
 Trainable params: 10,451 (40.82 KB)
 Non-trainable params: 0 (0.00 B)
```
### 1. Modell architektúra
**LSTM réteg (50 neuron): Megtanulja a trendeket és mintázatokat az időalapú adatainkban.**
- Paraméterek száma: 10,400 (ehhez tartoznak a bemeneti-kimeneti kapcsolatok és az LSTM belső állapotai).
**Dense réteg (1 neuron): Egyetlen kimenetet ad, ami a következő hét előrejelzése.**
- Paraméterek száma: 51.

### 2. Tanítás és validáció
**Tanulási folyamat (loss és val_loss):**
- Az Epoch 1 során a loss: 0.00045747, és a validációs val_loss: 0.0023. Ez azt jelenti, hogy kezdetben a modell nagyobb hibával dolgozott.
- Ahogy az epochok során haladunk, a validációs hiba (val_loss) folyamatosan csökken.

**Végső eredmény (20. epoch):**
- loss: 0.000017217 → A modell jól tanult az adatokból.
- val_loss: 0.0011 → A validációs hiba is alacsony, ami azt jelzi, hogy a modell nem túltanult, és képes az adatok általánosítására.

### 3. Mit jelent a "val_loss"?
- A validációs hiba a valós (tesztadatokhoz közeli) adatokon mért hiba. Az alacsony érték azt mutatja, hogy a modell nemcsak a tanító adatokra fókuszál, hanem általánosítható is.

## 3. Vizualizáció:
![Neuralis_halo_vizualizacioja](https://github.com/user-attachments/assets/a1c00662-77d8-4b35-aec0-e03e992da204)
### 1. Előrejelzés pontossága:
- Az előrejelzett értékek (narancssárga vonal) nagyon közel helyezkednek el a valós értékekhez (kék vonal), ami azt mutatja, hogy a modell jól megtanulta az adatok mintázatait.
- Különösen az egyenletes trendeknél látható, hogy a modell szinte tökéletesen követi az adatokat.

### 2. Esetleges eltérések:
- A sűrűbb, komplexebb változásoknál, például az adatcsúcsoknál, van némi eltérés. Ez arra utal, hogy a modell nem minden esetben képes teljesen pontosan előrejelezni a hirtelen növekedéseket vagy eséseket.



# 8. Következtetések
- A repülési trendek elemzése rávilágít a csúcsidőszakokra és az iparág helyreállásának mértékére.
- Az elemzés segíthet a légitársaságoknak optimalizálni az ütemezést, és az utasok számára is hasznos információkat nyújt.
