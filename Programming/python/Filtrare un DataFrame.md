---
owner: Daniele Andreis
created: November 08, 2025 – 8:43 AM
updated: 2025-11-08
tags: [python, pandas, dataframe]
---

# 📊 Filtrare un DataFrame in Pandas

Questa guida riassume le operazioni più comuni per filtrare un DataFrame (`df`) in **Pandas**.

---

## 1️⃣ Filtrare per data

Se l’indice del DataFrame è di tipo `DatetimeIndex`, si possono usare le **date come slicing**:

```python
# dal giorno start_date in avanti
df_filtered = df[start_date:]

# tra due date
df_filtered = df["2024-01-01":"2024-06-30"]

```

---

## 2️⃣ Filtrare con condizioni su colonne

```python
# righe in cui la colonna "precip" è maggiore di 0
df_filtered = df[df["precip"] > 0]

# più condizioni (AND con &, OR con |)
df_filtered = df[(df["temp"] > 10) & (df["precip"] == 0)]

```

---

## 3️⃣ Creare e usare una maschera (mask)

Una **mask** è una serie di valori booleani (`True`/`False`) che corrispondono alle righe.

```python
# esempio: tutte le righe in cui "precip" è zero
mask = df["precip"].eq(0)

# applico la mask
df_filtered = df[mask]

```

---

## 4️⃣ Mask con più condizioni

```python
# righe dove "temp" è > 15 E "precip" = 0
mask = (df["temp"] > 15) & (df["precip"] == 0)
df_filtered = df[mask]

```

---

## 5️⃣ Mask su intere righe

Controllare proprietà su tutte le colonne:

```python
# righe in cui tutta la riga = 0
mask = df.eq(0).all(axis=1)

# righe in cui tutti i valori sono uguali (escluse righe tutte NaN)
mask = (df.nunique(axis=1) == 1) & (df.notna().any(axis=1))

df_filtered = df[mask]

```

---

## 6️⃣ Indici delle righe filtrate

Se vuoi solo gli **indici** delle righe:

```python
indices = df[mask].index

```

---

## 7️⃣ Uso di `.loc` e `.iloc`

- **`.loc`**: selezione basata su **etichette** (label).
- **`.iloc`**: selezione basata su **posizioni** (indice numerico).

```python
# con loc → per etichette
df_filtered = df.loc["2024-01-01":"2024-01-31", ["precip", "temp"]]

# con iloc → per posizioni (righe 0-9, colonne 0-1)
df_filtered = df.iloc[0:10, 0:2]

```

---

## 8️⃣ Altri trucchi utili

```python
# query: sintassi SQL-like
df_filtered = df.query("temp > 15 and precip == 0")

# isin: valori contenuti in lista
df_filtered = df[df["station"].isin(["A", "B", "C"])]

```

---

✅ **Tip:** Puoi sempre combinare slicing e mask.

Esempio:

```python
df_filtered = df["2024-01-01":][mask]

```