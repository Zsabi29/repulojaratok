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
1. Adatok előkészítése:
- Az adatok normalizálása (pl. min-max scaling).
- Az adatok szekvenciákra bontása (pl. 4 heti adatokból a következő hetet jósoljuk).
2. Neurális hálózat modell felépítése:
- Egyszerű Keras vagy TensorFlow alapú modell, pl. egy LSTM vagy Dense rétegekből álló hálózat.
3. Tanítás és kiértékelés:
- A modellt betanítjuk az adatokra, majd validáljuk az eredményeket.
4. Vizualizáció:
- Az előrejelzések és a valós adatok összehasonlítása grafikonokon.

# 8. Következtetések
- A repülési trendek elemzése rávilágít a csúcsidőszakokra és az iparág helyreállásának mértékére.
- Az elemzés segíthet a légitársaságoknak optimalizálni az ütemezést, és az utasok számára is hasznos információkat nyújt.
