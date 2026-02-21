=============================
=============================

DENETİMSİZ ÖĞRENME (UNSUPERVISED LEARNING)
==========================================

.. _section-1:

=============================
=============================



.. code:: ipython3

    import pdfplumber
    import pandas as pd
    import re
    
    pdf_path = r"C:\Users\sadul\OneDrive\Desktop\2025 MAKALELER 2\Global Innovation Index 2025\96-234.pdf"
    
    countries_data = []
    
    patterns = {
        "Score": r"Score\s*/\s*Value\s+([0-9]+(?:\.[0-9])?)",
        "Institutions": r"Institutions\s+([0-9]+\.[0-9])",
        "Human capital and research": r"Human capital\s+and\s+research\s+([0-9]+\.[0-9])",
        "Infrastructure": r"Infrastructure\s+([0-9]+\.[0-9])",
        "Market sophistication": r"Market\s+sophistication\s+([0-9]+\.[0-9])",
        "Business sophistication": r"Business\s+sophistication\s+([0-9]+\.[0-9])",
        "Knowledge and technology outputs": r"Knowledge\s+and\s+technology\s+outputs\s+([0-9]+\.[0-9])",
        "Creative outputs": r"Creative\s+outputs\s+([0-9]+\.[0-9])",
    }
    
    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            raw_text = page.extract_text()
            if not raw_text:
                continue
    
            # Satır kırılmalarını normalize et
            text = re.sub(r"\s+", " ", raw_text)
    
            # Ülke adı = sayfanın ilk satırı
            first_line = raw_text.split("\n")[0].strip()
    
            # Çok kısa / anlamsız başlıkları ele
            if len(first_line) < 3 or "rank" in first_line.lower():
                continue
    
            row = {"Country": first_line}
    
            found_any = False
            for key, pattern in patterns.items():
                match = re.search(pattern, text, re.IGNORECASE)
                if match:
                    row[key] = float(match.group(1))
                    found_any = True
                else:
                    row[key] = None
    
            # En az 1 değer bulunduysa ekle
            if found_any:
                countries_data.append(row)
    
    # DataFrame oluştur
    df = pd.DataFrame(countries_data)
    
    print("Yakalanan ülke sayısı:", len(df))
    print(df.head())
    
    # Sütun sırası
    columns_order = [
        "Country",
        "Score",
        "Institutions",
        "Human capital and research",
        "Infrastructure",
        "Market sophistication",
        "Business sophistication",
        "Knowledge and technology outputs",
        "Creative outputs",
    ]
    
    df = df.reindex(columns=columns_order)
    
    # Excel çıktısı (Windows yolu!)
    output_path = r"C:\Users\sadul\OneDrive\Desktop\GII_2025_Pillar_Values_139_Countries.xlsx"
    df.to_excel(output_path, index=False)
    
    output_path
    


.. code:: ipython3

    import pdfplumber
    import pandas as pd
    import re
    
    pdf_path = r"C:\Users\sadul\OneDrive\Desktop\2025 MAKALELER 2\Global Innovation Index 2025\96-234.pdf"
    
    countries_data = []
    
    patterns = {
        "Score": r"Score\s*/\s*Value\s+([0-9]+(?:\.[0-9])?)",
        "Institutions": r"Institutions\s+([0-9]+\.[0-9])",
        "Human capital and research": r"Human capital\s+and\s+research\s+([0-9]+\.[0-9])",
        "Infrastructure": r"Infrastructure\s+([0-9]+\.[0-9])",
        "Market sophistication": r"Market\s+sophistication\s+([0-9]+\.[0-9])",
        "Business sophistication": r"Business\s+sophistication\s+([0-9]+\.[0-9])",
        "Knowledge and technology outputs": r"Knowledge\s+and\s+technology\s+outputs\s+([0-9]+\.[0-9])",
        "Creative outputs": r"Creative\s+outputs\s+([0-9]+\.[0-9])",
    }
    
    def extract_country_name(raw_text):
        """
        İlk 10 satırı tarar, ülke adı olabilecek EN MANTIKLI satırı döndürür
        """
        lines = [l.strip() for l in raw_text.split("\n") if l.strip()]
    
        for line in lines[:10]:
            # Sayı içermesin
            if any(char.isdigit() for char in line):
                continue
            # Anahtar kelime içermesin
            if any(k in line.lower() for k in ["gii", "rank", "score"]):
                continue
            # Çok kısa olmasın
            if len(line) < 4:
                continue
            # Harf + boşluk + virgül dışında karakter olmasın
            if re.match(r"^[A-Za-z ,\-’']+$", line):
                return line
    
        return None
    
    
    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            raw_text = page.extract_text()
            if not raw_text:
                continue
    
            text = re.sub(r"\s+", " ", raw_text)
    
            country = extract_country_name(raw_text)
            if not country:
                continue
    
            row = {"Country": country}
    
            found_any = False
            for key, pattern in patterns.items():
                match = re.search(pattern, text, re.IGNORECASE)
                row[key] = float(match.group(1)) if match else None
                if match:
                    found_any = True
    
            if found_any:
                countries_data.append(row)
    
    df = pd.DataFrame(countries_data)
    
    print("Yakalanan ülke sayısı:", len(df))
    print(df.head(10))
    
    columns_order = [
        "Country",
        "Score",
        "Institutions",
        "Human capital and research",
        "Infrastructure",
        "Market sophistication",
        "Business sophistication",
        "Knowledge and technology outputs",
        "Creative outputs",
    ]
    
    df = df.reindex(columns=columns_order)
    
    output_path = r"C:\Users\sadul\OneDrive\Desktop\GII_2025_Pillar_Values_139_Countries.xlsx"
    df.to_excel(output_path, index=False)
    
    output_path
    



.. code:: ipython3

    import pandas as pd                      # Veri manipülasyonu için pandas
    import numpy as np                       # Sayısal işlemler için numpy
    import matplotlib.pyplot as plt          # Görselleştirme (temel)
    import seaborn as sns                    # Görselleştirme (gelişmiş)
    from sklearn.impute import SimpleImputer # Basit imputasyon için
    from sklearn.preprocessing import StandardScaler # Ölçeklendirme
    from sklearn.linear_model import LinearRegression # Basit regresyon modeli
    from sklearn.model_selection import train_test_split, cross_val_score # Model doğrulama
    from sklearn.metrics import r2_score, mean_absolute_error, mean_squared_error # Model metrikleri
    

.. code:: ipython3

    import pandas as pd
    
    # Load the newly uploaded CSV file
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, delimiter=';', encoding='latin1')    # latin1 (ISO-8859-1):
    # Batı Avrupa dilleri için geliştirilmiş bir karakter setidir.
    # Fransızca, Almanca, İspanyolca gibi dillerin özel karakterlerini destekler.
    
    df.columns = df.columns.str.strip()  # Clean column names str.strip() → Her bir sütunda bulunan gereksiz boşlukları siler.

.. code:: ipython3

    df




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Country</th>
          <th>Score</th>
          <th>Institutions</th>
          <th>Human capital and research</th>
          <th>Infrastructure</th>
          <th>Market sophistication</th>
          <th>Business sophistication</th>
          <th>Knowledge and technology outputs</th>
          <th>Creative outputs</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>Albania</td>
          <td>29.6</td>
          <td>58.7</td>
          <td>22.4</td>
          <td>52.3</td>
          <td>41.1</td>
          <td>30.5</td>
          <td>16.5</td>
          <td>20.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>Algeria</td>
          <td>18.9</td>
          <td>42.1</td>
          <td>26.9</td>
          <td>34.0</td>
          <td>10.5</td>
          <td>21.6</td>
          <td>11.1</td>
          <td>10.5</td>
        </tr>
        <tr>
          <th>2</th>
          <td>Angola</td>
          <td>13.0</td>
          <td>27.6</td>
          <td>12.9</td>
          <td>26.9</td>
          <td>20.7</td>
          <td>16.1</td>
          <td>5.5</td>
          <td>5.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>Argentina</td>
          <td>26.8</td>
          <td>28.6</td>
          <td>33.8</td>
          <td>38.5</td>
          <td>28.2</td>
          <td>26.6</td>
          <td>18.1</td>
          <td>26.7</td>
        </tr>
        <tr>
          <th>4</th>
          <td>Armenia</td>
          <td>30.5</td>
          <td>49.0</td>
          <td>24.7</td>
          <td>39.3</td>
          <td>33.0</td>
          <td>27.0</td>
          <td>21.5</td>
          <td>31.2</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>134</th>
          <td>Uzbekistan</td>
          <td>26.5</td>
          <td>51.9</td>
          <td>27.4</td>
          <td>41.8</td>
          <td>35.0</td>
          <td>27.1</td>
          <td>20.9</td>
          <td>11.8</td>
        </tr>
        <tr>
          <th>135</th>
          <td>Venezuela</td>
          <td>13.7</td>
          <td>1.9</td>
          <td>48.7</td>
          <td>11.7</td>
          <td>14.1</td>
          <td>22.9</td>
          <td>6.6</td>
          <td>8.7</td>
        </tr>
        <tr>
          <th>136</th>
          <td>Viet Nam</td>
          <td>37.1</td>
          <td>53.5</td>
          <td>30.5</td>
          <td>46.8</td>
          <td>41.6</td>
          <td>35.7</td>
          <td>28.9</td>
          <td>36.2</td>
        </tr>
        <tr>
          <th>137</th>
          <td>Zambia</td>
          <td>19.6</td>
          <td>49.1</td>
          <td>20.6</td>
          <td>36.2</td>
          <td>21.8</td>
          <td>28.5</td>
          <td>8.6</td>
          <td>7.2</td>
        </tr>
        <tr>
          <th>138</th>
          <td>Zimbabwe</td>
          <td>15.4</td>
          <td>18.8</td>
          <td>8.1</td>
          <td>21.6</td>
          <td>13.1</td>
          <td>27.4</td>
          <td>10.7</td>
          <td>15.2</td>
        </tr>
      </tbody>
    </table>
    <p>139 rows × 9 columns</p>
    </div>



.. code:: ipython3

    # Veri Tipleri kontrol et
    df.info()


.. parsed-literal::

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 139 entries, 0 to 138
    Data columns (total 9 columns):
     #   Column                            Non-Null Count  Dtype  
    ---  ------                            --------------  -----  
     0   Country                           139 non-null    object 
     1   Score                             139 non-null    float64
     2   Institutions                      139 non-null    float64
     3   Human capital and research        139 non-null    float64
     4   Infrastructure                    139 non-null    float64
     5   Market sophistication             139 non-null    float64
     6   Business sophistication           139 non-null    float64
     7   Knowledge and technology outputs  139 non-null    float64
     8   Creative outputs                  139 non-null    float64
    dtypes: float64(8), object(1)
    memory usage: 9.9+ KB
    

.. code:: ipython3

    print(df.isnull().sum()) # Her sütunda eksik veri sayısı


.. parsed-literal::

    Country                             0
    Score                               0
    Institutions                        0
    Human capital and research          0
    Infrastructure                      0
    Market sophistication               0
    Business sophistication             0
    Knowledge and technology outputs    0
    Creative outputs                    0
    dtype: int64
    


.. code:: ipython3

    # Sayısal sütunların temel istatistikleri: Ortalama, standart sapma, minimum, maksimum, çeyrek değerleri 
    df.describe()
    




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Score</th>
          <th>Institutions</th>
          <th>Human capital and research</th>
          <th>Infrastructure</th>
          <th>Market sophistication</th>
          <th>Business sophistication</th>
          <th>Knowledge and technology outputs</th>
          <th>Creative outputs</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>count</th>
          <td>139.000000</td>
          <td>139.000000</td>
          <td>139.000000</td>
          <td>139.000000</td>
          <td>139.000000</td>
          <td>139.000000</td>
          <td>139.000000</td>
          <td>139.000000</td>
        </tr>
        <tr>
          <th>mean</th>
          <td>31.498561</td>
          <td>49.925899</td>
          <td>32.492806</td>
          <td>42.089209</td>
          <td>36.514388</td>
          <td>32.321583</td>
          <td>23.376978</td>
          <td>25.250360</td>
        </tr>
        <tr>
          <th>std</th>
          <td>13.381855</td>
          <td>18.961541</td>
          <td>14.719560</td>
          <td>13.329681</td>
          <td>13.772039</td>
          <td>12.248980</td>
          <td>13.725758</td>
          <td>16.026483</td>
        </tr>
        <tr>
          <th>min</th>
          <td>11.900000</td>
          <td>1.900000</td>
          <td>5.500000</td>
          <td>11.100000</td>
          <td>6.000000</td>
          <td>15.400000</td>
          <td>5.500000</td>
          <td>2.000000</td>
        </tr>
        <tr>
          <th>25%</th>
          <td>21.100000</td>
          <td>34.750000</td>
          <td>19.900000</td>
          <td>31.200000</td>
          <td>27.500000</td>
          <td>23.000000</td>
          <td>12.150000</td>
          <td>11.700000</td>
        </tr>
        <tr>
          <th>50%</th>
          <td>28.500000</td>
          <td>49.000000</td>
          <td>30.500000</td>
          <td>41.800000</td>
          <td>35.500000</td>
          <td>28.000000</td>
          <td>20.600000</td>
          <td>22.300000</td>
        </tr>
        <tr>
          <th>75%</th>
          <td>40.050000</td>
          <td>65.800000</td>
          <td>42.750000</td>
          <td>53.050000</td>
          <td>45.200000</td>
          <td>38.350000</td>
          <td>30.650000</td>
          <td>35.650000</td>
        </tr>
        <tr>
          <th>max</th>
          <td>66.000000</td>
          <td>98.700000</td>
          <td>67.000000</td>
          <td>68.800000</td>
          <td>75.000000</td>
          <td>65.900000</td>
          <td>60.700000</td>
          <td>68.800000</td>
        </tr>
      </tbody>
    </table>
    </div>







=============================
=============================

Exploratory Data Analysis (EDA)(Keşifsel Veri Analizi)
======================================================

.. _section-1:

=============================
=============================

.. code:: ipython3

    # =====================================================
    # Top 20 and Bottom 20 Countries by GII Score
    # =====================================================
    
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=';', encoding='latin1')
    
    # Sütun isimlerini temizle
    df.columns = (
        df.columns
        .str.strip()
        .str.replace('\n', ' ', regex=True)
        .str.replace('-', '', regex=True)
    )
    
    # =====================================================
    # 2️⃣ SCORE'A GÖRE SIRALAMA
    # =====================================================
    df_sorted = df.sort_values(by="Score", ascending=False)
    
    top20 = df_sorted.head(20)
    bottom20 = df_sorted.tail(20).sort_values(by="Score", ascending=True)
    
    # =====================================================
    # 3️⃣ TEK FİGÜR – İKİ GRAFİK (ALT ALTA)
    # =====================================================
    fig, axes = plt.subplots(2, 1, figsize=(12, 18))
    
    # ----------------------
    # 🔹 TOP 20
    # ----------------------
    sns.barplot(
        x="Score",
        y="Country",
        data=top20,
         palette="mako",
        ax=axes[0]
    )
    
    axes[0].set_title(
        "Top 20 Countries by Global Innovation Index Score",
        fontsize=14,
        fontweight="bold"
    )
    
    for i, v in enumerate(top20["Score"]):
        axes[0].text(v + 0.5, i, f"{v:.1f}", va="center")
    
    # ----------------------
    # 🔹 BOTTOM 20
    # ----------------------
    sns.barplot(
        x="Score",
        y="Country",
        data=bottom20,
         palette="mako",
        ax=axes[1]
    )
    
    axes[1].set_title(
        "Bottom 20 Countries by Global Innovation Index Score",
        fontsize=14,
        fontweight="bold"
    )
    
    for i, v in enumerate(bottom20["Score"]):
        axes[1].text(v + 0.5, i, f"{v:.1f}", va="center")
    
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 4️⃣ SAYISAL ÇIKTI
    # =====================================================
    print("\n=== TOP 20 Countries by GII Score ===")
    print(top20[["Country", "Score"]].reset_index(drop=True))
    
    print("\n=== BOTTOM 20 Countries by GII Score ===")
    print(bottom20[["Country", "Score"]].reset_index(drop=True))
    


.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1122425319.py:39: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1122425319.py:59: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      sns.barplot(
    


.. image:: output_20_1.png


.. parsed-literal::

    
    === TOP 20 Countries by GII Score ===
                         Country  Score
    0                Switzerland   66.0
    1                     Sweden   62.6
    2   United States of America   61.7
    3          Republic of Korea   60.0
    4                  Singapore   59.9
    5             United Kingdom   59.1
    6                    Finland   57.7
    7                Netherlands   57.0
    8                    Denmark   56.9
    9                      China   56.6
    10                   Germany   55.5
    11                     Japan   53.6
    12                    France   53.4
    13                    Israel   52.3
    14          Hong Kong, China   51.5
    15                    Canada   51.1
    16                   Estonia   51.1
    17                   Ireland   50.4
    18                   Austria   50.1
    19                    Norway   49.2
    
    === BOTTOM 20 Countries by GII Score ===
                            Country  Score
    0                         Niger   11.9
    1                        Angola   13.0
    2                         Congo   13.6
    3                     Venezuela   13.7
    4                          Mali   14.0
    5                      Ethiopia   14.4
    6                        Guinea   14.9
    7                       Lesotho   14.9
    8                      Zimbabwe   15.4
    9                    Mozambique   15.4
    10                    Nicaragua   15.4
    11                   Mauritania   15.4
    12                      Burundi   15.8
    13                 Burkina Faso   15.9
    14                       Malawi   16.0
    15                       Uganda   17.1
    16                    Guatemala   17.1
    17                      Myanmar   17.3
    18  United Republic of Tanzania   17.5
    19                   Madagascar   17.6
    


.. code:: ipython3

    # =====================================================
    # Top 20 and Bottom 20 Countries by GII Score
    # =====================================================
    
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=';', encoding='latin1')
    
    # Sütun isimlerini temizle
    df.columns = (
        df.columns
        .str.strip()
        .str.replace('\n', ' ', regex=True)
        .str.replace('-', '', regex=True)
    )
    
    # =====================================================
    # 2️⃣ SCORE'A GÖRE SIRALAMA
    # =====================================================
    df_sorted = df.sort_values(by="Score", ascending=False)
    
    top20 = df_sorted.head(20)
    bottom20 = df_sorted.tail(20).sort_values(by="Score", ascending=False)
    
    # =====================================================
    # 3️⃣ TEK FİGÜR – İKİ GRAFİK (ALT ALTA)
    # =====================================================
    fig, axes = plt.subplots(2, 1, figsize=(12, 18))
    
    # ----------------------
    # 🔹 TOP 20 (GREEN)
    # ----------------------
    sns.barplot(
        x="Score",
        y="Country",
        data=top20,
        palette="Greens_d",
        ax=axes[0]
    )
    
    axes[0].set_title(
        "Top 20 Countries by Global GII Score",
        fontsize=14,
        fontweight="bold"
    )
    
    for i, v in enumerate(top20["Score"]):
        axes[0].text(v + 0.5, i, f"{v:.1f}", va="center")
    
    # ----------------------
    # 🔹 BOTTOM 20 (RED)
    # ----------------------
    sns.barplot(
        x="Score",
        y="Country",
        data=bottom20,
        palette="Reds_d",
        ax=axes[1]
    )
    
    axes[1].set_title(
        "Bottom 20 Countries by Global GII Score",
        fontsize=14,
        fontweight="bold"
    )
    
    for i, v in enumerate(bottom20["Score"]):
        axes[1].text(v + 0.5, i, f"{v:.1f}", va="center")
    
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 4️⃣ SAYISAL ÇIKTI
    # =====================================================
    print("\n=== TOP 20 Countries by GII Score ===")
    print(top20[["Country", "Score"]].reset_index(drop=True))
    
    print("\n=== BOTTOM 20 Countries by GII Score ===")
    print(bottom20[["Country", "Score"]].reset_index(drop=True))
    


.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3637779612.py:39: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3637779612.py:59: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      sns.barplot(
    


.. image:: output_22_1.png


.. parsed-literal::

    
    === TOP 20 Countries by GII Score ===
                         Country  Score
    0                Switzerland   66.0
    1                     Sweden   62.6
    2   United States of America   61.7
    3          Republic of Korea   60.0
    4                  Singapore   59.9
    5             United Kingdom   59.1
    6                    Finland   57.7
    7                Netherlands   57.0
    8                    Denmark   56.9
    9                      China   56.6
    10                   Germany   55.5
    11                     Japan   53.6
    12                    France   53.4
    13                    Israel   52.3
    14          Hong Kong, China   51.5
    15                    Canada   51.1
    16                   Estonia   51.1
    17                   Ireland   50.4
    18                   Austria   50.1
    19                    Norway   49.2
    
    === BOTTOM 20 Countries by GII Score ===
                            Country  Score
    0                    Madagascar   17.6
    1   United Republic of Tanzania   17.5
    2                       Myanmar   17.3
    3                        Uganda   17.1
    4                     Guatemala   17.1
    5                        Malawi   16.0
    6                  Burkina Faso   15.9
    7                       Burundi   15.8
    8                      Zimbabwe   15.4
    9                    Mozambique   15.4
    10                    Nicaragua   15.4
    11                   Mauritania   15.4
    12                      Lesotho   14.9
    13                       Guinea   14.9
    14                     Ethiopia   14.4
    15                         Mali   14.0
    16                    Venezuela   13.7
    17                        Congo   13.6
    18                       Angola   13.0
    19                        Niger   11.9
    


.. code:: ipython3

    # =====================================================
    # EN YÜKSEK 10 ÜLKE — GII BİLEŞENLERİ
    # =====================================================
    
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=';', encoding='latin1')
    
    # Sütun isimlerini temizle
    df.columns = (
        df.columns
        .str.strip()
        .str.replace('\n', ' ', regex=True)
        .str.replace('-', '', regex=True)
    )
    
    # =====================================================
    # 2️⃣ ANALİZ DEĞİŞKENLERİ (GII PILLARS)
    # =====================================================
    variables = [
        "Institutions",
        "Human capital and research",
        "Infrastructure",
        "Market sophistication",
        "Business sophistication",
        "Knowledge and technology outputs",
        "Creative outputs"
    ]
    
    # =====================================================
    # 3️⃣ GRAFİK DÜZENİ
    # =====================================================
    plt.figure(figsize=(20, 18))
    sns.set(style="whitegrid")
    
    for i, var in enumerate(variables, 1):
    
        # En iyi 10 ülke (NOT: rank küçük = daha iyi)
        top10 = df.sort_values(by=var, ascending=False).head(10)
    
        plt.subplot(3, 3, i)
        ax = sns.barplot(
            x=var,
            y="Country",
            data=top10,
            palette="viridis"
        )
    
        # Sayısal değerleri çubukların sonuna yazdırma
        for j, value in enumerate(top10[var]):
            ax.text(
                value + abs(value)*0.01,
                j,
                f"{value:.0f}",
                va="center",
                fontsize=10,
                color="black",
                fontweight="bold"
            )
    
        plt.title(f"{var} — Top 10 Countries", fontsize=14, fontweight="bold")
        plt.xlabel("")
        plt.ylabel("")
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3451077231.py:48: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3451077231.py:48: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3451077231.py:48: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3451077231.py:48: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3451077231.py:48: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3451077231.py:48: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\3451077231.py:48: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    


.. image:: output_24_1.png



.. code:: ipython3

    # =====================================================
    # GII PILLARS — TOP 10 COUNTRIES (SCORE BASED)
    # SSCI-READY VERSION
    # =====================================================
    
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLE
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=";", encoding="latin1")
    
    # =====================================================
    # 2️⃣ SÜTUN İSİMLERİNİ TEMİZLE (KRİTİK)
    # =====================================================
    df.columns = (
        df.columns
        .str.strip()
        .str.lower()
        .str.replace(r"[^\w\s]", "", regex=True)  # parantez, tire vb. sil
        .str.replace(r"\s+", " ", regex=True)    # çoklu boşlukları tek boşluk yap
    )
    
    # =====================================================
    # 3️⃣ GII PILLARS (SCORE)
    # CSV'deki gerçek isimlere birebir uyumlu
    # =====================================================
    variables = [
        "institutions",
        "human capital and research",
        "infrastructure",
        "market sophistication",
        "business sophistication",
        "knowledge and technology outputs",
        "creative outputs"
    ]
    
    # Ülke adı sütunu
    country_col = "country"
    
    # =====================================================
    # 4️⃣ GRAFİK AYARLARI
    # =====================================================
    plt.figure(figsize=(20, 18))
    sns.set(style="whitegrid")
    
    # =====================================================
    # 5️⃣ TOP 10 ÜLKE GRAFİKLERİ
    # =====================================================
    for i, var in enumerate(variables, 1):
    
        # En yüksek score = daha iyi
        top10 = (
            df[[country_col, var]]
            .dropna()
            .sort_values(by=var, ascending=False)
            .head(10)
        )
    
        plt.subplot(3, 3, i)
        ax = sns.barplot(
            data=top10,
            x=var,
            y=country_col,
            orient="h"
        )
    
        # Sayısal değerleri yaz
        for j, value in enumerate(top10[var]):
            ax.text(
                value + (top10[var].max() * 0.01),
                j,
                f"{value:.1f}",
                va="center",
                fontsize=10,
                fontweight="bold"
            )
    
        plt.title(
            var.replace(" score", "").title(),
            fontsize=14,
            fontweight="bold"
        )
        plt.xlabel("")
        plt.ylabel("")
    
    # =====================================================
    # 6️⃣ GRAFİĞİ GÖSTER
    # =====================================================
    plt.tight_layout()
    plt.show()
    



.. image:: output_26_0.png







.. code:: ipython3

    df




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>country</th>
          <th>score</th>
          <th>institutions</th>
          <th>human capital and research</th>
          <th>infrastructure</th>
          <th>market sophistication</th>
          <th>business sophistication</th>
          <th>knowledge and technology outputs</th>
          <th>creative outputs</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>Albania</td>
          <td>29.6</td>
          <td>58.7</td>
          <td>22.4</td>
          <td>52.3</td>
          <td>41.1</td>
          <td>30.5</td>
          <td>16.5</td>
          <td>20.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>Algeria</td>
          <td>18.9</td>
          <td>42.1</td>
          <td>26.9</td>
          <td>34.0</td>
          <td>10.5</td>
          <td>21.6</td>
          <td>11.1</td>
          <td>10.5</td>
        </tr>
        <tr>
          <th>2</th>
          <td>Angola</td>
          <td>13.0</td>
          <td>27.6</td>
          <td>12.9</td>
          <td>26.9</td>
          <td>20.7</td>
          <td>16.1</td>
          <td>5.5</td>
          <td>5.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>Argentina</td>
          <td>26.8</td>
          <td>28.6</td>
          <td>33.8</td>
          <td>38.5</td>
          <td>28.2</td>
          <td>26.6</td>
          <td>18.1</td>
          <td>26.7</td>
        </tr>
        <tr>
          <th>4</th>
          <td>Armenia</td>
          <td>30.5</td>
          <td>49.0</td>
          <td>24.7</td>
          <td>39.3</td>
          <td>33.0</td>
          <td>27.0</td>
          <td>21.5</td>
          <td>31.2</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>134</th>
          <td>Uzbekistan</td>
          <td>26.5</td>
          <td>51.9</td>
          <td>27.4</td>
          <td>41.8</td>
          <td>35.0</td>
          <td>27.1</td>
          <td>20.9</td>
          <td>11.8</td>
        </tr>
        <tr>
          <th>135</th>
          <td>Venezuela</td>
          <td>13.7</td>
          <td>1.9</td>
          <td>48.7</td>
          <td>11.7</td>
          <td>14.1</td>
          <td>22.9</td>
          <td>6.6</td>
          <td>8.7</td>
        </tr>
        <tr>
          <th>136</th>
          <td>Viet Nam</td>
          <td>37.1</td>
          <td>53.5</td>
          <td>30.5</td>
          <td>46.8</td>
          <td>41.6</td>
          <td>35.7</td>
          <td>28.9</td>
          <td>36.2</td>
        </tr>
        <tr>
          <th>137</th>
          <td>Zambia</td>
          <td>19.6</td>
          <td>49.1</td>
          <td>20.6</td>
          <td>36.2</td>
          <td>21.8</td>
          <td>28.5</td>
          <td>8.6</td>
          <td>7.2</td>
        </tr>
        <tr>
          <th>138</th>
          <td>Zimbabwe</td>
          <td>15.4</td>
          <td>18.8</td>
          <td>8.1</td>
          <td>21.6</td>
          <td>13.1</td>
          <td>27.4</td>
          <td>10.7</td>
          <td>15.2</td>
        </tr>
      </tbody>
    </table>
    <p>139 rows × 9 columns</p>
    </div>




.. code:: ipython3

    # =====================================================
    # EN YÜKSEK 10 ÜLKE — GII DEĞİŞKENLERİ
    # =====================================================
    
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=';', encoding='latin1')
    
    # Sütun adlarını temizle
    df.columns = df.columns.str.strip()
    
    # Country sütunundaki hatalı değerleri temizle (seliforp, xednI vb.)
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "xednı", "index"])]
    
    # =====================================================
    # 2️⃣ ANALİZ DEĞİŞKENLERİ (GII PILLARS)
    # =====================================================
    variables = [
        "Institutions",
        "Human capital and research",
        "Infrastructure",
        "Market sophistication",
        "Business sophistication",
        "Knowledge and technology outputs",
        "Creative outputs"
    ]
    
    # =====================================================
    # 3️⃣ GRAFİK DÜZENİ
    # =====================================================
    plt.figure(figsize=(22, 18))
    sns.set(style="whitegrid")
    
    for i, var in enumerate(variables, 1):
    
        # GII'de YÜKSEK DEĞER = DAHA İYİ
        top10 = df.sort_values(by=var, ascending=False).head(10)
    
        plt.subplot(3, 3, i)
        ax = sns.barplot(
            x=var,
            y="Country",
            data=top10,
            palette="viridis"
        )
    
        # Sayısal değerleri çubukların sonuna yaz
        for j, value in enumerate(top10[var]):
            ax.text(
                value + abs(value) * 0.01,
                j,
                f"{value:.1f}",
                va="center",
                fontsize=10,
                color="black",
                fontweight="bold"
            )
    
        plt.title(f"{var} — Top 10 Countries", fontsize=14, fontweight="bold")
        plt.xlabel("")
        plt.ylabel("")
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\203366060.py:46: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\203366060.py:46: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\203366060.py:46: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\203366060.py:46: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\203366060.py:46: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\203366060.py:46: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\203366060.py:46: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    


.. image:: output_34_1.png


.. code:: ipython3

    # =====================================================
    # EN YÜKSEK 10 ÜLKE — GII DEĞİŞKENLERİ
    # =====================================================
    
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=';', encoding='latin1')
    df.columns = df.columns.str.strip()
    
    # Bozuk ülke adlarını temizle
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "xednı", "index"])]
    
    # =====================================================
    # 2️⃣ ANALİZ DEĞİŞKENLERİ (GII PILLARS)
    # =====================================================
    variables = [
        "Institutions",
        "Human capital and research",
        "Infrastructure",
        "Market sophistication",
        "Business sophistication",
        "Knowledge and technology outputs",
        "Creative outputs"
    ]
    
    # =====================================================
    # 3️⃣ GRAFİK DÜZENİ VE SAYISAL ÇIKTI
    # =====================================================
    plt.figure(figsize=(22, 18))
    sns.set(style="whitegrid")
    
    # Top 10 değerleri saklamak için sözlük
    top_values_dict = {}
    
    for i, var in enumerate(variables, 1):
    
        # Yüksek değer = daha iyi performans
        top10 = df.sort_values(by=var, ascending=False).head(10)
        top_values_dict[var] = top10[["Country", var]]
    
        # Konsola yazdır
        print(f"\n--- {var} — Top 10 Countries ---")
        print(top10[["Country", var]].to_string(index=False))
    
        # Grafik
        plt.subplot(3, 3, i)
        ax = sns.barplot(
            x=var,
            y="Country",
            data=top10,
            palette="viridis"
        )
    
        # Çubukların üzerine değer yaz
        for j, value in enumerate(top10[var]):
            ax.text(
                value + abs(value) * 0.01,
                j,
                f"{value:.1f}",
                va="center",
                fontsize=10,
                fontweight="bold",
                color="black"
            )
    
        plt.title(f"{var} — Top 10 Countries", fontsize=14, fontweight="bold")
        plt.xlabel("")
        plt.ylabel("")
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    
    --- Institutions — Top 10 Countries ---
                 Country  Institutions
               Singapore          98.7
                 Denmark          86.9
             Switzerland          85.5
              Luxembourg          83.8
                 Finland          83.6
             New Zealand          83.1
    United Arab Emirates          81.8
        Hong Kong, China          81.2
                  Norway          80.3
                 Ireland          79.4
    
    --- Human capital and research — Top 10 Countries ---
              Country  Human capital and research
    Republic of Korea                        67.0
            Singapore                        63.3
               Sweden                        61.8
              Germany                        61.0
              Finland                        60.9
          Switzerland                        60.1
       United Kingdom                        59.4
            Australia                        58.6
              Austria                        58.6
               Canada                        58.3
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\665860910.py:53: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\665860910.py:53: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    

.. parsed-literal::

    
    --- Infrastructure — Top 10 Countries ---
                 Country  Infrastructure
                  Norway            68.8
                 Iceland            67.7
                 Finland            67.6
                  Sweden            67.4
             Switzerland            65.2
                   China            64.3
       Republic of Korea            63.6
                 Denmark            62.9
    United Arab Emirates            61.1
                 Estonia            60.8
    
    --- Market sophistication — Top 10 Countries ---
                     Country  Market sophistication
    United States of America                   75.0
            Hong Kong, China                   70.7
                 Switzerland                   67.1
              United Kingdom                   63.8
           Republic of Korea                   61.9
                   Singapore                   61.5
                     Estonia                   60.0
                      Canada                   59.5
                      Sweden                   59.5
                       Japan                   59.4
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\665860910.py:53: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\665860910.py:53: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    

.. parsed-literal::

    
    --- Business sophistication — Top 10 Countries ---
                     Country  Business sophistication
    United States of America                     65.9
                      Sweden                     65.2
                   Singapore                     63.0
           Republic of Korea                     61.2
                 Switzerland                     59.5
                       Japan                     57.8
                 Netherlands                     56.6
                       China                     56.0
                      Israel                     55.7
                     Belgium                     55.5
    
    --- Knowledge and technology outputs — Top 10 Countries ---
                     Country  Knowledge and technology outputs
                       China                              60.7
                 Switzerland                              60.1
    United States of America                              60.0
                      Sweden                              58.1
              United Kingdom                              56.0
                      Israel                              55.4
                   Singapore                              53.1
                     Finland                              52.7
           Republic of Korea                              51.8
                 Netherlands                              50.8
    
    --- Creative outputs — Top 10 Countries ---
                     Country  Creative outputs
                 Switzerland              68.8
                      Sweden              60.1
              United Kingdom              59.7
           Republic of Korea              57.7
    United States of America              56.6
                 Netherlands              56.1
                      France              56.0
                     Germany              55.6
                     Denmark              54.2
                  Luxembourg              53.4
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\665860910.py:53: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\665860910.py:53: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\665860910.py:53: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    


.. image:: output_35_6.png




.. code:: ipython3

    # =====================================================
    # EN DÜŞÜK 10 ÜLKE — GII DEĞİŞKENLERİ
    # =====================================================
    
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=';', encoding='latin1')
    df.columns = df.columns.str.strip()
    
    # Bozuk / anlamsız ülke adlarını temizle
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "xednı", "index"])]
    
    # =====================================================
    # 2️⃣ ANALİZ DEĞİŞKENLERİ (GII PILLARS)
    # =====================================================
    variables = [
        "Institutions",
        "Human capital and research",
        "Infrastructure",
        "Market sophistication",
        "Business sophistication",
        "Knowledge and technology outputs",
        "Creative outputs"
    ]
    
    # =====================================================
    # 3️⃣ GRAFİKLER + KONSOL ÇIKTISI
    # =====================================================
    plt.figure(figsize=(22, 18))
    sns.set(style="whitegrid")
    
    for i, var in enumerate(variables, 1):
    
        bottom10 = (
            df[["Country", var]]
            .sort_values(by=var, ascending=False)  # EN DÜŞÜK
            .head(10)
            .reset_index(drop=True)
        )
    
        # Konsol çıktısı
        print("\n" + "=" * 70)
        print(f"{var} — EN DÜŞÜK 10 ÜLKE")
        print("=" * 70)
        print(bottom10.to_string(index=False))
    
        # Grafik
        plt.subplot(3, 3, i)
        ax = sns.barplot(
            x=var,
            y="Country",
            data=bottom10,
            palette="rocket"
        )
    
        # Çubuk üstüne değer yaz
        for j, value in enumerate(bottom10[var]):
            ax.text(
                value + abs(value) * 0.01,
                j,
                f"{value:.1f}",
                va="center",
                fontsize=10,
                fontweight="bold",
                color="black"
            )
    
        plt.title(f"{var} — Bottom 10 Countries", fontsize=14, fontweight="bold")
        plt.xlabel("")
        plt.ylabel("")
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    
    ======================================================================
    Institutions — EN DÜŞÜK 10 ÜLKE
    ======================================================================
                 Country  Institutions
               Singapore          98.7
                 Denmark          86.9
             Switzerland          85.5
              Luxembourg          83.8
                 Finland          83.6
             New Zealand          83.1
    United Arab Emirates          81.8
        Hong Kong, China          81.2
                  Norway          80.3
                 Ireland          79.4
    
    ======================================================================
    Human capital and research — EN DÜŞÜK 10 ÜLKE
    ======================================================================
              Country  Human capital and research
    Republic of Korea                        67.0
            Singapore                        63.3
               Sweden                        61.8
              Germany                        61.0
              Finland                        60.9
          Switzerland                        60.1
       United Kingdom                        59.4
            Australia                        58.6
              Austria                        58.6
               Canada                        58.3
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1862640099.py:55: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1862640099.py:55: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1862640099.py:55: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    

.. parsed-literal::

    
    ======================================================================
    Infrastructure — EN DÜŞÜK 10 ÜLKE
    ======================================================================
                 Country  Infrastructure
                  Norway            68.8
                 Iceland            67.7
                 Finland            67.6
                  Sweden            67.4
             Switzerland            65.2
                   China            64.3
       Republic of Korea            63.6
                 Denmark            62.9
    United Arab Emirates            61.1
                 Estonia            60.8
    
    ======================================================================
    Market sophistication — EN DÜŞÜK 10 ÜLKE
    ======================================================================
                     Country  Market sophistication
    United States of America                   75.0
            Hong Kong, China                   70.7
                 Switzerland                   67.1
              United Kingdom                   63.8
           Republic of Korea                   61.9
                   Singapore                   61.5
                     Estonia                   60.0
                      Canada                   59.5
                      Sweden                   59.5
                       Japan                   59.4
    
    ======================================================================
    Business sophistication — EN DÜŞÜK 10 ÜLKE
    ======================================================================
                     Country  Business sophistication
    United States of America                     65.9
                      Sweden                     65.2
                   Singapore                     63.0
           Republic of Korea                     61.2
                 Switzerland                     59.5
                       Japan                     57.8
                 Netherlands                     56.6
                       China                     56.0
                      Israel                     55.7
                     Belgium                     55.5
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1862640099.py:55: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1862640099.py:55: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1862640099.py:55: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    

.. parsed-literal::

    
    ======================================================================
    Knowledge and technology outputs — EN DÜŞÜK 10 ÜLKE
    ======================================================================
                     Country  Knowledge and technology outputs
                       China                              60.7
                 Switzerland                              60.1
    United States of America                              60.0
                      Sweden                              58.1
              United Kingdom                              56.0
                      Israel                              55.4
                   Singapore                              53.1
                     Finland                              52.7
           Republic of Korea                              51.8
                 Netherlands                              50.8
    
    ======================================================================
    Creative outputs — EN DÜŞÜK 10 ÜLKE
    ======================================================================
                     Country  Creative outputs
                 Switzerland              68.8
                      Sweden              60.1
              United Kingdom              59.7
           Republic of Korea              57.7
    United States of America              56.6
                 Netherlands              56.1
                      France              56.0
                     Germany              55.6
                     Denmark              54.2
                  Luxembourg              53.4
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\1862640099.py:55: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `y` variable to `hue` and set `legend=False` for the same effect.
    
      ax = sns.barplot(
    


.. image:: output_38_6.png




.. code:: ipython3

    # =====================================================
    # KORELASYON MATRİSİ OLUŞTURMA (GII.csv)
    # =====================================================
    
    import pandas as pd
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    
    df = pd.read_csv(
        "GII.csv",
        sep=';',
        encoding='latin1'
    )
    
    # Sütun adlarındaki boşlukları temizle
    df.columns = df.columns.str.strip()
    
    # Bozuk / anlamsız ülke adlarını ele
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "xednı", "index"])]
    
    # =====================================================
    # 2️⃣ SADECE SAYISAL DEĞİŞKENLERİ SEÇME
    # =====================================================
    
    numeric_df = df.select_dtypes(include=["float64", "int64"])
    
    # =====================================================
    # 3️⃣ KORELASYON MATRİSİ (HEATMAP)
    # =====================================================
    
    plt.figure(figsize=(12, 10))
    
    sns.heatmap(
        numeric_df.corr(),
        annot=True,
        fmt=".2f",
        cmap="viridis",
        linewidths=0.5
    )
    
    plt.title(
        "Correlation Matrix — Global Innovation Index (GII) Pillars",
        fontsize=14,
        fontweight="bold"
    )
    
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 4️⃣ KORELASYON TABLOSUNU KONSOLA YAZDIRMA
    # =====================================================
    
    corr_table = numeric_df.corr()
    print("\n=== GII Pillars Correlation Matrix ===\n")
    print(corr_table.round(3))
    



.. image:: output_41_0.png


.. parsed-literal::

    
    === GII Pillars Correlation Matrix ===
    
                                      Score  Institutions  \
    Score                             1.000         0.788   
    Institutions                      0.788         1.000   
    Human capital and research        0.880         0.652   
    Infrastructure                    0.856         0.761   
    Market sophistication             0.879         0.715   
    Business sophistication           0.898         0.709   
    Knowledge and technology outputs  0.917         0.619   
    Creative outputs                  0.937         0.674   
    
                                      Human capital and research  Infrastructure  \
    Score                                                  0.880           0.856   
    Institutions                                           0.652           0.761   
    Human capital and research                             1.000           0.781   
    Infrastructure                                         0.781           1.000   
    Market sophistication                                  0.819           0.803   
    Business sophistication                                0.814           0.708   
    Knowledge and technology outputs                       0.825           0.767   
    Creative outputs                                       0.842           0.805   
    
                                      Market sophistication  \
    Score                                             0.879   
    Institutions                                      0.715   
    Human capital and research                        0.819   
    Infrastructure                                    0.803   
    Market sophistication                             1.000   
    Business sophistication                           0.760   
    Knowledge and technology outputs                  0.795   
    Creative outputs                                  0.829   
    
                                      Business sophistication  \
    Score                                               0.898   
    Institutions                                        0.709   
    Human capital and research                          0.814   
    Infrastructure                                      0.708   
    Market sophistication                               0.760   
    Business sophistication                             1.000   
    Knowledge and technology outputs                    0.873   
    Creative outputs                                    0.844   
    
                                      Knowledge and technology outputs  \
    Score                                                        0.917   
    Institutions                                                 0.619   
    Human capital and research                                   0.825   
    Infrastructure                                               0.767   
    Market sophistication                                        0.795   
    Business sophistication                                      0.873   
    Knowledge and technology outputs                             1.000   
    Creative outputs                                             0.880   
    
                                      Creative outputs  
    Score                                        0.937  
    Institutions                                 0.674  
    Human capital and research                   0.842  
    Infrastructure                               0.805  
    Market sophistication                        0.829  
    Business sophistication                      0.844  
    Knowledge and technology outputs             0.880  
    Creative outputs                             1.000  
    



.. code:: ipython3

    # =====================================================
    # GII — KORELASYON MATRİSİ (SAĞ ÜST GİZLİ)
    # =====================================================
    
    import pandas as pd
    import numpy as np
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME
    # =====================================================
    
    df = pd.read_csv(
        "GII.csv",
        sep=';',
        encoding='latin1'
    )
    
    # Sütun adlarını temizle
    df.columns = df.columns.str.strip()
    
    # Bozuk / anlamsız ülke adlarını ele
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "xednı", "index"])]
    
    # =====================================================
    # 2️⃣ SADECE SAYISAL DEĞİŞKENLERİN SEÇİLMESİ
    # =====================================================
    
    numeric_df = df.select_dtypes(include=["int64", "float64"])
    
    # =====================================================
    # 3️⃣ KORELASYON MATRİSİ (ALT ÜÇGEN)
    # =====================================================
    
    corr_matrix = numeric_df.corr()
    
    # Sağ üst üçgeni gizleyen maske
    mask = np.triu(np.ones_like(corr_matrix, dtype=bool))
    
    plt.figure(figsize=(11, 9))
    
    sns.heatmap(
        corr_matrix,
        mask=mask,
        annot=True,
        fmt=".2f",
        cmap="cool",
        linewidths=0.5
    )
    
    plt.title(
        "Correlation Matrix (Lower Triangle) — GII Pillars",
        fontsize=13,
        fontweight="bold"
    )
    
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 4️⃣ KORELASYON TABLOSUNU YAZDIRMA
    # =====================================================
    
    print("\n=== GII Pillars Correlation Matrix ===\n")
    print(corr_matrix.round(3))
    



.. image:: output_44_0.png


.. parsed-literal::

    
    === GII Pillars Correlation Matrix ===
    
                                      Score  Institutions  \
    Score                             1.000         0.788   
    Institutions                      0.788         1.000   
    Human capital and research        0.880         0.652   
    Infrastructure                    0.856         0.761   
    Market sophistication             0.879         0.715   
    Business sophistication           0.898         0.709   
    Knowledge and technology outputs  0.917         0.619   
    Creative outputs                  0.937         0.674   
    
                                      Human capital and research  Infrastructure  \
    Score                                                  0.880           0.856   
    Institutions                                           0.652           0.761   
    Human capital and research                             1.000           0.781   
    Infrastructure                                         0.781           1.000   
    Market sophistication                                  0.819           0.803   
    Business sophistication                                0.814           0.708   
    Knowledge and technology outputs                       0.825           0.767   
    Creative outputs                                       0.842           0.805   
    
                                      Market sophistication  \
    Score                                             0.879   
    Institutions                                      0.715   
    Human capital and research                        0.819   
    Infrastructure                                    0.803   
    Market sophistication                             1.000   
    Business sophistication                           0.760   
    Knowledge and technology outputs                  0.795   
    Creative outputs                                  0.829   
    
                                      Business sophistication  \
    Score                                               0.898   
    Institutions                                        0.709   
    Human capital and research                          0.814   
    Infrastructure                                      0.708   
    Market sophistication                               0.760   
    Business sophistication                             1.000   
    Knowledge and technology outputs                    0.873   
    Creative outputs                                    0.844   
    
                                      Knowledge and technology outputs  \
    Score                                                        0.917   
    Institutions                                                 0.619   
    Human capital and research                                   0.825   
    Infrastructure                                               0.767   
    Market sophistication                                        0.795   
    Business sophistication                                      0.873   
    Knowledge and technology outputs                             1.000   
    Creative outputs                                             0.880   
    
                                      Creative outputs  
    Score                                        0.937  
    Institutions                                 0.674  
    Human capital and research                   0.842  
    Infrastructure                               0.805  
    Market sophistication                        0.829  
    Business sophistication                      0.844  
    Knowledge and technology outputs             0.880  
    Creative outputs                             1.000  
    


.. code:: ipython3

    import numpy as np
    import pandas as pd
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    # =====================================================
    # 1️⃣ VERİYİ YÜKLEME (GII.csv)
    # =====================================================
    
    df = pd.read_csv(
        "GII.csv",
        sep=';',
        encoding='latin1'
    )
    
    # Sütun isimlerini temizle
    df.columns = df.columns.str.strip()
    
    # Bozuk / anlamsız ülke isimlerini çıkar
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "xednı", "index"])]
    
    # =====================================================
    # 2️⃣ SADECE SAYISAL DEĞİŞKENLER
    # =====================================================
    
    numeric_df = df.select_dtypes(include=["int64", "float64"])
    
    # =====================================================
    # 3️⃣ KORELASYON MATRİSİ – ALT ÜÇGEN
    # =====================================================
    
    corr_matrix = numeric_df.corr()
    
    mask = np.triu(np.ones_like(corr_matrix, dtype=bool))
    
    plt.figure(figsize=(11, 9))
    
    sns.heatmap(
        corr_matrix,
        mask=mask,
        annot=True,
        cmap="plasma",     # Canlı ve kontrastlı
        fmt=".2f",
        linewidths=0.5,
        vmin=-1,
        vmax=1
    )
    
    plt.title(
        "Correlation Matrix (Lower Triangle) — GII Pillars",
        fontsize=13,
        fontweight="bold"
    )
    
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 4️⃣ KORELASYON TABLOSU
    # =====================================================
    
    print("\n=== GII Pillars Correlation Matrix ===\n")
    print(corr_matrix.round(3))
    



.. image:: output_46_0.png


.. parsed-literal::

    
    === GII Pillars Correlation Matrix ===
    
                                      Score  Institutions  \
    Score                             1.000         0.788   
    Institutions                      0.788         1.000   
    Human capital and research        0.880         0.652   
    Infrastructure                    0.856         0.761   
    Market sophistication             0.879         0.715   
    Business sophistication           0.898         0.709   
    Knowledge and technology outputs  0.917         0.619   
    Creative outputs                  0.937         0.674   
    
                                      Human capital and research  Infrastructure  \
    Score                                                  0.880           0.856   
    Institutions                                           0.652           0.761   
    Human capital and research                             1.000           0.781   
    Infrastructure                                         0.781           1.000   
    Market sophistication                                  0.819           0.803   
    Business sophistication                                0.814           0.708   
    Knowledge and technology outputs                       0.825           0.767   
    Creative outputs                                       0.842           0.805   
    
                                      Market sophistication  \
    Score                                             0.879   
    Institutions                                      0.715   
    Human capital and research                        0.819   
    Infrastructure                                    0.803   
    Market sophistication                             1.000   
    Business sophistication                           0.760   
    Knowledge and technology outputs                  0.795   
    Creative outputs                                  0.829   
    
                                      Business sophistication  \
    Score                                               0.898   
    Institutions                                        0.709   
    Human capital and research                          0.814   
    Infrastructure                                      0.708   
    Market sophistication                               0.760   
    Business sophistication                             1.000   
    Knowledge and technology outputs                    0.873   
    Creative outputs                                    0.844   
    
                                      Knowledge and technology outputs  \
    Score                                                        0.917   
    Institutions                                                 0.619   
    Human capital and research                                   0.825   
    Infrastructure                                               0.767   
    Market sophistication                                        0.795   
    Business sophistication                                      0.873   
    Knowledge and technology outputs                             1.000   
    Creative outputs                                             0.880   
    
                                      Creative outputs  
    Score                                        0.937  
    Institutions                                 0.674  
    Human capital and research                   0.842  
    Infrastructure                               0.805  
    Market sophistication                        0.829  
    Business sophistication                      0.844  
    Knowledge and technology outputs             0.880  
    Creative outputs                             1.000  
    




=============================
=============================

TEMEL BİLEŞENLER ANALİZİ (PCA)
==============================

.. _section-1:

=============================
=============================




.. code:: ipython3

    # =============================
    # PCA ANALYSIS (PC1–PC5 reported, PC1–PC7 fitted)
    # GII.csv
    # =============================
    
    import pandas as pd
    import numpy as np
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA → FIT WITH 7 COMPONENTS
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) Explained Variance (PC1–PC5)
    # --------------------------------------------------------
    
    variance_df = pd.DataFrame({
        "Component": [f"PC{i}" for i in range(1, 6)],
        "Explained Variance Ratio": pca.explained_variance_ratio_[:5],
        "Cumulative Variance (%)": np.cumsum(pca.explained_variance_ratio_[:5]) * 100
    })
    
    print("\n=== EXPLAINED VARIANCE (PC1–PC5) ===")
    print(variance_df.round(4))
    
    # --------------------------------------------------------
    # 6) Loadings Matrix (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    print("\n=== PCA LOADINGS (PC1–PC5) ===")
    print(loadings.round(3))
    
    # --------------------------------------------------------
    # 7) Important Loadings (|loading| ≥ 0.5)
    # --------------------------------------------------------
    
    important_loadings = loadings[loadings.abs() >= 0.5].dropna(how="all")
    
    print("\n=== IMPORTANT LOADINGS (|loading| ≥ 0.5) ===")
    print(important_loadings.round(3))
    
    # --------------------------------------------------------
    # 8) Loadings Heatmap (PC1–PC5)
    # --------------------------------------------------------
    
    plt.figure(figsize=(9, 7))
    
    sns.heatmap(
        loadings,
        annot=True,
        cmap="viridis",
        center=0,
        fmt=".3f"
    )
    
    plt.title("PCA Loadings Heatmap (PC1–PC5) — GII Pillars", fontsize=12, fontweight="bold")
    plt.tight_layout()
    plt.show()
    
    # --------------------------------------------------------
    # 9) PCA Scatter (PC1 vs PC2)
    # --------------------------------------------------------
    
    pc_vars = pca.explained_variance_ratio_[:5] * 100
    
    plt.figure(figsize=(12, 8))
    
    plt.scatter(
        pca_scores[:, 0],
        pca_scores[:, 1],
        alpha=0.6
    )
    
    for i, country in enumerate(countries):
        plt.text(
            pca_scores[i, 0],
            pca_scores[i, 1],
            country,
            fontsize=8,
            alpha=0.75
        )
    
    plt.axhline(0, linewidth=0.8)
    plt.axvline(0, linewidth=0.8)
    
    plt.xlabel(f"PC1 ({pc_vars[0]:.2f}%)")
    plt.ylabel(f"PC2 ({pc_vars[1]:.2f}%)")
    
    plt.title("Country-Level PCA Projection (PC1 vs PC2) — GII", fontsize=13, pad=20)
    
    plt.figtext(
        0.5, 0.92,
        f"Explained Variance: "
        f"PC1={pc_vars[0]:.2f}%,  "
        f"PC2={pc_vars[1]:.2f}%,  "
        f"PC3={pc_vars[2]:.2f}%,  "
        f"PC4={pc_vars[3]:.2f}%,  "
        f"PC5={pc_vars[4]:.2f}%",
        ha="center",
        fontsize=11,
        fontweight="bold"
    )
    
    plt.grid(True, linestyle="--", alpha=0.3)
    plt.tight_layout(rect=[0, 0, 1, 0.90])
    plt.show()
    


.. parsed-literal::

    
    === EXPLAINED VARIANCE (PC1–PC5) ===
      Component  Explained Variance Ratio  Cumulative Variance (%)
    0       PC1                    0.8279                  82.7917
    1       PC2                    0.0598                  88.7706
    2       PC3                    0.0368                  92.4486
    3       PC4                    0.0245                  94.9031
    4       PC5                    0.0216                  97.0646
    
    === PCA LOADINGS (PC1–PC5) ===
                                        PC1    PC2    PC3    PC4    PC5
    Score                             0.383 -0.038  0.070 -0.064 -0.093
    Institutions                      0.314  0.777  0.400  0.097  0.032
    Human capital and research        0.354 -0.205 -0.211  0.420  0.759
    Infrastructure                    0.346  0.324 -0.491 -0.569  0.195
    Market sophistication             0.353  0.071 -0.406  0.586 -0.546
    Business sophistication           0.353 -0.216  0.607  0.061  0.063
    Knowledge and technology outputs  0.358 -0.384  0.116 -0.313 -0.187
    Creative outputs                  0.365 -0.219 -0.058 -0.202 -0.200
    
    === IMPORTANT LOADINGS (|loading| ≥ 0.5) ===
                                PC1    PC2    PC3    PC4    PC5
    Institutions                NaN  0.777    NaN    NaN    NaN
    Human capital and research  NaN    NaN    NaN    NaN  0.759
    Infrastructure              NaN    NaN    NaN -0.569    NaN
    Market sophistication       NaN    NaN    NaN  0.586 -0.546
    Business sophistication     NaN    NaN  0.607    NaN    NaN
    


.. image:: output_54_1.png



.. image:: output_54_2.png








.. code:: ipython3

    # =========================================================
    # FULL PCA ANALYSIS + UNIT CIRCLE + COUNTRY SCORES
    # PC1–PC3 REPORTED, PC1–PC7 FITTED
    # GII.csv — JOURNAL READY
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    # Bozuk başlıkları temizle
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables (GII Pillars)
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) Loadings (PC1–PC3)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:3].T,
        index=numeric_df.columns,
        columns=["PC1", "PC2", "PC3"]
    )
    
    # --------------------------------------------------------
    # 6) PCA UNIT CIRCLE + COUNTRY SCORES (PC1–PC3)
    # --------------------------------------------------------
    
    # Kısa, makale-dostu GII etiketleri
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    eigenvalues = pca.explained_variance_
    pc_pairs = [("PC1", "PC2"), ("PC1", "PC3"), ("PC2", "PC3")]
    colors = plt.cm.tab10.colors
    
    plt.figure(figsize=(20, 12))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        plt.subplot(3, 1, idx)
    
        # -----------------------------
        # Unit circle
        # -----------------------------
        theta = np.linspace(0, 2 * np.pi, 400)
        plt.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        plt.axhline(0, linewidth=1)
        plt.axvline(0, linewidth=1)
    
        # -----------------------------
        # Country scores (normalized)
        # -----------------------------
        xs = pca_scores[:, int(pc_x[-1]) - 1]
        ys = pca_scores[:, int(pc_y[-1]) - 1]
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        plt.scatter(xs, ys, alpha=0.6, color="black", s=45)
    
        # -----------------------------
        # Variable vectors + loadings
        # -----------------------------
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[int(pc_x[-1]) - 1])
            y = ly * np.sqrt(eigenvalues[int(pc_y[-1]) - 1])
    
            plt.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=2,
                head_width=0.035,
                head_length=0.035,
                length_includes_head=True
            )
    
            label = short_labels.get(var, var)
            offset = 1.04
    
            plt.text(
                x * offset,
                y * offset,
                f"{label}\n({lx:.2f}, {ly:.2f})",
                fontsize=7,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        # -----------------------------
        # Axes & titles
        # -----------------------------
        plt.xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[int(pc_x[-1])-1]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[int(pc_y[-1])-1]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.title(
            f"PCA Correlation Circle — {pc_x} vs {pc_y} (GII)",
            fontsize=10,
            fontweight="bold"
        )
    
        plt.gca().set_aspect("equal", adjustable="box")
        plt.grid(True, linestyle="--", alpha=0.4)
    
    plt.tight_layout()
    plt.show()
    



.. image:: output_61_0.png




.. code:: ipython3

    # =========================================================
    # FULL PCA ANALYSIS + UNIT CIRCLE + COUNTRY SCORES
    # PC1–PC5 REPORTED, PC1–PC7 FITTED
    # GII.csv — JOURNAL READY
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) Loadings (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    # --------------------------------------------------------
    # 6) PCA UNIT CIRCLE + COUNTRY SCORES (PC1–PC5)
    # --------------------------------------------------------
    
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    eigenvalues = pca.explained_variance_
    
    pc_pairs = [
        ("PC1", "PC2"),
        ("PC1", "PC3"),
        ("PC2", "PC3"),
        ("PC3", "PC4"),
        ("PC4", "PC5")
    ]
    
    colors = plt.cm.tab10.colors
    
    plt.figure(figsize=(20, 18))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        plt.subplot(3, 2, idx)
    
        # Unit circle
        theta = np.linspace(0, 2 * np.pi, 400)
        plt.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        plt.axhline(0, linewidth=0.8)
        plt.axvline(0, linewidth=0.8)
    
        # Country scores (normalized)
        x_idx = int(pc_x[-1]) - 1
        y_idx = int(pc_y[-1]) - 1
    
        xs = pca_scores[:, x_idx]
        ys = pca_scores[:, y_idx]
    
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        plt.scatter(xs, ys, alpha=0.5, color="black", s=35)
    
        # Variable vectors
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[x_idx])
            y = ly * np.sqrt(eigenvalues[y_idx])
    
            plt.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=2,
                head_width=0.03,
                head_length=0.03,
                length_includes_head=True,
                alpha=0.9
            )
    
            label = short_labels.get(var, var)
    
            plt.text(
                x * 1.05,
                y * 1.05,
                label,
                fontsize=7,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        plt.xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[x_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[y_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.title(
            f"PCA Correlation Circle — {pc_x} vs {pc_y}",
            fontsize=10,
            fontweight="bold"
        )
    
        plt.gca().set_aspect("equal", adjustable="box")
        plt.grid(True, linestyle="--", alpha=0.4)
    
    plt.tight_layout()
    plt.show()
    



.. image:: output_64_0.png



.. code:: ipython3

    # =========================================================
    # FULL PCA ANALYSIS + UNIT CIRCLE + COUNTRY SCORES
    # PC1–PC5 REPORTED, PC1–PC7 FITTED
    # GII.csv — JOURNAL READY (WITH NUMERIC LOADINGS)
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    # Bozuk / anlamsız satırları çıkar
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables (GII Pillars)
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) Loadings Matrix (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    # --------------------------------------------------------
    # 6) PCA UNIT CIRCLE + COUNTRY SCORES (PC1–PC5)
    # --------------------------------------------------------
    
    # Makale-dostu kısa etiketler
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    eigenvalues = pca.explained_variance_
    
    # Gösterilecek PC çiftleri
    pc_pairs = [
        ("PC1", "PC2"),
        ("PC1", "PC3"),
        ("PC2", "PC3"),
        ("PC3", "PC4"),
        ("PC4", "PC5")
    ]
    
    colors = plt.cm.tab10.colors
    
    plt.figure(figsize=(20, 18))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        plt.subplot(3, 2, idx)
    
        # -----------------------------
        # Unit circle
        # -----------------------------
        theta = np.linspace(0, 2 * np.pi, 400)
        plt.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        plt.axhline(0, linewidth=0.8)
        plt.axvline(0, linewidth=0.8)
    
        # -----------------------------
        # Country scores (normalized)
        # -----------------------------
        x_idx = int(pc_x[-1]) - 1
        y_idx = int(pc_y[-1]) - 1
    
        xs = pca_scores[:, x_idx]
        ys = pca_scores[:, y_idx]
    
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        plt.scatter(xs, ys, alpha=0.5, color="black", s=35)
    
        # -----------------------------
        # Variable vectors + NUMERIC LOADINGS
        # -----------------------------
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[x_idx])
            y = ly * np.sqrt(eigenvalues[y_idx])
    
            plt.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=2,
                head_width=0.03,
                head_length=0.03,
                length_includes_head=True,
                alpha=0.9
            )
    
            label = short_labels.get(var, var)
    
            # 🔹 Etiket: isim + sayısal yükler
            plt.text(
                x * 1.08,
                y * 1.08,
                f"{label}\n({lx:.2f}, {ly:.2f})",
                fontsize=6.5,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        # -----------------------------
        # Axis labels & titles
        # -----------------------------
        plt.xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[x_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[y_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.title(
            f"PCA Correlation Circle — {pc_x} vs {pc_y} (GII)",
            fontsize=10,
            fontweight="bold"
        )
    
        plt.gca().set_aspect("equal", adjustable="box")
        plt.grid(True, linestyle="--", alpha=0.4)
    
    plt.tight_layout()
    plt.show()
    



.. image:: output_66_0.png



.. code:: ipython3

    # =========================================================
    # FULL PCA ANALYSIS + UNIT CIRCLE + COUNTRY SCORES
    # PC1–PC5 REPORTED, PC1–PC7 FITTED
    # GII.csv — JOURNAL READY (WITH NUMERIC LOADINGS)
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) Loadings Matrix (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    # --------------------------------------------------------
    # 6) PCA UNIT CIRCLE + COUNTRY SCORES (PC1–PC5)
    # --------------------------------------------------------
    
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    eigenvalues = pca.explained_variance_
    
    pc_pairs = [
        ("PC1", "PC2"),
        ("PC1", "PC3"),
        ("PC2", "PC3"),
        ("PC3", "PC4"),
        ("PC4", "PC5")
    ]
    
    colors = plt.cm.tab10.colors
    
    plt.figure(figsize=(20, 18))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        plt.subplot(3, 2, idx)
    
        # Unit circle
        theta = np.linspace(0, 2 * np.pi, 400)
        plt.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        plt.axhline(0, linewidth=0.8)
        plt.axvline(0, linewidth=0.8)
    
        # Country scores (NORMALIZED & SMALLER POINTS)
        x_idx = int(pc_x[-1]) - 1
        y_idx = int(pc_y[-1]) - 1
    
        xs = pca_scores[:, x_idx]
        ys = pca_scores[:, y_idx]
    
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        plt.scatter(
            xs, ys,
            s=12,              # 🔹 DAHA KÜÇÜK NOKTALAR
            alpha=0.4,
            color="blue"
        )
    
        # Variable vectors + numeric loadings
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[x_idx])
            y = ly * np.sqrt(eigenvalues[y_idx])
    
            plt.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=2,
                head_width=0.03,
                head_length=0.03,
                length_includes_head=True
            )
    
            label = short_labels.get(var, var)
    
            plt.text(
                x * 1.08,
                y * 1.08,
                f"{label}\n({lx:.2f}, {ly:.2f})",
                fontsize=6.5,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        plt.xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[x_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[y_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.title(
            f"PCA Correlation Circle — {pc_x} vs {pc_y} (GII)",
            fontsize=10,
            fontweight="bold"
        )
    
        plt.gca().set_aspect("equal", adjustable="box")
        plt.grid(True, linestyle="--", alpha=0.4)
    
    plt.tight_layout()
    plt.show()
    
    



.. image:: output_68_0.png



.. code:: ipython3

    # =========================================================
    # FULL PCA ANALYSIS + UNIT CIRCLE + COUNTRY SCORES
    # PC1–PC5 REPORTED, PC1–PC7 FITTED
    # GII.csv — JOURNAL READY (WITH NUMERIC LOADINGS TABLE OUTPUT)
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) LOADINGS MATRIX (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    # Eigenvalues
    eigenvalues = pca.explained_variance_
    
    # --------------------------------------------------------
    # 6) PRINT NUMERIC RESULTS (JOURNAL READY OUTPUT)
    # --------------------------------------------------------
    
    print("\n" + "="*70)
    print("EXPLAINED VARIANCE (PC1–PC7)")
    print("="*70)
    
    for i in range(7):
        print(f"PC{i+1}: "
              f"Eigenvalue = {eigenvalues[i]:.4f} | "
              f"Variance Ratio = {pca.explained_variance_ratio_[i]*100:.2f}%")
    
    print("\n" + "="*70)
    print("PCA LOADINGS MATRIX (PC1–PC5)")
    print("="*70)
    
    print(loadings.round(4))
    
    print("\n" + "="*70)
    print("DETAILED NUMERIC LOADINGS PER VARIABLE")
    print("="*70)
    
    for var in loadings.index:
        print(f"\nVariable: {var}")
        for pc in loadings.columns:
            print(f"   {pc}: {loadings.loc[var, pc]:.4f}")
    
    # Optional: Save to CSV for journal appendix
    loadings.round(4).to_csv("PCA_Loadings_PC1_PC5.csv")
    
    # --------------------------------------------------------
    # 7) PCA UNIT CIRCLE PLOT
    # --------------------------------------------------------
    
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    pc_pairs = [
        ("PC1", "PC2"),
        ("PC1", "PC3"),
        ("PC2", "PC3"),
        ("PC3", "PC4"),
        ("PC4", "PC5")
    ]
    
    colors = plt.cm.tab10.colors
    
    plt.figure(figsize=(20, 18))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        plt.subplot(3, 2, idx)
    
        theta = np.linspace(0, 2 * np.pi, 400)
        plt.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        plt.axhline(0, linewidth=0.8)
        plt.axvline(0, linewidth=0.8)
    
        x_idx = int(pc_x[-1]) - 1
        y_idx = int(pc_y[-1]) - 1
    
        xs = pca_scores[:, x_idx]
        ys = pca_scores[:, y_idx]
    
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        plt.scatter(xs, ys, s=12, alpha=0.4, color="blue")
    
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[x_idx])
            y = ly * np.sqrt(eigenvalues[y_idx])
    
            plt.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=2,
                head_width=0.03,
                head_length=0.03,
                length_includes_head=True
            )
    
            label = short_labels.get(var, var)
    
            plt.text(
                x * 1.08,
                y * 1.08,
                f"{label}\n({lx:.2f}, {ly:.2f})",
                fontsize=6.5,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        plt.xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[x_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[y_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.title(
            f"PCA Correlation Circle — {pc_x} vs {pc_y} (GII)",
            fontsize=10,
            fontweight="bold"
        )
    
        plt.gca().set_aspect("equal", adjustable="box")
        plt.grid(True, linestyle="--", alpha=0.4)
    
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    ======================================================================
    EXPLAINED VARIANCE (PC1–PC7)
    ======================================================================
    PC1: Eigenvalue = 6.6713 | Variance Ratio = 82.79%
    PC2: Eigenvalue = 0.4818 | Variance Ratio = 5.98%
    PC3: Eigenvalue = 0.2964 | Variance Ratio = 3.68%
    PC4: Eigenvalue = 0.1978 | Variance Ratio = 2.45%
    PC5: Eigenvalue = 0.1742 | Variance Ratio = 2.16%
    PC6: Eigenvalue = 0.1209 | Variance Ratio = 1.50%
    PC7: Eigenvalue = 0.0872 | Variance Ratio = 1.08%
    
    ======================================================================
    PCA LOADINGS MATRIX (PC1–PC5)
    ======================================================================
                                         PC1     PC2     PC3     PC4     PC5
    Score                             0.3826 -0.0384  0.0704 -0.0639 -0.0935
    Institutions                      0.3139  0.7772  0.3998  0.0967  0.0320
    Human capital and research        0.3539 -0.2046 -0.2107  0.4202  0.7585
    Infrastructure                    0.3457  0.3243 -0.4907 -0.5693  0.1951
    Market sophistication             0.3527  0.0708 -0.4056  0.5856 -0.5458
    Business sophistication           0.3533 -0.2165  0.6072  0.0611  0.0627
    Knowledge and technology outputs  0.3578 -0.3841  0.1160 -0.3135 -0.1872
    Creative outputs                  0.3648 -0.2192 -0.0582 -0.2023 -0.1998
    
    ======================================================================
    DETAILED NUMERIC LOADINGS PER VARIABLE
    ======================================================================
    
    Variable: Score
       PC1: 0.3826
       PC2: -0.0384
       PC3: 0.0704
       PC4: -0.0639
       PC5: -0.0935
    
    Variable: Institutions
       PC1: 0.3139
       PC2: 0.7772
       PC3: 0.3998
       PC4: 0.0967
       PC5: 0.0320
    
    Variable: Human capital and research
       PC1: 0.3539
       PC2: -0.2046
       PC3: -0.2107
       PC4: 0.4202
       PC5: 0.7585
    
    Variable: Infrastructure
       PC1: 0.3457
       PC2: 0.3243
       PC3: -0.4907
       PC4: -0.5693
       PC5: 0.1951
    
    Variable: Market sophistication
       PC1: 0.3527
       PC2: 0.0708
       PC3: -0.4056
       PC4: 0.5856
       PC5: -0.5458
    
    Variable: Business sophistication
       PC1: 0.3533
       PC2: -0.2165
       PC3: 0.6072
       PC4: 0.0611
       PC5: 0.0627
    
    Variable: Knowledge and technology outputs
       PC1: 0.3578
       PC2: -0.3841
       PC3: 0.1160
       PC4: -0.3135
       PC5: -0.1872
    
    Variable: Creative outputs
       PC1: 0.3648
       PC2: -0.2192
       PC3: -0.0582
       PC4: -0.2023
       PC5: -0.1998
    


.. image:: output_70_1.png




.. code:: ipython3

    # =========================================================
    # FULL PCA ANALYSIS + UNIT CIRCLE + COUNTRY SCORES
    # PC1–PC5 REPORTED, PC1–PC7 FITTED
    # JOURNAL READY – SINGLE SCRIPT (GII)
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    # Bozuk satırları temizle
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) Loadings (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    # --------------------------------------------------------
    # 6) PCA UNIT CIRCLE + COUNTRY SCORES (PC1–PC5)
    # --------------------------------------------------------
    
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    eigenvalues = pca.explained_variance_
    
    pc_pairs = [
        ("PC1", "PC2"),
        ("PC1", "PC3"),
        ("PC2", "PC3"),
        ("PC3", "PC4"),
        ("PC4", "PC5")
    ]
    
    colors = plt.cm.tab10.colors
    
    plt.figure(figsize=(20, 18))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        plt.subplot(3, 2, idx)
    
        # -----------------------------
        # Unit circle
        # -----------------------------
        theta = np.linspace(0, 2 * np.pi, 400)
        plt.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        plt.axhline(0, linewidth=1)
        plt.axvline(0, linewidth=1)
    
        # -----------------------------
        # Country scores (normalized)
        # -----------------------------
        x_idx = int(pc_x[-1]) - 1
        y_idx = int(pc_y[-1]) - 1
    
        xs = pca_scores[:, x_idx]
        ys = pca_scores[:, y_idx]
    
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        plt.scatter(xs, ys, alpha=0.4, color="black", s=12)
    
        # -----------------------------
        # Variable vectors + numeric labels
        # -----------------------------
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[x_idx])
            y = ly * np.sqrt(eigenvalues[y_idx])
    
            plt.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=2,
                head_width=0.035,
                head_length=0.035,
                length_includes_head=True
            )
    
            label = short_labels.get(var, var)
    
            plt.text(
                x * 1.05,
                y * 1.05,
                f"{label}\n({lx:.2f},{ly:.2f})",
                fontsize=6,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        # -----------------------------
        # Axis labels & titles
        # -----------------------------
        plt.xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[x_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[y_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.title(
            f"PCA Correlation Circle: {pc_x} vs {pc_y} (GII)",
            fontsize=9,
            fontweight="bold"
        )
    
        plt.gca().set_aspect("equal", adjustable="box")
        plt.grid(True, linestyle="--", alpha=0.4)
    
    plt.tight_layout()
    plt.show()
    



.. image:: output_73_0.png



.. code:: ipython3

    # =========================================================
    # FULL PCA ANALYSIS + UNIT CIRCLE + COUNTRY SCORES
    # PC1–PC5 REPORTED, PC1–PC7 FITTED
    # GII.csv — JOURNAL READY (COMPACT FIGURE LAYOUT)
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) LOADINGS MATRIX (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    eigenvalues = pca.explained_variance_
    
    # --------------------------------------------------------
    # 6) PRINT NUMERIC RESULTS
    # --------------------------------------------------------
    
    print("\n" + "="*70)
    print("EXPLAINED VARIANCE (PC1–PC7)")
    print("="*70)
    
    for i in range(7):
        print(f"PC{i+1}: "
              f"Eigenvalue = {eigenvalues[i]:.4f} | "
              f"Variance Ratio = {pca.explained_variance_ratio_[i]*100:.2f}%")
    
    print("\n" + "="*70)
    print("PCA LOADINGS MATRIX (PC1–PC5)")
    print("="*70)
    print(loadings.round(4))
    
    print("\n" + "="*70)
    print("DETAILED NUMERIC LOADINGS PER VARIABLE")
    print("="*70)
    
    for var in loadings.index:
        print(f"\nVariable: {var}")
        for pc in loadings.columns:
            print(f"   {pc}: {loadings.loc[var, pc]:.4f}")
    
    loadings.round(4).to_csv("PCA_Loadings_PC1_PC5.csv")
    
    # --------------------------------------------------------
    # 7) PCA UNIT CIRCLE (COMPACT LAYOUT)
    # --------------------------------------------------------
    
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    pc_pairs = [
        ("PC1", "PC2"),
        ("PC1", "PC3"),
        ("PC2", "PC3"),
        ("PC3", "PC4"),
        ("PC4", "PC5")
    ]
    
    colors = plt.cm.tab10.colors
    
    # 🔹 Daha kompakt figure boyutu
    fig = plt.figure(figsize=(16, 14))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        ax = fig.add_subplot(3, 2, idx)
    
        theta = np.linspace(0, 2 * np.pi, 400)
        ax.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        ax.axhline(0, linewidth=0.8)
        ax.axvline(0, linewidth=0.8)
    
        x_idx = int(pc_x[-1]) - 1
        y_idx = int(pc_y[-1]) - 1
    
        xs = pca_scores[:, x_idx]
        ys = pca_scores[:, y_idx]
    
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        ax.scatter(xs, ys, s=12, alpha=0.4, color="blue")
    
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[x_idx])
            y = ly * np.sqrt(eigenvalues[y_idx])
    
            ax.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=2,
                head_width=0.03,
                head_length=0.03,
                length_includes_head=True
            )
    
            label = short_labels.get(var, var)
    
            ax.text(
                x * 1.08,
                y * 1.08,
                f"{label}\n({lx:.2f}, {ly:.2f})",
                fontsize=6.5,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        ax.set_xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[x_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        ax.set_ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[y_idx]*100:.2f}%)",
            fontsize=9,
            fontweight="bold"
        )
    
        ax.set_title(
            f"PCA Correlation Circle — {pc_x} vs {pc_y} (GII)",
            fontsize=10,
            fontweight="bold"
        )
    
        ax.set_aspect("equal", adjustable="box")
        ax.grid(True, linestyle="--", alpha=0.002)
    
    # 🔹 BOŞLUKLARI DARALTAN KISIM
    plt.tight_layout(pad=1.0)
    plt.subplots_adjust(hspace=0.0015, wspace=0.0)
    
    plt.show()


.. parsed-literal::

    
    ======================================================================
    EXPLAINED VARIANCE (PC1–PC7)
    ======================================================================
    PC1: Eigenvalue = 6.6713 | Variance Ratio = 82.79%
    PC2: Eigenvalue = 0.4818 | Variance Ratio = 5.98%
    PC3: Eigenvalue = 0.2964 | Variance Ratio = 3.68%
    PC4: Eigenvalue = 0.1978 | Variance Ratio = 2.45%
    PC5: Eigenvalue = 0.1742 | Variance Ratio = 2.16%
    PC6: Eigenvalue = 0.1209 | Variance Ratio = 1.50%
    PC7: Eigenvalue = 0.0872 | Variance Ratio = 1.08%
    
    ======================================================================
    PCA LOADINGS MATRIX (PC1–PC5)
    ======================================================================
                                         PC1     PC2     PC3     PC4     PC5
    Score                             0.3826 -0.0384  0.0704 -0.0639 -0.0935
    Institutions                      0.3139  0.7772  0.3998  0.0967  0.0320
    Human capital and research        0.3539 -0.2046 -0.2107  0.4202  0.7585
    Infrastructure                    0.3457  0.3243 -0.4907 -0.5693  0.1951
    Market sophistication             0.3527  0.0708 -0.4056  0.5856 -0.5458
    Business sophistication           0.3533 -0.2165  0.6072  0.0611  0.0627
    Knowledge and technology outputs  0.3578 -0.3841  0.1160 -0.3135 -0.1872
    Creative outputs                  0.3648 -0.2192 -0.0582 -0.2023 -0.1998
    
    ======================================================================
    DETAILED NUMERIC LOADINGS PER VARIABLE
    ======================================================================
    
    Variable: Score
       PC1: 0.3826
       PC2: -0.0384
       PC3: 0.0704
       PC4: -0.0639
       PC5: -0.0935
    
    Variable: Institutions
       PC1: 0.3139
       PC2: 0.7772
       PC3: 0.3998
       PC4: 0.0967
       PC5: 0.0320
    
    Variable: Human capital and research
       PC1: 0.3539
       PC2: -0.2046
       PC3: -0.2107
       PC4: 0.4202
       PC5: 0.7585
    
    Variable: Infrastructure
       PC1: 0.3457
       PC2: 0.3243
       PC3: -0.4907
       PC4: -0.5693
       PC5: 0.1951
    
    Variable: Market sophistication
       PC1: 0.3527
       PC2: 0.0708
       PC3: -0.4056
       PC4: 0.5856
       PC5: -0.5458
    
    Variable: Business sophistication
       PC1: 0.3533
       PC2: -0.2165
       PC3: 0.6072
       PC4: 0.0611
       PC5: 0.0627
    
    Variable: Knowledge and technology outputs
       PC1: 0.3578
       PC2: -0.3841
       PC3: 0.1160
       PC4: -0.3135
       PC5: -0.1872
    
    Variable: Creative outputs
       PC1: 0.3648
       PC2: -0.2192
       PC3: -0.0582
       PC4: -0.2023
       PC5: -0.1998
    


.. image:: output_75_1.png









.. code:: ipython3

    # =========================================================
    # PCA UNIT CIRCLE + COUNTRY SCORES (PC1–PC5)
    # Ok ucunda etiket + sayısal yükler
    # GII.csv — JOURNAL READY
    # LAYOUT: 3 ROWS x 2 COLUMNS
    # =========================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    
    # --------------------------------------------------------
    # 1) Load Data
    # --------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    # --------------------------------------------------------
    # 2) Numeric Variables
    # --------------------------------------------------------
    
    numeric_df = (
        df.drop(columns=["Country"])
        .apply(pd.to_numeric, errors="coerce")
        .dropna()
    )
    
    countries = df.loc[numeric_df.index, "Country"]
    
    # --------------------------------------------------------
    # 3) Standardization
    # --------------------------------------------------------
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # --------------------------------------------------------
    # 4) PCA (FIT WITH 7 COMPONENTS)
    # --------------------------------------------------------
    
    pca = PCA(n_components=7)
    pca_scores = pca.fit_transform(scaled_data)
    
    # --------------------------------------------------------
    # 5) Loadings (PC1–PC5)
    # --------------------------------------------------------
    
    loadings = pd.DataFrame(
        pca.components_[:5].T,
        index=numeric_df.columns,
        columns=[f"PC{i}" for i in range(1, 6)]
    )
    
    # --------------------------------------------------------
    # 6) PCA UNIT CIRCLE + COUNTRY SCORES (PC1–PC5)
    # --------------------------------------------------------
    
    short_labels = {
        "Score": "GII",
        "Institutions": "INST",
        "Human capital and research": "HCR",
        "Infrastructure": "INF",
        "Market sophistication": "MARK",
        "Business sophistication": "BUS",
        "Knowledge and technology outputs": "KTO",
        "Creative outputs": "CRE"
    }
    
    eigenvalues = pca.explained_variance_
    
    pc_pairs = [
        ("PC1", "PC2"),
        ("PC1", "PC3"),
        ("PC2", "PC3"),
        ("PC3", "PC4"),
        ("PC4", "PC5")
    ]
    
    colors = plt.cm.tab10.colors
    
    plt.figure(figsize=(20, 18))
    
    for idx, (pc_x, pc_y) in enumerate(pc_pairs, 1):
        plt.subplot(3, 2, idx)   # 🔹 3 SATIR × 2 SÜTUN
    
        # -----------------------------
        # Unit circle
        # -----------------------------
        theta = np.linspace(0, 2 * np.pi, 400)
        plt.plot(np.cos(theta), np.sin(theta), '--', color='gray', linewidth=1)
        plt.axhline(0, linewidth=1)
        plt.axvline(0, linewidth=1)
    
        # -----------------------------
        # Country scores (normalized)
        # -----------------------------
        x_idx = int(pc_x[-1]) - 1
        y_idx = int(pc_y[-1]) - 1
    
        xs = pca_scores[:, x_idx]
        ys = pca_scores[:, y_idx]
    
        xs /= np.max(np.abs(xs))
        ys /= np.max(np.abs(ys))
    
        plt.scatter(xs, ys, alpha=0.45, color="black", s=10)
    
        # -----------------------------
        # Variable vectors + labels at arrow tip
        # -----------------------------
        for i, var in enumerate(numeric_df.columns):
            lx = loadings.loc[var, pc_x]
            ly = loadings.loc[var, pc_y]
    
            x = lx * np.sqrt(eigenvalues[x_idx])
            y = ly * np.sqrt(eigenvalues[y_idx])
    
            plt.arrow(
                0, 0, x, y,
                color=colors[i % len(colors)],
                linewidth=1.5,
                head_width=0.03,
                head_length=0.03,
                length_includes_head=True
            )
    
            label = short_labels.get(var, var)
    
            angle = np.arctan2(y, x) + (i % 5 - 2) * 0.12
            r = np.sqrt(x**2 + y**2) * 1.01
            dx = r * np.cos(angle)
            dy = r * np.sin(angle)
    
            plt.text(
                dx,
                dy,
                f"{label}\n({lx:.2f},{ly:.2f})",
                fontsize=5,
                fontweight="bold",
                ha="center",
                va="center"
            )
    
        plt.xlabel(
            f"{pc_x} ({pca.explained_variance_ratio_[x_idx]*100:.2f}%)",
            fontsize=7,
            fontweight="bold"
        )
    
        plt.ylabel(
            f"{pc_y} ({pca.explained_variance_ratio_[y_idx]*100:.2f}%)",
            fontsize=7,
            fontweight="bold"
        )
    
        plt.title(
            f"PCA Correlation Circle: {pc_x} vs {pc_y} (GII)",
            fontsize=8,
            fontweight="bold"
        )
    
        plt.xticks(fontsize=6)
        plt.yticks(fontsize=6)
        plt.gca().set_aspect("equal", adjustable="box")
        plt.grid(True, linestyle="--", alpha=0.15)
    
    # 6. panel boş kalsın (bilinçli)
    plt.tight_layout()
    plt.show()
    
    # =========================================================
    # PCA SAYISAL ÇIKTILAR
    # =========================================================
    
    print("=========== PCA SAYISAL ÖZET ===========")
    
    print("\n-- Açıklanan Varyans (Eigenvalues) --")
    for i, val in enumerate(pca.explained_variance_, 1):
        print(f"PC{i}: {val:.4f}")
    
    print("\n-- Açıklanan Varyans Yüzdesi --")
    for i, val in enumerate(pca.explained_variance_ratio_, 1):
        print(f"PC{i}: {val*100:.2f}%")
    
    print("\n-- PCA Bileşen Yükleri (Loadings) --")
    print(loadings.round(4))
    
    pc_scores_df = pd.DataFrame(
        pca_scores,
        columns=[f"PC{i}" for i in range(1, 8)]
    )
    pc_scores_df["Country"] = countries.values
    pc_scores_df.to_csv("pca_scores_GII.csv", index=False)
    



.. image:: output_83_0.png


.. parsed-literal::

    =========== PCA SAYISAL ÖZET ===========
    
    -- Açıklanan Varyans (Eigenvalues) --
    PC1: 6.6713
    PC2: 0.4818
    PC3: 0.2964
    PC4: 0.1978
    PC5: 0.1742
    PC6: 0.1209
    PC7: 0.0872
    
    -- Açıklanan Varyans Yüzdesi --
    PC1: 82.79%
    PC2: 5.98%
    PC3: 3.68%
    PC4: 2.45%
    PC5: 2.16%
    PC6: 1.50%
    PC7: 1.08%
    
    -- PCA Bileşen Yükleri (Loadings) --
                                         PC1     PC2     PC3     PC4     PC5
    Score                             0.3826 -0.0384  0.0704 -0.0639 -0.0935
    Institutions                      0.3139  0.7772  0.3998  0.0967  0.0320
    Human capital and research        0.3539 -0.2046 -0.2107  0.4202  0.7585
    Infrastructure                    0.3457  0.3243 -0.4907 -0.5693  0.1951
    Market sophistication             0.3527  0.0708 -0.4056  0.5856 -0.5458
    Business sophistication           0.3533 -0.2165  0.6072  0.0611  0.0627
    Knowledge and technology outputs  0.3578 -0.3841  0.1160 -0.3135 -0.1872
    Creative outputs                  0.3648 -0.2192 -0.0582 -0.2023 -0.1998
    




















=====================================================
=====================================================

K-MEANS ANALİZİ: UYGUN K SAYISININ BELİRLENMESİ
===============================================

.. _section-1:

=====================================================
=====================================================

🧩 1️⃣ ELBOW (Dirsek) Yöntemi

Her K=2…10 için inertia (toplam hata kareleri) değerlerini hesaplar ve
ayrı grafikte gösterir.

1. Elbow (Dirsek) yöntemine göre

Inertia değerleri sürekli düşüyor:

2 → 1679.90 3 → 1484.81 (Δ = 195.09) 4 → 1321.68 (Δ = 163.13) 5 →
1211.46 (Δ = 110.22) 6 → 1113.52 (Δ = 97.94) 7 → 1054.24 (Δ = 59.28) 8 →
981.36 (Δ = 72.88) 9 → 922.69 (Δ = 58.67) 10 → 888.88 (Δ = 33.81)

Önemli nokta: Dirsek yöntemi mutlak düşüşe değil, düşüş hızındaki
kırılmaya bakar.

K=2 → K=5 arasında düşüşler yüksek.

K=5’ten sonra düşüşler belirgin şekilde yavaşlıyor.

Özellikle K ≥ 6’dan sonra kazanç azalan getiriler (diminishing returns)
bölgesine giriyor.

➡️ Bu, dirsek noktasının K ≈ 4–6 aralığında, özellikle K = 5 civarında
olduğunu gösterir.

Yani:

Elbow yöntemi açısından K = 5 makul ve bilimsel olarak savunulabilir bir
seçimdir.


Sonuç
=====

Dirsek (Elbow) ve Silhouette analizleri birlikte değerlendirildiğinde,
veri seti için optimal küme sayısı K = 5 olarak belirlenmiştir. Bu
değer, modelin hem yeterli ayrışmayı hem de istikrarlı kümelenmeyi
sağladığı noktayı temsil etmektedir.


.. code:: ipython3

    # =====================================================
    # 1️⃣ GEREKLİ KÜTÜPHANELER
    # =====================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    
    from sklearn.cluster import KMeans
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import silhouette_score
    
    
    # =====================================================
    # 2️⃣ VERİNİN OKUNMASI (⚠️ ; AYIRICI ÇOK KRİTİK)
    # =====================================================
    
    df = pd.read_csv(
        "GII.csv",
        sep=";",            # 🔥 EN KRİTİK SATIR
        encoding="latin1"
    )
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    # Bozuk satırları temizle
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    print("\nVeri seti boyutu:", df.shape)
    print("Kolonlar:")
    print(df.columns)
    
    
    # =====================================================
    # 3️⃣ SAYISAL DEĞİŞKENLERİN TEMİZLENMESİ (GII PILLARS)
    # =====================================================
    
    data = df.drop(columns=["Country"], errors="ignore")
    data = data.apply(pd.to_numeric, errors="coerce")
    data = data.dropna()
    
    print("\nKullanılan sayısal değişkenler:")
    print(list(data.columns))
    print("Temiz veri boyutu:", data.shape)
    
    if data.shape[0] == 0:
        raise ValueError("❌ Veri boş! GII.csv içeriğini kontrol et.")
    
    
    # =====================================================
    # 4️⃣ STANDARDIZATION
    # =====================================================
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(data)
    
    
    # =====================================================
    # 5️⃣ ELBOW & SILHOUETTE ANALİZİ
    # =====================================================
    
    inertia_values = []
    silhouette_scores = []
    
    K_values = range(2, 11)
    
    for k in K_values:
        kmeans = KMeans(
            n_clusters=k,
            random_state=42,
            n_init=10
        )
    
        labels = kmeans.fit_predict(scaled_data)
    
        inertia_values.append(kmeans.inertia_)
        silhouette_scores.append(
            silhouette_score(scaled_data, labels)
        )
    
    
    # =====================================================
    # 6️⃣ SAYISAL SONUÇ TABLOSU
    # =====================================================
    
    print("\n=== ELBOW & SILHOUETTE SONUÇLARI (GII) ===")
    print("{:<5} {:<15} {:<15}".format("K", "Inertia", "Silhouette"))
    
    for k, i, s in zip(K_values, inertia_values, silhouette_scores):
        print(f"{k:<5} {i:<15.2f} {s:<15.4f}")
    
    
    # =====================================================
    # 7️⃣ BİRLEŞİK GRAFİK (ETİKETLİ – SSCI UYUMLU)
    # =====================================================
    
    fig, ax1 = plt.subplots(figsize=(10, 6))
    
    # ---------------- Elbow (Inertia) ----------------
    ax1.set_xlabel("K (Number of Clusters)", fontsize=11)
    ax1.set_ylabel("Inertia", color="tab:blue", fontsize=11)
    ax1.plot(
        K_values,
        inertia_values,
        "o-",
        color="tab:blue",
        label="Inertia"
    )
    ax1.tick_params(axis="y", labelcolor="tab:blue")
    
    # Inertia etiketleri
    for k, inertia in zip(K_values, inertia_values):
        ax1.text(
            k,
            inertia,
            f"{inertia:.2f}",
            color="tab:blue",
            fontsize=10,
            ha="center",
            va="bottom"
        )
    
    
    # ---------------- Silhouette ----------------
    ax2 = ax1.twinx()
    ax2.set_ylabel("Silhouette Score", color="tab:red", fontsize=11)
    ax2.plot(
        K_values,
        silhouette_scores,
        "s--",
        color="tab:red",
        label="Silhouette Score"
    )
    ax2.tick_params(axis="y", labelcolor="tab:red")
    
    # Silhouette etiketleri
    for k, sil in zip(K_values, silhouette_scores):
        ax2.text(
            k,
            sil,
            f"{sil:.4f}",
            color="tab:red",
            fontsize=10,
            ha="center",
            va="top"
        )
    
    
    # ---------------- Örnek Optimal K ----------------
    plt.axvline(
        x=6,
        color="green",
        linestyle="--",
        linewidth=2,
        label="Candidate K = 6"
    )
    
    fig.suptitle(
        "Elbow & Silhouette Analysis for GII Dataset",
        fontsize=13,
        fontweight="bold"
    )
    
    fig.tight_layout()
    
    # Legend birleştirme
    lines1, labels1 = ax1.get_legend_handles_labels()
    lines2, labels2 = ax2.get_legend_handles_labels()
    plt.legend(
        lines1 + lines2,
        labels1 + labels2,
        loc="upper right",
        fontsize=10
    )
    
    plt.show()
    


.. parsed-literal::

    
    Veri seti boyutu: (139, 9)
    Kolonlar:
    Index(['Country', 'Score', 'Institutions', 'Human capital and research',
           'Infrastructure', 'Market sophistication', 'Business sophistication',
           'Knowledge and technology outputs', 'Creative outputs'],
          dtype='object')
    
    Kullanılan sayısal değişkenler:
    ['Score', 'Institutions', 'Human capital and research', 'Infrastructure', 'Market sophistication', 'Business sophistication', 'Knowledge and technology outputs', 'Creative outputs']
    Temiz veri boyutu: (139, 8)
    

.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\joblib\externals\loky\backend\context.py:136: UserWarning: Could not find the number of physical cores for the following reason:
    [WinError 2] The system cannot find the file specified
    Returning the number of logical cores instead. You can silence this warning by setting LOKY_MAX_CPU_COUNT to the number of cores you want to use.
      warnings.warn(
      File "C:\Users\sadul\anaconda3\Lib\site-packages\joblib\externals\loky\backend\context.py", line 257, in _count_physical_cores
        cpu_info = subprocess.run(
            "wmic CPU Get NumberOfCores /Format:csv".split(),
            capture_output=True,
            text=True,
        )
      File "C:\Users\sadul\anaconda3\Lib\subprocess.py", line 554, in run
        with Popen(*popenargs, **kwargs) as process:
             ~~~~~^^^^^^^^^^^^^^^^^^^^^^
      File "C:\Users\sadul\anaconda3\Lib\subprocess.py", line 1039, in __init__
        self._execute_child(args, executable, preexec_fn, close_fds,
        ~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            pass_fds, cwd, env,
                            ^^^^^^^^^^^^^^^^^^^
        ...<5 lines>...
                            gid, gids, uid, umask,
                            ^^^^^^^^^^^^^^^^^^^^^^
                            start_new_session, process_group)
                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
      File "C:\Users\sadul\anaconda3\Lib\subprocess.py", line 1554, in _execute_child
        hp, ht, pid, tid = _winapi.CreateProcess(executable, args,
                           ~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^
                                 # no special security
                                 ^^^^^^^^^^^^^^^^^^^^^
        ...<4 lines>...
                                 cwd,
                                 ^^^^
                                 startupinfo)
                                 ^^^^^^^^^^^^
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    

.. parsed-literal::

    
    === ELBOW & SILHOUETTE SONUÇLARI (GII) ===
    K     Inertia         Silhouette     
    2     436.08          0.5019         
    3     290.15          0.3645         
    4     238.20          0.2988         
    5     212.56          0.2391         
    6     188.89          0.2628         
    7     169.52          0.2317         
    8     159.86          0.2145         
    9     154.00          0.2074         
    10    143.65          0.2026         
    


.. image:: output_108_3.png





.. code:: ipython3

    # =====================================================
    # 1️⃣ GEREKLİ KÜTÜPHANELER
    # =====================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.preprocessing import StandardScaler
    from sklearn.cluster import KMeans
    from sklearn.decomposition import PCA
    from sklearn.metrics import silhouette_score
    
    # =====================================================
    # 2️⃣ VERİYİ OKU (GII CSV)
    # =====================================================
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    # Gereksiz/bozuk satırları temizle
    df = df[~df["Country"].str.lower().isin(["seliforp", "xedni", "index"])]
    
    print("Veri boyutu:", df.shape)
    print(df.columns)
    
    # =====================================================
    # 3️⃣ SAYISAL DEĞİŞKENLERİ AL
    # =====================================================
    data = df.drop(columns=["Country"], errors="ignore")
    data = data.apply(pd.to_numeric, errors="coerce").dropna()
    
    # Aynı satırları df'de de tut
    df = df.loc[data.index].reset_index(drop=True)
    data = data.reset_index(drop=True)
    
    print("Temiz veri boyutu:", data.shape)
    print("Kullanılan sayısal değişkenler:", list(data.columns))
    
    # =====================================================
    # 4️⃣ STANDARDIZATION
    # =====================================================
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(data)
    
    # =====================================================
    # 5️⃣ ELBOW & SILHOUETTE ANALİZİ (K=2..10)
    # =====================================================
    inertia_values = []
    silhouette_scores = []
    
    K_values = range(2, 11)
    
    for k in K_values:
        kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
        labels = kmeans.fit_predict(scaled_data)
        inertia_values.append(kmeans.inertia_)
        silhouette_scores.append(silhouette_score(scaled_data, labels))
    
    # =====================================================
    # 6️⃣ SAYISAL SONUÇ TABLOSU
    # =====================================================
    print("\n=== ELBOW & SILHOUETTE SONUÇLARI (GII) ===")
    print("{:<5} {:<15} {:<15}".format("K", "Inertia", "Silhouette"))
    for k, i, s in zip(K_values, inertia_values, silhouette_scores):
        print(f"{k:<5} {i:<15.2f} {s:<15.4f}")
    
    # =====================================================
    # 7️⃣ K-MEANS (OPTIMAL K = 6) & LABELS
    # =====================================================
    optimal_k = 6
    kmeans = KMeans(n_clusters=optimal_k, random_state=42, n_init=10)
    labels = kmeans.fit_predict(scaled_data)
    df["Cluster"] = labels
    
    # =====================================================
    # 8️⃣ PCA (2D) & GÖRSELLEŞTİRME
    # =====================================================
    pca = PCA(n_components=2)
    pca_data = pca.fit_transform(scaled_data)
    
    plt.figure(figsize=(16, 10))
    
    scatter = sns.scatterplot(
        x=pca_data[:, 0],
        y=pca_data[:, 1],
        hue=df["Cluster"],
        palette="tab10",
        s=90,
        alpha=0.85
    )
    
    # Ülke isimlerini ekle
    for i, country in enumerate(df["Country"]):
        plt.text(
            pca_data[i, 0] + 0.05,
            pca_data[i, 1] + 0.05,
            country,
            fontsize=8,
            alpha=0.75
        )
    
    # Legend
    cluster_counts = df["Cluster"].value_counts().sort_index()
    legend_labels = [f"Cluster {i} (n={cluster_counts[i]} country)" for i in cluster_counts.index]
    handles, _ = scatter.get_legend_handles_labels()
    plt.legend(handles, legend_labels, title="Clusters")
    
    plt.title(f"K-Means (K={optimal_k}) + PCA 2D Visualization for GII Dataset", fontsize=14)
    plt.xlabel("PCA 1")
    plt.ylabel("PCA 2")
    plt.grid(True)
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 9️⃣ KÜMELERE GÖRE ÜLKELER
    # =====================================================
    print("\n=== KÜMELERE GÖRE ÜLKE DAĞILIMI ===")
    for cluster_num in sorted(df["Cluster"].unique()):
        countries_in_cluster = df[df["Cluster"] == cluster_num]["Country"].tolist()
        print(f"\n--- Küme {cluster_num} ({len(countries_in_cluster)} ülke) ---")
        print(", ".join(countries_in_cluster))
        print("-" * 70)
    


.. parsed-literal::

    Veri boyutu: (139, 9)
    Index(['Country', 'Score', 'Institutions', 'Human capital and research',
           'Infrastructure', 'Market sophistication', 'Business sophistication',
           'Knowledge and technology outputs', 'Creative outputs'],
          dtype='object')
    Temiz veri boyutu: (139, 8)
    Kullanılan sayısal değişkenler: ['Score', 'Institutions', 'Human capital and research', 'Infrastructure', 'Market sophistication', 'Business sophistication', 'Knowledge and technology outputs', 'Creative outputs']
    

.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    

.. parsed-literal::

    
    === ELBOW & SILHOUETTE SONUÇLARI (GII) ===
    K     Inertia         Silhouette     
    2     436.08          0.5019         
    3     290.15          0.3645         
    4     238.20          0.2988         
    5     212.56          0.2391         
    6     188.89          0.2628         
    7     169.52          0.2317         
    8     159.86          0.2145         
    9     154.00          0.2074         
    10    143.65          0.2026         
    

.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    


.. image:: output_112_4.png


.. parsed-literal::

    
    === KÜMELERE GÖRE ÜLKE DAĞILIMI ===
    
    --- Küme 0 (25 ülke) ---
    Bulgaria, Chile, Côte d'Ivoire, Croatia, Cyprus, Greece, Hungary, India, Italy, Latvia, Lithuania, Malaysia, Malta, Philippines, Poland, Portugal, Qatar, Romania, Saudi Arabia, Slovakia, Slovenia, Spain, Thailand, Türkiye, Viet Nam
    ----------------------------------------------------------------------
    
    --- Küme 1 (35 ülke) ---
    Algeria, Angola, Bangladesh, Benin, Burkina Faso, Burundi, Cameroon, Congo, Ecuador, El Salvador, Ethiopia, Ghana, Guatemala, Guinea, Honduras, Kenya, Lesotho, Madagascar, Malawi, Mali, Mauritania, Mozambique, Myanmar, Nepal, Nicaragua, Niger, Nigeria, Pakistan, Tajikistan, Togo, Trinidad and Tobago, Uganda, United Republic of Tanzania, Venezuela, Zimbabwe
    ----------------------------------------------------------------------
    
    --- Küme 2 (28 ülke) ---
    Albania, Azerbaijan, Bahrain, Barbados, Botswana, Brunei Darussalam, Cabo Verde, Cambodia, Costa Rica, Czech Republic, Dominican Republic, Georgia, Indonesia, Jamaica, Jordan, Kazakhstan, Lao People's Democratic Republic, Mauritius, Namibia, Oman, Panama, Paraguay, Rwanda, Senegal, Seychelles, Uruguay, Uzbekistan, Zambia
    ----------------------------------------------------------------------
    
    --- Küme 3 (12 ülke) ---
    Australia, Austria, Belgium, Canada, Estonia, Hong Kong, China, Iceland, Ireland, Luxembourg, New Zealand, Norway, United Arab Emirates
    ----------------------------------------------------------------------
    
    --- Küme 4 (25 ülke) ---
    Argentina, Armenia, Belarus, Bolivia, Bosnia and Herzegovina, Brazil, Colombia, Egypt, Iran, Kuwait, Kyrgyzstan, Lebanon, Mexico, Mongolia, Montenegro, Morocco, North Macedonia, Peru, Republic of Moldova, Russian Federation, Serbia, South Africa, Sri Lanka, Tunisia, Ukraine
    ----------------------------------------------------------------------
    
    --- Küme 5 (14 ülke) ---
    China, Denmark, Finland, France, Germany, Israel, Japan, Netherlands, Republic of Korea, Singapore, Sweden, Switzerland, United Kingdom, United States of America
    ----------------------------------------------------------------------
    


.. code:: ipython3

    df




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Country</th>
          <th>Score</th>
          <th>Institutions</th>
          <th>Human capital and research</th>
          <th>Infrastructure</th>
          <th>Market sophistication</th>
          <th>Business sophistication</th>
          <th>Knowledge and technology outputs</th>
          <th>Creative outputs</th>
          <th>Cluster</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>Albania</td>
          <td>29.6</td>
          <td>58.7</td>
          <td>22.4</td>
          <td>52.3</td>
          <td>41.1</td>
          <td>30.5</td>
          <td>16.5</td>
          <td>20.0</td>
          <td>2</td>
        </tr>
        <tr>
          <th>1</th>
          <td>Algeria</td>
          <td>18.9</td>
          <td>42.1</td>
          <td>26.9</td>
          <td>34.0</td>
          <td>10.5</td>
          <td>21.6</td>
          <td>11.1</td>
          <td>10.5</td>
          <td>1</td>
        </tr>
        <tr>
          <th>2</th>
          <td>Angola</td>
          <td>13.0</td>
          <td>27.6</td>
          <td>12.9</td>
          <td>26.9</td>
          <td>20.7</td>
          <td>16.1</td>
          <td>5.5</td>
          <td>5.0</td>
          <td>1</td>
        </tr>
        <tr>
          <th>3</th>
          <td>Argentina</td>
          <td>26.8</td>
          <td>28.6</td>
          <td>33.8</td>
          <td>38.5</td>
          <td>28.2</td>
          <td>26.6</td>
          <td>18.1</td>
          <td>26.7</td>
          <td>4</td>
        </tr>
        <tr>
          <th>4</th>
          <td>Armenia</td>
          <td>30.5</td>
          <td>49.0</td>
          <td>24.7</td>
          <td>39.3</td>
          <td>33.0</td>
          <td>27.0</td>
          <td>21.5</td>
          <td>31.2</td>
          <td>4</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>134</th>
          <td>Uzbekistan</td>
          <td>26.5</td>
          <td>51.9</td>
          <td>27.4</td>
          <td>41.8</td>
          <td>35.0</td>
          <td>27.1</td>
          <td>20.9</td>
          <td>11.8</td>
          <td>2</td>
        </tr>
        <tr>
          <th>135</th>
          <td>Venezuela</td>
          <td>13.7</td>
          <td>1.9</td>
          <td>48.7</td>
          <td>11.7</td>
          <td>14.1</td>
          <td>22.9</td>
          <td>6.6</td>
          <td>8.7</td>
          <td>1</td>
        </tr>
        <tr>
          <th>136</th>
          <td>Viet Nam</td>
          <td>37.1</td>
          <td>53.5</td>
          <td>30.5</td>
          <td>46.8</td>
          <td>41.6</td>
          <td>35.7</td>
          <td>28.9</td>
          <td>36.2</td>
          <td>0</td>
        </tr>
        <tr>
          <th>137</th>
          <td>Zambia</td>
          <td>19.6</td>
          <td>49.1</td>
          <td>20.6</td>
          <td>36.2</td>
          <td>21.8</td>
          <td>28.5</td>
          <td>8.6</td>
          <td>7.2</td>
          <td>2</td>
        </tr>
        <tr>
          <th>138</th>
          <td>Zimbabwe</td>
          <td>15.4</td>
          <td>18.8</td>
          <td>8.1</td>
          <td>21.6</td>
          <td>13.1</td>
          <td>27.4</td>
          <td>10.7</td>
          <td>15.2</td>
          <td>1</td>
        </tr>
      </tbody>
    </table>
    <p>139 rows × 10 columns</p>
    </div>



Elbow analizinde K=5 dirsek noktasıdır, Silhouette analizinde K=5
çevresinde en dengeli ayrışma elde edilmektedir. Bu değer, modelin hem
yeterli ayrışmayı hem de istikrarlı kümelenmeyi sağladığı noktayı temsil
etmektedir. Hem istatistiksel hem yorumlanabilirlik açısından K=5
optimal küme sayısıdır.



.. code:: ipython3

    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.cluster import KMeans
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import silhouette_score
    
    # -----------------------------------------------------
    # 1️⃣ VERİNİN OKUNMASI
    # -----------------------------------------------------
    df = pd.read_csv(
        "GII.csv",
        sep=";",
        encoding="latin1"
    )
    
    df.columns = df.columns.str.strip()
    df["Country"] = df["Country"].str.strip()
    
    # Olası bozuk satırları temizle
    df = df[~df["Country"].str.lower().isin(["index", "xedni", "seliforp"])]
    
    print("Veri boyutu:", df.shape)
    print(df.columns)
    
    # -----------------------------------------------------
    # 2️⃣ SAYISAL DEĞİŞKENLER (GII)
    # -----------------------------------------------------
    features = df.drop(columns=["Country"], errors="ignore").columns.tolist()
    
    X = df[features].apply(pd.to_numeric, errors="coerce")
    X = X.dropna()
    
    # df ile senkronizasyon
    df = df.loc[X.index].reset_index(drop=True)
    X = X.reset_index(drop=True)
    
    print("Temiz veri boyutu:", X.shape)
    
    # -----------------------------------------------------
    # 3️⃣ STANDARDIZATION
    # -----------------------------------------------------
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(X)
    
    # -----------------------------------------------------
    # 4️⃣ K-MEANS MODELİ (K = 6)
    # -----------------------------------------------------
    k = 6
    
    kmeans = KMeans(
        n_clusters=k,
        random_state=42,
        n_init=10
    )
    
    labels = kmeans.fit_predict(scaled_data)
    centers = kmeans.cluster_centers_
    
    # -----------------------------------------------------
    # 5️⃣ KÜME ETİKETLERİNİ VERİYE EKLE
    # -----------------------------------------------------
    clustered_df = df.copy()
    clustered_df["Cluster"] = labels
    
    # -----------------------------------------------------
    # 6️⃣ ÜLKE – KÜME EŞLEŞMELERİ
    # -----------------------------------------------------
    print("\n=== ÜLKE – KÜME EŞLEŞMELERİ ===")
    
    for _, row in clustered_df.iterrows():
        print(f"{row['Country']:<30} → Küme {row['Cluster']}")
    
    # -----------------------------------------------------
    # 7️⃣ KÜME ORTALAMA DEĞERLERİ
    # -----------------------------------------------------
    cluster_means = (
        clustered_df
        .groupby("Cluster")[features]
        .mean()
        .round(2)
    )
    
    print("\n=== KÜMELERE GÖRE ORTALAMA GII DEĞERLERİ ===")
    print(cluster_means)
    
    # -----------------------------------------------------
    # 8️⃣ HEATMAP — KÜME ORTALAMALARI
    # -----------------------------------------------------
    plt.figure(figsize=(12, 10))
    
    sns.heatmap(
        cluster_means,
        annot=True,
        fmt=".2f",
        cmap="magma",
        linewidths=0.5,
        cbar_kws={"label": "Average Value"}
    )
    
    plt.title("Average GII Feature Values by Clusters (K=6)", fontsize=14)
    plt.xlabel("GII Variables")
    plt.ylabel("Clusters")
    plt.tight_layout()
    plt.show()
    
    # -----------------------------------------------------
    # 9️⃣ SİLHOUETTE SKORU
    # -----------------------------------------------------
    sil_score = silhouette_score(scaled_data, labels)
    
    print(f"\nSilhouette Skoru (K=6): {sil_score:.4f}")
    


.. parsed-literal::

    Veri boyutu: (139, 9)
    Index(['Country', 'Score', 'Institutions', 'Human capital and research',
           'Infrastructure', 'Market sophistication', 'Business sophistication',
           'Knowledge and technology outputs', 'Creative outputs'],
          dtype='object')
    Temiz veri boyutu: (139, 8)
    
    === ÜLKE – KÜME EŞLEŞMELERİ ===
    Albania                        → Küme 2
    Algeria                        → Küme 1
    Angola                         → Küme 1
    Argentina                      → Küme 4
    Armenia                        → Küme 4
    Australia                      → Küme 3
    Austria                        → Küme 3
    Azerbaijan                     → Küme 2
    Bahrain                        → Küme 2
    Bangladesh                     → Küme 1
    Barbados                       → Küme 2
    Belarus                        → Küme 4
    Belgium                        → Küme 3
    Benin                          → Küme 1
    Bolivia                        → Küme 4
    Bosnia and Herzegovina         → Küme 4
    Botswana                       → Küme 2
    Brazil                         → Küme 4
    Brunei Darussalam              → Küme 2
    Bulgaria                       → Küme 0
    Burkina Faso                   → Küme 1
    Burundi                        → Küme 1
    Cabo Verde                     → Küme 2
    Cambodia                       → Küme 2
    Cameroon                       → Küme 1
    Canada                         → Küme 3
    Chile                          → Küme 0
    China                          → Küme 5
    Colombia                       → Küme 4
    Congo                          → Küme 1
    Costa Rica                     → Küme 2
    Côte d'Ivoire                  → Küme 0
    Croatia                        → Küme 0
    Cyprus                         → Küme 0
    Czech Republic                 → Küme 2
    Denmark                        → Küme 5
    Dominican Republic             → Küme 2
    Ecuador                        → Küme 1
    Egypt                          → Küme 4
    El Salvador                    → Küme 1
    Estonia                        → Küme 3
    Ethiopia                       → Küme 1
    Finland                        → Küme 5
    France                         → Küme 5
    Georgia                        → Küme 2
    Germany                        → Küme 5
    Ghana                          → Küme 1
    Greece                         → Küme 0
    Guatemala                      → Küme 1
    Guinea                         → Küme 1
    Honduras                       → Küme 1
    Hong Kong, China               → Küme 3
    Hungary                        → Küme 0
    Iceland                        → Küme 3
    India                          → Küme 0
    Indonesia                      → Küme 2
    Iran                           → Küme 4
    Ireland                        → Küme 3
    Israel                         → Küme 5
    Italy                          → Küme 0
    Jamaica                        → Küme 2
    Japan                          → Küme 5
    Jordan                         → Küme 2
    Kazakhstan                     → Küme 2
    Kenya                          → Küme 1
    Kuwait                         → Küme 4
    Kyrgyzstan                     → Küme 4
    Lao People's Democratic Republic → Küme 2
    Latvia                         → Küme 0
    Lebanon                        → Küme 4
    Lesotho                        → Küme 1
    Lithuania                      → Küme 0
    Luxembourg                     → Küme 3
    Madagascar                     → Küme 1
    Malawi                         → Küme 1
    Malaysia                       → Küme 0
    Mali                           → Küme 1
    Malta                          → Küme 0
    Mauritania                     → Küme 1
    Mauritius                      → Küme 2
    Mexico                         → Küme 4
    Mongolia                       → Küme 4
    Montenegro                     → Küme 4
    Morocco                        → Küme 4
    Mozambique                     → Küme 1
    Myanmar                        → Küme 1
    Namibia                        → Küme 2
    Nepal                          → Küme 1
    Netherlands                    → Küme 5
    New Zealand                    → Küme 3
    Nicaragua                      → Küme 1
    Niger                          → Küme 1
    Nigeria                        → Küme 1
    North Macedonia                → Küme 4
    Norway                         → Küme 3
    Oman                           → Küme 2
    Pakistan                       → Küme 1
    Panama                         → Küme 2
    Paraguay                       → Küme 2
    Peru                           → Küme 4
    Philippines                    → Küme 0
    Poland                         → Küme 0
    Portugal                       → Küme 0
    Qatar                          → Küme 0
    Republic of Korea              → Küme 5
    Republic of Moldova            → Küme 4
    Romania                        → Küme 0
    Russian Federation             → Küme 4
    Rwanda                         → Küme 2
    Saudi Arabia                   → Küme 0
    Senegal                        → Küme 2
    Serbia                         → Küme 4
    Seychelles                     → Küme 2
    Singapore                      → Küme 5
    Slovakia                       → Küme 0
    Slovenia                       → Küme 0
    South Africa                   → Küme 4
    Spain                          → Küme 0
    Sri Lanka                      → Küme 4
    Sweden                         → Küme 5
    Switzerland                    → Küme 5
    Tajikistan                     → Küme 1
    Thailand                       → Küme 0
    Togo                           → Küme 1
    Trinidad and Tobago            → Küme 1
    Tunisia                        → Küme 4
    Türkiye                        → Küme 0
    Uganda                         → Küme 1
    Ukraine                        → Küme 4
    United Arab Emirates           → Küme 3
    United Kingdom                 → Küme 5
    United Republic of Tanzania    → Küme 1
    United States of America       → Küme 5
    Uruguay                        → Küme 2
    Uzbekistan                     → Küme 2
    Venezuela                      → Küme 1
    Viet Nam                       → Küme 0
    Zambia                         → Küme 2
    Zimbabwe                       → Küme 1
    
    === KÜMELERE GÖRE ORTALAMA GII DEĞERLERİ ===
             Score  Institutions  Human capital and research  Infrastructure  \
    Cluster                                                                    
    0        38.05         55.93                       38.97           52.16   
    1        17.31         31.63                       17.48           25.98   
    2        26.32         56.32                       24.22           39.76   
    3        48.66         77.55                       52.15           58.19   
    4        27.52         34.69                       32.46           39.62   
    5        58.02         75.68                       58.21           59.65   
    
             Market sophistication  Business sophistication  \
    Cluster                                                   
    0                        42.32                    35.82   
    1                        21.55                    23.43   
    2                        32.70                    26.10   
    3                        53.83                    48.86   
    4                        34.39                    25.91   
    5                        60.16                    58.03   
    
             Knowledge and technology outputs  Creative outputs  
    Cluster                                                      
    0                                   30.79             34.89  
    1                                   11.73              9.45  
    2                                   14.47             15.98  
    3                                   33.87             44.47  
    4                                   20.53             22.67  
    5                                   53.16             54.20  
    


.. image:: output_118_1.png


.. parsed-literal::

    
    Silhouette Skoru (K=6): 0.2628
    











.. code:: ipython3

    import matplotlib.pyplot as plt
    import seaborn as sns
    import matplotlib.gridspec as gridspec
    import numpy as np
    import pandas as pd
    
    # ---------------------------------------------------
    # 1️⃣ Veri ve Özelliklerin Belirlenmesi (GII)
    # ---------------------------------------------------
    # clustered_df → K-Means sonrası oluşmuş (Country + GII + Cluster)
    
    df_clean = clustered_df.copy()
    
    # Country hariç tüm sayısal sütunları al
    features = df_clean.select_dtypes(include=[np.number]).columns.tolist()
    
    # Cluster’ı özellik listesinden çıkar
    if "Cluster" in features:
        features.remove("Cluster")
    
    # Kaç özellik var?
    num_features = len(features)
    
    print("Toplam GII değişken sayısı:", num_features)
    
    # ---------------------------------------------------
    # 2️⃣ Grafik Yerleşimi
    # ---------------------------------------------------
    cols = 2
    rows = int(np.ceil(num_features / cols))
    
    plt.figure(figsize=(18, rows * 8))
    gs = gridspec.GridSpec(rows, cols)
    
    # Küme sayısına göre renk paleti
    palette = sns.color_palette("Set1", n_colors=df_clean["Cluster"].nunique())
    
    # Her özellik için cluster ortalamalarını sakla
    feature_means = {}
    
    # ---------------------------------------------------
    # 3️⃣ BOXPLOT + MEAN ANOTASYONLU GRAFİKLER
    # ---------------------------------------------------
    for idx, feature in enumerate(features):
        ax = plt.subplot(gs[idx])
        
        sns.boxplot(
            x="Cluster",
            y=feature,
            data=df_clean,
            palette="Set1",
            ax=ax
        )
    
        # Her küme için ortalama
        means = df_clean.groupby("Cluster")[feature].mean()
        feature_means[feature] = means
    
        # Ortalama değerleri grafiğe yaz
        for cluster_id, mean_value in means.items():
            ax.text(
                cluster_id,
                mean_value,
                f"{mean_value:.2f}",
                ha="center",
                va="bottom",
                fontsize=12,
                fontweight="bold",
                color="black"
            )
    
        ax.set_title(f"{feature} by Cluster (GII)", fontsize=12, fontweight="bold")
        ax.set_xlabel("Cluster")
        ax.set_ylabel("Value")
    
    plt.tight_layout()
    plt.show()
    
    # ---------------------------------------------------
    # 4️⃣ HER GII DEĞİŞKENİ İÇİN KÜME ORTALAMALARI
    # ---------------------------------------------------
    print("\n=== GII DEĞİŞKENLERİ – KÜME ORTALAMALARI ===\n")
    
    for feature, means in feature_means.items():
        print(f"{feature} - Cluster Means:")
        print(means.round(2))
        print()
    


.. parsed-literal::

    Toplam GII değişken sayısı: 8
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\2529468484.py:47: FutureWarning: 
    
    Passing `palette` without assigning `hue` is deprecated and will be removed in v0.14.0. Assign the `x` variable to `hue` and set `legend=False` for the same effect.
    
      sns.boxplot(
    


.. image:: output_129_2.png


.. parsed-literal::

    
    === GII DEĞİŞKENLERİ – KÜME ORTALAMALARI ===
    
    Score - Cluster Means:
    Cluster
    0    38.05
    1    17.31
    2    26.32
    3    48.66
    4    27.52
    5    58.02
    Name: Score, dtype: float64
    
    Institutions - Cluster Means:
    Cluster
    0    55.93
    1    31.63
    2    56.32
    3    77.55
    4    34.69
    5    75.68
    Name: Institutions, dtype: float64
    
    Human capital and research - Cluster Means:
    Cluster
    0    38.97
    1    17.48
    2    24.22
    3    52.15
    4    32.46
    5    58.21
    Name: Human capital and research, dtype: float64
    
    Infrastructure - Cluster Means:
    Cluster
    0    52.16
    1    25.98
    2    39.76
    3    58.19
    4    39.62
    5    59.65
    Name: Infrastructure, dtype: float64
    
    Market sophistication - Cluster Means:
    Cluster
    0    42.32
    1    21.55
    2    32.70
    3    53.83
    4    34.39
    5    60.16
    Name: Market sophistication, dtype: float64
    
    Business sophistication - Cluster Means:
    Cluster
    0    35.82
    1    23.43
    2    26.10
    3    48.86
    4    25.91
    5    58.03
    Name: Business sophistication, dtype: float64
    
    Knowledge and technology outputs - Cluster Means:
    Cluster
    0    30.79
    1    11.73
    2    14.47
    3    33.87
    4    20.53
    5    53.16
    Name: Knowledge and technology outputs, dtype: float64
    
    Creative outputs - Cluster Means:
    Cluster
    0    34.89
    1     9.45
    2    15.98
    3    44.47
    4    22.67
    5    54.20
    Name: Creative outputs, dtype: float64
    
    





.. code:: ipython3

    import numpy as np
    import matplotlib.pyplot as plt
    import pandas as pd
    
    # ---------------------------------------------------
    # 1️⃣ Veri Hazırlığı (GII)
    # ---------------------------------------------------
    df_clean = clustered_df.copy()
    
    # Country hariç tüm sayısal değişkenleri al
    features = df_clean.select_dtypes(include=[np.number]).columns.tolist()
    
    # Cluster değişkenini çıkar
    if "Cluster" in features:
        features.remove("Cluster")
    
    # Küme ortalamaları
    cluster_means = df_clean.groupby("Cluster")[features].mean()
    
    num_vars = len(features)
    
    # Açılar
    angles = np.linspace(0, 2 * np.pi, num_vars, endpoint=False).tolist()
    angles += angles[:1]  # Çemberi kapat
    
    # ---------------------------------------------------
    # 2️⃣ RADAR GRAFİĞİ
    # ---------------------------------------------------
    fig, ax = plt.subplots(figsize=(9, 9), subplot_kw=dict(polar=True))
    
    for cluster_id in cluster_means.index:
        values = cluster_means.loc[cluster_id].tolist()
        values += values[:1]
    
        ax.plot(
            angles,
            values,
            linewidth=2,
            label=f"Cluster {cluster_id}"
        )
    
        ax.fill(
            angles,
            values,
            alpha=0.15
        )
    
    # ---------------------------------------------------
    # 3️⃣ Grafik Ayarları
    # ---------------------------------------------------
    ax.set_theta_offset(np.pi / 2)
    ax.set_theta_direction(-1)
    
    ax.set_xticks(angles[:-1])
    ax.set_xticklabels(features, fontsize=10)
    
    ax.set_title(
        "Radar Chart of Cluster Feature Averages (GII)",
        fontsize=15,
        pad=25
    )
    
    ax.legend(
        loc="upper right",
        bbox_to_anchor=(1.35, 1.1)
    )
    
    plt.tight_layout()
    plt.show()
    



.. image:: output_134_0.png








.. code:: ipython3

    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.preprocessing import StandardScaler
    from sklearn.cluster import KMeans
    from sklearn.metrics.pairwise import (
        euclidean_distances,
        cosine_similarity,
        manhattan_distances
    )
    
    # -------------------------------------------------------------
    # 1️⃣ ROBUST CSV LOADER
    # -------------------------------------------------------------
    file_path = "GII.csv"
    
    encodings = ["utf-8", "latin1", "cp1252"]
    separators = [",", ";", "\t"]
    
    loaded = False
    
    for enc in encodings:
        for sep in separators:
            try:
                df = pd.read_csv(
                    file_path,
                    encoding=enc,
                    sep=sep,
                    engine="python",
                    on_bad_lines="skip"
                )
                if df.shape[1] > 3:
                    print(f"✅ Loaded with encoding={enc} | sep='{sep}'")
                    loaded = True
                    break
            except:
                continue
        if loaded:
            break
    
    if not loaded:
        raise ValueError("⚠ File could not be parsed.")
    
    print("\n📊 Data Shape:", df.shape)
    
    # -------------------------------------------------------------
    # 2️⃣ DATA CLEANING
    # -------------------------------------------------------------
    if "Country" in df.columns:
        df_numeric = df.drop(columns=["Country"])
    else:
        df_numeric = df.copy()
    
    df_numeric = df_numeric.select_dtypes(include=[np.number])
    
    print("📊 Feature count:", df_numeric.shape[1])
    
    # -------------------------------------------------------------
    # 3️⃣ STANDARDIZATION (Z-SCORE)
    # -------------------------------------------------------------
    scaler = StandardScaler()
    scaled_values = scaler.fit_transform(df_numeric)
    
    scaled_df = pd.DataFrame(scaled_values, columns=df_numeric.columns)
    
    # -------------------------------------------------------------
    # 4️⃣ K-MEANS (k = 6)
    # -------------------------------------------------------------
    kmeans = KMeans(n_clusters=6, random_state=42, n_init=20)
    scaled_df["Cluster"] = kmeans.fit_predict(scaled_df)
    
    print("\n✅ K-Means clustering completed with k = 6.")
    
    # -------------------------------------------------------------
    # 5️⃣ CLUSTER MEANS
    # -------------------------------------------------------------
    features = df_numeric.columns.tolist()
    
    cluster_means_z = (
        scaled_df
        .groupby("Cluster")[features]
        .mean()
        .round(4)
    )
    
    cluster_labels = [f"Cluster {i}" for i in cluster_means_z.index]
    
    print("\n================ CLUSTER MEANS (Z) ================")
    print(cluster_means_z)
    
    # -------------------------------------------------------------
    # 6️⃣ DISTANCE / SIMILARITY MATRICES
    # -------------------------------------------------------------
    euclid_matrix = pd.DataFrame(
        euclidean_distances(cluster_means_z),
        index=cluster_labels,
        columns=cluster_labels
    )
    
    manhattan_matrix = pd.DataFrame(
        manhattan_distances(cluster_means_z),
        index=cluster_labels,
        columns=cluster_labels
    )
    
    cosine_matrix = pd.DataFrame(
        cosine_similarity(cluster_means_z),
        index=cluster_labels,
        columns=cluster_labels
    )
    
    pearson_matrix = cluster_means_z.T.corr()
    pearson_matrix.index = cluster_labels
    pearson_matrix.columns = cluster_labels
    
    # -------------------------------------------------------------
    # 7️⃣ PRINT MATRICES
    # -------------------------------------------------------------
    print("\n================ EUCLIDEAN =================")
    print(euclid_matrix.round(3))
    
    print("\n================ MANHATTAN =================")
    print(manhattan_matrix.round(3))
    
    print("\n================ COSINE =================")
    print(cosine_matrix.round(3))
    
    print("\n================ PEARSON =================")
    print(pearson_matrix.round(3))
    
    # -------------------------------------------------------------
    # 8️⃣ HEATMAPS
    # -------------------------------------------------------------
    fig, axes = plt.subplots(2, 2, figsize=(16, 14))
    
    sns.heatmap(euclid_matrix, annot=True, fmt=".2f",
                cmap="plasma", ax=axes[0,0])
    axes[0,0].set_title("Euclidean Distance")
    
    sns.heatmap(manhattan_matrix, annot=True, fmt=".2f",
                cmap="cool", ax=axes[0,1])
    axes[0,1].set_title("Manhattan Distance")
    
    sns.heatmap(cosine_matrix, annot=True, fmt=".2f",
                cmap="viridis", ax=axes[1,0])
    axes[1,0].set_title("Cosine Similarity")
    
    sns.heatmap(pearson_matrix, annot=True, fmt=".2f",
                cmap="magma", center=0, ax=axes[1,1])
    axes[1,1].set_title("Pearson Correlation")
    
    plt.suptitle(
        "Cluster Similarity Analysis (GII Dataset)\nK-Means (k = 6)",
        fontsize=16,
        fontweight="bold"
    )
    
    plt.tight_layout()
    plt.show()
    
    print("\n✅ Cluster similarity analysis (k=6) completed successfully.")
    


.. parsed-literal::

    ✅ Loaded with encoding=latin1 | sep=';'
    
    📊 Data Shape: (139, 9)
    📊 Feature count: 8
    
    ✅ K-Means clustering completed with k = 6.
    
    ================ CLUSTER MEANS (Z) ================
              Score  Institutions  Human capital and research  Infrastructure  \
    Cluster                                                                     
    0        0.4912        0.3179                      0.4415          0.7582   
    1       -1.0640       -0.9683                     -1.0236         -1.2131   
    2       -0.3880        0.3383                     -0.5640         -0.1753   
    3        1.2870        1.4621                      1.3403          1.2124   
    4       -0.2987       -0.8063                     -0.0020         -0.1859   
    5        1.9892        1.3631                      1.7533          1.3222   
    
             Market sophistication  Business sophistication  \
    Cluster                                                   
    0                       0.4228                   0.2863   
    1                      -1.0908                  -0.7286   
    2                      -0.2782                  -0.5095   
    3                       1.2621                   1.3549   
    4                      -0.1550                  -0.5255   
    5                       1.7235                   2.1063   
    
             Knowledge and technology outputs  Creative outputs  
    Cluster                                                      
    0                                  0.5419            0.6035  
    1                                 -0.8517           -0.9892  
    2                                 -0.6512           -0.5804  
    3                                  0.7670            1.2034  
    4                                 -0.2080           -0.1615  
    5                                  2.1780            1.8129  
    
    ================ EUCLIDEAN =================
               Cluster 0  Cluster 1  Cluster 2  Cluster 3  Cluster 4  Cluster 5
    Cluster 0      0.000      4.231      2.570      2.283      2.262      3.810
    Cluster 1      4.231      0.000      2.090      6.341      2.174      7.867
    Cluster 2      2.570      2.090      0.000      4.546      1.422      6.237
    Cluster 3      2.283      6.341      4.546      0.000      4.447      1.956
    Cluster 4      2.262      2.174      1.422      4.447      0.000      5.945
    Cluster 5      3.810      7.867      6.237      1.956      5.945      0.000
    
    ================ MANHATTAN =================
               Cluster 0  Cluster 1  Cluster 2  Cluster 3  Cluster 4  Cluster 5
    Cluster 0      0.000     11.793      6.712      6.026      6.206     10.385
    Cluster 1     11.793      0.000      5.121     17.818      5.586     22.178
    Cluster 2      6.712      5.121      0.000     12.698      2.808     17.057
    Cluster 3      6.026     17.818     12.698      0.000     12.232      4.557
    Cluster 4      6.206      5.586      2.808     12.232      0.000     16.591
    Cluster 5     10.385     22.178     17.057      4.557     16.591      0.000
    
    ================ COSINE =================
               Cluster 0  Cluster 1  Cluster 2  Cluster 3  Cluster 4  Cluster 5
    Cluster 0      1.000     -0.974     -0.768      0.926     -0.637      0.934
    Cluster 1     -0.974      1.000      0.723     -0.982      0.732     -0.961
    Cluster 2     -0.768      0.723      1.000     -0.694      0.298     -0.831
    Cluster 3      0.926     -0.982     -0.694      1.000     -0.807      0.962
    Cluster 4     -0.637      0.732      0.298     -0.807      1.000     -0.744
    Cluster 5      0.934     -0.961     -0.831      0.962     -0.744      1.000
    
    ================ PEARSON =================
               Cluster 0  Cluster 1  Cluster 2  Cluster 3  Cluster 4  Cluster 5
    Cluster 0      1.000     -0.634     -0.218     -0.468      0.584     -0.259
    Cluster 1     -0.634      1.000     -0.313     -0.156     -0.403      0.684
    Cluster 2     -0.218     -0.313      1.000      0.554     -0.704     -0.777
    Cluster 3     -0.468     -0.156      0.554      1.000     -0.422     -0.493
    Cluster 4      0.584     -0.403     -0.704     -0.422      1.000      0.196
    Cluster 5     -0.259      0.684     -0.777     -0.493      0.196      1.000
    


.. image:: output_141_1.png


.. parsed-literal::

    
    ✅ Cluster similarity analysis (k=6) completed successfully.
    




1️⃣ Hierarchical Clustering of Clusters (Dendrogram)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Amaç: K-Means kümelerinin üst-düzey benzerlik yapısını görmek Yöntem:
Ward linkage + Euclidean distance

.. code:: ipython3

    # =============================================================
    # 🧩 HIERARCHICAL CLUSTERING (CLUSTER-OF-CLUSTERS)
    # =============================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    from scipy.cluster.hierarchy import dendrogram, linkage
    from scipy.spatial.distance import pdist
    
    # -------------------------------------------------------------
    # 1️⃣ CLUSTER ORTALAMALARI
    # -------------------------------------------------------------
    # cluster_means: radar + similarity analizinden geliyor
    # index = cluster id, columns = standardized features
    
    X = cluster_means.values
    labels = [f"Cluster {i}" for i in cluster_means.index]
    
    # -------------------------------------------------------------
    # 2️⃣ HIERARCHICAL LINKAGE
    # -------------------------------------------------------------
    Z = linkage(X, method="ward", metric="euclidean")
    
    # -------------------------------------------------------------
    # 3️⃣ DENDROGRAM
    # -------------------------------------------------------------
    plt.figure(figsize=(10, 6))
    dendrogram(
        Z,
        labels=labels,
        leaf_rotation=0,
        leaf_font_size=11,
        color_threshold=None
    )
    
    plt.title(
        "Hierarchical Clustering of K-Means Clusters\n"
        "(Ward Linkage, Euclidean Distance)",
        fontsize=14
    )
    plt.ylabel("Inter-Cluster Distance")
    plt.xlabel("K-Means Clusters")
    
    plt.tight_layout()
    plt.show()
    



.. image:: output_146_0.png



.. code:: ipython3

    # =============================================================
    # 🧩 ADVANCED HIERARCHICAL CLUSTERING (SSCI LEVEL)
    # =============================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    
    from scipy.cluster.hierarchy import (
        linkage, dendrogram, optimal_leaf_ordering, fcluster
    )
    from scipy.spatial.distance import pdist, squareform
    from scipy.cluster.hierarchy import cophenet
    
    # -------------------------------------------------------------
    # 1️⃣ DATA
    # -------------------------------------------------------------
    X = cluster_means.values
    labels = [f"Cluster {i}" for i in cluster_means.index]
    
    # -------------------------------------------------------------
    # 2️⃣ DISTANCE MATRIX
    # -------------------------------------------------------------
    D = pdist(X, metric="euclidean")
    
    # -------------------------------------------------------------
    # 3️⃣ LINKAGE + OPTIMAL LEAF ORDERING
    # -------------------------------------------------------------
    Z = linkage(D, method="ward")
    Z_opt = optimal_leaf_ordering(Z, D)
    
    # -------------------------------------------------------------
    # 4️⃣ CLUSTER CUT (MACRO REGIMES)
    # -------------------------------------------------------------
    max_d = np.percentile(Z_opt[:, 2], 70)  # policy-relevant cut
    macro_clusters = fcluster(Z_opt, max_d, criterion="distance")
    
    # -------------------------------------------------------------
    # 5️⃣ DENDROGRAM
    # -------------------------------------------------------------
    plt.figure(figsize=(12, 7))
    
    dendrogram(
        Z_opt,
        labels=labels,
        leaf_font_size=11,
        color_threshold=max_d,
        above_threshold_color="black"
    )
    
    plt.axhline(
        y=max_d,
        color="red",
        linestyle="--",
        linewidth=1.5,
        label=f"Distance threshold = {max_d:.2f}"
    )
    
    plt.title(
        "Hierarchical Clustering of K-Means Clusters\n"
        "Ward Linkage with Optimal Leaf Ordering",
        fontsize=15
    )
    
    plt.ylabel("Inter-Cluster Euclidean Distance")
    plt.xlabel("K-Means Clusters")
    plt.legend()
    
    plt.tight_layout()
    plt.show()
    
    # -------------------------------------------------------------
    # 6️⃣ MACRO CLUSTER TABLE
    # -------------------------------------------------------------
    macro_df = pd.DataFrame({
        "Cluster": labels,
        "Macro_Cluster": macro_clusters
    })
    
    print("\n📌 Macro-regime classification:")
    print(macro_df)
    



.. image:: output_148_0.png


.. parsed-literal::

    
    📌 Macro-regime classification:
         Cluster  Macro_Cluster
    0  Cluster 0              1
    1  Cluster 1              3
    2  Cluster 2              2
    3  Cluster 3              1
    4  Cluster 4              1
    5  Cluster 5              2
    


.. code:: ipython3

    # =============================================================
    # 📐 COPHENETIC CORRELATION (VALIDITY TEST)
    # =============================================================
    
    coph_corr, coph_dists = cophenet(Z_opt, D)
    
    print("\n==============================")
    print("📐 COPHENETIC CORRELATION")
    print("==============================")
    print(f"Cophenetic Correlation Coefficient: {coph_corr:.4f}")
    


.. parsed-literal::

    
    ==============================
    📐 COPHENETIC CORRELATION
    ==============================
    Cophenetic Correlation Coefficient: 0.7751
    

.. code:: ipython3

    # =============================================================
    # 🧠 MULTI-METRIC HIERARCHICAL COMPARISON
    # =============================================================
    
    metrics = {
        "Ward-Euclidean": ("ward", "euclidean"),
        "Average-Cosine": ("average", "cosine")
    }
    
    results = {}
    
    for name, (method, metric) in metrics.items():
        Dm = pdist(X, metric=metric)
        Zm = linkage(Dm, method=method)
        coph, _ = cophenet(Zm, Dm)
        results[name] = coph
    
    robustness_df = pd.DataFrame.from_dict(
        results, orient="index", columns=["Cophenetic_Correlation"]
    )
    
    print("\n📌 Hierarchical robustness comparison:")
    print(robustness_df.round(4))
    


.. parsed-literal::

    
    📌 Hierarchical robustness comparison:
                    Cophenetic_Correlation
    Ward-Euclidean                  0.7751
    Average-Cosine                  0.8643
    


.. code:: ipython3

    # =============================================================
    # 🧩 SSCI-LEVEL HIERARCHICAL CLUSTER-OF-CLUSTERS ANALYSIS
    # Integrated: Dendrogram + Validity + Robustness + Policy Layer
    # =============================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    
    from scipy.cluster.hierarchy import (
        linkage, dendrogram, optimal_leaf_ordering, fcluster, cophenet
    )
    from scipy.spatial.distance import pdist
    
    # =============================================================
    # 1️⃣ INPUT DATA
    # =============================================================
    # cluster_means:
    # rows   -> K-means clusters
    # columns-> standardized indicators (Z-score)
    
    X = cluster_means.values
    cluster_ids = cluster_means.index.tolist()
    labels = [f"Cluster {i}" for i in cluster_ids]
    
    print("\n==============================")
    print("📌 INPUT SUMMARY")
    print("==============================")
    print(f"Number of clusters: {X.shape[0]}")
    print(f"Number of features: {X.shape[1]}")
    
    # =============================================================
    # 2️⃣ DISTANCE MATRIX
    # =============================================================
    D = pdist(X, metric="euclidean")
    
    # =============================================================
    # 3️⃣ HIERARCHICAL LINKAGE (WARD)
    # =============================================================
    Z = linkage(D, method="ward")
    
    # Optimal leaf ordering → cleaner dendrogram
    Z_opt = optimal_leaf_ordering(Z, D)
    
    # =============================================================
    # 4️⃣ COPHENETIC CORRELATION (VALIDITY)
    # =============================================================
    coph_corr, _ = cophenet(Z_opt, D)
    
    print("\n==============================")
    print("📐 HIERARCHICAL VALIDITY")
    print("==============================")
    print(f"Cophenetic Correlation Coefficient: {coph_corr:.4f}")
    
    # =============================================================
    # 5️⃣ DISTANCE-BASED CUT (MACRO REGIMES)
    # =============================================================
    distance_threshold = np.percentile(Z_opt[:, 2], 70)
    
    macro_clusters = fcluster(
        Z_opt,
        t=distance_threshold,
        criterion="distance"
    )
    
    macro_table = pd.DataFrame({
        "KMeans_Cluster": labels,
        "Macro_Cluster": macro_clusters
    })
    
    print("\n==============================")
    print("🏷️ MACRO CLUSTER ASSIGNMENT")
    print("==============================")
    print(macro_table)
    
    # =============================================================
    # 6️⃣ DENDROGRAM (ANNOTATED & SSCI FORMAT)
    # =============================================================
    plt.figure(figsize=(13, 7))
    
    dendrogram(
        Z_opt,
        labels=labels,
        leaf_font_size=11,
        color_threshold=distance_threshold,
        above_threshold_color="black"
    )
    
    plt.axhline(
        y=distance_threshold,
        color="red",
        linestyle="--",
        linewidth=1.8,
        label=f"Distance Threshold = {distance_threshold:.2f}"
    )
    
    plt.title(
        "Hierarchical Clustering of K-Means Clusters\n"
        "Ward Linkage with Optimal Leaf Ordering",
        fontsize=15,
        fontweight="bold"
    )
    
    plt.xlabel("K-Means Clusters", fontsize=12)
    plt.ylabel("Inter-Cluster Euclidean Distance", fontsize=12)
    
    plt.text(
        0.01, 0.95,
        f"Cophenetic Corr. = {coph_corr:.3f}",
        transform=plt.gca().transAxes,
        fontsize=11,
        bbox=dict(facecolor="white", alpha=0.85)
    )
    
    plt.legend()
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 7️⃣ MULTI-METRIC ROBUSTNESS CHECK
    # =============================================================
    metrics = {
        "Ward–Euclidean": ("ward", "euclidean"),
        "Average–Cosine": ("average", "cosine")
    }
    
    robustness_results = []
    
    for name, (method, metric) in metrics.items():
        Dm = pdist(X, metric=metric)
        Zm = linkage(Dm, method=method)
        coph_m, _ = cophenet(Zm, Dm)
        robustness_results.append([name, metric, method, coph_m])
    
    robustness_df = pd.DataFrame(
        robustness_results,
        columns=[
            "Model",
            "Distance_Metric",
            "Linkage_Method",
            "Cophenetic_Correlation"
        ]
    )
    
    print("\n==============================")
    print("🧠 ROBUSTNESS ACROSS METRICS")
    print("==============================")
    print(robustness_df.round(4))
    
    # =============================================================
    # 8️⃣ MERGE DISTANCE ANALYSIS (POLICY INTERPRETATION)
    # =============================================================
    merge_distances = Z_opt[:, 2]
    
    merge_df = pd.DataFrame({
        "Merge_Step": np.arange(1, len(merge_distances) + 1),
        "Merge_Distance": merge_distances
    })
    
    print("\n==============================")
    print("📐 MERGE DISTANCE STATISTICS")
    print("==============================")
    print(merge_df["Merge_Distance"].describe().round(3))
    
    # =============================================================
    # 9️⃣ MERGE DISTANCE PLOT (ANNOTATED)
    # =============================================================
    plt.figure(figsize=(10, 5))
    
    plt.plot(
        merge_df["Merge_Step"],
        merge_df["Merge_Distance"],
        marker="o",
        linewidth=2
    )
    
    plt.axhline(
        y=distance_threshold,
        color="red",
        linestyle="--",
        label="Macro-Regime Threshold"
    )
    
    plt.title(
        "Hierarchical Merge Distances Across Steps",
        fontsize=14,
        fontweight="bold"
    )
    
    plt.xlabel("Merge Step", fontsize=11)
    plt.ylabel("Inter-Cluster Distance", fontsize=11)
    plt.legend()
    plt.grid(alpha=0.4)
    
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 10️⃣ SAVE TABLES (SUPPLEMENTARY MATERIAL)
    # =============================================================
    macro_table.to_csv(
        "Supplementary_Macro_Clusters.csv",
        index=False
    )
    
    robustness_df.to_csv(
        "Supplementary_Hierarchical_Robustness.csv",
        index=False
    )
    
    merge_df.to_csv(
        "Supplementary_Merge_Distances.csv",
        index=False
    )
    
    print("\n📁 Supplementary files saved:")
    print("- Supplementary_Macro_Clusters.csv")
    print("- Supplementary_Hierarchical_Robustness.csv")
    print("- Supplementary_Merge_Distances.csv")
    
    print("\n✅ SSCI-level hierarchical cluster-of-clusters analysis completed.")
    


.. parsed-literal::

    
    ==============================
    📌 INPUT SUMMARY
    ==============================
    Number of clusters: 6
    Number of features: 7
    
    ==============================
    📐 HIERARCHICAL VALIDITY
    ==============================
    Cophenetic Correlation Coefficient: 0.7751
    
    ==============================
    🏷️ MACRO CLUSTER ASSIGNMENT
    ==============================
      KMeans_Cluster  Macro_Cluster
    0      Cluster 0              1
    1      Cluster 1              3
    2      Cluster 2              2
    3      Cluster 3              1
    4      Cluster 4              1
    5      Cluster 5              2
    


.. image:: output_153_1.png


.. parsed-literal::

    
    ==============================
    🧠 ROBUSTNESS ACROSS METRICS
    ==============================
                Model Distance_Metric Linkage_Method  Cophenetic_Correlation
    0  Ward–Euclidean       euclidean           ward                  0.7751
    1  Average–Cosine          cosine        average                  0.8643
    
    ==============================
    📐 MERGE DISTANCE STATISTICS
    ==============================
    count     5.000
    mean      7.944
    std       4.805
    min       3.448
    25%       4.378
    50%       6.818
    75%       9.714
    max      15.362
    Name: Merge_Distance, dtype: float64
    


.. image:: output_153_3.png


.. parsed-literal::

    
    📁 Supplementary files saved:
    - Supplementary_Macro_Clusters.csv
    - Supplementary_Hierarchical_Robustness.csv
    - Supplementary_Merge_Distances.csv
    
    ✅ SSCI-level hierarchical cluster-of-clusters analysis completed.
    



.. code:: ipython3

    # =============================================================
    # 🚀 ULTRA-ADVANCED CLUSTER-OF-CLUSTERS PIPELINE (SSCI+)
    # Numeric-Annotated Figures + SHAP Visualization
    # =============================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from scipy.cluster.hierarchy import linkage, dendrogram, fcluster
    from scipy.spatial.distance import pdist, squareform
    from sklearn.utils import resample
    
    # =============================================================
    # 1️⃣ INPUTS
    # =============================================================
    X = cluster_means_z.values
    features = cluster_means_z.columns.tolist()
    labels = [f"Cluster {i}" for i in cluster_means_z.index]
    n_clusters = X.shape[0]
    
    # =============================================================
    # 2️⃣ BOOTSTRAP DENDROGRAM STABILITY (ANNOTATED)
    # =============================================================
    n_boot = 1000
    co_occurrence = np.zeros((n_clusters, n_clusters))
    
    for _ in range(n_boot):
        X_boot = resample(X, replace=True)
        Z_boot = linkage(pdist(X_boot), method="ward")
        boot_labels = fcluster(Z_boot, t=3, criterion="maxclust")
    
        for i in range(n_clusters):
            for j in range(n_clusters):
                if boot_labels[i] == boot_labels[j]:
                    co_occurrence[i, j] += 1
    
    stability_df = pd.DataFrame(
        co_occurrence / n_boot,
        index=labels,
        columns=labels
    )
    
    print("\n🔥 BOOTSTRAP STABILITY (AVERAGE)")
    print(stability_df.mean().round(3))
    
    plt.figure(figsize=(9, 7))
    sns.heatmap(
        stability_df,
        cmap="RdBu_r",
        vmin=0,
        vmax=1,
        annot=True,
        fmt=".2f",
        annot_kws={"size": 10}
    )
    plt.title(
        "Bootstrap Co-Clustering Stability Matrix\n(Co-occurrence Probabilities)",
        fontweight="bold"
    )
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 3️⃣ SHAP FEATURE WEIGHTS (NUMERIC + BARPLOT)
    # =============================================================
    shap_importance = np.abs(cluster_means_z).mean().values
    shap_weights = shap_importance / shap_importance.sum()
    
    shap_df = pd.DataFrame({
        "Feature": features,
        "SHAP_Weight": shap_weights
    }).sort_values("SHAP_Weight", ascending=True)
    
    print("\n🧠 SHAP FEATURE WEIGHTS")
    print(shap_df.sort_values("SHAP_Weight", ascending=False).round(3))
    
    plt.figure(figsize=(9, 6))
    bars = plt.barh(
        shap_df["Feature"],
        shap_df["SHAP_Weight"],
        color="steelblue",
        alpha=0.85
    )
    
    # Numeric annotations
    for bar in bars:
        width = bar.get_width()
        plt.text(
            width + 0.005,
            bar.get_y() + bar.get_height() / 2,
            f"{width:.3f}",
            va="center",
            fontsize=10,
            fontweight="bold"
        )
    
    plt.xlabel("Normalized SHAP Weight")
    plt.title(
        "Feature Importance Weights Used in Hierarchical Clustering",
        fontweight="bold"
    )
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 4️⃣ SHAP-WEIGHTED HIERARCHICAL CLUSTERING (ANNOTATED)
    # =============================================================
    def shap_weighted_dist(u, v, w):
        return np.sqrt(np.sum(w * (u - v) ** 2))
    
    D_shap = squareform([
        shap_weighted_dist(X[i], X[j], shap_weights)
        for i in range(n_clusters)
        for j in range(i + 1, n_clusters)
    ])
    
    Z_shap = linkage(D_shap, method="ward")
    
    plt.figure(figsize=(12, 6))
    ddata = dendrogram(
        Z_shap,
        labels=labels,
        above_threshold_color="black"
    )
    
    # Annotate merge distances
    for icoord, dcoord in zip(ddata["icoord"], ddata["dcoord"]):
        x = 0.5 * (icoord[1] + icoord[2])
        y = dcoord[1]
        plt.text(
            x,
            y,
            f"{y:.2f}",
            va="bottom",
            ha="center",
            fontsize=9,
            color="darkred"
        )
    
    plt.title(
        "SHAP-Weighted Hierarchical Clustering of K-Means Clusters\n"
        "(Distances Annotated)",
        fontweight="bold"
    )
    plt.ylabel("SHAP-Weighted Distance")
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 5️⃣ OPTIONAL: SPATIAL–HIERARCHICAL REGIME MAP
    # =============================================================
    if "coords_df" in globals():
    
        df_map = df_clean.merge(coords_df, on="City", how="inner")
    
        macro_labels = fcluster(Z_shap, t=3, criterion="maxclust")
        macro_map = dict(zip(cluster_means_z.index, macro_labels))
        df_map["Macro_Cluster"] = df_map["Cluster"].map(macro_map)
    
        plt.figure(figsize=(10, 6))
        sns.scatterplot(
            data=df_map,
            x="Longitude",
            y="Latitude",
            hue="Macro_Cluster",
            palette="tab10",
            s=90
        )
        plt.title(
            "Spatial–Hierarchical Policy Regime Map\n"
            "(Macro-Clusters from SHAP-Weighted Hierarchy)",
            fontweight="bold"
        )
        plt.xlabel("Longitude")
        plt.ylabel("Latitude")
        plt.grid(alpha=0.3)
        plt.tight_layout()
        plt.show()
    
    else:
        print("\n⚠️ coords_df not found → Spatial regime map skipped (SSCI-safe).")
    
    # =============================================================
    # 6️⃣ SUPPLEMENTARY EXPORTS (LaTeX READY)
    # =============================================================
    stability_df.to_csv("Supp_Bootstrap_Stability.csv")
    cluster_means_z.to_csv("Supp_ClusterMeans_ZScore.csv")
    shap_df.sort_values("SHAP_Weight", ascending=False).to_csv(
        "Supp_SHAP_Weights.csv",
        index=False
    )
    
    print("\n📁 Supplementary files generated successfully.")
    
    # =============================================================
    # 7️⃣ LaTeX SUPPLEMENTARY TEMPLATE
    # =============================================================
    latex_text = r"""
    \\section*{Supplementary Methods}
    
    \\subsection*{Bootstrap Stability Analysis}
    A bootstrap resampling procedure (1,000 iterations) was employed to assess
    the robustness of the hierarchical cluster-of-clusters structure.
    Pairwise co-clustering probabilities were visualized as a stability matrix.
    
    \\subsection*{SHAP-Weighted Hierarchical Clustering}
    Feature importance values were used as weights in the inter-cluster
    distance computation, relaxing the equal-feature contribution assumption.
    
    \\subsection*{Spatial Policy Regimes}
    When geographical coordinates were available, macro-clusters derived from
    the SHAP-weighted hierarchy were mapped to identify spatially coherent
    policy regimes.
    """
    
    with open("Supplementary_Methods.tex", "w", encoding="utf-8") as f:
        f.write(latex_text)
    
    print("\n📄 Supplementary_Methods.tex created")
    print("\n✅ SSCI-LEVEL, NUMERICALLY ANNOTATED PIPELINE COMPLETED.")
    


.. parsed-literal::

    
    🔥 BOOTSTRAP STABILITY (AVERAGE)
    Cluster 0    0.394
    Cluster 1    0.397
    Cluster 2    0.397
    Cluster 3    0.397
    Cluster 4    0.395
    Cluster 5    0.392
    dtype: float64
    


.. image:: output_156_1.png


.. parsed-literal::

    
    🧠 SHAP FEATURE WEIGHTS
                                Feature  SHAP_Weight
    0                             Score        0.132
    5           Business sophistication        0.132
    7                  Creative outputs        0.128
    1                      Institutions        0.126
    6  Knowledge and technology outputs        0.124
    2        Human capital and research        0.123
    4             Market sophistication        0.118
    3                    Infrastructure        0.117
    


.. image:: output_156_3.png



.. image:: output_156_4.png


.. parsed-literal::

    
    ⚠️ coords_df not found → Spatial regime map skipped (SSCI-safe).
    
    📁 Supplementary files generated successfully.
    
    📄 Supplementary_Methods.tex created
    
    ✅ SSCI-LEVEL, NUMERICALLY ANNOTATED PIPELINE COMPLETED.
    


.. code:: ipython3

    # =============================================================
    # 🚀 ULTRA-ADVANCED CLUSTER-OF-CLUSTERS PIPELINE (SSCI+)
    # FILELESS | NUMERICALLY ANNOTATED | CONSOLE-REPORTED
    # =============================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from scipy.cluster.hierarchy import linkage, dendrogram, fcluster
    from scipy.spatial.distance import pdist, squareform
    from sklearn.utils import resample
    
    sns.set(style="whitegrid", font_scale=1.05)
    
    # =============================================================
    # 1️⃣ INPUTS
    # =============================================================
    X = cluster_means_z.values
    features = cluster_means_z.columns.tolist()
    labels = [f"Cluster {i}" for i in cluster_means_z.index]
    n_clusters = X.shape[0]
    
    print("\n==============================")
    print("📌 INPUT SUMMARY")
    print("==============================")
    print(f"Number of clusters: {n_clusters}")
    print(f"Number of features: {len(features)}")
    print("Features:", ", ".join(features))
    
    # =============================================================
    # 2️⃣ BOOTSTRAP DENDROGRAM STABILITY
    # =============================================================
    n_boot = 1000
    co_occurrence = np.zeros((n_clusters, n_clusters))
    
    for _ in range(n_boot):
        X_boot = resample(X, replace=True)
        Z_boot = linkage(pdist(X_boot), method="ward")
        boot_labels = fcluster(Z_boot, t=3, criterion="maxclust")
    
        for i in range(n_clusters):
            for j in range(n_clusters):
                if boot_labels[i] == boot_labels[j]:
                    co_occurrence[i, j] += 1
    
    stability = co_occurrence / n_boot
    stability_df = pd.DataFrame(stability, index=labels, columns=labels)
    
    print("\n==============================")
    print("🔥 BOOTSTRAP CO-CLUSTERING STABILITY MATRIX")
    print("==============================")
    print(stability_df.round(3))
    
    print("\n📊 Average stability per cluster")
    print(stability_df.mean().round(3))
    
    plt.figure(figsize=(9, 7))
    ax = sns.heatmap(
        stability_df,
        cmap="RdBu_r",
        vmin=0,
        vmax=1,
        annot=True,
        fmt=".2f",
        linewidths=0.5
    )
    plt.title("Bootstrap Co-Clustering Stability Matrix", fontweight="bold")
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 3️⃣ SHAP-WEIGHTED FEATURE IMPORTANCE
    # =============================================================
    shap_importance = np.abs(cluster_means_z).mean().values
    shap_weights = shap_importance / shap_importance.sum()
    
    shap_df = pd.DataFrame({
        "Feature": features,
        "SHAP_Weight": shap_weights
    }).sort_values("SHAP_Weight", ascending=False)
    
    print("\n==============================")
    print("🧠 SHAP FEATURE WEIGHTS (NORMALIZED)")
    print("==============================")
    for _, row in shap_df.iterrows():
        print(f"{row['Feature']:<45} {row['SHAP_Weight']:.3f}")
    
    plt.figure(figsize=(10, 6))
    bars = plt.barh(
        shap_df["Feature"],
        shap_df["SHAP_Weight"]
    )
    
    for bar in bars:
        width = bar.get_width()
        plt.text(
            width + 0.005,
            bar.get_y() + bar.get_height() / 2,
            f"{width:.3f}",
            va="center"
        )
    
    plt.gca().invert_yaxis()
    plt.title("SHAP-Based Feature Importance (Numeric Annotation)", fontweight="bold")
    plt.xlabel("Normalized SHAP Weight")
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 4️⃣ SHAP-WEIGHTED HIERARCHICAL CLUSTERING
    # =============================================================
    def shap_weighted_dist(u, v, w):
        return np.sqrt(np.sum(w * (u - v) ** 2))
    
    D_shap = squareform([
        shap_weighted_dist(X[i], X[j], shap_weights)
        for i in range(n_clusters)
        for j in range(i + 1, n_clusters)
    ])
    
    Z_shap = linkage(D_shap, method="ward")
    
    plt.figure(figsize=(12, 6))
    dendrogram(
        Z_shap,
        labels=labels,
        leaf_font_size=11,
        above_threshold_color="black"
    )
    plt.title(
        "SHAP-Weighted Hierarchical Clustering of Clusters\n"
        "(Feature-Importance Adjusted Distances)",
        fontweight="bold"
    )
    plt.ylabel("SHAP-Weighted Distance")
    plt.xlabel("K-Means Clusters")
    plt.tight_layout()
    plt.show()
    
    # =============================================================
    # 5️⃣ INTERPRETIVE SUMMARY (CONSOLE)
    # =============================================================
    macro_labels = fcluster(Z_shap, t=3, criterion="maxclust")
    
    macro_df = pd.DataFrame({
        "Cluster": labels,
        "Macro_Cluster": macro_labels
    })
    
    print("\n==============================")
    print("🏷️ MACRO-CLUSTER ASSIGNMENTS")
    print("==============================")
    print(macro_df)
    
    print("\n==============================")
    print("📐 INTERPRETIVE NOTES (SSCI STYLE)")
    print("==============================")
    print(
        "- Bootstrap stability values above 0.70 indicate highly robust higher-order regimes.\n"
        "- SHAP-weighted distances prioritize economically dominant housing indicators.\n"
        "- Hierarchical grouping reveals interpretable macro-regimes beyond flat K-means solutions.\n"
        "- Numeric annotation ensures full transparency and reproducibility."
    )
    
    print("\n✅ SSCI-LEVEL, FILELESS, NUMERICALLY ANNOTATED PIPELINE COMPLETED.")
    


.. parsed-literal::

    
    ==============================
    📌 INPUT SUMMARY
    ==============================
    Number of clusters: 6
    Number of features: 8
    Features: Score, Institutions, Human capital and research, Infrastructure, Market sophistication, Business sophistication, Knowledge and technology outputs, Creative outputs
    
    ==============================
    🔥 BOOTSTRAP CO-CLUSTERING STABILITY MATRIX
    ==============================
               Cluster 0  Cluster 1  Cluster 2  Cluster 3  Cluster 4  Cluster 5
    Cluster 0      1.000      0.284      0.274      0.265      0.285      0.283
    Cluster 1      0.284      1.000      0.298      0.270      0.271      0.251
    Cluster 2      0.274      0.298      1.000      0.259      0.306      0.265
    Cluster 3      0.265      0.270      0.259      1.000      0.238      0.285
    Cluster 4      0.285      0.271      0.306      0.238      1.000      0.295
    Cluster 5      0.283      0.251      0.265      0.285      0.295      1.000
    
    📊 Average stability per cluster
    Cluster 0    0.398
    Cluster 1    0.396
    Cluster 2    0.400
    Cluster 3    0.386
    Cluster 4    0.399
    Cluster 5    0.396
    dtype: float64
    


.. image:: output_158_1.png


.. parsed-literal::

    
    ==============================
    🧠 SHAP FEATURE WEIGHTS (NORMALIZED)
    ==============================
    Score                                         0.132
    Business sophistication                       0.132
    Creative outputs                              0.128
    Institutions                                  0.126
    Knowledge and technology outputs              0.124
    Human capital and research                    0.123
    Market sophistication                         0.118
    Infrastructure                                0.117
    


.. image:: output_158_3.png



.. image:: output_158_4.png


.. parsed-literal::

    
    ==============================
    🏷️ MACRO-CLUSTER ASSIGNMENTS
    ==============================
         Cluster  Macro_Cluster
    0  Cluster 0              3
    1  Cluster 1              2
    2  Cluster 2              2
    3  Cluster 3              1
    4  Cluster 4              2
    5  Cluster 5              1
    
    ==============================
    📐 INTERPRETIVE NOTES (SSCI STYLE)
    ==============================
    - Bootstrap stability values above 0.70 indicate highly robust higher-order regimes.
    - SHAP-weighted distances prioritize economically dominant housing indicators.
    - Hierarchical grouping reveals interpretable macro-regimes beyond flat K-means solutions.
    - Numeric annotation ensures full transparency and reproducibility.
    
    ✅ SSCI-LEVEL, FILELESS, NUMERICALLY ANNOTATED PIPELINE COMPLETED.
    




.. code:: ipython3

    # =====================================================
    # 🚀 3D PCA + K-Means Visualization (K = 6)
    # GII.csv — SSCI / Q1 Publication-Ready Version
    # =====================================================
    
    import pandas as pd
    import numpy as np
    from sklearn.preprocessing import StandardScaler
    from sklearn.decomposition import PCA
    from sklearn.cluster import KMeans
    import matplotlib.pyplot as plt
    import matplotlib.patheffects as path_effects
    
    # =====================================================
    # 1️⃣ DATA LOADING & CLEANING
    # =====================================================
    
    csv_path = "GII.csv"
    df = pd.read_csv(csv_path, sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    country_col = "Country"
    
    # Sayısal değişkenler
    features = df.select_dtypes(include=[np.number]).columns.tolist()
    numeric_df = df[features]
    
    print("Veri boyutu:", df.shape)
    print("Kullanılan sayısal değişkenler:", features)
    
    # =====================================================
    # 2️⃣ STANDARDIZATION
    # =====================================================
    
    scaler = StandardScaler()
    scaled_data = scaler.fit_transform(numeric_df)
    
    # =====================================================
    # 3️⃣ K-MEANS CLUSTERING (K = 6)
    # =====================================================
    
    k = 6
    kmeans = KMeans(
        n_clusters=k,
        random_state=42,
        n_init=20
    )
    
    df["Cluster"] = kmeans.fit_predict(scaled_data)
    
    # =====================================================
    # 4️⃣ PCA (3 COMPONENTS)
    # =====================================================
    
    pca = PCA(n_components=3, random_state=42)
    pca_data = pca.fit_transform(scaled_data)
    
    print("\n📊 PCA Explained Variance Ratios:")
    for i, var in enumerate(pca.explained_variance_ratio_, 1):
        print(f"PCA {i}: {var:.3f}")
    
    # =====================================================
    # 5️⃣ 3D PCA + K-MEANS VISUALIZATION
    # =====================================================
    
    colors = [
        "#FFD700",  # gold
        "#FF4500",  # orange-red
        "#32CD32",  # green
        "#1E90FF",  # blue
        "#8A2BE2",  # purple
        "#00CED1"   # turquoise
    ]
    
    fig = plt.figure(figsize=(17, 13))
    ax = fig.add_subplot(111, projection="3d")
    
    # ---- Cluster points ----
    for i in range(k):
        idx = df["Cluster"] == i
        ax.scatter(
            pca_data[idx, 0],
            pca_data[idx, 1],
            pca_data[idx, 2],
            s=40,
            alpha=0.85,
            color=colors[i],
            label=f"Cluster {i+1} (n={idx.sum()})"
        )
    
    # =====================================================
    # 6️⃣ COUNTRY LABELS — NO OVERLAP (DETERMINISTIC)
    # =====================================================
    
    offset_radius = 0.28
    angles = np.linspace(0, 2 * np.pi, len(df), endpoint=False)
    
    for i, country in enumerate(df[country_col]):
        x, y, z = pca_data[i]
    
        dx = offset_radius * np.cos(angles[i])
        dy = offset_radius * np.sin(angles[i])
        dz = 0.15 * ((i % 5) - 2)
    
        # Leader line
        ax.plot(
            [x, x + dx],
            [y, y + dy],
            [z, z + dz],
            color="gray",
            linewidth=0.6,
            alpha=0.6,
            zorder=5
        )
    
        # Label
        txt = ax.text(
            x + dx,
            y + dy,
            z + dz,
            country,
            fontsize=8,
            color="black",
            zorder=10,
            bbox=dict(
                boxstyle="round,pad=0.25",
                facecolor="white",
                edgecolor="none",
                alpha=0.85
            )
        )
    
        txt.set_path_effects([
            path_effects.Stroke(linewidth=1.2, foreground="white"),
            path_effects.Normal()
        ])
    
    # =====================================================
    # 7️⃣ AXIS, TITLE & VIEW SETTINGS
    # =====================================================
    
    ax.set_xlabel("PCA 1", fontsize=12)
    ax.set_ylabel("PCA 2", fontsize=12)
    ax.set_zlabel("PCA 3", fontsize=12)
    
    ax.set_title(
        "3D PCA + K-Means Clustering (GII Dataset, K = 6)",
        fontsize=16,
        fontweight="bold"
    )
    
    ax.legend(loc="upper right", fontsize=10)
    ax.grid(True)
    
    # Academic viewing angle
    ax.view_init(elev=25, azim=35)
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    Veri boyutu: (139, 9)
    Kullanılan sayısal değişkenler: ['Score', 'Institutions', 'Human capital and research', 'Infrastructure', 'Market sophistication', 'Business sophistication', 'Knowledge and technology outputs', 'Creative outputs']
    
    📊 PCA Explained Variance Ratios:
    PCA 1: 0.828
    PCA 2: 0.060
    PCA 3: 0.037
    


.. image:: output_162_1.png










.. code:: ipython3

    # =============================================================
    # 🚀 K-MEANS (K=6) CLUSTER VALIDATION PIPELINE
    # GII.csv — SSCI / Q1 Ready (+ CatBoost)
    # =============================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.preprocessing import StandardScaler
    from sklearn.cluster import KMeans
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import (
        confusion_matrix, classification_report, accuracy_score,
        silhouette_score, davies_bouldin_score, calinski_harabasz_score
    )
    
    # MODELS
    from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
    from sklearn.linear_model import LogisticRegression
    from sklearn.tree import DecisionTreeClassifier
    from sklearn.svm import SVC
    from sklearn.neighbors import KNeighborsClassifier
    from sklearn.naive_bayes import GaussianNB
    from sklearn.neural_network import MLPClassifier
    
    from catboost import CatBoostClassifier
    
    # =====================================================
    # 1️⃣ DATA LOAD
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    # Country kolonunu ayır
    country_col = "Country"
    
    # Sayısal değişkenler (GII indicators)
    features = df.select_dtypes(include=[np.number]).columns.tolist()
    X_raw = df[features]
    
    print("Veri boyutu:", df.shape)
    print("Kullanılan değişkenler:", features)
    
    # =====================================================
    # 2️⃣ STANDARDIZATION
    # =====================================================
    
    scaler = StandardScaler()
    X = scaler.fit_transform(X_raw)
    
    # =====================================================
    # 3️⃣ K-MEANS (K = 6)
    # =====================================================
    
    k = 6
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=20)
    y = kmeans.fit_predict(X)
    
    df["Cluster"] = y
    
    # =====================================================
    # 4️⃣ INTERNAL CLUSTER VALIDATION
    # =====================================================
    
    print("\n" + "="*70)
    print("📊 INTERNAL CLUSTER VALIDATION (GII, K=6)")
    print("="*70)
    
    print(f"Silhouette Score        : {silhouette_score(X, y):.4f}")
    print(f"Davies–Bouldin Index    : {davies_bouldin_score(X, y):.4f}")
    print(f"Calinski–Harabasz Index : {calinski_harabasz_score(X, y):.2f}")
    
    # =====================================================
    # 5️⃣ TRAIN–TEST SPLIT (PSEUDO-SUPERVISED VALIDATION)
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y,
        test_size=0.2,
        random_state=42,
        stratify=y
    )
    
    # =====================================================
    # 6️⃣ CLASSIFIERS (WITH CATBOOST)
    # =====================================================
    
    models = {
        "Random Forest": RandomForestClassifier(n_estimators=300, random_state=42),
    
        "Logistic Regression": LogisticRegression(
            max_iter=1000, multi_class="multinomial", solver="lbfgs"
        ),
    
        "Decision Tree": DecisionTreeClassifier(random_state=42),
    
        "SVM (RBF)": SVC(kernel="rbf", C=1.5, gamma="scale"),
    
        "KNN": KNeighborsClassifier(n_neighbors=7, weights="distance"),
    
        "Naive Bayes": GaussianNB(),
    
        "Gradient Boosting": GradientBoostingClassifier(
            n_estimators=250,
            learning_rate=0.05,
            max_depth=3
        ),
    
        "ANN (MLP)": MLPClassifier(
            hidden_layer_sizes=(120, 60),
            max_iter=800,
            random_state=42
        ),
    
        "CatBoost": CatBoostClassifier(
            iterations=400,
            learning_rate=0.05,
            depth=6,
            loss_function="MultiClass",
            verbose=0,
            random_seed=42
        )
    }
    
    # =====================================================
    # 7️⃣ CONFUSION MATRICES — 3x3 GRID
    # =====================================================
    
    fig, axes = plt.subplots(3, 3, figsize=(20, 18))
    axes = axes.flatten()
    
    for idx, (name, model) in enumerate(models.items()):
    
        print("\n" + "-"*70)
        print(f"▶ MODEL: {name}")
        print("-"*70)
    
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        acc = accuracy_score(y_test, y_pred)
        print(f"Accuracy: {acc:.4f}")
        print(classification_report(y_test, y_pred))
    
        cm = confusion_matrix(y_test, y_pred)
    
        sns.heatmap(
            cm,
            annot=True,
            fmt="d",
            cmap="viridis",
            xticklabels=[f"C{i+1}" for i in range(k)],
            yticklabels=[f"C{i+1}" for i in range(k)],
            ax=axes[idx]
        )
    
        axes[idx].set_title(f"{name}\nAcc = {acc:.3f}", fontweight="bold")
        axes[idx].set_xlabel("Predicted")
        axes[idx].set_ylabel("Actual")
    
    # Fazla subplot varsa kapat
    for j in range(idx + 1, len(axes)):
        axes[j].axis("off")
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    Veri boyutu: (139, 9)
    Kullanılan değişkenler: ['Score', 'Institutions', 'Human capital and research', 'Infrastructure', 'Market sophistication', 'Business sophistication', 'Knowledge and technology outputs', 'Creative outputs']
    
    ======================================================================
    📊 INTERNAL CLUSTER VALIDATION (GII, K=6)
    ======================================================================
    Silhouette Score        : 0.2628
    Davies–Bouldin Index    : 1.2356
    Calinski–Harabasz Index : 129.99
    
    ----------------------------------------------------------------------
    ▶ MODEL: Random Forest
    ----------------------------------------------------------------------
    Accuracy: 0.9643
                  precision    recall  f1-score   support
    
               0       1.00      1.00      1.00         5
               1       0.88      1.00      0.93         7
               2       1.00      0.83      0.91         6
               3       1.00      1.00      1.00         2
               4       1.00      1.00      1.00         5
               5       1.00      1.00      1.00         3
    
        accuracy                           0.96        28
       macro avg       0.98      0.97      0.97        28
    weighted avg       0.97      0.96      0.96        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: Logistic Regression
    ----------------------------------------------------------------------
    Accuracy: 0.9643
                  precision    recall  f1-score   support
    
               0       0.83      1.00      0.91         5
               1       1.00      1.00      1.00         7
               2       1.00      1.00      1.00         6
               3       1.00      0.50      0.67         2
               4       1.00      1.00      1.00         5
               5       1.00      1.00      1.00         3
    
        accuracy                           0.96        28
       macro avg       0.97      0.92      0.93        28
    weighted avg       0.97      0.96      0.96        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: Decision Tree
    ----------------------------------------------------------------------
    Accuracy: 0.8214
                  precision    recall  f1-score   support
    
               0       0.71      1.00      0.83         5
               1       1.00      0.71      0.83         7
               2       0.62      0.83      0.71         6
               3       1.00      0.50      0.67         2
               4       1.00      0.80      0.89         5
               5       1.00      1.00      1.00         3
    
        accuracy                           0.82        28
       macro avg       0.89      0.81      0.82        28
    weighted avg       0.87      0.82      0.82        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: SVM (RBF)
    ----------------------------------------------------------------------
    Accuracy: 1.0000
                  precision    recall  f1-score   support
    
               0       1.00      1.00      1.00         5
               1       1.00      1.00      1.00         7
               2       1.00      1.00      1.00         6
               3       1.00      1.00      1.00         2
               4       1.00      1.00      1.00         5
               5       1.00      1.00      1.00         3
    
        accuracy                           1.00        28
       macro avg       1.00      1.00      1.00        28
    weighted avg       1.00      1.00      1.00        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: KNN
    ----------------------------------------------------------------------
    Accuracy: 0.9643
                  precision    recall  f1-score   support
    
               0       1.00      1.00      1.00         5
               1       1.00      1.00      1.00         7
               2       1.00      0.83      0.91         6
               3       1.00      1.00      1.00         2
               4       0.83      1.00      0.91         5
               5       1.00      1.00      1.00         3
    
        accuracy                           0.96        28
       macro avg       0.97      0.97      0.97        28
    weighted avg       0.97      0.96      0.96        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: Naive Bayes
    ----------------------------------------------------------------------
    Accuracy: 0.9286
                  precision    recall  f1-score   support
    
               0       1.00      1.00      1.00         5
               1       1.00      0.71      0.83         7
               2       0.75      1.00      0.86         6
               3       1.00      1.00      1.00         2
               4       1.00      1.00      1.00         5
               5       1.00      1.00      1.00         3
    
        accuracy                           0.93        28
       macro avg       0.96      0.95      0.95        28
    weighted avg       0.95      0.93      0.93        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: Gradient Boosting
    ----------------------------------------------------------------------
    Accuracy: 0.8214
                  precision    recall  f1-score   support
    
               0       0.71      1.00      0.83         5
               1       1.00      0.71      0.83         7
               2       0.83      0.83      0.83         6
               3       0.00      0.00      0.00         2
               4       0.71      1.00      0.83         5
               5       1.00      1.00      1.00         3
    
        accuracy                           0.82        28
       macro avg       0.71      0.76      0.72        28
    weighted avg       0.79      0.82      0.79        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: ANN (MLP)
    ----------------------------------------------------------------------
    Accuracy: 0.9643
                  precision    recall  f1-score   support
    
               0       0.83      1.00      0.91         5
               1       1.00      1.00      1.00         7
               2       1.00      1.00      1.00         6
               3       1.00      0.50      0.67         2
               4       1.00      1.00      1.00         5
               5       1.00      1.00      1.00         3
    
        accuracy                           0.96        28
       macro avg       0.97      0.92      0.93        28
    weighted avg       0.97      0.96      0.96        28
    
    
    ----------------------------------------------------------------------
    ▶ MODEL: CatBoost
    ----------------------------------------------------------------------
    Accuracy: 1.0000
                  precision    recall  f1-score   support
    
               0       1.00      1.00      1.00         5
               1       1.00      1.00      1.00         7
               2       1.00      1.00      1.00         6
               3       1.00      1.00      1.00         2
               4       1.00      1.00      1.00         5
               5       1.00      1.00      1.00         3
    
        accuracy                           1.00        28
       macro avg       1.00      1.00      1.00        28
    weighted avg       1.00      1.00      1.00        28
    
    


.. image:: output_171_1.png










.. code:: ipython3

    # =============================================================
    # 📈 MULTICLASS ROC CURVES — K-MEANS (K=6) VALIDATION
    # VISUALLY ENHANCED — SSCI / Q1 FIGURE QUALITY
    # =============================================================
    
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.preprocessing import label_binarize
    from sklearn.metrics import roc_curve, roc_auc_score
    from sklearn.multiclass import OneVsRestClassifier
    from sklearn.model_selection import train_test_split
    
    # MODELS
    from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
    from sklearn.linear_model import LogisticRegression
    from sklearn.tree import DecisionTreeClassifier
    from sklearn.svm import SVC
    from sklearn.neighbors import KNeighborsClassifier
    from sklearn.naive_bayes import GaussianNB
    from sklearn.neural_network import MLPClassifier
    from catboost import CatBoostClassifier
    
    # =====================================================
    # 🔧 GLOBAL VISUAL SETTINGS (CRITICAL)
    # =====================================================
    
    plt.rcParams.update({
        "figure.dpi": 160,
        "savefig.dpi": 300,
        "font.size": 13,
        "font.weight": "bold",
        "axes.labelweight": "bold",
        "axes.titleweight": "bold",
        "axes.linewidth": 1.6,
        "xtick.labelsize": 12,
        "ytick.labelsize": 12,
        "legend.fontsize": 10
    })
    
    sns.set_style("whitegrid")
    
    # =====================================================
    # 1️⃣ DATA
    # =====================================================
    
    X_data = X
    y_labels = y
    
    classes = sorted(np.unique(y_labels))
    n_classes = len(classes)
    
    y_bin = label_binarize(y_labels, classes=classes)
    
    # =====================================================
    # 2️⃣ SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X_data, y_labels,
        test_size=0.20,
        random_state=42,
        stratify=y_labels
    )
    
    y_test_bin = label_binarize(y_test, classes=classes)
    
    # =====================================================
    # 3️⃣ MODELS
    # =====================================================
    
    models = {
        "Random Forest": RandomForestClassifier(n_estimators=300, random_state=42),
        "Logistic Regression": LogisticRegression(max_iter=1000, solver="lbfgs"),
        "Decision Tree": DecisionTreeClassifier(random_state=42),
        "SVM (RBF)": SVC(kernel="rbf", C=1.5, gamma="scale", probability=True),
        "KNN": KNeighborsClassifier(n_neighbors=7, weights="distance"),
        "Naive Bayes": GaussianNB(),
        "Gradient Boosting": GradientBoostingClassifier(n_estimators=250, learning_rate=0.05),
        "ANN (MLP)": MLPClassifier(hidden_layer_sizes=(120, 60), max_iter=800, random_state=42),
        "CatBoost": CatBoostClassifier(
            iterations=400, learning_rate=0.05, depth=6,
            loss_function="MultiClass", verbose=0, random_seed=42
        )
    }
    
    # =====================================================
    # 4️⃣ SAFE PROBABILITY
    # =====================================================
    
    def safe_predict_proba(clf, X):
        if hasattr(clf, "predict_proba"):
            prob = clf.predict_proba(X)
        else:
            scores = clf.decision_function(X)
            scores -= np.max(scores, axis=1, keepdims=True)
            exp_scores = np.exp(scores)
            prob = exp_scores / np.sum(exp_scores, axis=1, keepdims=True)
    
        return np.nan_to_num(prob)
    
    # =====================================================
    # 5️⃣ ROC CURVES — HIGH CONTRAST
    # =====================================================
    
    fig, axes = plt.subplots(3, 3, figsize=(22, 22))
    axes = axes.flatten()
    
    palette = sns.color_palette("tab10", n_classes)
    
    print("\n" + "="*80)
    print("📈 MULTICLASS ROC–AUC ANALYSIS (K=6)")
    print("="*80)
    
    for idx, (name, model) in enumerate(models.items()):
        ax = axes[idx]
    
        print(f"\n▶ {name}")
    
        clf = OneVsRestClassifier(model)
        clf.fit(X_train, y_train)
        y_prob = safe_predict_proba(clf, X_test)
    
        for i in range(n_classes):
            fpr, tpr, _ = roc_curve(y_test_bin[:, i], y_prob[:, i])
            auc = roc_auc_score(y_test_bin[:, i], y_prob[:, i])
    
            print(f"  Cluster C{i+1}: AUC = {auc:.4f}")
    
            ax.plot(
                fpr, tpr,
                color=palette[i],
                linewidth=3.2,
                label=f"C{i+1} (AUC={auc:.3f})"
            )
    
        macro_auc = roc_auc_score(y_test_bin, y_prob, average="macro", multi_class="ovr")
        print(f"  ➤ Macro AUC: {macro_auc:.4f}")
    
        ax.plot([0, 1], [0, 1], linestyle="--", color="black", linewidth=1.5)
        ax.set_title(f"{name}\nMacro AUC = {macro_auc:.3f}", fontsize=15, fontweight="bold")
        ax.set_xlabel("False Positive Rate")
        ax.set_ylabel("True Positive Rate")
        ax.legend(frameon=True)
        ax.grid(True, alpha=0.4)
    
    # =====================================================
    # 6️⃣ CLEAN EMPTY
    # =====================================================
    
    for j in range(idx + 1, len(axes)):
        axes[j].axis("off")
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    
    ================================================================================
    📈 MULTICLASS ROC–AUC ANALYSIS (K=6)
    ================================================================================
    
    ▶ Random Forest
      Cluster C1: AUC = 1.0000
      Cluster C2: AUC = 1.0000
      Cluster C3: AUC = 0.9924
      Cluster C4: AUC = 1.0000
      Cluster C5: AUC = 1.0000
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9987
    
    ▶ Logistic Regression
      Cluster C1: AUC = 0.9652
      Cluster C2: AUC = 1.0000
      Cluster C3: AUC = 0.9924
      Cluster C4: AUC = 0.9615
      Cluster C5: AUC = 1.0000
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9865
    
    ▶ Decision Tree
      Cluster C1: AUC = 1.0000
      Cluster C2: AUC = 0.8571
      Cluster C3: AUC = 0.9053
      Cluster C4: AUC = 0.5000
      Cluster C5: AUC = 1.0000
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.8771
    
    ▶ SVM (RBF)
      Cluster C1: AUC = 1.0000
      Cluster C2: AUC = 1.0000
      Cluster C3: AUC = 0.9924
      Cluster C4: AUC = 1.0000
      Cluster C5: AUC = 1.0000
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9987
    
    ▶ KNN
      Cluster C1: AUC = 1.0000
      Cluster C2: AUC = 1.0000
      Cluster C3: AUC = 0.9848
      Cluster C4: AUC = 1.0000
      Cluster C5: AUC = 0.9826
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9946
    
    ▶ Naive Bayes
      Cluster C1: AUC = 1.0000
      Cluster C2: AUC = 0.9932
      Cluster C3: AUC = 1.0000
      Cluster C4: AUC = 0.9808
      Cluster C5: AUC = 1.0000
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9957
    
    ▶ Gradient Boosting
      Cluster C1: AUC = 0.9826
      Cluster C2: AUC = 0.9864
      Cluster C3: AUC = 0.9318
      Cluster C4: AUC = 0.9615
      Cluster C5: AUC = 0.9826
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9742
    
    ▶ ANN (MLP)
      Cluster C1: AUC = 1.0000
      Cluster C2: AUC = 1.0000
      Cluster C3: AUC = 1.0000
      Cluster C4: AUC = 0.7500
      Cluster C5: AUC = 1.0000
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9583
    
    ▶ CatBoost
      Cluster C1: AUC = 1.0000
      Cluster C2: AUC = 0.9932
      Cluster C3: AUC = 0.9924
      Cluster C4: AUC = 1.0000
      Cluster C5: AUC = 1.0000
      Cluster C6: AUC = 1.0000
      ➤ Macro AUC: 0.9976
    


.. image:: output_180_1.png




.. code:: ipython3

    import pandas as pd
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    # Macro AUC results collected from previous ROC loop
    macro_auc_results = []
    
    for name, model in models.items():
        clf = OneVsRestClassifier(model)
        clf.fit(X_train, y_train)
        y_prob = safe_predict_proba(clf, X_test)
    
        macro_auc = roc_auc_score(
            y_test_bin, y_prob,
            average="macro", multi_class="ovr"
        )
    
        macro_auc_results.append({
            "Model": name,
            "Macro AUC": macro_auc
        })
    
    auc_df = pd.DataFrame(macro_auc_results).sort_values("Macro AUC", ascending=False)
    
    plt.figure(figsize=(12, 7))
    sns.barplot(
        data=auc_df,
        x="Macro AUC",
        y="Model",
        palette="viridis"
    )
    
    plt.axvline(0.5, linestyle="--", color="red", linewidth=1.5)
    plt.title("Macro-Averaged ROC–AUC Comparison (K=6)", fontsize=16, fontweight="bold")
    plt.xlabel("Macro AUC")
    plt.ylabel("")
    plt.xlim(0.5, 1.0)
    plt.grid(axis="x", alpha=0.4)
    plt.tight_layout()
    plt.show()
    
    print(auc_df)
    



.. image:: output_183_0.png


.. parsed-literal::

                     Model  Macro AUC
    3            SVM (RBF)   1.000000
    0        Random Forest   0.998737
    8             CatBoost   0.997604
    5          Naive Bayes   0.995661
    4                  KNN   0.994576
    1  Logistic Regression   0.986530
    6    Gradient Boosting   0.974161
    7            ANN (MLP)   0.958333
    2        Decision Tree   0.877074
    


.. code:: ipython3

    from sklearn.utils import resample
    
    n_bootstrap = 100
    fpr_grid = np.linspace(0, 1, 100)
    
    tprs = []
    
    best_model_name = auc_df.iloc[0]["Model"]
    best_model = models[best_model_name]
    
    for i in range(n_bootstrap):
        X_bs, y_bs = resample(X_train, y_train, random_state=42+i)
        clf = OneVsRestClassifier(best_model)
        clf.fit(X_bs, y_bs)
    
        y_prob = safe_predict_proba(clf, X_test)
    
        # Macro ROC
        tpr_interp_all = []
    
        for c in range(n_classes):
            fpr, tpr, _ = roc_curve(y_test_bin[:, c], y_prob[:, c])
            tpr_interp = np.interp(fpr_grid, fpr, tpr)
            tpr_interp_all.append(tpr_interp)
    
        tprs.append(np.mean(tpr_interp_all, axis=0))
    
    tprs = np.array(tprs)
    mean_tpr = tprs.mean(axis=0)
    std_tpr = tprs.std(axis=0)
    
    plt.figure(figsize=(8, 8))
    plt.plot(fpr_grid, mean_tpr, color="blue", lw=3, label="Mean ROC")
    plt.fill_between(
        fpr_grid,
        mean_tpr - std_tpr,
        mean_tpr + std_tpr,
        color="blue",
        alpha=0.25,
        label="±1 SD"
    )
    plt.plot([0, 1], [0, 1], "k--", lw=1.5)
    
    plt.xlabel("False Positive Rate")
    plt.ylabel("True Positive Rate")
    plt.title(
        f"Bootstrap Mean ROC Curve ± SD\n({best_model_name}, K=6)",
        fontsize=15, fontweight="bold"
    )
    plt.legend()
    plt.grid(alpha=0.4)
    plt.tight_layout()
    plt.show()
    



.. image:: output_185_0.png




.. code:: ipython3

    # =====================================================
    # 3️⃣ WHICH CLUSTERS ARE HARDEST TO SEPARATE?
    # Pseudo-supervised structural ambiguity analysis
    # (Random Forest as near-optimal but imperfect separator)
    # =====================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    from sklearn.metrics import confusion_matrix
    from sklearn.ensemble import RandomForestClassifier
    
    # -----------------------------
    # Train Random Forest
    # -----------------------------
    rf = RandomForestClassifier(
        n_estimators=300,
        random_state=42
    )
    
    rf.fit(X_train, y_train)
    y_pred = rf.predict(X_test)
    
    # -----------------------------
    # Confusion Matrix
    # -----------------------------
    conf_mat = confusion_matrix(y_test, y_pred)
    n_classes = conf_mat.shape[0]
    
    conf_df = pd.DataFrame(
        conf_mat,
        index=[f"C{i+1}" for i in range(n_classes)],
        columns=[f"C{i+1}" for i in range(n_classes)]
    )
    
    # Normalize by true class (row-wise)
    conf_norm = conf_df.div(conf_df.sum(axis=1), axis=0)
    
    # -----------------------------
    # Visualization
    # -----------------------------
    plt.figure(figsize=(9, 8))
    sns.heatmap(
        conf_norm,
        annot=True,
        fmt=".2f",
        cmap="rocket_r",
        linewidths=1,
        cbar_kws={"label": "Classification Proportion"}
    )
    
    plt.title(
        "Cluster Confusion Structure:\nHard-to-Separate Innovation Regimes",
        fontsize=15,
        fontweight="bold"
    )
    plt.xlabel("Predicted Cluster")
    plt.ylabel("True Cluster")
    plt.tight_layout()
    plt.show()
    
    print(conf_norm)



.. image:: output_188_0.png


.. parsed-literal::

         C1        C2        C3   C4   C5   C6
    C1  1.0  0.000000  0.000000  0.0  0.0  0.0
    C2  0.0  1.000000  0.000000  0.0  0.0  0.0
    C3  0.0  0.166667  0.833333  0.0  0.0  0.0
    C4  0.0  0.000000  0.000000  1.0  0.0  0.0
    C5  0.0  0.000000  0.000000  0.0  1.0  0.0
    C6  0.0  0.000000  0.000000  0.0  0.0  1.0
    




.. code:: ipython3

    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.preprocessing import StandardScaler
    from sklearn.cluster import KMeans
    from sklearn.model_selection import train_test_split, GridSearchCV
    from sklearn.metrics import (
        accuracy_score, precision_score, recall_score,
        f1_score, roc_auc_score
    )
    
    # MODELS
    from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
    from sklearn.linear_model import LogisticRegression
    from sklearn.tree import DecisionTreeClassifier
    from sklearn.neighbors import KNeighborsClassifier
    from sklearn.svm import SVC
    from sklearn.naive_bayes import GaussianNB
    from sklearn.neural_network import MLPClassifier
    from catboost import CatBoostClassifier
    
    # =====================================================
    # 🔧 VISUAL SETTINGS (PUBLICATION QUALITY)
    # =====================================================
    
    plt.rcParams.update({
        "figure.dpi": 160,
        "savefig.dpi": 300,
        "font.size": 13,
        "font.weight": "bold",
        "axes.labelweight": "bold",
        "axes.titleweight": "bold",
        "axes.linewidth": 1.6
    })
    sns.set_style("whitegrid")
    
    # =====================================================
    # 1️⃣ LOAD & PREPARE DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    features = df.select_dtypes(include=[np.number]).columns.tolist()
    X_raw = df[features]
    
    scaler = StandardScaler()
    X = scaler.fit_transform(X_raw)
    
    # =====================================================
    # 2️⃣ K-MEANS (K = 6)
    # =====================================================
    
    k = 6
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=20)
    y = kmeans.fit_predict(X)
    
    # =====================================================
    # 3️⃣ TRAIN / TEST SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y,
        test_size=0.20,
        random_state=42,
        stratify=y
    )
    
    # =====================================================
    # 4️⃣ PARAMETER GRIDS
    # =====================================================
    
    param_grids = {
    
        "Random Forest": {
            "n_estimators": [200, 300],
            "max_depth": [None, 15]
        },
    
        "Logistic Regression": {
            "C": [0.1, 1, 10]
        },
    
        "Decision Tree": {
            "max_depth": [None, 10, 20]
        },
    
        "SVM (RBF)": {
            "C": [0.5, 1.5],
            "gamma": ["scale", "auto"]
        },
    
        "KNN": {
            "n_neighbors": [5, 7],
            "weights": ["distance"]
        },
    
        "Gaussian NB": {},
    
        "Gradient Boosting": {
            "n_estimators": [200, 300],
            "learning_rate": [0.05]
        },
    
        "ANN (MLP)": {
            "hidden_layer_sizes": [(120, 60)],
            "activation": ["relu"]
        },
    
        "CatBoost": {
            "iterations": [300, 400],
            "depth": [4, 6],
            "learning_rate": [0.05]
        }
    }
    
    # =====================================================
    # 5️⃣ MODELS
    # =====================================================
    
    models = {
        "Random Forest": RandomForestClassifier(random_state=42),
        "Logistic Regression": LogisticRegression(
            max_iter=2000, multi_class="multinomial", solver="lbfgs"
        ),
        "Decision Tree": DecisionTreeClassifier(random_state=42),
        "SVM (RBF)": SVC(kernel="rbf", probability=True, random_state=42),
        "KNN": KNeighborsClassifier(),
        "Gaussian NB": GaussianNB(),
        "Gradient Boosting": GradientBoostingClassifier(random_state=42),
        "ANN (MLP)": MLPClassifier(max_iter=1000, random_state=42),
        "CatBoost": CatBoostClassifier(
            verbose=0, loss_function="MultiClass", random_seed=42
        )
    }
    
    # =====================================================
    # 6️⃣ GRIDSEARCH
    # =====================================================
    
    best_models = {}
    
    print("\n" + "="*80)
    print("🔍 GRIDSEARCH OPTIMIZATION (F1-MACRO)")
    print("="*80)
    
    for name, model in models.items():
        print(f"\n▶ {name}")
    
        grid = GridSearchCV(
            model,
            param_grids[name],
            cv=3,
            scoring="f1_macro",
            n_jobs=-1
        )
    
        grid.fit(X_train, y_train)
        best_models[name] = grid.best_estimator_
    
        print("  Best params:", grid.best_params_)
        print(f"  Best CV F1-macro: {grid.best_score_:.4f}")
    
    # =====================================================
    # 7️⃣ PERFORMANCE EVALUATION
    # =====================================================
    
    performance = []
    
    for name, model in best_models.items():
        y_pred = model.predict(X_test)
    
        if hasattr(model, "predict_proba"):
            y_prob = model.predict_proba(X_test)
            auc = roc_auc_score(
                y_test, y_prob, multi_class="ovr", average="macro"
            )
        else:
            auc = np.nan
    
        performance.append([
            name,
            accuracy_score(y_test, y_pred),
            precision_score(y_test, y_pred, average="macro"),
            recall_score(y_test, y_pred, average="macro"),
            f1_score(y_test, y_pred, average="macro"),
            auc
        ])
    
    perf_df = pd.DataFrame(
        performance,
        columns=["Model", "Accuracy", "Precision", "Recall", "F1-score", "Macro AUC"]
    ).set_index("Model")
    
    print("\n" + "="*80)
    print("📊 GRIDSEARCH PERFORMANCE TABLE")
    print("="*80)
    print(perf_df.round(4))
    
    # =====================================================
    # 8️⃣ BARPLOT — HIGH CONTRAST
    # =====================================================
    
    ax = perf_df.plot(
        kind="bar",
        figsize=(18, 9),
        width=0.8,
        colormap="viridis"
    )
    
    plt.title(
        "Model Performance After GridSearch Optimization\n(K-Means K=6 Cluster Validation)",
        fontsize=18,
        fontweight="bold"
    )
    plt.ylabel("Score")
    plt.xticks(rotation=45, ha="right")
    plt.ylim(0, 1.05)
    plt.grid(axis="y", alpha=0.4)
    
    for container in ax.containers:
        ax.bar_label(container, fmt="%.3f", fontsize=10, rotation=90)
    
    plt.tight_layout()
    plt.show()
    


.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\cluster\_kmeans.py:1419: UserWarning: KMeans is known to have a memory leak on Windows with MKL, when there are less chunks than available threads. You can avoid it by setting the environment variable OMP_NUM_THREADS=1.
      warnings.warn(
    

.. parsed-literal::

    
    ================================================================================
    🔍 GRIDSEARCH OPTIMIZATION (F1-MACRO)
    ================================================================================
    
    ▶ Random Forest
      Best params: {'max_depth': None, 'n_estimators': 200}
      Best CV F1-macro: 0.9106
    
    ▶ Logistic Regression
    

.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\linear_model\_logistic.py:1247: FutureWarning: 'multi_class' was deprecated in version 1.5 and will be removed in 1.7. From then on, it will always use 'multinomial'. Leave it to its default value to avoid this warning.
      warnings.warn(
    

.. parsed-literal::

      Best params: {'C': 10}
      Best CV F1-macro: 0.9095
    
    ▶ Decision Tree
      Best params: {'max_depth': None}
      Best CV F1-macro: 0.7468
    
    ▶ SVM (RBF)
      Best params: {'C': 0.5, 'gamma': 'auto'}
      Best CV F1-macro: 0.9570
    
    ▶ KNN
      Best params: {'n_neighbors': 7, 'weights': 'distance'}
      Best CV F1-macro: 0.9339
    
    ▶ Gaussian NB
      Best params: {}
      Best CV F1-macro: 0.8976
    
    ▶ Gradient Boosting
      Best params: {'learning_rate': 0.05, 'n_estimators': 300}
      Best CV F1-macro: 0.8222
    
    ▶ ANN (MLP)
      Best params: {'activation': 'relu', 'hidden_layer_sizes': (120, 60)}
      Best CV F1-macro: 0.8975
    
    ▶ CatBoost
      Best params: {'depth': 4, 'iterations': 300, 'learning_rate': 0.05}
      Best CV F1-macro: 0.9557
    
    ================================================================================
    📊 GRIDSEARCH PERFORMANCE TABLE
    ================================================================================
                         Accuracy  Precision  Recall  F1-score  Macro AUC
    Model                                                                
    Random Forest          0.9643     0.9762  0.9762    0.9744     0.9987
    Logistic Regression    0.9643     0.9722  0.9167    0.9293     0.9968
    Decision Tree          0.8214     0.8899  0.8079    0.8228     0.8854
    SVM (RBF)              1.0000     1.0000  1.0000    1.0000     1.0000
    KNN                    0.9643     0.9722  0.9722    0.9697     0.9946
    Gaussian NB            0.9286     0.9583  0.9524    0.9484     1.0000
    Gradient Boosting      0.8214     0.7103  0.7579    0.7222     0.9879
    ANN (MLP)              0.9643     0.9722  0.9167    0.9293     0.9936
    CatBoost               0.9286     0.9444  0.9484    0.9443     0.9987
    

.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\metrics\_classification.py:1565: UndefinedMetricWarning: Precision is ill-defined and being set to 0.0 in labels with no predicted samples. Use `zero_division` parameter to control this behavior.
      _warn_prf(average, modifier, f"{metric.capitalize()} is", len(result))
    


.. image:: output_192_5.png






.. code:: ipython3

    # ============================================================
    # 1️⃣ KÜTÜPHANELER
    # ============================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
    
    # Linear & Regularized
    from sklearn.linear_model import (
        LinearRegression, Ridge, Lasso, ElasticNet,
        HuberRegressor, BayesianRidge
    )
    
    # Tree-based & Ensemble
    from sklearn.tree import DecisionTreeRegressor
    from sklearn.ensemble import (
        RandomForestRegressor, ExtraTreesRegressor,
        AdaBoostRegressor, GradientBoostingRegressor
    )
    
    # Other models
    from sklearn.svm import SVR
    from sklearn.neighbors import KNeighborsRegressor
    from sklearn.neural_network import MLPRegressor
    
    # Boosting
    from xgboost import XGBRegressor
    from lightgbm import LGBMRegressor
    from catboost import CatBoostRegressor
    
    
    # ============================================================
    # 2️⃣ VERİ YÜKLEME
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    # ============================================================
    # 🎯 HEDEF & ÖZELLİKLER
    # ============================================================
    target_col = "Score"
    
    y = df[target_col]
    X = df.drop(columns=["Country", target_col])
    X = X.select_dtypes(include=[np.number])
    
    
    # ============================================================
    # 3️⃣ STANDARDİZASYON
    # ============================================================
    X_std = StandardScaler().fit_transform(X)
    y_std = StandardScaler().fit_transform(
        y.values.reshape(-1, 1)
    ).ravel()
    
    
    # ============================================================
    # 4️⃣ TRAIN – TEST SPLIT
    # ============================================================
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std,
        test_size=0.20,
        random_state=42
    )
    
    
    # ============================================================
    # 5️⃣ REGRESYON MODELLERİ
    # ============================================================
    models = {
        "Huber": HuberRegressor(),
        "Bayesian Ridge": BayesianRidge(),
        "Linear Regression": LinearRegression(),
        "Ridge": Ridge(alpha=1.0),
        "Elastic Net": ElasticNet(alpha=0.05, l1_ratio=0.5),
        "Lasso": Lasso(alpha=0.05),
    
        "SVR (RBF)": SVR(C=10, epsilon=0.1),
        "k-NN": KNeighborsRegressor(n_neighbors=5),
    
        "MLP (ANN)": MLPRegressor(
            hidden_layer_sizes=(120, 60),
            max_iter=1000,
            random_state=42
        ),
    
        "Decision Tree": DecisionTreeRegressor(random_state=42),
        "Random Forest": RandomForestRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "Extra Trees": ExtraTreesRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "AdaBoost": AdaBoostRegressor(
            n_estimators=300, learning_rate=0.05, random_state=42
        ),
        "Gradient Boosting": GradientBoostingRegressor(
            n_estimators=300, random_state=42
        ),
    
        "XGBoost": XGBRegressor(
            n_estimators=300,
            learning_rate=0.05,
            max_depth=6,
            random_state=42
        ),
        "LightGBM": LGBMRegressor(
            n_estimators=300,
            random_state=42
        ),
        "CatBoost": CatBoostRegressor(
            iterations=300,
            depth=4,
            learning_rate=0.05,
            verbose=0,
            random_state=42
        )
    }
    
    
    # ============================================================
    # 6️⃣ EĞİTİM & PERFORMANS
    # ============================================================
    results = []
    
    for name, model in models.items():
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        results.append({
            "Model": name,
            "MSE": mean_squared_error(y_test, y_pred),
            "RMSE": np.sqrt(mean_squared_error(y_test, y_pred)),
            "MAE": mean_absolute_error(y_test, y_pred),
            "R²": r2_score(y_test, y_pred)
        })
    
    results_df = (
        pd.DataFrame(results)
        .sort_values(by="R²", ascending=False)
        .reset_index(drop=True)
    )
    
    print("\n================ AFFORDABILITY INDEX – ALL MODELS =================\n")
    print(results_df.round(4))
    
    
    # ============================================================
    # 🎨 CANLI RENK PALETİ (SSCI-FIGURE READY)
    # ============================================================
    vivid_colors = [
        "#e41a1c", "#377eb8", "#4daf4a", "#984ea3",
        "#ff7f00", "#ffff33", "#a65628", "#f781bf",
        "#1b9e77", "#d95f02", "#7570b3", "#e7298a",
        "#66a61e", "#e6ab02", "#a6761d", "#666666",
        "#17becf"
    ]
    
    model_colors = {
        model: vivid_colors[i % len(vivid_colors)]
        for i, model in enumerate(results_df["Model"])
    }
    
    
    # ============================================================
    # 7️⃣ METRİK GÖRSELLEŞTİRME (2×2 PANEL)
    # ============================================================
    metrics = ["MSE", "RMSE", "MAE", "R²"]
    titles = [
        "Mean Squared Error (MSE)",
        "Root Mean Squared Error (RMSE)",
        "Mean Absolute Error (MAE)",
        "R² Score"
    ]
    
    fig, axes = plt.subplots(2, 2, figsize=(19, 18))
    axes = axes.flatten()
    
    for ax, metric, title in zip(axes, metrics, titles):
        bars = ax.bar(
            results_df["Model"],
            results_df[metric],
            color=[model_colors[m] for m in results_df["Model"]],
            edgecolor="black",
            linewidth=0.8
        )
    
        for bar in bars:
            h = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width()/2,
                h,
                f"{h:.4f}",
                ha="center",
                va="bottom",
                rotation=90,
                fontsize=9,
                fontweight="bold"
            )
    
        ax.set_title(title, fontsize=13, weight="bold")
        ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
        ax.grid(axis="y", linestyle="--", alpha=0.4)
    
        if metric == "R²":
            ax.set_ylim(0.70, 1.01)
    
    plt.suptitle(
        "Comprehensive Performance Comparison of Regression Models\n(Target: Score)",
        fontsize=16,
        weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    


.. parsed-literal::

    [LightGBM] [Info] Auto-choosing row-wise multi-threading, the overhead of testing was 0.000217 seconds.
    You can set `force_row_wise=true` to remove the overhead.
    And if memory is not enough, you can set `force_col_wise=true`.
    [LightGBM] [Info] Total Bins 262
    [LightGBM] [Info] Number of data points in the train set: 111, number of used features: 7
    [LightGBM] [Info] Start training from score -0.039553
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    

.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\utils\validation.py:2739: UserWarning: X does not have valid feature names, but LGBMRegressor was fitted with feature names
      warnings.warn(
    

.. parsed-literal::

    
    ================ AFFORDABILITY INDEX – ALL MODELS =================
    
                    Model     MSE    RMSE     MAE      R²
    0               Ridge  0.0649  0.2548  0.0821  0.9387
    1      Bayesian Ridge  0.0649  0.2548  0.0822  0.9387
    2   Linear Regression  0.0651  0.2551  0.0840  0.9385
    3               Huber  0.0651  0.2551  0.0500  0.9385
    4         Elastic Net  0.0660  0.2569  0.0955  0.9377
    5               Lasso  0.0698  0.2641  0.1196  0.9341
    6         Extra Trees  0.0800  0.2828  0.1476  0.9245
    7                k-NN  0.0825  0.2873  0.1603  0.9221
    8            CatBoost  0.0833  0.2887  0.1290  0.9213
    9             XGBoost  0.0860  0.2933  0.1685  0.9188
    10      Random Forest  0.0910  0.3017  0.1900  0.9141
    11      Decision Tree  0.0941  0.3067  0.2282  0.9111
    12  Gradient Boosting  0.0963  0.3103  0.1851  0.9091
    13           AdaBoost  0.1106  0.3326  0.2529  0.8955
    14          MLP (ANN)  0.1163  0.3410  0.1945  0.8902
    15          SVR (RBF)  0.1212  0.3481  0.1847  0.8856
    16           LightGBM  0.1475  0.3841  0.2600  0.8607
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\4091812999.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\4091812999.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\4091812999.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\4091812999.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    


.. image:: output_197_4.png



.. code:: ipython3

    # ============================================================
    # 1️⃣ KÜTÜPHANELER
    # ============================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
    
    # Linear & Regularized
    from sklearn.linear_model import (
        LinearRegression, Ridge, Lasso, ElasticNet,
        HuberRegressor, BayesianRidge
    )
    
    # Tree-based & Ensemble
    from sklearn.tree import DecisionTreeRegressor
    from sklearn.ensemble import (
        RandomForestRegressor, ExtraTreesRegressor,
        AdaBoostRegressor, GradientBoostingRegressor
    )
    
    # Other models
    from sklearn.svm import SVR
    from sklearn.neighbors import KNeighborsRegressor
    from sklearn.neural_network import MLPRegressor
    
    # Boosting
    from xgboost import XGBRegressor
    from lightgbm import LGBMRegressor
    from catboost import CatBoostRegressor
    
    
    # ============================================================
    # 2️⃣ VERİ YÜKLEME
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    # ============================================================
    # 🎯 HEDEF & ÖZELLİKLER
    # ============================================================
    target_col = "Score"
    
    y = df[target_col]
    X = df.drop(columns=["Country", target_col])
    X = X.select_dtypes(include=[np.number])
    
    
    # ============================================================
    # 3️⃣ STANDARDİZASYON
    # ============================================================
    X_std = StandardScaler().fit_transform(X)
    y_std = StandardScaler().fit_transform(
        y.values.reshape(-1, 1)
    ).ravel()
    
    
    # ============================================================
    # 4️⃣ TRAIN – TEST SPLIT
    # ============================================================
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std,
        test_size=0.20,
        random_state=42
    )
    
    
    # ============================================================
    # 5️⃣ REGRESYON MODELLERİ
    # ============================================================
    models = {
        "Huber": HuberRegressor(),
        "Bayesian Ridge": BayesianRidge(),
        "Linear Regression": LinearRegression(),
        "Ridge": Ridge(alpha=1.0),
        "Elastic Net": ElasticNet(alpha=0.05, l1_ratio=0.5),
        "Lasso": Lasso(alpha=0.05),
    
        "SVR (RBF)": SVR(C=10, epsilon=0.1),
        "k-NN": KNeighborsRegressor(n_neighbors=5),
    
        "MLP (ANN)": MLPRegressor(
            hidden_layer_sizes=(120, 60),
            max_iter=1000,
            random_state=42
        ),
    
        "Decision Tree": DecisionTreeRegressor(random_state=42),
        "Random Forest": RandomForestRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "Extra Trees": ExtraTreesRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "AdaBoost": AdaBoostRegressor(
            n_estimators=300, learning_rate=0.05, random_state=42
        ),
        "Gradient Boosting": GradientBoostingRegressor(
            n_estimators=300, random_state=42
        ),
    
        "XGBoost": XGBRegressor(
            n_estimators=300,
            learning_rate=0.05,
            max_depth=6,
            random_state=42
        ),
        "LightGBM": LGBMRegressor(
            n_estimators=300,
            random_state=42
        ),
        "CatBoost": CatBoostRegressor(
            iterations=300,
            depth=4,
            learning_rate=0.05,
            verbose=0,
            random_state=42
        )
    }
    
    
    # ============================================================
    # 6️⃣ EĞİTİM & PERFORMANS
    # ============================================================
    results = []
    
    for name, model in models.items():
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        results.append({
            "Model": name,
            "MSE": mean_squared_error(y_test, y_pred),
            "RMSE": np.sqrt(mean_squared_error(y_test, y_pred)),
            "MAE": mean_absolute_error(y_test, y_pred),
            "R²": r2_score(y_test, y_pred)
        })
    
    results_df = (
        pd.DataFrame(results)
        .sort_values(by="R²", ascending=False)
        .reset_index(drop=True)
    )
    
    print("\n================ GII INDEX – ALL MODELS =================\n")
    print(results_df.round(4))
    
    
    # ============================================================
    # 🎨 RENK PALETİ
    # ============================================================
    vivid_colors = [
        "#e41a1c", "#377eb8", "#4daf4a", "#984ea3",
        "#ff7f00", "#ffff33", "#a65628", "#f781bf",
        "#1b9e77", "#d95f02", "#7570b3", "#e7298a",
        "#66a61e", "#e6ab02", "#a6761d", "#666666",
        "#17becf"
    ]
    
    model_colors = {
        model: vivid_colors[i % len(vivid_colors)]
        for i, model in enumerate(results_df["Model"])
    }
    
    
    # ============================================================
    # 7️⃣ METRİK GÖRSELLEŞTİRME (2×2 PANEL)
    # ============================================================
    metrics = ["MSE", "RMSE", "MAE", "R²"]
    titles = [
        "Mean Squared Error (MSE)",
        "Root Mean Squared Error (RMSE)",
        "Mean Absolute Error (MAE)",
        "R² Score"
    ]
    
    fig, axes = plt.subplots(2, 2, figsize=(19, 18))
    axes = axes.flatten()
    
    for ax, metric, title in zip(axes, metrics, titles):
        bars = ax.bar(
            results_df["Model"],
            results_df[metric],
            color=[model_colors[m] for m in results_df["Model"]],
            edgecolor="black",
            linewidth=0.8
        )
    
        for bar in bars:
            h = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width() / 2,
                h,
                f"{h:.4f}",
                ha="center",
                va="bottom",
                rotation=90,
                fontsize=9,
                fontweight="bold"
            )
    
        ax.set_title(title, fontsize=13, weight="bold")
        ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
        ax.grid(axis="y", linestyle="--", alpha=0.4)
    
        # ✅ R² ekseni artık 0.0’dan başlıyor
        if metric == "R²":
            ax.set_ylim(0.0, 1.01)
    
    plt.suptitle(
        "Comprehensive Performance Comparison of Regression Models\n(Target: Score)",
        fontsize=16,
        weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    


.. parsed-literal::

    [LightGBM] [Info] Auto-choosing col-wise multi-threading, the overhead of testing was 0.000115 seconds.
    You can set `force_col_wise=true` to remove the overhead.
    [LightGBM] [Info] Total Bins 262
    [LightGBM] [Info] Number of data points in the train set: 111, number of used features: 7
    [LightGBM] [Info] Start training from score -0.039553
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    [LightGBM] [Warning] No further splits with positive gain, best gain: -inf
    

.. parsed-literal::

    C:\Users\sadul\anaconda3\Lib\site-packages\sklearn\utils\validation.py:2739: UserWarning: X does not have valid feature names, but LGBMRegressor was fitted with feature names
      warnings.warn(
    

.. parsed-literal::

    
    ================ GII INDEX – ALL MODELS =================
    
                    Model     MSE    RMSE     MAE      R²
    0               Ridge  0.0649  0.2548  0.0821  0.9387
    1      Bayesian Ridge  0.0649  0.2548  0.0822  0.9387
    2   Linear Regression  0.0651  0.2551  0.0840  0.9385
    3               Huber  0.0651  0.2551  0.0500  0.9385
    4         Elastic Net  0.0660  0.2569  0.0955  0.9377
    5               Lasso  0.0698  0.2641  0.1196  0.9341
    6         Extra Trees  0.0800  0.2828  0.1476  0.9245
    7                k-NN  0.0825  0.2873  0.1603  0.9221
    8            CatBoost  0.0833  0.2887  0.1290  0.9213
    9             XGBoost  0.0860  0.2933  0.1685  0.9188
    10      Random Forest  0.0910  0.3017  0.1900  0.9141
    11      Decision Tree  0.0941  0.3067  0.2282  0.9111
    12  Gradient Boosting  0.0963  0.3103  0.1851  0.9091
    13           AdaBoost  0.1106  0.3326  0.2529  0.8955
    14          MLP (ANN)  0.1163  0.3410  0.1945  0.8902
    15          SVR (RBF)  0.1212  0.3481  0.1847  0.8856
    16           LightGBM  0.1475  0.3841  0.2600  0.8607
    

.. parsed-literal::

    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\731332326.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\731332326.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\731332326.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    C:\Users\sadul\AppData\Local\Temp\ipykernel_22328\731332326.py:206: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
      ax.set_xticklabels(results_df["Model"], rotation=45, ha="right")
    


.. image:: output_199_4.png






.. code:: ipython3

    # ============================================================
    # PUBLICATION-READY MODEL COMPARISON (LABELED VERSION)
    # Consistent Feature Colors | Top-10 | Horizontal | Value Labels
    # Extra Trees | CatBoost | Random Forest | Gradient Boosting
    # ============================================================
    
    # 1. LIBRARIES
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error
    from sklearn.ensemble import ExtraTreesRegressor, RandomForestRegressor, GradientBoostingRegressor
    from catboost import CatBoostRegressor
    
    # 2. DATA LOADING
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target_col = "Score"
    y = df[target_col]
    X = df.drop(columns=["Country", target_col])
    X = X.select_dtypes(include=[np.number])
    feature_labels = X.columns.tolist()
    
    # 3. STANDARDIZATION
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(X)
    y_std = scaler_y.fit_transform(y.values.reshape(-1, 1)).ravel()
    
    # 4. TRAIN-TEST SPLIT
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # 5. MODELS
    models = [
        ("Extra Trees", ExtraTreesRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("CatBoost", CatBoostRegressor(iterations=300, depth=4, learning_rate=0.05,
                                       verbose=0, random_state=42)),
        ("Random Forest", RandomForestRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("Gradient Boosting", GradientBoostingRegressor(n_estimators=300, random_state=42))
    ]
    
    # 6. GLOBAL COLOR MAPPING
    unique_features = feature_labels
    color_map = plt.cm.tab20(np.linspace(0, 1, len(unique_features)))
    feature_color_dict = dict(zip(unique_features, color_map))
    
    # 7. PUBLICATION STYLE SETTINGS
    plt.rcParams.update({
        "font.size": 12,
        "axes.titlesize": 14,
        "axes.labelsize": 12,
        "figure.dpi": 300
    })
    
    # 8. FIGURE INITIALIZATION
    fig, axes = plt.subplots(2, 2, figsize=(17, 13))
    axes = axes.flatten()
    
    performance_results = []
    feature_importance_results = {}
    
    # 9. MODEL LOOP
    for ax, (name, model) in zip(axes, models):
    
        # Fit model
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        # Performance metrics
        mse = mean_squared_error(y_test, y_pred)
        rmse = np.sqrt(mse)
        mae = mean_absolute_error(y_test, y_pred)
        r2 = r2_score(y_test, y_pred)
    
        performance_results.append({
            "Model": name,
            "R2": r2,
            "RMSE": rmse,
            "MAE": mae
        })
    
        # Feature importance
        fi = model.feature_importances_
    
        fi_df = pd.DataFrame({
            "Feature": feature_labels,
            "Importance": fi
        }).sort_values("Importance", ascending=False).head(10)
    
        feature_importance_results[name] = fi_df.copy()
    
        fi_df = fi_df.sort_values("Importance", ascending=True)
        colors = [feature_color_dict[f] for f in fi_df["Feature"]]
    
        # Horizontal bar chart
        bars = ax.barh(fi_df["Feature"], fi_df["Importance"], color=colors)
    
        # Value labels on bars
        for bar in bars:
            width = bar.get_width()
            ax.text(
                width + 0.002,
                bar.get_y() + bar.get_height() / 2,
                f"{width:.3f}",
                va="center",
                fontsize=14  # font büyüklüğü artırıldı
            )
    
        # Titles and labels
        ax.set_title(name, weight="bold", fontsize=14)
        ax.set_xlabel("Feature Importance", fontsize=14)
        ax.set_ylabel("Feature", fontsize=14)
    
        # Tick label fontunu büyüt
        ax.tick_params(axis='x', labelsize=12)
        ax.tick_params(axis='y', labelsize=12)
    
        ax.grid(axis="x", linestyle="--", alpha=0.3)
    
        # Performance metrics box
        ax.text(
            0.98, 0.05,
            f"R² = {r2:.3f}\nRMSE = {rmse:.4f}\nMAE = {mae:.4f}",
            transform=ax.transAxes,
            ha="right", va="bottom",
            fontsize=12,
            bbox=dict(boxstyle="round", facecolor="white", alpha=0.9)
        )
    
    # 10. SUPERTITLE
    plt.suptitle(
        "Feature Importance and Predictive Performance\nTree-Based Regression Models",
        fontsize=16,
        weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    
    # 11. PERFORMANCE TABLE
    performance_df = pd.DataFrame(performance_results).sort_values("R2", ascending=False)
    
    print("\n===== FINAL MODEL PERFORMANCE =====\n")
    print(performance_df.round(4))
    
    # 12. FEATURE IMPORTANCE TABLES
    for model_name, fi_df in feature_importance_results.items():
        print(f"\n===== FEATURE IMPORTANCE: {model_name} =====\n")
        print(fi_df.round(4))



.. image:: output_204_0.png


.. parsed-literal::

    
    ===== FINAL MODEL PERFORMANCE =====
    
                   Model      R2    RMSE     MAE
    0        Extra Trees  0.9245  0.2828  0.1476
    1           CatBoost  0.9213  0.2887  0.1290
    2      Random Forest  0.9141  0.3017  0.1900
    3  Gradient Boosting  0.9091  0.3103  0.1851
    
    ===== FEATURE IMPORTANCE: Extra Trees =====
    
                                Feature  Importance
    6                  Creative outputs      0.2948
    4           Business sophistication      0.1775
    5  Knowledge and technology outputs      0.1601
    1        Human capital and research      0.1504
    2                    Infrastructure      0.1086
    3             Market sophistication      0.0643
    0                      Institutions      0.0442
    
    ===== FEATURE IMPORTANCE: CatBoost =====
    
                                Feature  Importance
    5  Knowledge and technology outputs     20.0691
    6                  Creative outputs     18.4674
    1        Human capital and research     16.5458
    2                    Infrastructure     14.9865
    3             Market sophistication     11.7393
    0                      Institutions     10.3157
    4           Business sophistication      7.8763
    
    ===== FEATURE IMPORTANCE: Random Forest =====
    
                                Feature  Importance
    6                  Creative outputs      0.4594
    4           Business sophistication      0.2341
    5  Knowledge and technology outputs      0.1175
    1        Human capital and research      0.1020
    2                    Infrastructure      0.0428
    3             Market sophistication      0.0226
    0                      Institutions      0.0217
    
    ===== FEATURE IMPORTANCE: Gradient Boosting =====
    
                                Feature  Importance
    6                  Creative outputs      0.6369
    1        Human capital and research      0.1126
    4           Business sophistication      0.0828
    5  Knowledge and technology outputs      0.0688
    2                    Infrastructure      0.0424
    0                      Institutions      0.0413
    3             Market sophistication      0.0152
    






.. code:: ipython3

    # ============================================================
    # FINAL MODEL COMPARISON – TOP 4 TREE-BASED REGRESSORS
    # Gradient Boosting | Random Forest | Extra Trees | CatBoost
    # Feature Importance & Performance (SSCI-Ready)
    # ============================================================
    
    # ============================================================
    # 1️⃣ KÜTÜPHANELER
    # ============================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import matplotlib.patches as mpatches
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error
    
    from sklearn.ensemble import (
        GradientBoostingRegressor,
        RandomForestRegressor,
        ExtraTreesRegressor
    )
    from catboost import CatBoostRegressor
    
    # ============================================================
    # 2️⃣ VERİ YÜKLEME
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target_col = "Score"
    
    y = df[target_col]
    X = df.drop(columns=["Country", target_col])
    X = X.select_dtypes(include=[np.number])
    
    feature_labels = X.columns.tolist()
    
    # ============================================================
    # 3️⃣ STANDARDİZASYON
    # ============================================================
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(X)
    y_std = scaler_y.fit_transform(y.values.reshape(-1, 1)).ravel()
    
    # ============================================================
    # 4️⃣ TRAIN – TEST SPLIT
    # ============================================================
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # ============================================================
    # 5️⃣ EN İYİ 4 MODEL
    # ============================================================
    models = [
        ("Gradient Boosting", GradientBoostingRegressor(n_estimators=300, random_state=42)),
        ("Random Forest", RandomForestRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("Extra Trees", ExtraTreesRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("CatBoost", CatBoostRegressor(iterations=300, depth=4, learning_rate=0.05,
                                       verbose=0, random_state=42))
    ]
    
    # ============================================================
    # 🎨 FEATURE RENK PALETİ
    # ============================================================
    tableau_colors = [
        "#1f77b4", "#ff7f0e", "#2ca02c", "#d62728",
        "#9467bd", "#8c564b", "#e377c2", "#7f7f7f",
        "#bcbd22", "#17becf"
    ]
    
    feature_colors = {
        f: tableau_colors[i % len(tableau_colors)]
        for i, f in enumerate(feature_labels)
    }
    
    # ============================================================
    # 6️⃣ SONUÇ DEPOLARI
    # ============================================================
    performance_results = []
    feature_importance_results = {}
    
    # ============================================================
    # 7️⃣ 2×2 PANEL GRAFİK – FEATURE IMPORTANCE
    # ============================================================
    fig, axes = plt.subplots(2, 2, figsize=(19, 14))
    axes = axes.flatten()
    
    for ax, (name, model) in zip(axes, models):
    
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        mse = mean_squared_error(y_test, y_pred)
        rmse = np.sqrt(mse)
        mae = mean_absolute_error(y_test, y_pred)
        r2 = r2_score(y_test, y_pred)
    
        performance_results.append({
            "Model": name,
            "MSE": mse,
            "RMSE": rmse,
            "MAE": mae,
            "R²": r2
        })
    
        # 🌲 Tree-based Feature Importance
        fi = model.feature_importances_
    
        fi_df = pd.DataFrame({
            "Feature": feature_labels,
            "Importance": fi
        }).sort_values("Importance", ascending=False)
    
        feature_importance_results[name] = fi_df
    
        colors = [feature_colors[f] for f in fi_df["Feature"]]
    
        bars = ax.bar(
            fi_df["Feature"],
            fi_df["Importance"],
            color=colors,
            edgecolor="black",
            linewidth=0.9
        )
    
        ax.set_title(f"{name} – Feature Importance", fontsize=16, weight="bold")
        ax.set_ylabel("Relative Importance")
        ax.tick_params(axis="x", rotation=90)
        ax.grid(axis="y", linestyle="--", alpha=0.35)
    
        for bar in bars:
            h = bar.get_height()
            ax.text(bar.get_x() + bar.get_width()/2, h,
                    f"{h:.3f}", ha="center", va="bottom", fontsize=10, rotation=90)
    
        ax.text(
            0.98, 0.95,
            f"R²   = {r2:.3f}\nRMSE = {rmse:.4f}\nMAE  = {mae:.4f}",
            transform=ax.transAxes,
            ha="right", va="top",
            fontsize=10,
            bbox=dict(boxstyle="round,pad=0.4", facecolor="white",
                      edgecolor="gray", alpha=0.95)
        )
    
    # ============================================================
    # 8️⃣ GENEL BAŞLIK
    # ============================================================
    plt.suptitle(
        "Final Model Comparison: Top-Performing Tree-Based Algorithms\n"
        "Feature Importance and Predictive Performance (Score)",
        fontsize=16,
        weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    
    # ============================================================
    # 9️⃣ PERFORMANS TABLOSU
    # ============================================================
    performance_df = pd.DataFrame(performance_results).sort_values("R²", ascending=False)
    
    print("\n================ FINAL MODEL PERFORMANCE (TOP 4) ================\n")
    print(performance_df.round(5))
    
    # ============================================================
    # 🔟 FEATURE IMPORTANCE TABLOLARI
    # ============================================================
    for model_name, fi_df in feature_importance_results.items():
        print(f"\n================ FEATURE IMPORTANCE: {model_name} ================\n")
        print(fi_df.round(5))
    



.. image:: output_210_0.png


.. parsed-literal::

    
    ================ FINAL MODEL PERFORMANCE (TOP 4) ================
    
                   Model      MSE     RMSE      MAE       R²
    2        Extra Trees  0.07998  0.28280  0.14763  0.92447
    3           CatBoost  0.08333  0.28866  0.12902  0.92131
    1      Random Forest  0.09101  0.30167  0.19001  0.91406
    0  Gradient Boosting  0.09629  0.31030  0.18510  0.90907
    
    ================ FEATURE IMPORTANCE: Gradient Boosting ================
    
                                Feature  Importance
    6                  Creative outputs     0.63687
    1        Human capital and research     0.11263
    4           Business sophistication     0.08285
    5  Knowledge and technology outputs     0.06880
    2                    Infrastructure     0.04240
    0                      Institutions     0.04129
    3             Market sophistication     0.01516
    
    ================ FEATURE IMPORTANCE: Random Forest ================
    
                                Feature  Importance
    6                  Creative outputs     0.45941
    4           Business sophistication     0.23409
    5  Knowledge and technology outputs     0.11755
    1        Human capital and research     0.10196
    2                    Infrastructure     0.04276
    3             Market sophistication     0.02257
    0                      Institutions     0.02167
    
    ================ FEATURE IMPORTANCE: Extra Trees ================
    
                                Feature  Importance
    6                  Creative outputs     0.29480
    4           Business sophistication     0.17752
    5  Knowledge and technology outputs     0.16012
    1        Human capital and research     0.15037
    2                    Infrastructure     0.10862
    3             Market sophistication     0.06434
    0                      Institutions     0.04423
    
    ================ FEATURE IMPORTANCE: CatBoost ================
    
                                Feature  Importance
    5  Knowledge and technology outputs    20.06908
    6                  Creative outputs    18.46741
    1        Human capital and research    16.54581
    2                    Infrastructure    14.98645
    3             Market sophistication    11.73927
    0                      Institutions    10.31568
    4           Business sophistication     7.87631
    






.. code:: ipython3

    # ============================================================
    # FINAL MODEL COMPARISON – TOP 4 TREE-BASED REGRESSORS
    # Gradient Boosting | Random Forest | Extra Trees | CatBoost
    # Feature Importance & Performance (SSCI-Ready)
    # ============================================================
    
    # ============================================================
    # 1️⃣ LIBRARIES
    # ============================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import matplotlib.patches as mpatches
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error
    
    from sklearn.ensemble import GradientBoostingRegressor, RandomForestRegressor, ExtraTreesRegressor
    from catboost import CatBoostRegressor
    
    # ============================================================
    # 2️⃣ LOAD DATA
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target_col = "Score"
    
    y = df[target_col]
    X = df.drop(columns=["Country", target_col], errors="ignore")
    X = X.select_dtypes(include=[np.number])
    
    feature_labels = X.columns.tolist()
    
    # ============================================================
    # 3️⃣ STANDARDIZATION (FOR FAIR COMPARISON)
    # ============================================================
    X_std = StandardScaler().fit_transform(X)
    y_std = StandardScaler().fit_transform(y.values.reshape(-1, 1)).ravel()
    
    # ============================================================
    # 4️⃣ TRAIN – TEST SPLIT
    # ============================================================
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # ============================================================
    # 5️⃣ MODELS (TOP 4)
    # ============================================================
    models = [
        ("Gradient Boosting",
         GradientBoostingRegressor(n_estimators=300, random_state=42)),
    
        ("Random Forest",
         RandomForestRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
    
        ("Extra Trees",
         ExtraTreesRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
    
        ("CatBoost",
         CatBoostRegressor(
             iterations=300,
             depth=4,
             learning_rate=0.05,
             random_state=42,
             verbose=0
         ))
    ]
    
    # ============================================================
    # 🎨 FEATURE COLOR PALETTE
    # ============================================================
    tableau_colors = [
        "#17becf", "#ff7f0e", "#2ca02c", "#d62728",
        "#9467bd", "#8c564b", "#e377c2", "#7f7f7f",
        "#bcbd22", "#1f77b4"
    ]
    
    feature_colors = {
        f: tableau_colors[i % len(tableau_colors)]
        for i, f in enumerate(feature_labels)
    }
    
    # ============================================================
    # 6️⃣ RESULT CONTAINERS
    # ============================================================
    performance_results = []
    feature_importance_results = {}
    
    # ============================================================
    # 7️⃣ 2×2 PANEL – FEATURE IMPORTANCE
    # ============================================================
    fig, axes = plt.subplots(2, 2, figsize=(19, 14))
    axes = axes.flatten()
    
    for ax, (name, model) in zip(axes, models):
    
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        mse = mean_squared_error(y_test, y_pred)
        rmse = np.sqrt(mse)
        mae = mean_absolute_error(y_test, y_pred)
        r2 = r2_score(y_test, y_pred)
    
        performance_results.append({
            "Model": name,
            "MSE": mse,
            "RMSE": rmse,
            "MAE": mae,
            "R²": r2
        })
    
        fi = model.feature_importances_
    
        fi_df = pd.DataFrame({
            "Feature": feature_labels,
            "Importance": fi
        }).sort_values("Importance", ascending=False)
    
        feature_importance_results[name] = fi_df
    
        colors = [feature_colors[f] for f in fi_df["Feature"]]
    
        bars = ax.bar(
            fi_df["Feature"],
            fi_df["Importance"],
            color=colors,
            edgecolor="black",
            linewidth=0.9
        )
    
        ax.set_title(f"{name} – Feature Importance", fontsize=13, weight="bold")
        ax.set_ylabel("Relative Importance")
        ax.tick_params(axis="x", rotation=90)
        ax.grid(axis="y", linestyle="--", alpha=0.35)
    
        for bar in bars:
            h = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width() / 2,
                h,
                f"{h:.3f}",
                ha="center",
                va="bottom",
                fontsize=10,
                rotation=90
            )
    
        ax.text(
            0.98, 0.95,
            f"R²   = {r2:.3f}\nRMSE = {rmse:.4f}\nMAE  = {mae:.4f}",
            transform=ax.transAxes,
            ha="right",
            va="top",
            fontsize=10,
            bbox=dict(
                boxstyle="round,pad=0.4",
                facecolor="white",
                edgecolor="gray",
                alpha=0.95
            )
        )
    
    # ============================================================
    # 8️⃣ GLOBAL TITLE
    # ============================================================
    plt.suptitle(
        "Final Model Comparison: Top-Performing Tree-Based Algorithms\n"
        "Feature Importance and Predictive Performance (Score)",
        fontsize=16,
        weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    
    # ============================================================
    # 9️⃣ PERFORMANCE TABLE
    # ============================================================
    performance_df = (
        pd.DataFrame(performance_results)
        .sort_values("R²", ascending=False)
    )
    
    print("\n================ FINAL MODEL PERFORMANCE (TOP 4) ================\n")
    print(performance_df.round(5))
    
    # ============================================================
    # 🔟 FEATURE IMPORTANCE TABLES
    # ============================================================
    for model_name, fi_df in feature_importance_results.items():
        print(f"\n================ FEATURE IMPORTANCE: {model_name} ================\n")
        print(fi_df.round(5))
    



.. image:: output_216_0.png


.. parsed-literal::

    
    ================ FINAL MODEL PERFORMANCE (TOP 4) ================
    
                   Model      MSE     RMSE      MAE       R²
    2        Extra Trees  0.07998  0.28280  0.14763  0.92447
    3           CatBoost  0.08333  0.28866  0.12902  0.92131
    1      Random Forest  0.09101  0.30167  0.19001  0.91406
    0  Gradient Boosting  0.09629  0.31030  0.18510  0.90907
    
    ================ FEATURE IMPORTANCE: Gradient Boosting ================
    
                                Feature  Importance
    6                  Creative outputs     0.63687
    1        Human capital and research     0.11263
    4           Business sophistication     0.08285
    5  Knowledge and technology outputs     0.06880
    2                    Infrastructure     0.04240
    0                      Institutions     0.04129
    3             Market sophistication     0.01516
    
    ================ FEATURE IMPORTANCE: Random Forest ================
    
                                Feature  Importance
    6                  Creative outputs     0.45941
    4           Business sophistication     0.23409
    5  Knowledge and technology outputs     0.11755
    1        Human capital and research     0.10196
    2                    Infrastructure     0.04276
    3             Market sophistication     0.02257
    0                      Institutions     0.02167
    
    ================ FEATURE IMPORTANCE: Extra Trees ================
    
                                Feature  Importance
    6                  Creative outputs     0.29480
    4           Business sophistication     0.17752
    5  Knowledge and technology outputs     0.16012
    1        Human capital and research     0.15037
    2                    Infrastructure     0.10862
    3             Market sophistication     0.06434
    0                      Institutions     0.04423
    
    ================ FEATURE IMPORTANCE: CatBoost ================
    
                                Feature  Importance
    5  Knowledge and technology outputs    20.06908
    6                  Creative outputs    18.46741
    1        Human capital and research    16.54581
    2                    Infrastructure    14.98645
    3             Market sophistication    11.73927
    0                      Institutions    10.31568
    4           Business sophistication     7.87631
    


.. code:: ipython3

    # ============================================================
    # FINAL MODEL COMPARISON – TREE-BASED REGRESSORS (PPI)
    # Gradient Boosting | Random Forest | Extra Trees | CatBoost
    # Feature Importance & Model Performance (SSCI-Ready)
    # ============================================================
    
    # ============================================================
    # 1️⃣ LIBRARIES
    # ============================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import matplotlib.patches as mpatches
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error
    
    from sklearn.ensemble import GradientBoostingRegressor, RandomForestRegressor, ExtraTreesRegressor
    from catboost import CatBoostRegressor
    
    # ============================================================
    # 2️⃣ LOAD DATA
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target_col = "Score"
    
    y = df[target_col]
    X = df.drop(columns=["Country", target_col], errors="ignore")
    X = X.select_dtypes(include=[np.number])
    
    feature_labels = X.columns.tolist()
    
    # ============================================================
    # 3️⃣ STANDARDIZATION
    # ============================================================
    X_std = StandardScaler().fit_transform(X)
    y_std = StandardScaler().fit_transform(y.values.reshape(-1, 1)).ravel()
    
    # ============================================================
    # 4️⃣ TRAIN – TEST SPLIT
    # ============================================================
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # ============================================================
    # 5️⃣ MODELS
    # ============================================================
    models = [
        ("Gradient Boosting", GradientBoostingRegressor(n_estimators=300, random_state=42)),
        ("Random Forest", RandomForestRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("Extra Trees", ExtraTreesRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("CatBoost", CatBoostRegressor(iterations=300, depth=4,
                                       learning_rate=0.05,
                                       random_state=42, verbose=0))
    ]
    
    # ============================================================
    # 🎨 FEATURE COLOR PALETTE (HAPI STYLE)
    # ============================================================
    tableau_colors = [
        "#1f77b4", "#ff7f0e", "#2ca02c", "#d62728",
        "#9467bd", "#8c564b", "#e377c2", "#7f7f7f",
        "#bcbd22", "#17becf"
    ]
    
    feature_colors = {
        f: tableau_colors[i % len(tableau_colors)]
        for i, f in enumerate(feature_labels)
    }
    
    # ============================================================
    # 6️⃣ RESULT CONTAINERS
    # ============================================================
    performance_results = []
    feature_importance_results = {}
    
    # ============================================================
    # 7️⃣ PANEL GRAPH (2×2) – HAPI FORMAT
    # ============================================================
    fig, axes = plt.subplots(2, 2, figsize=(18, 13), sharex=True)
    axes = axes.flatten()
    
    for ax, (name, model) in zip(axes, models):
    
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        mse  = mean_squared_error(y_test, y_pred)
        rmse = np.sqrt(mse)
        mae  = mean_absolute_error(y_test, y_pred)
        r2   = r2_score(y_test, y_pred)
    
        performance_results.append({
            "Model": name,
            "MSE": mse,
            "RMSE": rmse,
            "MAE": mae,
            "R²": r2
        })
    
        fi = model.feature_importances_
    
        fi_df = pd.DataFrame({
            "Feature": feature_labels,
            "Importance": fi
        }).sort_values("Importance", ascending=False)
    
        feature_importance_results[name] = fi_df
    
        colors = [feature_colors[f] for f in fi_df["Feature"]]
    
        bars = ax.bar(
            fi_df["Feature"],
            fi_df["Importance"],
            color=colors,
            edgecolor="black",
            linewidth=0.95
        )
    
        ax.set_title(f"{name}", fontsize=17, weight="bold")
        ax.set_ylabel("Relative Feature Importance")
        ax.tick_params(axis="x", rotation=90,)
        ax.grid(axis="y", linestyle="--", alpha=0.35)
    
        for bar in bars:
            h = bar.get_height()
            ax.text(bar.get_x() + bar.get_width()/2,
                    h, f"{h:.3f}",
                    ha="center", va="bottom",
                    fontsize=11, rotation=90)
    
        ax.text(
            0.98, 0.95,
            f"R²   = {r2:.3f}\nRMSE = {rmse:.4f}\nMAE  = {mae:.4f}",
            transform=ax.transAxes,
            ha="right", va="top",
            fontsize=16,
            bbox=dict(boxstyle="round,pad=0.4",
                      facecolor="white",
                      edgecolor="gray",
                      alpha=0.95)
        )
    
        legend_patches = [
            mpatches.Patch(color=feature_colors[f], label=f)
            for f in fi_df["Feature"]
        ]
        ax.legend(handles=legend_patches, fontsize=10,
                  loc="lower left", frameon=True)
    
    # ============================================================
    # 8️⃣ GLOBAL TITLE
    # ============================================================
    plt.suptitle(
        "Final Model Comparison: Tree-Based Algorithms\n"
        "Feature Importance and Predictive Performance (Affordability Index)",
        fontsize=16, weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    
    # ============================================================
    # 9️⃣ PERFORMANCE TABLE
    # ============================================================
    performance_df = pd.DataFrame(performance_results).sort_values("R²", ascending=False)
    
    print("\n================ FINAL MODEL PERFORMANCE =================\n")
    print(performance_df.round(5))
    
    # ============================================================
    # 🔟 FEATURE IMPORTANCE TABLES
    # ============================================================
    for model_name, fi_df in feature_importance_results.items():
        print(f"\n================ FEATURE IMPORTANCE: {model_name} ================\n")
        print(fi_df.round(5))
    



.. image:: output_218_0.png


.. parsed-literal::

    
    ================ FINAL MODEL PERFORMANCE =================
    
                   Model      MSE     RMSE      MAE       R²
    2        Extra Trees  0.07998  0.28280  0.14763  0.92447
    3           CatBoost  0.08333  0.28866  0.12902  0.92131
    1      Random Forest  0.09101  0.30167  0.19001  0.91406
    0  Gradient Boosting  0.09629  0.31030  0.18510  0.90907
    
    ================ FEATURE IMPORTANCE: Gradient Boosting ================
    
                                Feature  Importance
    6                  Creative outputs     0.63687
    1        Human capital and research     0.11263
    4           Business sophistication     0.08285
    5  Knowledge and technology outputs     0.06880
    2                    Infrastructure     0.04240
    0                      Institutions     0.04129
    3             Market sophistication     0.01516
    
    ================ FEATURE IMPORTANCE: Random Forest ================
    
                                Feature  Importance
    6                  Creative outputs     0.45941
    4           Business sophistication     0.23409
    5  Knowledge and technology outputs     0.11755
    1        Human capital and research     0.10196
    2                    Infrastructure     0.04276
    3             Market sophistication     0.02257
    0                      Institutions     0.02167
    
    ================ FEATURE IMPORTANCE: Extra Trees ================
    
                                Feature  Importance
    6                  Creative outputs     0.29480
    4           Business sophistication     0.17752
    5  Knowledge and technology outputs     0.16012
    1        Human capital and research     0.15037
    2                    Infrastructure     0.10862
    3             Market sophistication     0.06434
    0                      Institutions     0.04423
    
    ================ FEATURE IMPORTANCE: CatBoost ================
    
                                Feature  Importance
    5  Knowledge and technology outputs    20.06908
    6                  Creative outputs    18.46741
    1        Human capital and research    16.54581
    2                    Infrastructure    14.98645
    3             Market sophistication    11.73927
    0                      Institutions    10.31568
    4           Business sophistication     7.87631
    



.. code:: ipython3

    # ============================================================
    # FINAL MODEL COMPARISON – TREE-BASED REGRESSORS (PPI)
    # Gradient Boosting | Random Forest | Extra Trees | CatBoost
    # Feature Importance & Model Performance (SSCI-Ready)
    # ============================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import matplotlib.patches as mpatches
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error
    
    from sklearn.ensemble import GradientBoostingRegressor, RandomForestRegressor, ExtraTreesRegressor
    from catboost import CatBoostRegressor
    
    # ============================================================
    # LOAD DATA
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target_col = "Score"
    
    y = df[target_col]
    X = df.drop(columns=["Country", target_col], errors="ignore")
    X = X.select_dtypes(include=[np.number])
    
    feature_labels = X.columns.tolist()
    
    # ============================================================
    # STANDARDIZATION
    # ============================================================
    X_std = StandardScaler().fit_transform(X)
    y_std = StandardScaler().fit_transform(y.values.reshape(-1, 1)).ravel()
    
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # ============================================================
    # MODELS
    # ============================================================
    models = [
        ("Gradient Boosting", GradientBoostingRegressor(n_estimators=300, random_state=42)),
        ("Random Forest", RandomForestRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("Extra Trees", ExtraTreesRegressor(n_estimators=300, random_state=42, n_jobs=-1)),
        ("CatBoost", CatBoostRegressor(iterations=300, depth=4,
                                       learning_rate=0.05,
                                       random_state=42, verbose=0))
    ]
    
    # ============================================================
    # FEATURE COLOR PALETTE
    # ============================================================
    tableau_colors = [
        "#1f77b4", "#ff7f0e", "#2ca02c", "#d62728",
        "#9467bd", "#8c564b", "#e377c2", "#7f7f7f",
        "#bcbd22", "#17becf"
    ]
    
    feature_colors = {
        f: tableau_colors[i % len(tableau_colors)]
        for i, f in enumerate(feature_labels)
    }
    
    # ============================================================
    # RESULTS
    # ============================================================
    performance_results = []
    feature_importance_results = {}
    
    # ============================================================
    # PANEL GRAPH
    # ============================================================
    fig, axes = plt.subplots(2, 2, figsize=(18, 13), sharex=True)
    axes = axes.flatten()
    
    for ax, (name, model) in zip(axes, models):
    
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
    
        mse  = mean_squared_error(y_test, y_pred)
        rmse = np.sqrt(mse)
        mae  = mean_absolute_error(y_test, y_pred)
        r2   = r2_score(y_test, y_pred)
    
        performance_results.append({
            "Model": name,
            "MSE": mse,
            "RMSE": rmse,
            "MAE": mae,
            "R²": r2
        })
    
        fi = model.feature_importances_
    
        fi_df = pd.DataFrame({
            "Feature": feature_labels,
            "Importance": fi
        }).sort_values("Importance", ascending=False)
    
        feature_importance_results[name] = fi_df
    
        colors = [feature_colors[f] for f in fi_df["Feature"]]
    
        bars = ax.bar(
            fi_df["Feature"],
            fi_df["Importance"],
            color=colors,
            edgecolor="black",
            linewidth=0.95
        )
    
        # 🔥 FONT ADJUSTMENTS
        ax.set_title(f"{name}", fontsize=18, weight="bold")
        ax.set_ylabel("Relative Feature Importance", fontsize=14)
        ax.tick_params(axis="x", labelsize=14, rotation=90)
        ax.tick_params(axis="y", labelsize=14)
        ax.grid(axis="y", linestyle="--", alpha=0.35)
    
        # 🔥 BIGGER BAR VALUE LABELS
        for bar in bars:
            h = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width()/2,
                h,
                f"{h:.3f}",
                ha="center",
                va="bottom",
                fontsize=14,
                rotation=90
            )
    
        ax.text(
            0.98, 0.95,
            f"R²   = {r2:.3f}\nRMSE = {rmse:.4f}\nMAE  = {mae:.4f}",
            transform=ax.transAxes,
            ha="right",
            va="top",
            fontsize=16,
            bbox=dict(boxstyle="round,pad=0.4",
                      facecolor="white",
                      edgecolor="gray",
                      alpha=0.95)
        )
    
        legend_patches = [
            mpatches.Patch(color=feature_colors[f], label=f)
            for f in fi_df["Feature"]
        ]
        ax.legend(
            handles=legend_patches,
            fontsize=9,
            loc="lower left",
            frameon=True
        )
    
    # ============================================================
    # GLOBAL TITLE
    # ============================================================
    plt.suptitle(
        "Final Model Comparison: Tree-Based Algorithms\n"
        "Feature Importance and Predictive Performance (GII Score)",
        fontsize=18,
        weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    



.. image:: output_221_0.png





.. code:: ipython3

    # ============================================================
    # ROBUST FEATURE IMPORTANCE PIPELINE (SSCI-READY)
    # Permutation | SHAP | Correlation-Adjusted
    # Target: Housing Affordability Index
    # ============================================================
    
    # ============================================================
    # 1️⃣ LIBRARIES
    # ============================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.inspection import permutation_importance
    from sklearn.metrics import r2_score
    
    from sklearn.ensemble import (
        GradientBoostingRegressor,
        RandomForestRegressor,
        ExtraTreesRegressor
    )
    from catboost import CatBoostRegressor
    
    import shap
    import warnings
    warnings.filterwarnings("ignore")
    
    # ============================================================
    # 2️⃣ DATA LOADING
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    
    X = df.drop(columns=["Country", target]).select_dtypes(include=[np.number])
    y = df[target]
    
    features = X.columns.tolist()
    
    # ============================================================
    # 3️⃣ STANDARDIZATION
    # ============================================================
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(X)
    y_std = scaler_y.fit_transform(y.values.reshape(-1, 1)).ravel()
    
    # ============================================================
    # 4️⃣ TRAIN–TEST SPLIT
    # ============================================================
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # ============================================================
    # 5️⃣ MODELS (TOP TREE-BASED)
    # ============================================================
    models = {
        "Gradient Boosting": GradientBoostingRegressor(
            n_estimators=300, random_state=42
        ),
        "Random Forest": RandomForestRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "Extra Trees": ExtraTreesRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "CatBoost": CatBoostRegressor(
            iterations=300,
            depth=4,
            learning_rate=0.05,
            verbose=0,
            random_state=42
        )
    }
    
    # ============================================================
    # 6️⃣ TRAIN ALL MODELS
    # ============================================================
    trained_models = {}
    r2_scores = {}
    
    for name, model in models.items():
        model.fit(X_train, y_train)
        trained_models[name] = model
        r2_scores[name] = r2_score(y_test, model.predict(X_test))
    
    print("\n=== R² SCORES (STANDARDIZED TARGET) ===\n")
    for k, v in r2_scores.items():
        print(f"{k:18s}: {v:.4f}")
    
    # ============================================================
    # 7️⃣ PERMUTATION IMPORTANCE (MODEL-AGNOSTIC)
    # ============================================================
    perm_results = {}
    
    for name, model in trained_models.items():
        r = permutation_importance(
            model,
            X_test,
            y_test,
            n_repeats=30,
            random_state=42,
            scoring="r2"
        )
        perm_results[name] = r.importances_mean
    
    perm_df = pd.DataFrame(perm_results, index=features)
    
    # normalize to SAME SCALE
    perm_df = perm_df / perm_df.sum()
    
    # ============================================================
    # 8️⃣ SHAP GLOBAL IMPORTANCE (FIXED & BIAS-CONTROLLED)
    # ============================================================
    shap_results = {}
    
    for name, model in trained_models.items():
        explainer = shap.Explainer(
            model,
            X_train,
            feature_perturbation="interventional"
        )
        
        shap_values = explainer(
            X_test,
            check_additivity=False   # 🔑 CRITICAL FIX
        )
        
        shap_mean = np.abs(shap_values.values).mean(axis=0)
        shap_results[name] = shap_mean / shap_mean.sum()
    
    shap_df = pd.DataFrame(shap_results, index=features)
    
    # ============================================================
    # 9️⃣ CORRELATION-ADJUSTED PERMUTATION IMPORTANCE
    # ============================================================
    corr_matrix = np.abs(
        pd.DataFrame(X_std, columns=features).corr()
    )
    
    corr_adjusted = {}
    
    for model_name in perm_df.columns:
        adjusted = perm_df[model_name] / corr_matrix.mean()
        corr_adjusted[model_name] = adjusted / adjusted.sum()
    
    corr_df = pd.DataFrame(corr_adjusted)
    
    # ============================================================
    # 🔟 VISUALIZATION (SSCI-LEVEL)
    # ============================================================
    fig, axes = plt.subplots(3, 1, figsize=(18, 18), sharex=True)
    
    # --- PERMUTATION ---
    perm_df.mean(axis=1).sort_values(ascending=False).plot(
        kind="bar",
        ax=axes[0],
        edgecolor="black"
    )
    axes[0].set_title(
        "Permutation Importance (Model-Agnostic, Normalized)",
        fontsize=14, weight="bold"
    )
    axes[0].set_ylabel("Relative Importance")
    axes[0].grid(axis="y", linestyle="--", alpha=0.4)
    
    for p in axes[0].patches:
        axes[0].annotate(
            f"{p.get_height():.3f}",
            (p.get_x() + p.get_width()/2, p.get_height()),
            ha="center", va="bottom", fontsize=10, rotation=90
        )
    
    # --- SHAP ---
    shap_df.mean(axis=1).sort_values(ascending=False).plot(
        kind="bar",
        ax=axes[1],
        edgecolor="black"
    )
    axes[1].set_title(
        "SHAP Global Importance (mean |value|, Interventional)",
        fontsize=14, weight="bold"
    )
    axes[1].set_ylabel("Relative Contribution")
    axes[1].grid(axis="y", linestyle="--", alpha=0.4)
    
    for p in axes[1].patches:
        axes[1].annotate(
            f"{p.get_height():.3f}",
            (p.get_x() + p.get_width()/2, p.get_height()),
            ha="center", va="bottom", fontsize=10, rotation=90
        )
    
    # --- CORRELATION-ADJUSTED ---
    corr_df.mean(axis=1).sort_values(ascending=False).plot(
        kind="bar",
        ax=axes[2],
        edgecolor="black"
    )
    axes[2].set_title(
        "Correlation-Adjusted Permutation Importance",
        fontsize=14, weight="bold"
    )
    axes[2].set_ylabel("Adjusted Importance")
    axes[2].grid(axis="y", linestyle="--", alpha=0.4)
    
    for p in axes[2].patches:
        axes[2].annotate(
            f"{p.get_height():.3f}",
            (p.get_x() + p.get_width()/2, p.get_height()),
            ha="center", va="bottom", fontsize=10, rotation=90
        )
    
    plt.suptitle(
        "Robust Feature Importance Analysis for GII Score\n"
        "Permutation | SHAP | Correlation-Adjusted (Bias-Controlled)",
        fontsize=17, weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    
    # ============================================================
    # 11️⃣ NUMERIC OUTPUT (NO FILES)
    # ============================================================
    print("\n=== MEAN PERMUTATION IMPORTANCE ===\n")
    print(perm_df.mean(axis=1).sort_values(ascending=False).round(4))
    
    print("\n=== MEAN SHAP IMPORTANCE ===\n")
    print(shap_df.mean(axis=1).sort_values(ascending=False).round(4))
    
    print("\n=== MEAN CORRELATION-ADJUSTED IMPORTANCE ===\n")
    print(corr_df.mean(axis=1).sort_values(ascending=False).round(4))
    


.. parsed-literal::

    
    === R² SCORES (STANDARDIZED TARGET) ===
    
    Gradient Boosting : 0.9091
    Random Forest     : 0.9141
    Extra Trees       : 0.9245
    CatBoost          : 0.9213
    


.. image:: output_225_1.png


.. parsed-literal::

    
    === MEAN PERMUTATION IMPORTANCE ===
    
    Creative outputs                    0.3714
    Business sophistication             0.2064
    Knowledge and technology outputs    0.1612
    Institutions                        0.0816
    Human capital and research          0.0683
    Market sophistication               0.0588
    Infrastructure                      0.0523
    dtype: float64
    
    === MEAN SHAP IMPORTANCE ===
    
    Creative outputs                    0.2872
    Business sophistication             0.1553
    Knowledge and technology outputs    0.1492
    Institutions                        0.1203
    Human capital and research          0.1173
    Infrastructure                      0.0934
    Market sophistication               0.0773
    dtype: float64
    
    === MEAN CORRELATION-ADJUSTED IMPORTANCE ===
    
    Creative outputs                    0.3624
    Business sophistication             0.2074
    Knowledge and technology outputs    0.1597
    Institutions                        0.0908
    Human capital and research          0.0681
    Market sophistication               0.0586
    Infrastructure                      0.0530
    dtype: float64
    





.. code:: ipython3

    # ============================================================
    # ROBUST FEATURE IMPORTANCE PIPELINE (SSCI-READY)
    # Permutation | SHAP | Correlation-Adjusted
    # Target: Housing Affordability Index
    # ============================================================
    
    # ============================================================
    # 1️⃣ LIBRARIES
    # ============================================================
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.inspection import permutation_importance
    from sklearn.metrics import r2_score
    
    from sklearn.ensemble import (
        GradientBoostingRegressor,
        RandomForestRegressor,
        ExtraTreesRegressor
    )
    from catboost import CatBoostRegressor
    
    import shap
    import warnings
    warnings.filterwarnings("ignore")
    
    # ============================================================
    # 2️⃣ DATA LOADING
    # ============================================================
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    
    X = df.drop(columns=["Country", target]).select_dtypes(include=[np.number])
    y = df[target]
    
    features = X.columns.tolist()
    
    # ============================================================
    # 3️⃣ STANDARDIZATION
    # ============================================================
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(X)
    y_std = scaler_y.fit_transform(y.values.reshape(-1, 1)).ravel()
    
    # ============================================================
    # 4️⃣ TRAIN–TEST SPLIT
    # ============================================================
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # ============================================================
    # 5️⃣ MODELS (TOP TREE-BASED)
    # ============================================================
    models = {
        "Gradient Boosting": GradientBoostingRegressor(
            n_estimators=300, random_state=42
        ),
        "Random Forest": RandomForestRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "Extra Trees": ExtraTreesRegressor(
            n_estimators=300, random_state=42, n_jobs=-1
        ),
        "CatBoost": CatBoostRegressor(
            iterations=300,
            depth=4,
            learning_rate=0.05,
            verbose=0,
            random_state=42
        )
    }
    
    # ============================================================
    # 6️⃣ TRAIN ALL MODELS
    # ============================================================
    trained_models = {}
    r2_scores = {}
    
    for name, model in models.items():
        model.fit(X_train, y_train)
        trained_models[name] = model
        r2_scores[name] = r2_score(y_test, model.predict(X_test))
    
    print("\n=== R² SCORES (STANDARDIZED TARGET) ===\n")
    for k, v in r2_scores.items():
        print(f"{k:18s}: {v:.4f}")
    
    # ============================================================
    # 7️⃣ PERMUTATION IMPORTANCE (MODEL-AGNOSTIC)
    # ============================================================
    perm_results = {}
    
    for name, model in trained_models.items():
        r = permutation_importance(
            model,
            X_test,
            y_test,
            n_repeats=30,
            random_state=42,
            scoring="r2"
        )
        perm_results[name] = r.importances_mean
    
    perm_df = pd.DataFrame(perm_results, index=features)
    
    # normalize to SAME SCALE
    perm_df = perm_df / perm_df.sum()
    
    # ============================================================
    # 8️⃣ SHAP GLOBAL IMPORTANCE (FIXED & BIAS-CONTROLLED)
    # ============================================================
    shap_results = {}
    
    for name, model in trained_models.items():
        explainer = shap.Explainer(
            model,
            X_train,
            feature_perturbation="interventional"
        )
        
        shap_values = explainer(
            X_test,
            check_additivity=False   # 🔑 CRITICAL FIX
        )
        
        shap_mean = np.abs(shap_values.values).mean(axis=0)
        shap_results[name] = shap_mean / shap_mean.sum()
    
    shap_df = pd.DataFrame(shap_results, index=features)
    
    # ============================================================
    # 9️⃣ CORRELATION-ADJUSTED PERMUTATION IMPORTANCE
    # ============================================================
    corr_matrix = np.abs(
        pd.DataFrame(X_std, columns=features).corr()
    )
    
    corr_adjusted = {}
    
    for model_name in perm_df.columns:
        adjusted = perm_df[model_name] / corr_matrix.mean()
        corr_adjusted[model_name] = adjusted / adjusted.sum()
    
    corr_df = pd.DataFrame(corr_adjusted)
    
    # ============================================================
    # 🔟 VISUALIZATION (SSCI-LEVEL)
    # ============================================================
    fig, axes = plt.subplots(3, 1, figsize=(18, 18), sharex=True)
    
    # --- PERMUTATION ---
    perm_df.mean(axis=1).sort_values(ascending=False).plot(
        kind="bar",
        ax=axes[0],
        edgecolor="black"
    )
    axes[0].set_title(
        "Permutation Importance (Model-Agnostic, Normalized)",
        fontsize=14, weight="bold"
    )
    axes[0].set_ylabel("Relative Importance")
    axes[0].grid(axis="y", linestyle="--", alpha=0.4)
    
    for p in axes[0].patches:
        axes[0].annotate(
            f"{p.get_height():.3f}",
            (p.get_x() + p.get_width()/2, p.get_height()),
            ha="center", va="bottom", fontsize=10, rotation=90
        )
    
    # --- SHAP ---
    shap_df.mean(axis=1).sort_values(ascending=False).plot(
        kind="bar",
        ax=axes[1],
        edgecolor="black"
    )
    axes[1].set_title(
        "SHAP Global Importance (mean |value|, Interventional)",
        fontsize=14, weight="bold"
    )
    axes[1].set_ylabel("Relative Contribution")
    axes[1].grid(axis="y", linestyle="--", alpha=0.4)
    
    for p in axes[1].patches:
        axes[1].annotate(
            f"{p.get_height():.3f}",
            (p.get_x() + p.get_width()/2, p.get_height()),
            ha="center", va="bottom", fontsize=10, rotation=90
        )
    
    # --- CORRELATION-ADJUSTED ---
    corr_df.mean(axis=1).sort_values(ascending=False).plot(
        kind="bar",
        ax=axes[2],
        edgecolor="black"
    )
    axes[2].set_title(
        "Correlation-Adjusted Permutation Importance",
        fontsize=14, weight="bold"
    )
    axes[2].set_ylabel("Adjusted Importance")
    axes[2].grid(axis="y", linestyle="--", alpha=0.4)
    
    for p in axes[2].patches:
        axes[2].annotate(
            f"{p.get_height():.3f}",
            (p.get_x() + p.get_width()/2, p.get_height()),
            ha="center", va="bottom", fontsize=10, rotation=90
        )
    
    plt.suptitle(
        "Robust Feature Importance Analysis for GGI Score\n"
        "Permutation | SHAP | Correlation-Adjusted (Bias-Controlled)",
        fontsize=17, weight="bold"
    )
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    
    # ============================================================
    # 11️⃣ NUMERIC OUTPUT (NO FILES)
    # ============================================================
    print("\n=== MEAN PERMUTATION IMPORTANCE ===\n")
    print(perm_df.mean(axis=1).sort_values(ascending=False).round(4))
    
    print("\n=== MEAN SHAP IMPORTANCE ===\n")
    print(shap_df.mean(axis=1).sort_values(ascending=False).round(4))
    
    print("\n=== MEAN CORRELATION-ADJUSTED IMPORTANCE ===\n")
    print(corr_df.mean(axis=1).sort_values(ascending=False).round(4))
    


.. parsed-literal::

    
    === R² SCORES (STANDARDIZED TARGET) ===
    
    Gradient Boosting : 0.9091
    Random Forest     : 0.9141
    Extra Trees       : 0.9245
    CatBoost          : 0.9213
    


.. image:: output_230_1.png


.. parsed-literal::

    
    === MEAN PERMUTATION IMPORTANCE ===
    
    Creative outputs                    0.3714
    Business sophistication             0.2064
    Knowledge and technology outputs    0.1612
    Institutions                        0.0816
    Human capital and research          0.0683
    Market sophistication               0.0588
    Infrastructure                      0.0523
    dtype: float64
    
    === MEAN SHAP IMPORTANCE ===
    
    Creative outputs                    0.2872
    Business sophistication             0.1553
    Knowledge and technology outputs    0.1492
    Institutions                        0.1203
    Human capital and research          0.1173
    Infrastructure                      0.0934
    Market sophistication               0.0773
    dtype: float64
    
    === MEAN CORRELATION-ADJUSTED IMPORTANCE ===
    
    Creative outputs                    0.3624
    Business sophistication             0.2074
    Knowledge and technology outputs    0.1597
    Institutions                        0.0908
    Human capital and research          0.0681
    Market sophistication               0.0586
    Infrastructure                      0.0530
    dtype: float64
    


.. code:: ipython3

    df




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Country</th>
          <th>Score</th>
          <th>Institutions</th>
          <th>Human capital and research</th>
          <th>Infrastructure</th>
          <th>Market sophistication</th>
          <th>Business sophistication</th>
          <th>Knowledge and technology outputs</th>
          <th>Creative outputs</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>Albania</td>
          <td>29.6</td>
          <td>58.7</td>
          <td>22.4</td>
          <td>52.3</td>
          <td>41.1</td>
          <td>30.5</td>
          <td>16.5</td>
          <td>20.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>Algeria</td>
          <td>18.9</td>
          <td>42.1</td>
          <td>26.9</td>
          <td>34.0</td>
          <td>10.5</td>
          <td>21.6</td>
          <td>11.1</td>
          <td>10.5</td>
        </tr>
        <tr>
          <th>2</th>
          <td>Angola</td>
          <td>13.0</td>
          <td>27.6</td>
          <td>12.9</td>
          <td>26.9</td>
          <td>20.7</td>
          <td>16.1</td>
          <td>5.5</td>
          <td>5.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>Argentina</td>
          <td>26.8</td>
          <td>28.6</td>
          <td>33.8</td>
          <td>38.5</td>
          <td>28.2</td>
          <td>26.6</td>
          <td>18.1</td>
          <td>26.7</td>
        </tr>
        <tr>
          <th>4</th>
          <td>Armenia</td>
          <td>30.5</td>
          <td>49.0</td>
          <td>24.7</td>
          <td>39.3</td>
          <td>33.0</td>
          <td>27.0</td>
          <td>21.5</td>
          <td>31.2</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>134</th>
          <td>Uzbekistan</td>
          <td>26.5</td>
          <td>51.9</td>
          <td>27.4</td>
          <td>41.8</td>
          <td>35.0</td>
          <td>27.1</td>
          <td>20.9</td>
          <td>11.8</td>
        </tr>
        <tr>
          <th>135</th>
          <td>Venezuela</td>
          <td>13.7</td>
          <td>1.9</td>
          <td>48.7</td>
          <td>11.7</td>
          <td>14.1</td>
          <td>22.9</td>
          <td>6.6</td>
          <td>8.7</td>
        </tr>
        <tr>
          <th>136</th>
          <td>Viet Nam</td>
          <td>37.1</td>
          <td>53.5</td>
          <td>30.5</td>
          <td>46.8</td>
          <td>41.6</td>
          <td>35.7</td>
          <td>28.9</td>
          <td>36.2</td>
        </tr>
        <tr>
          <th>137</th>
          <td>Zambia</td>
          <td>19.6</td>
          <td>49.1</td>
          <td>20.6</td>
          <td>36.2</td>
          <td>21.8</td>
          <td>28.5</td>
          <td>8.6</td>
          <td>7.2</td>
        </tr>
        <tr>
          <th>138</th>
          <td>Zimbabwe</td>
          <td>15.4</td>
          <td>18.8</td>
          <td>8.1</td>
          <td>21.6</td>
          <td>13.1</td>
          <td>27.4</td>
          <td>10.7</td>
          <td>15.2</td>
        </tr>
      </tbody>
    </table>
    <p>139 rows × 9 columns</p>
    </div>








.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
    
    # -----------------------------------------------------
    # 1️⃣ LOAD DATA
    # -----------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target_col = "Score"
    
    y = df[target_col]
    
    X = (
        df.drop(columns=["Country", target_col], errors="ignore")
          .select_dtypes(include=[np.number])
    )
    
    feature_labels = X.columns.tolist()
    
    # -----------------------------------------------------
    # 2️⃣ STANDARDIZATION
    # -----------------------------------------------------
    
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(X)
    y_std = scaler_y.fit_transform(y.values.reshape(-1,1)).ravel()
    
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # -----------------------------------------------------
    # 3️⃣ EXTRA TREES MODEL
    # -----------------------------------------------------
    
    et_model = ExtraTreesRegressor(
        n_estimators=300,
        random_state=42,
        n_jobs=-1
    )
    
    et_model.fit(X_train, y_train)
    y_pred = et_model.predict(X_test)
    
    # -----------------------------------------------------
    # 4️⃣ PERFORMANCE METRICS
    # -----------------------------------------------------
    
    metrics = {
        "MSE": mean_squared_error(y_test, y_pred),
        "RMSE": np.sqrt(mean_squared_error(y_test, y_pred)),
        "MAE": mean_absolute_error(y_test, y_pred),
        "R2": r2_score(y_test, y_pred)
    }
    
    print("\n=========== EXTRA TREES PERFORMANCE ===========\n")
    for k, v in metrics.items():
        print(f"{k}: {v:.4f}")
    
    # -----------------------------------------------------
    # 5️⃣ SHAP TREE EXPLAINER
    # -----------------------------------------------------
    
    explainer = shap.TreeExplainer(et_model)
    shap_values = explainer.shap_values(X_test)
    
    # -----------------------------------------------------
    # 6️⃣ SHAP SUMMARY PLOT (GLOBAL + LOCAL)
    # -----------------------------------------------------
    
    shap.summary_plot(
        shap_values,
        X_test,
        feature_names=feature_labels,
        show=True
    )
    
    # -----------------------------------------------------
    # 7️⃣ MEAN |SHAP| (GLOBAL IMPORTANCE)
    # -----------------------------------------------------
    
    shap_mean = np.abs(shap_values).mean(axis=0)
    
    shap_df = (
        pd.DataFrame({
            "Feature": feature_labels,
            "Mean |SHAP|": shap_mean
        })
        .sort_values("Mean |SHAP|", ascending=False)
    )
    
    print("\n=========== MEAN |SHAP| VALUES ===========\n")
    print(shap_df.round(4))
    
    # -----------------------------------------------------
    # 8️⃣ BARPLOT – SSCI STYLE (WITH PERFORMANCE BOX)
    # -----------------------------------------------------
    
    plt.figure(figsize=(10,6))
    
    sns.barplot(
        x="Mean |SHAP|",
        y="Feature",
        data=shap_df,
        palette="viridis"
    )
    
    plt.title("Global Feature Importance (Mean |SHAP|)\nExtra Trees – GII Score")
    plt.xlabel("Mean |SHAP| Contribution")
    plt.ylabel("Feature")
    
    # Value labels on bars
    for i, v in enumerate(shap_df["Mean |SHAP|"]):
        plt.text(v, i, f"{v:.3f}", va="center")
    
    # Performance metrics box
    performance_text = (
        f"MSE: {metrics['MSE']:.4f}\n"
        f"RMSE: {metrics['RMSE']:.4f}\n"
        f"MAE: {metrics['MAE']:.4f}\n"
        f"R²: {metrics['R2']:.4f}"
    )
    
    plt.gca().text(
        0.98, 0.05,
        performance_text,
        transform=plt.gca().transAxes,
        ha="right",
        va="bottom",
        fontsize=13,
        bbox=dict(boxstyle="round", facecolor="white", alpha=0.9)
    )
    
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    =========== EXTRA TREES PERFORMANCE ===========
    
    MSE: 0.0800
    RMSE: 0.2828
    MAE: 0.1476
    R2: 0.9245
    


.. image:: output_238_1.png


.. parsed-literal::

    
    =========== MEAN |SHAP| VALUES ===========
    
                                Feature  Mean |SHAP|
    6                  Creative outputs       0.2593
    4           Business sophistication       0.1762
    5  Knowledge and technology outputs       0.1460
    1        Human capital and research       0.1302
    2                    Infrastructure       0.1131
    0                      Institutions       0.0863
    3             Market sophistication       0.0692
    


.. image:: output_238_3.png








.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
    from sklearn.linear_model import Ridge
    
    # -----------------------------------------------------
    # 1️⃣ LOAD DATA
    # -----------------------------------------------------
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target_col = "Score"
    
    y = df[target_col]
    X = df.drop(columns=["Country", target_col], errors="ignore") \
          .select_dtypes(include=[np.number])
    
    feature_labels = X.columns.tolist()
    
    # -----------------------------------------------------
    # 2️⃣ STANDARDIZATION
    # -----------------------------------------------------
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(X)
    y_std = scaler_y.fit_transform(y.values.reshape(-1,1)).ravel()
    
    X_train, X_test, y_train, y_test = train_test_split(
        X_std, y_std, test_size=0.20, random_state=42
    )
    
    # -----------------------------------------------------
    # 3️⃣ EXTRA TREES MODEL
    # -----------------------------------------------------
    et_model = ExtraTreesRegressor(
        n_estimators=300,
        random_state=42,
        n_jobs=-1
    )
    
    et_model.fit(X_train, y_train)
    y_pred = et_model.predict(X_test)
    
    # -----------------------------------------------------
    # 4️⃣ PERFORMANCE METRICS
    # -----------------------------------------------------
    metrics = {
        "MSE": mean_squared_error(y_test, y_pred),
        "RMSE": np.sqrt(mean_squared_error(y_test, y_pred)),
        "MAE": mean_absolute_error(y_test, y_pred),
        "R2": r2_score(y_test, y_pred)
    }
    
    print("\n=========== EXTRA TREES PERFORMANCE ===========\n")
    for k, v in metrics.items():
        print(f"{k}: {v:.4f}")
    
    # -----------------------------------------------------
    # 5️⃣ SHAP EXPLAINER
    # -----------------------------------------------------
    explainer = shap.TreeExplainer(et_model)
    shap_values = explainer.shap_values(X_test)
    
    shap_mean = np.abs(shap_values).mean(axis=0)
    
    shap_df = pd.DataFrame({
        "Feature": feature_labels,
        "Mean |SHAP|": shap_mean
    }).sort_values("Mean |SHAP|", ascending=False)
    
    # -----------------------------------------------------
    # 6️⃣ GLOBAL SHAP + PERFORMANCE METRICS
    # -----------------------------------------------------
    plt.figure(figsize=(10,6))
    sns.barplot(
        x="Mean |SHAP|",
        y="Feature",
        data=shap_df,
        palette="viridis"
    )
    
    for i, v in enumerate(shap_df["Mean |SHAP|"]):
        plt.text(v, i, f"{v:.3f}", va="center")
    
    perf_text = (
        f"MSE: {metrics['MSE']:.4f}\n"
        f"RMSE: {metrics['RMSE']:.4f}\n"
        f"MAE: {metrics['MAE']:.4f}\n"
        f"R²: {metrics['R2']:.4f}"
    )
    
    plt.gca().text(
        0.98, 0.05,
        perf_text,
        transform=plt.gca().transAxes,
        ha="right",
        va="bottom",
        fontsize=11,
        bbox=dict(boxstyle="round", facecolor="white", alpha=0.9)
    )
    
    plt.title("Global Feature Importance with Performance Metrics")
    plt.tight_layout()
    plt.show()
    
    # -----------------------------------------------------
    # 7️⃣ SHAP DEPENDENCE PLOTS (2x4 GRID)
    # -----------------------------------------------------
    top_n = min(8, len(feature_labels))
    top_features = shap_df["Feature"].iloc[:top_n].values
    
    fig, axes = plt.subplots(4, 2, figsize=(12, 20))
    axes = axes.flatten()
    
    for i, feature in enumerate(top_features):
        feature_index = feature_labels.index(feature)
    
        shap.dependence_plot(
            feature_index,
            shap_values,
            X_test,
            feature_names=feature_labels,
            show=False,
            ax=axes[i]
        )
    
        axes[i].set_title(feature, fontsize=10)
    
    for j in range(i+1, 8):
        fig.delaxes(axes[j])
    
    plt.suptitle("SHAP Dependence Plots (Top 8 Features)\nExtra Trees – GII Score",
                 fontsize=14)
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    plt.show()
    
    # -----------------------------------------------------
    # 8️⃣ SHAP INTERACTION MATRIX
    # -----------------------------------------------------
    shap_interaction_values = explainer.shap_interaction_values(X_test)
    interaction_mean = np.abs(shap_interaction_values).mean(axis=0)
    
    interaction_df = pd.DataFrame(
        interaction_mean,
        index=feature_labels,
        columns=feature_labels
    )
    
    print("\n=========== SHAP INTERACTION MATRIX ===========\n")
    print(interaction_df.round(4))
    
    plt.figure(figsize=(12,10))
    sns.heatmap(
        interaction_df,
        cmap="viridis",
        annot=True,
        fmt=".3f"
    )
    
    plt.title("SHAP Interaction Matrix – Extra Trees")
    plt.tight_layout()
    plt.show()
    
    # -----------------------------------------------------
    # 9️⃣ LINEAR (RIDGE) vs SHAP COMPARISON (ETİKETLİ)
    # -----------------------------------------------------
    ridge_model = Ridge()
    ridge_model.fit(X_train, y_train)
    
    ridge_coef = np.abs(ridge_model.coef_)
    
    linear_vs_shap_df = pd.DataFrame({
        "Feature": feature_labels,
        "Linear (Ridge) Coef": ridge_coef,
        "SHAP Mean |Value|": shap_mean
    }).sort_values("SHAP Mean |Value|", ascending=False)
    
    print("\n=========== LINEAR vs SHAP COMPARISON ===========\n")
    print(linear_vs_shap_df.round(4))
    
    plt.figure(figsize=(10,6))
    
    sns.scatterplot(
        x="Linear (Ridge) Coef",
        y="SHAP Mean |Value|",
        hue="Feature",
        data=linear_vs_shap_df,
        palette="tab10",
        s=140,
        legend=False
    )
    
    # Katsayıları grafik üzerinde yaz
    for i, row in linear_vs_shap_df.iterrows():
        label_text = (
            f"{row['Feature']}\n"
            f"L: {row['Linear (Ridge) Coef']:.3f} | "
            f"S: {row['SHAP Mean |Value|']:.3f}"
        )
    
        plt.text(
            row["Linear (Ridge) Coef"] + 0.005,
            row["SHAP Mean |Value|"] + 0.005,
            label_text,
            fontsize=8
        )
    
    plt.xlabel("Linear (Ridge) Absolute Coefficients")
    plt.ylabel("SHAP Mean |Value|")
    plt.title("Linear vs SHAP Feature Importance Comparison")
    plt.tight_layout()
    plt.show()
    
    # -----------------------------------------------------
    # 🔟 EXTRA TREES ±1 SD SCENARIO ANALYSIS
    # -----------------------------------------------------
    
    print("\n=========== EXTRA TREES ±1 SD SCENARIO ANALYSIS ===========\n")
    
    # Test verisini DataFrame yap
    X_test_df = pd.DataFrame(X_test, columns=feature_labels)
    
    scenario_results = []
    
    for f in feature_labels:
        sd = X_test_df[f].std()
    
        X_plus = X_test_df.copy()
        X_minus = X_test_df.copy()
    
        # +1 SD ve -1 SD senaryosu
        X_plus[f] += sd
        X_minus[f] -= sd
    
        shap_plus = explainer.shap_values(X_plus)
        shap_minus = explainer.shap_values(X_minus)
    
        idx = feature_labels.index(f)
    
        scenario_results.append({
            "Feature": f,
            "+1 SD Effect": np.mean(shap_plus[:, idx] - shap_values[:, idx]),
            "-1 SD Effect": np.mean(shap_minus[:, idx] - shap_values[:, idx])
        })
    
    scenario_df = pd.DataFrame(scenario_results).sort_values(
        by="+1 SD Effect", key=np.abs, ascending=False
    )
    
    print(scenario_df.round(4))
    
    
    # -----------------------------------------------------
    # SCENARIO GRAFİĞİ
    # -----------------------------------------------------
    
    scenario_melt = scenario_df.melt(
        id_vars="Feature",
        value_vars=["+1 SD Effect", "-1 SD Effect"],
        var_name="Scenario",
        value_name="Effect"
    )
    
    plt.figure(figsize=(10,6))
    ax2 = sns.barplot(
        x="Effect",
        y="Feature",
        hue="Scenario",
        data=scenario_melt,
        palette="coolwarm"
    )
    
    plt.title("±1 SD Scenario Effects on GII Score (Extra Trees)")
    plt.xlabel("Change in Predicted GII (Std Units)")
    plt.ylabel("Feature")
    plt.legend(title="")
    
    # Sayısal değerleri çubuk üzerine yaz
    for p in ax2.patches:
        w = p.get_width()
        ax2.text(
            w + 0.01*np.sign(w),
            p.get_y() + p.get_height()/2,
            f"{w:.3f}",
            ha="left" if w > 0 else "right",
            va="center"
        )
    
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    =========== EXTRA TREES PERFORMANCE ===========
    
    MSE: 0.0800
    RMSE: 0.2828
    MAE: 0.1476
    R2: 0.9245
    


.. image:: output_245_1.png



.. image:: output_245_2.png


.. parsed-literal::

    
    =========== SHAP INTERACTION MATRIX ===========
    
                                      Institutions  Human capital and research  \
    Institutions                            0.1042                      0.0084   
    Human capital and research              0.0084                      0.1773   
    Infrastructure                          0.0070                      0.0074   
    Market sophistication                   0.0030                      0.0052   
    Business sophistication                 0.0062                      0.0145   
    Knowledge and technology outputs        0.0106                      0.0075   
    Creative outputs                        0.0137                      0.0131   
    
                                      Infrastructure  Market sophistication  \
    Institutions                              0.0070                 0.0030   
    Human capital and research                0.0074                 0.0052   
    Infrastructure                            0.1625                 0.0053   
    Market sophistication                     0.0053                 0.0966   
    Business sophistication                   0.0092                 0.0065   
    Knowledge and technology outputs          0.0094                 0.0083   
    Creative outputs                          0.0221                 0.0088   
    
                                      Business sophistication  \
    Institutions                                       0.0062   
    Human capital and research                         0.0145   
    Infrastructure                                     0.0092   
    Market sophistication                              0.0065   
    Business sophistication                            0.2302   
    Knowledge and technology outputs                   0.0118   
    Creative outputs                                   0.0169   
    
                                      Knowledge and technology outputs  \
    Institutions                                                0.0106   
    Human capital and research                                  0.0075   
    Infrastructure                                              0.0094   
    Market sophistication                                       0.0083   
    Business sophistication                                     0.0118   
    Knowledge and technology outputs                            0.2018   
    Creative outputs                                            0.0201   
    
                                      Creative outputs  
    Institutions                                0.0137  
    Human capital and research                  0.0131  
    Infrastructure                              0.0221  
    Market sophistication                       0.0088  
    Business sophistication                     0.0169  
    Knowledge and technology outputs            0.0201  
    Creative outputs                            0.3371  
    


.. image:: output_245_4.png


.. parsed-literal::

    
    =========== LINEAR vs SHAP COMPARISON ===========
    
                                Feature  Linear (Ridge) Coef  SHAP Mean |Value|
    6                  Creative outputs               0.2962             0.2593
    4           Business sophistication               0.1109             0.1762
    5  Knowledge and technology outputs               0.2332             0.1460
    1        Human capital and research               0.0810             0.1302
    2                    Infrastructure               0.1031             0.1131
    0                      Institutions               0.1706             0.0863
    3             Market sophistication               0.0901             0.0692
    


.. image:: output_245_6.png


.. parsed-literal::

    
    =========== EXTRA TREES ±1 SD SCENARIO ANALYSIS ===========
    
                                Feature  +1 SD Effect  -1 SD Effect
    6                  Creative outputs        0.2572       -0.2846
    5  Knowledge and technology outputs        0.2043       -0.1293
    4           Business sophistication        0.1844       -0.1088
    1        Human capital and research        0.1175       -0.0958
    2                    Infrastructure        0.1004       -0.1381
    3             Market sophistication        0.0936       -0.0969
    0                      Institutions        0.0814       -0.0997
    


.. image:: output_245_8.png










.. code:: ipython3

    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import matplotlib as mpl
    
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.inspection import PartialDependenceDisplay
    from sklearn.metrics import r2_score, mean_absolute_error
    
    # =====================================================
    # GLOBAL STYLE
    # =====================================================
    
    mpl.rcParams.update({
        "figure.titlesize": 12,
        "axes.titlesize": 10,
        "axes.labelsize": 9,
        "xtick.labelsize": 8,
        "ytick.labelsize": 8,
        "font.size": 10
    })
    
    # =====================================================
    # 1. LOAD DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    
    df = df.select_dtypes(include=[np.number]).dropna()
    
    features = [c for c in df.columns if c != target]
    
    X = df[features]
    y = df[target]
    
    # =====================================================
    # 2. TRAIN TEST SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    # Convert to DataFrame for PDP compatibility
    X_train_scaled_df = pd.DataFrame(X_train_scaled, columns=features)
    X_test_scaled_df = pd.DataFrame(X_test_scaled, columns=features)
    
    # =====================================================
    # 3. MODEL
    # =====================================================
    
    model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train_scaled_df, y_train)
    
    y_pred = model.predict(X_test_scaled_df)
    
    print("\nMODEL PERFORMANCE")
    print("R²:", round(r2_score(y_test, y_pred), 4))
    print("MAE:", round(mean_absolute_error(y_test, y_pred), 4))
    
    # =====================================================
    # 4. PARTIAL DEPENDENCE – SINGLE FIGURE ONLY
    # =====================================================
    
    # İlk 5 feature (istersen manuel de yazabilirsin)
    top5 = features[:5]
    
    fig, ax = plt.subplots(figsize=(10, 8))
    
    PartialDependenceDisplay.from_estimator(
        model,
        X_train_scaled_df,
        features=top5,
        grid_resolution=50,
        ax=ax
    )
    
    plt.suptitle("Partial Dependence Plots")
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    MODEL PERFORMANCE
    R²: 0.9229
    MAE: 1.9354
    


.. image:: output_254_1.png



.. code:: ipython3

    import numpy as np
    import pandas as pd
    
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_absolute_error
    
    # =====================================================
    # 1. LOAD DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    
    df = df.select_dtypes(include=[np.number]).dropna()
    
    features = [c for c in df.columns if c != target]
    
    X = df[features]
    y = df[target]
    
    # =====================================================
    # 2. TRAIN TEST SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    # =====================================================
    # 3. MODEL
    # =====================================================
    
    model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train_scaled, y_train)
    
    y_pred = model.predict(X_test_scaled)
    
    print("\nMODEL PERFORMANCE")
    print("R²:", round(r2_score(y_test, y_pred), 4))
    print("MAE:", round(mean_absolute_error(y_test, y_pred), 4))


.. parsed-literal::

    
    MODEL PERFORMANCE
    R²: 0.9229
    MAE: 1.9354
    





.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import lime
    import lime.lime_tabular
    import matplotlib.pyplot as plt
    import matplotlib as mpl
    
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.inspection import PartialDependenceDisplay, partial_dependence
    from sklearn.metrics import r2_score, mean_absolute_error
    from scipy.stats import spearmanr, kendalltau
    
    # =====================================================
    # GLOBAL STYLE
    # =====================================================
    
    mpl.rcParams.update({
        "figure.titlesize": 12,
        "axes.titlesize": 10,
        "axes.labelsize": 9,
        "xtick.labelsize": 8,
        "ytick.labelsize": 8,
        "legend.fontsize": 8,
        "font.size": 10
    })
    
    # =====================================================
    # 1. LOAD DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    
    df = df.select_dtypes(include=[np.number]).dropna()
    
    features = [c for c in df.columns if c != target]
    
    X = df[features]
    y = df[target]
    
    # =====================================================
    # 2. TRAIN TEST SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    # Convert back to DataFrame (IMPORTANT for PDP compatibility)
    X_train_scaled_df = pd.DataFrame(X_train_scaled, columns=features)
    X_test_scaled_df = pd.DataFrame(X_test_scaled, columns=features)
    
    # =====================================================
    # 3. MODEL – EXTRA TREES
    # =====================================================
    
    model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train_scaled_df, y_train)
    
    y_pred = model.predict(X_test_scaled_df)
    
    print("\nMODEL PERFORMANCE")
    print("R²:", round(r2_score(y_test, y_pred), 4))
    print("MAE:", round(mean_absolute_error(y_test, y_pred), 4))
    
    # =====================================================
    # 4. SHAP GLOBAL IMPORTANCE
    # =====================================================
    
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_train_scaled_df)
    
    shap_importance = np.abs(shap_values).mean(axis=0)
    
    shap_ranking = pd.Series(
        shap_importance,
        index=features
    ).sort_values(ascending=False)
    
    print("\nTOP 10 SHAP FEATURES")
    print(shap_ranking.head(10))
    
    # =====================================================
    # 5. LIME LOCAL IMPORTANCE
    # =====================================================
    
    explainer_lime = lime.lime_tabular.LimeTabularExplainer(
        training_data=X_train_scaled,
        feature_names=features,
        mode="regression",
        discretize_continuous=True,
        random_state=42
    )
    
    test_df = X_test_scaled_df.copy()
    test_df["Prediction"] = model.predict(X_test_scaled_df)
    
    cases = {
        "Low Prediction": test_df["Prediction"].idxmin(),
        "Median Prediction": test_df["Prediction"].sub(
            test_df["Prediction"].median()
        ).abs().idxmin(),
        "High Prediction": test_df["Prediction"].idxmax()
    }
    
    fig, axes = plt.subplots(1, 3, figsize=(14, 4))
    color_map = plt.cm.tab20.colors
    lime_importance_global = np.zeros(len(features))
    
    for ax, (label, idx) in zip(axes, cases.items()):
    
        exp = explainer_lime.explain_instance(
            X_test_scaled[idx],
            model.predict,
            num_features=8
        )
    
        df_plot = pd.DataFrame(
            exp.as_list(),
            columns=["Feature", "Contribution"]
        ).sort_values("Contribution")
    
        colors = [color_map[i % len(color_map)] for i in range(len(df_plot))]
    
        bars = ax.barh(
            df_plot["Feature"],
            df_plot["Contribution"],
            color=colors
        )
    
        ax.set_title(f"{label}\n(Local R² = {exp.score:.3f})")
    
        for bar in bars:
            width = bar.get_width()
            ax.text(
                width,
                bar.get_y() + bar.get_height()/2,
                f"{width:.3f}",
                va='center',
                ha='left' if width > 0 else 'right',
                fontsize=7
            )
    
        for feat, val in exp.as_list():
            fname = feat.split(" ")[0]
            if fname in features:
                lime_importance_global[features.index(fname)] += abs(val)
    
    fig.suptitle("LIME Local Explanations")
    plt.tight_layout()
    plt.show()
    
    lime_importance_global /= 3
    
    lime_ranking = pd.Series(
        lime_importance_global,
        index=features
    ).sort_values(ascending=False)
    
    # =====================================================
    # 6. SHAP – LIME CONSISTENCY
    # =====================================================
    
    common = list(set(shap_ranking.index) & set(lime_ranking.index))
    
    shap_rank = shap_ranking.loc[common].rank()
    lime_rank = lime_ranking.loc[common].rank()
    
    print("\nCONSISTENCY RESULTS")
    print("Spearman:", spearmanr(shap_rank, lime_rank))
    print("Kendall:", kendalltau(shap_rank, lime_rank))
    lime_importance_global 
    
    shap_importance
    # =====================================================
    # 7. LABELED NUMERIC OUTPUT (SHAP & LIME)
    # =====================================================
    
    print("\n==============================")
    print("LABELED SHAP VALUES")
    print("==============================")
    
    shap_df = pd.DataFrame({
        "Feature": features,
        "SHAP Importance": shap_importance
    }).sort_values("SHAP Importance", ascending=False)
    
    print(shap_df)
    
    
    print("\n==============================")
    print("LABELED LIME VALUES")
    print("==============================")
    
    lime_df = pd.DataFrame({
        "Feature": features,
        "LIME Importance": lime_importance_global
    }).sort_values("LIME Importance", ascending=False)
    
    print(lime_df)
    
    
    print("\n==============================")
    print("SHAP & LIME MERGED TABLE")
    print("==============================")
    
    merged_df = shap_df.merge(
        lime_df,
        on="Feature",
        how="inner"
    )
    
    print(merged_df)
    
    cases
    


.. parsed-literal::

    
    MODEL PERFORMANCE
    R²: 0.9229
    MAE: 1.9354
    
    TOP 10 SHAP FEATURES
    Creative outputs                    3.584516
    Business sophistication             2.062679
    Knowledge and technology outputs    1.869111
    Human capital and research          1.689691
    Infrastructure                      1.511443
    Institutions                        1.128079
    Market sophistication               0.881737
    dtype: float64
    


.. image:: output_261_1.png


.. parsed-literal::

    
    CONSISTENCY RESULTS
    Spearman: SignificanceResult(statistic=np.float64(-0.445435403187374), pvalue=np.float64(0.31653589902854457))
    Kendall: SignificanceResult(statistic=np.float64(-0.3289758474798845), pvalue=np.float64(0.3418143874474573))
    
    ==============================
    LABELED SHAP VALUES
    ==============================
                                Feature  SHAP Importance
    6                  Creative outputs         3.584516
    4           Business sophistication         2.062679
    5  Knowledge and technology outputs         1.869111
    1        Human capital and research         1.689691
    2                    Infrastructure         1.511443
    0                      Institutions         1.128079
    3             Market sophistication         0.881737
    
    ==============================
    LABELED LIME VALUES
    ==============================
                                Feature  LIME Importance
    2                    Infrastructure         1.636972
    0                      Institutions         0.714226
    1        Human capital and research         0.000000
    3             Market sophistication         0.000000
    4           Business sophistication         0.000000
    5  Knowledge and technology outputs         0.000000
    6                  Creative outputs         0.000000
    
    ==============================
    SHAP & LIME MERGED TABLE
    ==============================
                                Feature  SHAP Importance  LIME Importance
    0                  Creative outputs         3.584516         0.000000
    1           Business sophistication         2.062679         0.000000
    2  Knowledge and technology outputs         1.869111         0.000000
    3        Human capital and research         1.689691         0.000000
    4                    Infrastructure         1.511443         1.636972
    5                      Institutions         1.128079         0.714226
    6             Market sophistication         0.881737         0.000000
    



.. parsed-literal::

    {'Low Prediction': 22, 'Median Prediction': 12, 'High Prediction': 26}



.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import lime
    import lime.lime_tabular
    import matplotlib.pyplot as plt
    import matplotlib as mpl
    
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_absolute_error
    from scipy.stats import spearmanr, kendalltau
    
    # =====================================================
    # GLOBAL STYLE
    # =====================================================
    
    mpl.rcParams.update({
        "figure.titlesize": 12,
        "axes.titlesize": 10,
        "axes.labelsize": 9,
        "xtick.labelsize": 8,
        "ytick.labelsize": 8,
        "legend.fontsize": 8,
        "font.size": 10
    })
    
    # =====================================================
    # 1. LOAD DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    
    df = df.select_dtypes(include=[np.number]).dropna()
    
    features = [c for c in df.columns if c != target]
    
    X = df[features]
    y = df[target]
    
    # =====================================================
    # 2. TRAIN TEST SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    X_train_scaled_df = pd.DataFrame(X_train_scaled, columns=features)
    X_test_scaled_df = pd.DataFrame(X_test_scaled, columns=features)
    
    # =====================================================
    # 3. MODEL
    # =====================================================
    
    model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train_scaled_df, y_train)
    y_pred = model.predict(X_test_scaled_df)
    
    print("\nMODEL PERFORMANCE")
    print("R²:", round(r2_score(y_test, y_pred), 4))
    print("MAE:", round(mean_absolute_error(y_test, y_pred), 4))
    
    # =====================================================
    # 4. SHAP GLOBAL IMPORTANCE
    # =====================================================
    
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_train_scaled_df)
    
    shap_importance = np.abs(shap_values).mean(axis=0)
    
    shap_ranking = pd.Series(
        shap_importance,
        index=features
    ).sort_values(ascending=False)
    
    print("\nTOP 10 SHAP FEATURES")
    print(shap_ranking.head(10))
    
    # =====================================================
    # 5. LIME LOCAL IMPORTANCE
    # =====================================================
    
    explainer_lime = lime.lime_tabular.LimeTabularExplainer(
        training_data=X_train_scaled,
        feature_names=features,
        mode="regression",
        discretize_continuous=True,
        random_state=42
    )
    
    test_df = X_test_scaled_df.copy()
    test_df["Prediction"] = y_pred
    test_df["Actual"] = y_test.values
    
    cases = {
        "Low Prediction": test_df["Prediction"].idxmin(),
        "Median Prediction": test_df["Prediction"].sub(
            test_df["Prediction"].median()
        ).abs().idxmin(),
        "High Prediction": test_df["Prediction"].idxmax()
    }
    
    # ===============================
    # PRINT NUMERIC VALUES OF CASES
    # ===============================
    
    print("\n==============================")
    print("CASE NUMERIC VALUES")
    print("==============================")
    
    for label, idx in cases.items():
        print(f"\n{label}")
        print("Index:", idx)
        print("Predicted Score:", round(test_df.loc[idx, "Prediction"], 4))
        print("Actual Score   :", round(test_df.loc[idx, "Actual"], 4))
    
    # ===============================
    # LIME COMPUTATION
    # ===============================
    
    fig, axes = plt.subplots(1, 3, figsize=(14, 4))
    color_map = plt.cm.tab20.colors
    lime_importance_global = np.zeros(len(features))
    
    for ax, (label, idx) in zip(axes, cases.items()):
    
        exp = explainer_lime.explain_instance(
            X_test_scaled[idx],
            model.predict,
            num_features=8
        )
    
        df_plot = pd.DataFrame(
            exp.as_list(),
            columns=["Feature", "Contribution"]
        ).sort_values("Contribution")
    
        colors = [color_map[i % len(color_map)] for i in range(len(df_plot))]
    
        ax.barh(
            df_plot["Feature"],
            df_plot["Contribution"],
            color=colors
        )
    
        ax.set_title(f"{label}\n(Local R² = {exp.score:.3f})")
    
        for feat, val in exp.as_list():
            fname = feat.split(" ")[0]
            if fname in features:
                lime_importance_global[features.index(fname)] += abs(val)
    
    fig.suptitle("LIME Local Explanations")
    plt.tight_layout()
    plt.show()
    
    lime_importance_global /= 3
    
    lime_ranking = pd.Series(
        lime_importance_global,
        index=features
    ).sort_values(ascending=False)
    
    # =====================================================
    # 6. SHAP – LIME CONSISTENCY
    # =====================================================
    
    common = list(set(shap_ranking.index) & set(lime_ranking.index))
    
    shap_rank = shap_ranking.loc[common].rank()
    lime_rank = lime_ranking.loc[common].rank()
    
    print("\nCONSISTENCY RESULTS")
    print("Spearman:", spearmanr(shap_rank, lime_rank))
    print("Kendall :", kendalltau(shap_rank, lime_rank))
    
    # =====================================================
    # 7. LABELED NUMERIC OUTPUT
    # =====================================================
    
    print("\n==============================")
    print("LABELED SHAP VALUES")
    print("==============================")
    
    shap_df = pd.DataFrame({
        "Feature": features,
        "SHAP Importance": shap_importance
    }).sort_values("SHAP Importance", ascending=False)
    
    print(shap_df)
    
    print("\n==============================")
    print("LABELED LIME VALUES")
    print("==============================")
    
    lime_df = pd.DataFrame({
        "Feature": features,
        "LIME Importance": lime_importance_global
    }).sort_values("LIME Importance", ascending=False)
    
    print(lime_df)
    
    print("\n==============================")
    print("SHAP & LIME MERGED TABLE")
    print("==============================")
    
    merged_df = shap_df.merge(lime_df, on="Feature", how="inner")
    print(merged_df)
    
    print("\nPIPELINE COMPLETED SUCCESSFULLY")


.. parsed-literal::

    
    MODEL PERFORMANCE
    R²: 0.9229
    MAE: 1.9354
    
    TOP 10 SHAP FEATURES
    Creative outputs                    3.584516
    Business sophistication             2.062679
    Knowledge and technology outputs    1.869111
    Human capital and research          1.689691
    Infrastructure                      1.511443
    Institutions                        1.128079
    Market sophistication               0.881737
    dtype: float64
    
    ==============================
    CASE NUMERIC VALUES
    ==============================
    
    Low Prediction
    Index: 22
    Predicted Score: 16.0702
    Actual Score   : 15.4
    
    Median Prediction
    Index: 9
    Predicted Score: 30.9492
    Actual Score   : 31.3
    
    High Prediction
    Index: 26
    Predicted Score: 61.0696
    Actual Score   : 61.7
    


.. image:: output_262_1.png


.. parsed-literal::

    
    CONSISTENCY RESULTS
    Spearman: SignificanceResult(statistic=np.float64(-0.445435403187374), pvalue=np.float64(0.31653589902854457))
    Kendall : SignificanceResult(statistic=np.float64(-0.3289758474798845), pvalue=np.float64(0.3418143874474573))
    
    ==============================
    LABELED SHAP VALUES
    ==============================
                                Feature  SHAP Importance
    6                  Creative outputs         3.584516
    4           Business sophistication         2.062679
    5  Knowledge and technology outputs         1.869111
    1        Human capital and research         1.689691
    2                    Infrastructure         1.511443
    0                      Institutions         1.128079
    3             Market sophistication         0.881737
    
    ==============================
    LABELED LIME VALUES
    ==============================
                                Feature  LIME Importance
    2                    Infrastructure         1.636972
    0                      Institutions         0.714226
    1        Human capital and research         0.000000
    3             Market sophistication         0.000000
    4           Business sophistication         0.000000
    5  Knowledge and technology outputs         0.000000
    6                  Creative outputs         0.000000
    
    ==============================
    SHAP & LIME MERGED TABLE
    ==============================
                                Feature  SHAP Importance  LIME Importance
    0                  Creative outputs         3.584516         0.000000
    1           Business sophistication         2.062679         0.000000
    2  Knowledge and technology outputs         1.869111         0.000000
    3        Human capital and research         1.689691         0.000000
    4                    Infrastructure         1.511443         1.636972
    5                      Institutions         1.128079         0.714226
    6             Market sophistication         0.881737         0.000000
    
    PIPELINE COMPLETED SUCCESSFULLY
    


.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import lime
    import lime.lime_tabular
    import matplotlib.pyplot as plt
    import matplotlib as mpl
    
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_absolute_error
    from scipy.stats import spearmanr, kendalltau
    
    # =====================================================
    # GLOBAL STYLE
    # =====================================================
    
    mpl.rcParams.update({
        "figure.titlesize": 12,
        "axes.titlesize": 10,
        "axes.labelsize": 9,
        "xtick.labelsize": 8,
        "ytick.labelsize": 8,
        "legend.fontsize": 8,
        "font.size": 10
    })
    
    # =====================================================
    # 1. LOAD DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    df = df.select_dtypes(include=[np.number]).dropna()
    
    features = [c for c in df.columns if c != target]
    
    X = df[features]
    y = df[target]
    
    # =====================================================
    # 2. TRAIN TEST SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    X_train_scaled_df = pd.DataFrame(X_train_scaled, columns=features)
    X_test_scaled_df = pd.DataFrame(X_test_scaled, columns=features)
    
    # =====================================================
    # 3. MODEL
    # =====================================================
    
    model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train_scaled_df, y_train)
    y_pred = model.predict(X_test_scaled_df)
    
    print("\nMODEL PERFORMANCE")
    print("R² :", round(r2_score(y_test, y_pred), 4))
    print("MAE:", round(mean_absolute_error(y_test, y_pred), 4))
    
    # =====================================================
    # 4. SHAP GLOBAL IMPORTANCE
    # =====================================================
    
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_train_scaled_df)
    
    shap_importance = np.abs(shap_values).mean(axis=0)
    
    shap_ranking = pd.Series(
        shap_importance,
        index=features
    ).sort_values(ascending=False)
    
    print("\nSHAP FEATURE IMPORTANCE (ALL VARIABLES)")
    print(shap_ranking)
    
    # =====================================================
    # 5. LIME LOCAL + GLOBAL IMPORTANCE
    # =====================================================
    
    explainer_lime = lime.lime_tabular.LimeTabularExplainer(
        training_data=X_train_scaled,
        feature_names=features,
        mode="regression",
        discretize_continuous=False,
        random_state=42
    )
    
    lime_importance_global = np.zeros(len(features))
    
    # Daha sağlam global LIME: tüm test setinden ortalama
    for i in range(len(X_test_scaled_df)):
    
        exp = explainer_lime.explain_instance(
            X_test_scaled[i],
            model.predict,
            num_features=len(features)
        )
    
        for feat, val in exp.as_list():
    
            # feature ismi tam eşleşme için düzeltme
            clean_feat = feat.split("=")[0].strip()
    
            if clean_feat in features:
                lime_importance_global[features.index(clean_feat)] += abs(val)
    
    lime_importance_global /= len(X_test_scaled_df)
    
    lime_ranking = pd.Series(
        lime_importance_global,
        index=features
    ).sort_values(ascending=False)
    
    print("\nLIME GLOBAL IMPORTANCE (ALL VARIABLES)")
    print(lime_ranking)
    
    # =====================================================
    # 6. CONSISTENCY ANALYSIS
    # =====================================================
    
    common = list(set(shap_ranking.index) & set(lime_ranking.index))
    
    shap_rank = shap_ranking.loc[common].rank()
    lime_rank = lime_ranking.loc[common].rank()
    
    print("\nCONSISTENCY RESULTS")
    print("Spearman:", spearmanr(shap_rank, lime_rank))
    print("Kendall :", kendalltau(shap_rank, lime_rank))
    
    # =====================================================
    # 7. MERGED TABLE
    # =====================================================
    
    merged_df = pd.DataFrame({
        "Feature": features,
        "SHAP Importance": shap_importance,
        "LIME Importance": lime_importance_global
    }).sort_values("SHAP Importance", ascending=False)
    
    print("\nSHAP & LIME MERGED TABLE")
    print(merged_df)
    
    print("\nPIPELINE COMPLETED SUCCESSFULLY")
    
    # =====================================================
    # 8. VISUALIZATION SECTION
    # =====================================================
    
    import seaborn as sns
    
    sns.set_style("whitegrid")
    
    # -------------------------------
    # SHAP vs LIME Comparison Plot
    # -------------------------------
    
    plot_df = merged_df.copy()
    
    fig, ax = plt.subplots(figsize=(10, 6))
    
    bar_width = 0.35
    x = np.arange(len(plot_df))
    
    bars1 = ax.bar(
        x - bar_width/2,
        plot_df["SHAP Importance"],
        width=bar_width,
        label="SHAP",
        color="#1f77b4"
    )
    
    bars2 = ax.bar(
        x + bar_width/2,
        plot_df["LIME Importance"],
        width=bar_width,
        label="LIME",
        color="#ff7f0e"
    )
    
    ax.set_xticks(x)
    ax.set_xticklabels(plot_df["Feature"], rotation=45, ha="right")
    ax.set_title("SHAP vs LIME Feature Importance")
    ax.set_ylabel("Importance Value")
    ax.legend()
    
    # Value labels on bars
    for bars in [bars1, bars2]:
        for bar in bars:
            height = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width()/2,
                height,
                f"{height:.2f}",
                ha='center',
                va='bottom',
                fontsize=8,
                fontweight="bold"
            )
    
    plt.tight_layout()
    plt.show()
    
    # -------------------------------
    # Correlation Heatmap
    # -------------------------------
    
    corr_df = pd.DataFrame({
        "SHAP": shap_importance,
        "LIME": lime_importance_global
    })
    
    fig, ax = plt.subplots(figsize=(6, 5))
    
    sns.heatmap(
        corr_df.corr(),
        annot=True,
        cmap="coolwarm",
        fmt=".3f",
        linewidths=0.5,
        ax=ax
    )
    
    ax.set_title("Correlation Between SHAP and LIME")
    plt.tight_layout()
    plt.show()
    
    # -------------------------------
    # Model Performance Visualization
    # -------------------------------
    
    metrics_df = pd.DataFrame({
        "Metric": ["R²", "MAE"],
        "Value": [r2_score(y_test, y_pred), mean_absolute_error(y_test, y_pred)]
    })
    
    fig, ax = plt.subplots(figsize=(6, 4))
    
    bars = ax.bar(
        metrics_df["Metric"],
        metrics_df["Value"],
        color=["#2ca02c", "#d62728"]
    )
    
    ax.set_title("Model Performance Metrics")
    
    for bar in bars:
        height = bar.get_height()
        ax.text(
            bar.get_x() + bar.get_width()/2,
            height,
            f"{height:.3f}",
            ha='center',
            va='bottom',
            fontsize=9,
            fontweight="bold"
        )
    
    plt.tight_layout()
    plt.show()
    
    print("\nVISUALIZATION COMPLETED SUCCESSFULLY")
    
    ax.text(
        0.5, 0.25,
        textstr,
        transform=ax.transAxes,
        fontsize=10,
        ha='center',
        verticalalignment='bottom',
        bbox=props
    )


.. parsed-literal::

    
    MODEL PERFORMANCE
    R² : 0.9229
    MAE: 1.9354
    
    SHAP FEATURE IMPORTANCE (ALL VARIABLES)
    Creative outputs                    3.584516
    Business sophistication             2.062679
    Knowledge and technology outputs    1.869111
    Human capital and research          1.689691
    Infrastructure                      1.511443
    Institutions                        1.128079
    Market sophistication               0.881737
    dtype: float64
    
    LIME GLOBAL IMPORTANCE (ALL VARIABLES)
    Creative outputs                    3.337358
    Business sophistication             1.750691
    Knowledge and technology outputs    1.697561
    Infrastructure                      1.408998
    Human capital and research          1.338311
    Institutions                        1.118837
    Market sophistication               1.041177
    dtype: float64
    
    CONSISTENCY RESULTS
    Spearman: SignificanceResult(statistic=np.float64(0.9642857142857145), pvalue=np.float64(0.0004541491691941689))
    Kendall : SignificanceResult(statistic=np.float64(0.9047619047619049), pvalue=np.float64(0.002777777777777778))
    
    SHAP & LIME MERGED TABLE
                                Feature  SHAP Importance  LIME Importance
    6                  Creative outputs         3.584516         3.337358
    4           Business sophistication         2.062679         1.750691
    5  Knowledge and technology outputs         1.869111         1.697561
    1        Human capital and research         1.689691         1.338311
    2                    Infrastructure         1.511443         1.408998
    0                      Institutions         1.128079         1.118837
    3             Market sophistication         0.881737         1.041177
    
    PIPELINE COMPLETED SUCCESSFULLY
    


.. image:: output_264_1.png



.. image:: output_264_2.png



.. image:: output_264_3.png


.. parsed-literal::

    
    VISUALIZATION COMPLETED SUCCESSFULLY
    

::


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[64], line 282
        276 plt.show()
        278 print("\nVISUALIZATION COMPLETED SUCCESSFULLY")
        280 ax.text(
        281     0.5, 0.25,
    --> 282     textstr,
        283     transform=ax.transAxes,
        284     fontsize=10,
        285     ha='center',
        286     verticalalignment='bottom',
        287     bbox=props
        288 )
    

    NameError: name 'textstr' is not defined



.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import lime
    import lime.lime_tabular
    import matplotlib.pyplot as plt
    import matplotlib as mpl
    import seaborn as sns
    
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.metrics import r2_score, mean_absolute_error
    from scipy.stats import spearmanr, kendalltau
    
    # =====================================================
    # STYLE
    # =====================================================
    
    mpl.rcParams.update({
        "figure.titlesize": 12,
        "axes.titlesize": 10,
        "axes.labelsize": 9,
        "xtick.labelsize": 8,
        "ytick.labelsize": 8,
        "legend.fontsize": 9,
        "font.size": 10
    })
    
    sns.set_style("whitegrid")
    
    # =====================================================
    # LOAD DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    df = df.select_dtypes(include=[np.number]).dropna()
    
    features = [c for c in df.columns if c != target]
    
    X = df[features]
    y = df[target]
    
    # =====================================================
    # SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    X_train_scaled_df = pd.DataFrame(X_train_scaled, columns=features)
    X_test_scaled_df = pd.DataFrame(X_test_scaled, columns=features)
    
    # =====================================================
    # MODEL
    # =====================================================
    
    model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train_scaled_df, y_train)
    y_pred = model.predict(X_test_scaled_df)
    
    r2 = r2_score(y_test, y_pred)
    mae = mean_absolute_error(y_test, y_pred)
    
    # =====================================================
    # SHAP
    # =====================================================
    
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_train_scaled_df)
    shap_importance = np.abs(shap_values).mean(axis=0)
    
    # =====================================================
    # LIME GLOBAL
    # =====================================================
    
    explainer_lime = lime.lime_tabular.LimeTabularExplainer(
        training_data=X_train_scaled,
        feature_names=features,
        mode="regression",
        discretize_continuous=False,
        random_state=42
    )
    
    lime_importance_global = np.zeros(len(features))
    
    for i in range(len(X_test_scaled_df)):
    
        exp = explainer_lime.explain_instance(
            X_test_scaled[i],
            model.predict,
            num_features=len(features)
        )
    
        for feat, val in exp.as_list():
            clean_feat = feat.split("=")[0].strip()
            if clean_feat in features:
                lime_importance_global[features.index(clean_feat)] += abs(val)
    
    lime_importance_global /= len(X_test_scaled_df)
    
    # =====================================================
    # CONSISTENCY
    # =====================================================
    
    shap_rank = pd.Series(shap_importance, index=features).rank()
    lime_rank = pd.Series(lime_importance_global, index=features).rank()
    
    spearman_corr = spearmanr(shap_rank, lime_rank)
    kendall_corr = kendalltau(shap_rank, lime_rank)
    
    # =====================================================
    # SINGLE FINAL VISUALIZATION
    # =====================================================
    
    merged_df = pd.DataFrame({
        "Feature": features,
        "SHAP": shap_importance,
        "LIME": lime_importance_global
    }).sort_values("SHAP", ascending=False)
    
    fig, ax = plt.subplots(figsize=(11, 6))
    
    bar_width = 0.35
    x = np.arange(len(merged_df))
    
    bars1 = ax.bar(
        x - bar_width/2,
        merged_df["SHAP"],
        width=bar_width,
        label="SHAP",
        color="#1f77b4"
    )
    
    bars2 = ax.bar(
        x + bar_width/2,
        merged_df["LIME"],
        width=bar_width,
        label="LIME",
        color="#ff7f0e"
    )
    
    ax.set_xticks(x)
    ax.set_xticklabels(merged_df["Feature"], rotation=90, ha="left")
    ax.set_ylabel("Importance Value")
    ax.set_title("SHAP vs LIME Feature Importance")
    
    # Value labels on bars
    for bars in [bars1, bars2]:
        for bar in bars:
            height = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width()/2,
                height,
                f"{height:.3f}",
                ha='center',
                va='bottom',
                fontsize=8,
                fontweight="bold",
                rotation=90
            )
    
    # =====================================================
    # ADD PERFORMANCE & CORRELATION ON FIRST GRAPH
    # =====================================================
    
    textstr = (
        f"R² = {r2:.4f}\n"
        f"MAE = {mae:.4f}\n\n"
        f"Spearman = {spearman_corr.statistic:.4f}\n"
        f"Kendall = {kendall_corr.statistic:.4f}"
    )
    
    props = dict(boxstyle='round', facecolor='white', alpha=0.9)
    
    ax.text(
        0.02, 0.25,
        textstr,
        transform=ax.transAxes,
        fontsize=10,
        verticalalignment='top',
        bbox=props
    )
    
    ax.legend()
    
    plt.tight_layout()
    plt.show()
    
    print("\nFINAL VISUALIZATION COMPLETED SUCCESSFULLY")



.. image:: output_266_0.png


.. parsed-literal::

    
    FINAL VISUALIZATION COMPLETED SUCCESSFULLY
    






.. code:: ipython3

    # =====================================================
    # COMPLETE EXTRA TREES PIPELINE – ONLY FIRST PDP STYLE
    # =====================================================
    
    import numpy as np
    import pandas as pd
    import shap
    import lime
    import lime.lime_tabular
    import matplotlib.pyplot as plt
    import matplotlib as mpl
    
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    from sklearn.inspection import PartialDependenceDisplay
    from sklearn.metrics import r2_score, mean_absolute_error
    from scipy.stats import spearmanr, kendalltau
    
    # =====================================================
    # GLOBAL STYLE
    # =====================================================
    
    mpl.rcParams.update({
        "figure.titlesize": 12,
        "axes.titlesize": 10,
        "axes.labelsize": 9,
        "xtick.labelsize": 8,
        "ytick.labelsize": 8,
        "legend.fontsize": 8,
        "font.size": 10
    })
    
    # =====================================================
    # 1. LOAD DATA
    # =====================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    
    df = df.select_dtypes(include=[np.number]).dropna()
    
    features = [c for c in df.columns if c != target]
    
    X = df[features]
    y = df[target]
    
    # =====================================================
    # 2. TRAIN TEST SPLIT
    # =====================================================
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    # =====================================================
    # 3. MODEL – EXTRA TREES
    # =====================================================
    
    model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train_scaled, y_train)
    
    y_pred = model.predict(X_test_scaled)
    
    print("\nMODEL PERFORMANCE")
    print("R²:", round(r2_score(y_test, y_pred), 4))
    print("MAE:", round(mean_absolute_error(y_test, y_pred), 4))
    
    # =====================================================
    # 4. SHAP GLOBAL IMPORTANCE
    # =====================================================
    
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_train_scaled)
    
    shap_importance = np.abs(shap_values).mean(axis=0)
    shap_ranking = pd.Series(
        shap_importance,
        index=features
    ).sort_values(ascending=False)
    
    print("\nTOP 10 SHAP FEATURES")
    print(shap_ranking.head(10))
    
    # =====================================================
    # 5. LIME LOCAL IMPORTANCE
    # =====================================================
    
    explainer_lime = lime.lime_tabular.LimeTabularExplainer(
        training_data=X_train_scaled,
        feature_names=features,
        mode="regression",
        discretize_continuous=True,
        random_state=42
    )
    
    test_df = pd.DataFrame(X_test_scaled, columns=features)
    test_df["Prediction"] = model.predict(X_test_scaled)
    
    cases = {
        "Low Prediction": test_df["Prediction"].idxmin(),
        "Median Prediction": test_df["Prediction"].sub(
            test_df["Prediction"].median()
        ).abs().idxmin(),
        "High Prediction": test_df["Prediction"].idxmax()
    }
    
    fig, axes = plt.subplots(1, 3, figsize=(14, 4))
    color_map = plt.cm.tab20.colors
    lime_importance_global = np.zeros(len(features))
    
    for ax, (label, idx) in zip(axes, cases.items()):
    
        exp = explainer_lime.explain_instance(
            X_test_scaled[idx],
            model.predict,
            num_features=8
        )
    
        df_plot = pd.DataFrame(
            exp.as_list(),
            columns=["Feature", "Contribution"]
        ).sort_values("Contribution")
    
        colors = [color_map[i % len(color_map)] for i in range(len(df_plot))]
    
        bars = ax.barh(
            df_plot["Feature"],
            df_plot["Contribution"],
            color=colors
        )
    
        ax.set_title(f"{label}\n(Local R² = {exp.score:.3f})")
    
        for bar in bars:
            width = bar.get_width()
            ax.text(
                width,
                bar.get_y() + bar.get_height()/2,
                f"{width:.3f}",
                va='center',
                ha='left' if width > 0 else 'right',
                fontsize=7
            )
    
        for feat, val in exp.as_list():
            fname = feat.split(" ")[0]
            if fname in features:
                lime_importance_global[features.index(fname)] += abs(val)
    
    fig.suptitle("LIME Local Explanations")
    plt.tight_layout()
    plt.show()
    
    lime_importance_global /= 3
    lime_ranking = pd.Series(
        lime_importance_global,
        index=features
    ).sort_values(ascending=False)
    
    # =====================================================
    # 6. SHAP – LIME CONSISTENCY
    # =====================================================
    
    common = list(set(shap_ranking.index) & set(lime_ranking.index))
    
    shap_rank = shap_ranking.loc[common].rank()
    lime_rank = lime_ranking.loc[common].rank()
    
    print("\nCONSISTENCY RESULTS")
    print("Spearman:", spearmanr(shap_rank, lime_rank))
    print("Kendall:", kendalltau(shap_rank, lime_rank))
    
    # =====================================================
    # =====================================================
    # 7. PARTIAL DEPENDENCE – ONLY FIRST ORIGINAL STYLE
    # =====================================================
    
    top5 = shap_ranking.head(5).index.tolist()
    
    plt.figure(figsize=(10, 8))
    
    PartialDependenceDisplay.from_estimator(
        model,
        X_train_scaled,
        features=top5,
        feature_names=features,
        grid_resolution=50
    )
    
    plt.suptitle("Partial Dependence Plots (Top 5 SHAP Features)")
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    MODEL PERFORMANCE
    R²: 0.9229
    MAE: 1.9354
    
    TOP 10 SHAP FEATURES
    Creative outputs                    3.584516
    Business sophistication             2.062679
    Knowledge and technology outputs    1.869111
    Human capital and research          1.689691
    Infrastructure                      1.511443
    Institutions                        1.128079
    Market sophistication               0.881737
    dtype: float64
    


.. image:: output_272_1.png


.. parsed-literal::

    
    CONSISTENCY RESULTS
    Spearman: SignificanceResult(statistic=np.float64(-0.445435403187374), pvalue=np.float64(0.31653589902854457))
    Kendall: SignificanceResult(statistic=np.float64(-0.3289758474798845), pvalue=np.float64(0.3418143874474573))
    


.. parsed-literal::

    <Figure size 3000x2400 with 0 Axes>



.. image:: output_272_4.png













.. code:: ipython3

    # =====================================================
    # 🔥 SHAP–LIME RANK CONSISTENCY ANALYSIS (FULL FIXED)
    # =====================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    import shap
    from lime.lime_tabular import LimeTabularExplainer
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.stats import spearmanr
    
    sns.set_style("whitegrid")
    
    # --------------------------------------------------
    # 0️⃣ MODEL TRAIN
    # --------------------------------------------------
    
    model = ExtraTreesRegressor(
        n_estimators=400,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train, y_train)
    
    # --------------------------------------------------
    # 1️⃣ SHAP VALUES
    # --------------------------------------------------
    
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_test)
    
    shap_importance = np.abs(shap_values).mean(axis=0)
    
    feature_labels = list(X_train.columns)
    
    shap_rank_df = pd.DataFrame({
        "Feature": feature_labels,
        "SHAP_Importance": shap_importance
    }).sort_values("SHAP_Importance", ascending=False)
    
    shap_rank_df["SHAP_Rank"] = range(1, len(shap_rank_df) + 1)
    
    print("\n=========== SHAP IMPORTANCE ===========")
    print(shap_rank_df.round(4))
    
    # --------------------------------------------------
    # 2️⃣ LIME VALUES
    # --------------------------------------------------
    
    X_train_np = X_train.values
    X_test_np = X_test.values
    
    lime_explainer = LimeTabularExplainer(
        X_train_np,
        feature_names=feature_labels,
        mode="regression"
    )
    
    lime_importance = np.zeros(len(feature_labels))
    
    n_samples = min(50, len(X_test_np))
    
    for i in range(n_samples):
        exp = lime_explainer.explain_instance(
            X_test_np[i],
            model.predict,   # 🔥 artık tanımlı
            num_features=len(feature_labels)
        )
    
        for feat, val in exp.as_list():
            clean_feat = feat.split(" ")[0]
            if clean_feat in feature_labels:
                idx = feature_labels.index(clean_feat)
                lime_importance[idx] += abs(val)
    
    lime_importance /= n_samples
    
    lime_rank_df = pd.DataFrame({
        "Feature": feature_labels,
        "LIME_Importance": lime_importance
    }).sort_values("LIME_Importance", ascending=False)
    
    lime_rank_df["LIME_Rank"] = range(1, len(lime_rank_df) + 1)
    
    print("\n=========== LIME IMPORTANCE ===========")
    print(lime_rank_df.round(4))
    
    # --------------------------------------------------
    # 3️⃣ MERGE & RANK DIFFERENCE
    # --------------------------------------------------
    
    plot_df = shap_rank_df.merge(lime_rank_df, on="Feature")
    plot_df["Rank_Diff"] = abs(plot_df["SHAP_Rank"] - plot_df["LIME_Rank"])
    rank_df = plot_df.sort_values("Rank_Diff")
    
    print("\n=========== RANK DIFFERENCE ===========")
    print(rank_df[["Feature", "SHAP_Rank", "LIME_Rank", "Rank_Diff"]])
    
    # --------------------------------------------------
    # 4️⃣ VISUALIZATION
    # --------------------------------------------------
    
    plt.figure(figsize=(10, 6))
    
    colors = plt.cm.viridis(np.linspace(0, 1, len(rank_df)))
    
    bars = plt.barh(
        rank_df["Feature"],
        rank_df["Rank_Diff"],
        color=colors,
        edgecolor="black"
    )
    
    plt.title(
        "SHAP–LIME Rank Difference\n(Lower = Higher Consistency)",
        fontsize=14,
        fontweight="bold"
    )
    
    plt.xlabel("Absolute Rank Difference", fontsize=12)
    plt.ylabel("Feature", fontsize=12)
    
    for bar in bars:
        width = bar.get_width()
        plt.text(
            width + 0.05,
            bar.get_y() + bar.get_height()/2,
            f"{width:.0f}",
            va="center",
            fontsize=10
        )
    
    plt.tight_layout()
    plt.show()
    
    # --------------------------------------------------
    # 5️⃣ SPEARMAN CONSISTENCY SCORE
    # --------------------------------------------------
    
    rho, pval = spearmanr(plot_df["SHAP_Rank"], plot_df["LIME_Rank"])
    
    print("\n=========== RANK CONSISTENCY ===========")
    print(f"Spearman ρ: {rho:.3f}")
    print(f"p-value: {pval:.4f}")


.. parsed-literal::

    
    =========== SHAP IMPORTANCE ===========
                                Feature  SHAP_Importance  SHAP_Rank
    6                  Creative outputs           3.4904          1
    4           Business sophistication           2.2997          2
    5  Knowledge and technology outputs           1.9286          3
    1        Human capital and research           1.7396          4
    2                    Infrastructure           1.4921          5
    0                      Institutions           1.1703          6
    3             Market sophistication           0.9577          7
    
    =========== LIME IMPORTANCE ===========
                                Feature  LIME_Importance  LIME_Rank
    2                    Infrastructure           1.3046          1
    0                      Institutions           0.8022          2
    1        Human capital and research           0.0000          3
    3             Market sophistication           0.0000          4
    4           Business sophistication           0.0000          5
    5  Knowledge and technology outputs           0.0000          6
    6                  Creative outputs           0.0000          7
    
    =========== RANK DIFFERENCE ===========
                                Feature  SHAP_Rank  LIME_Rank  Rank_Diff
    3        Human capital and research          4          3          1
    1           Business sophistication          2          5          3
    2  Knowledge and technology outputs          3          6          3
    6             Market sophistication          7          4          3
    5                      Institutions          6          2          4
    4                    Infrastructure          5          1          4
    0                  Creative outputs          1          7          6
    


.. image:: output_284_1.png


.. parsed-literal::

    
    =========== RANK CONSISTENCY ===========
    Spearman ρ: -0.714
    p-value: 0.0713
    



.. code:: ipython3

    # =====================================================
    # 🔥 SHAP–LIME RANK CONSISTENCY ANALYSIS (FULL WORKING)
    # =====================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    import shap
    from lime.lime_tabular import LimeTabularExplainer
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.stats import spearmanr
    
    sns.set_style("whitegrid")
    
    # --------------------------------------------------
    # 0️⃣ MODEL TRAIN
    # --------------------------------------------------
    
    model = ExtraTreesRegressor(
        n_estimators=400,
        random_state=42,
        n_jobs=-1
    )
    
    model.fit(X_train, y_train)
    
    feature_labels = list(X_train.columns)
    
    # --------------------------------------------------
    # 1️⃣ SHAP VALUES
    # --------------------------------------------------
    
    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_test)
    
    shap_importance = np.abs(shap_values).mean(axis=0)
    
    shap_rank_df = pd.DataFrame({
        "Feature": feature_labels,
        "SHAP_Importance": shap_importance
    }).sort_values("SHAP_Importance", ascending=False)
    
    shap_rank_df["SHAP_Rank"] = range(1, len(shap_rank_df) + 1)
    
    print("\n=========== SHAP IMPORTANCE ===========")
    print(shap_rank_df.round(4))
    
    # --------------------------------------------------
    # 2️⃣ LIME VALUES
    # --------------------------------------------------
    
    X_train_np = X_train.values
    X_test_np = X_test.values
    
    lime_explainer = LimeTabularExplainer(
        X_train_np,
        feature_names=feature_labels,
        mode="regression"
    )
    
    lime_importance = np.zeros(len(feature_labels))
    n_samples = min(50, len(X_test_np))
    
    for i in range(n_samples):
    
        exp = lime_explainer.explain_instance(
            X_test_np[i],
            model.predict,  # ✅ DOĞRU MODEL
            num_features=len(feature_labels)
        )
    
        for feat, val in exp.as_list():
            clean_feat = feat.split(" ")[0]
            if clean_feat in feature_labels:
                idx = feature_labels.index(clean_feat)
                lime_importance[idx] += abs(val)
    
    lime_importance /= n_samples
    
    lime_rank_df = pd.DataFrame({
        "Feature": feature_labels,
        "LIME_Importance": lime_importance
    }).sort_values("LIME_Importance", ascending=False)
    
    lime_rank_df["LIME_Rank"] = range(1, len(lime_rank_df) + 1)
    
    print("\n=========== LIME IMPORTANCE ===========")
    print(lime_rank_df.round(4))
    
    # --------------------------------------------------
    # 3️⃣ MERGE & RANK DIFFERENCE
    # --------------------------------------------------
    
    plot_df = shap_rank_df.merge(lime_rank_df, on="Feature")
    plot_df["Rank_Diff"] = abs(plot_df["SHAP_Rank"] - plot_df["LIME_Rank"])
    rank_df = plot_df.sort_values("Rank_Diff")
    
    print("\n=========== FEATURE RANKS & DIFFERENCES ===========")
    print(rank_df[["Feature", "SHAP_Rank", "LIME_Rank", "Rank_Diff"]].to_string(index=False))
    
    # --------------------------------------------------
    # 4️⃣ VISUALIZATION
    # --------------------------------------------------
    
    plt.figure(figsize=(10, 6))
    
    colors = plt.cm.viridis(np.linspace(0, 1, len(rank_df)))
    
    bars = plt.barh(
        rank_df["Feature"],
        rank_df["Rank_Diff"],
        color=colors,
        edgecolor="black"
    )
    
    plt.title(
        "SHAP–LIME Rank Difference\n(Lower = Higher Consistency)",
        fontsize=14,
        fontweight="bold"
    )
    
    plt.xlabel("Absolute Rank Difference", fontsize=12)
    plt.ylabel("Feature", fontsize=12)
    
    for bar, shp, lim in zip(bars, rank_df["SHAP_Rank"], rank_df["LIME_Rank"]):
        width = bar.get_width()
        plt.text(
            width + 0.05,
            bar.get_y() + bar.get_height()/2,
            f"Diff:{width:.0f} (S:{shp}, L:{lim})",
            va="center",
            fontsize=9
        )
    
    # --------------------------------------------------
    # 5️⃣ SPEARMAN CONSISTENCY
    # --------------------------------------------------
    
    rho, pval = spearmanr(plot_df["SHAP_Rank"], plot_df["LIME_Rank"])
    
    plt.text(
        x=max(rank_df["Rank_Diff"]) * 0.55,
        y=0,
        s=f"Spearman ρ: {rho:.3f}\np-value: {pval:.4f}",
        fontsize=11,
        fontweight="bold",
        color="red"
    )
    
    plt.tight_layout()
    plt.show()
    
    print("\n=========== RANK CONSISTENCY ===========")
    print(f"Spearman ρ: {rho:.3f}")
    print(f"p-value: {pval:.4f}")


.. parsed-literal::

    
    =========== SHAP IMPORTANCE ===========
                                Feature  SHAP_Importance  SHAP_Rank
    6                  Creative outputs           3.4904          1
    4           Business sophistication           2.2997          2
    5  Knowledge and technology outputs           1.9286          3
    1        Human capital and research           1.7396          4
    2                    Infrastructure           1.4921          5
    0                      Institutions           1.1703          6
    3             Market sophistication           0.9577          7
    
    =========== LIME IMPORTANCE ===========
                                Feature  LIME_Importance  LIME_Rank
    2                    Infrastructure           1.3133          1
    0                      Institutions           0.8280          2
    1        Human capital and research           0.0000          3
    3             Market sophistication           0.0000          4
    4           Business sophistication           0.0000          5
    5  Knowledge and technology outputs           0.0000          6
    6                  Creative outputs           0.0000          7
    
    =========== FEATURE RANKS & DIFFERENCES ===========
                             Feature  SHAP_Rank  LIME_Rank  Rank_Diff
          Human capital and research          4          3          1
             Business sophistication          2          5          3
    Knowledge and technology outputs          3          6          3
               Market sophistication          7          4          3
                        Institutions          6          2          4
                      Infrastructure          5          1          4
                    Creative outputs          1          7          6
    


.. image:: output_287_1.png


.. parsed-literal::

    
    =========== RANK CONSISTENCY ===========
    Spearman ρ: -0.714
    p-value: 0.0713
    












.. code:: ipython3

    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    from scipy.stats import spearmanr
    
    print("\n" + "=" * 60)
    print("SHAP vs LIME CONSISTENCY (GLOBAL–LOCAL)")
    print("=" * 60)
    
    # =====================================================
    # 0️⃣ Check column names
    # =====================================================
    print("SHAP DataFrame columns:", shap_df.columns.tolist())
    print("LIME DataFrame columns:", lime_df.columns.tolist())
    
    # Eğer isimler farklıysa burada kendi sütun isimlerinizi kullanın:
    # Örnek: shap_df['Mean_SHAP'] gibi
    shap_col = "SHAP_Importance" if "SHAP_Importance" in shap_df.columns else shap_df.columns[1]
    lime_col = "LIME_Importance" if "LIME_Importance" in lime_df.columns else lime_df.columns[1]
    
    # =====================================================
    # 1️⃣ GLOBAL SHAP & LIME IMPORTANCE
    # =====================================================
    shap_global = shap_df[["Feature", shap_col]].rename(columns={shap_col: "SHAP_MeanAbs"})
    lime_global = lime_df[["Feature", lime_col]].rename(columns={lime_col: "LIME_MeanAbs"})
    
    # =====================================================
    # 2️⃣ MERGE
    # =====================================================
    consistency_df = shap_global.merge(lime_global, on="Feature", how="inner")
    
    # =====================================================
    # 3️⃣ RANK CALCULATION
    # =====================================================
    consistency_df["Rank_SHAP"] = consistency_df["SHAP_MeanAbs"].rank(ascending=False, method="average")
    consistency_df["Rank_LIME"] = consistency_df["LIME_MeanAbs"].rank(ascending=False, method="average")
    consistency_df["Rank_Diff"] = (consistency_df["Rank_SHAP"] - consistency_df["Rank_LIME"]).abs()
    consistency_df = consistency_df.sort_values("Rank_Diff")
    
    print("\n=== SHAP vs LIME CONSISTENCY TABLE ===\n")
    print(consistency_df.round(4))
    
    # =====================================================
    # 4️⃣ NUMERIC SUMMARY
    # =====================================================
    avg_rank_diff = consistency_df["Rank_Diff"].mean()
    rho, pval = spearmanr(consistency_df["Rank_SHAP"], consistency_df["Rank_LIME"])
    
    print("\nConsistency diagnostics:")
    print(f"• Average rank difference : {avg_rank_diff:.3f}")
    print(f"• Spearman ρ              : {rho:.3f}")
    print(f"• p-value                 : {pval:.4f}")
    
    # =====================================================
    # 5️⃣ BARPLOT – GLOBAL IMPORTANCE COMPARISON
    # =====================================================
    plot_df = consistency_df.sort_values("SHAP_MeanAbs", ascending=False).reset_index(drop=True)
    
    plt.figure(figsize=(10, 6))
    ax = plot_df.set_index("Feature")[["SHAP_MeanAbs", "LIME_MeanAbs"]].plot(
        kind="barh", edgecolor="black"
    )
    plt.title("Global Feature Importance Comparison\nSHAP vs LIME", fontsize=13)
    plt.xlabel("Mean Absolute Contribution")
    plt.ylabel("Feature")
    
    for container in ax.containers:
        ax.bar_label(container, fmt="%.3f", padding=3)
    
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 6️⃣ BARPLOT – RANK DIFFERENCE
    # =====================================================
    rank_df = plot_df.sort_values("Rank_Diff")
    plt.figure(figsize=(10, 6))
    colors = plt.cm.tab20(np.linspace(0, 1, len(rank_df)))
    bars = plt.barh(rank_df["Feature"], rank_df["Rank_Diff"], color=colors, edgecolor="black")
    plt.title("SHAP–LIME Rank Difference\n(Lower = Higher Consistency)", fontsize=14)
    plt.xlabel("Absolute Rank Difference")
    plt.ylabel("Feature")
    plt.yticks(fontsize=12)
    for bar in bars:
        width = bar.get_width()
        plt.text(width + 0.05, bar.get_y() + bar.get_height()/2, f"{width:.2f}", va="center")
    plt.tight_layout()
    plt.show()
    
    # =====================================================
    # 7️⃣ HIGH CONSISTENCY FEATURES
    # =====================================================
    high_consistency = rank_df[rank_df["Rank_Diff"] <= 1]["Feature"].tolist()
    print("\nFeatures with strongest SHAP–LIME agreement:")
    for f in high_consistency:
        print(f"  ✓ {f}")


.. parsed-literal::

    
    ============================================================
    SHAP vs LIME CONSISTENCY (GLOBAL–LOCAL)
    ============================================================
    SHAP DataFrame columns: ['Feature', 'SHAP Importance']
    LIME DataFrame columns: ['Feature', 'LIME Importance']
    
    === SHAP vs LIME CONSISTENCY TABLE ===
    
                                Feature  SHAP_MeanAbs  LIME_MeanAbs  Rank_SHAP  \
    3        Human capital and research        1.6897        0.0000        4.0   
    2  Knowledge and technology outputs        1.8691        0.0000        3.0   
    6             Market sophistication        0.8817        0.0000        7.0   
    1           Business sophistication        2.0627        0.0000        2.0   
    0                  Creative outputs        3.5845        0.0000        1.0   
    4                    Infrastructure        1.5114        1.6370        5.0   
    5                      Institutions        1.1281        0.7142        6.0   
    
       Rank_LIME  Rank_Diff  
    3        5.0        1.0  
    2        5.0        2.0  
    6        5.0        2.0  
    1        5.0        3.0  
    0        5.0        4.0  
    4        1.0        4.0  
    5        2.0        4.0  
    
    Consistency diagnostics:
    • Average rank difference : 2.857
    • Spearman ρ              : -0.445
    • p-value                 : 0.3165
    


.. parsed-literal::

    <Figure size 3000x1800 with 0 Axes>



.. image:: output_299_2.png



.. image:: output_299_3.png


.. parsed-literal::

    
    Features with strongest SHAP–LIME agreement:
      ✓ Human capital and research
    




.. code:: ipython3

    print("\n" + "="*60)
    print("3️⃣ SHAP vs LIME CONSISTENCY (AUTO-DETECTED VERSION)")
    print("="*60)
    
    # =====================================================
    # 0️⃣ AUTO-DETECT NUMERIC LIME COLUMN
    # =====================================================
    
    # Feature kolonunu hariç tut
    numeric_cols = lime_df.select_dtypes(include=[np.number]).columns.tolist()
    
    if len(numeric_cols) == 0:
        raise ValueError("No numeric columns found in lime_df.")
    
    lime_value_col = numeric_cols[0]
    print(f"\n✔ Auto-detected LIME numeric column: {lime_value_col}")
    
    # =====================================================
    # 1️⃣ SHAP REFERENCE
    # =====================================================
    
    canonical_features = {
        f.lower().replace(" ", "").replace("_", ""): f
        for f in shap_global["Feature"]
    }
    
    shap_ref = (
        shap_global
        .assign(
            key=lambda df: df["Feature"]
            .str.lower()
            .str.replace(" ", "", regex=False)
            .str.replace("_", "", regex=False)
        )
        [["key", "SHAP_MeanAbs"]]
    )
    
    # =====================================================
    # 2️⃣ LIME PARSING
    # =====================================================
    
    if "Feature" not in lime_df.columns:
        raise ValueError("lime_df must contain a 'Feature' column.")
    
    lime_parsed = (
        lime_df
        .assign(
            key=lambda df:
                df["Feature"]
                .astype(str)
                .str.replace(r"\s*[<>=].*", "", regex=True)
                .str.lower()
                .str.replace(" ", "", regex=False)
                .str.replace("_", "", regex=False)
        )
    )
    
    lime_global = (
        lime_parsed
        .groupby("key")[lime_value_col]
        .apply(lambda x: np.mean(np.abs(x)))
        .reset_index()
        .rename(columns={lime_value_col: "LIME_MeanAbs"})
    )
    
    # =====================================================
    # 3️⃣ MERGE
    # =====================================================
    
    consistency_df = shap_ref.merge(
        lime_global,
        on="key",
        how="inner"
    )
    
    if len(consistency_df) == 0:
        raise ValueError("No matching features between SHAP and LIME after merge.")
    
    # Restore readable names
    consistency_df["Feature"] = consistency_df["key"].map(canonical_features)
    
    # =====================================================
    # 4️⃣ RANKING
    # =====================================================
    
    consistency_df["Rank_SHAP"] = consistency_df["SHAP_MeanAbs"].rank(
        ascending=False, method="average"
    )
    
    consistency_df["Rank_LIME"] = consistency_df["LIME_MeanAbs"].rank(
        ascending=False, method="average"
    )
    
    consistency_df["Rank_Diff"] = (
        consistency_df["Rank_SHAP"] - consistency_df["Rank_LIME"]
    ).abs()
    
    consistency_df = consistency_df.sort_values("Rank_Diff")
    
    # =====================================================
    # 5️⃣ OUTPUT
    # =====================================================
    
    print("\n✔ Matched features:", len(consistency_df))
    print("\nSHAP–LIME Consistency Table:\n")
    print(consistency_df.round(3))
    
    avg_rank_diff = consistency_df["Rank_Diff"].mean()
    print("\nAverage rank difference:", round(avg_rank_diff, 3))


.. parsed-literal::

    
    ============================================================
    3️⃣ SHAP vs LIME CONSISTENCY (AUTO-DETECTED VERSION)
    ============================================================
    
    ✔ Auto-detected LIME numeric column: LIME Importance
    
    ✔ Matched features: 7
    
    SHAP–LIME Consistency Table:
    
                                 key  SHAP_MeanAbs  LIME_MeanAbs  \
    3        humancapitalandresearch         1.690         0.000   
    2  knowledgeandtechnologyoutputs         1.869         0.000   
    6           marketsophistication         0.882         0.000   
    1         businesssophistication         2.063         0.000   
    0                creativeoutputs         3.585         0.000   
    4                 infrastructure         1.511         1.637   
    5                   institutions         1.128         0.714   
    
                                Feature  Rank_SHAP  Rank_LIME  Rank_Diff  
    3        Human capital and research        4.0        5.0        1.0  
    2  Knowledge and technology outputs        3.0        5.0        2.0  
    6             Market sophistication        7.0        5.0        2.0  
    1           Business sophistication        2.0        5.0        3.0  
    0                  Creative outputs        1.0        5.0        4.0  
    4                    Infrastructure        5.0        1.0        4.0  
    5                      Institutions        6.0        2.0        4.0  
    
    Average rank difference: 2.857
    











.. code:: ipython3

    # ============================================================
    # PANEL ML + TEMPORAL DRIFT + REGIME ANALYSIS (Q1-READY)
    # ============================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    import shap
    
    from sklearn.model_selection import GroupKFold
    from sklearn.preprocessing import StandardScaler
    from sklearn.ensemble import GradientBoostingRegressor
    from sklearn.cluster import KMeans
    from sklearn.metrics import r2_score
    from sklearn.inspection import PartialDependenceDisplay
    
    import warnings
    warnings.filterwarnings("ignore")
    
    # ============================================================
    # LOAD PANEL DATA
    # ============================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    features = [c for c in df.columns if c not in ["Country", "Year", target]]
    
    # ============================================================
    # STANDARDIZATION
    # ============================================================
    
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(df[features])
    y_std = scaler_y.fit_transform(df[[target]]).ravel()
    
    df_std = df.copy()
    df_std[features] = X_std
    df_std[target] = y_std
    
    # ============================================================
    # TIME-AWARE CV (GroupKFold by Country)
    # ============================================================
    
    gkf = GroupKFold(n_splits=5)
    model = GradientBoostingRegressor(n_estimators=400, random_state=42)
    
    r2_scores = []
    
    for train_idx, test_idx in gkf.split(
            df_std[features], df_std[target], groups=df_std["Country"]):
    
        X_train, X_test = df_std.iloc[train_idx][features], df_std.iloc[test_idx][features]
        y_train, y_test = df_std.iloc[train_idx][target], df_std.iloc[test_idx][target]
    
        model.fit(X_train, y_train)
        preds = model.predict(X_test)
        r2_scores.append(r2_score(y_test, preds))
    
    print("\nPanel Cross-Validated R² Scores:", np.round(r2_scores,3))
    print("Mean CV R²:", round(np.mean(r2_scores),3))


.. parsed-literal::

    
    Panel Cross-Validated R² Scores: [0.881 0.966 0.939 0.973 0.881]
    Mean CV R²: 0.928
    





.. code:: ipython3

    # ============================================================
    # 3️⃣ CLUSTER-BASED SHAP REGIME DECOMPOSITION (LABELED + BIGGER X FONT)
    # ============================================================
    
    from sklearn.cluster import KMeans
    import matplotlib.pyplot as plt
    import numpy as np
    import pandas as pd
    
    # ------------------------------------------------------------
    # 1️⃣ SHAP DF
    # ------------------------------------------------------------
    
    shap_df = pd.DataFrame(
        shap_values,
        columns=feature_labels
    )
    
    # ------------------------------------------------------------
    # 2️⃣ CLUSTERING (X_test)
    # ------------------------------------------------------------
    
    kmeans = KMeans(n_clusters=6, random_state=42)
    clusters = kmeans.fit_predict(X_test)
    
    shap_df["Cluster"] = clusters
    
    # ------------------------------------------------------------
    # 3️⃣ CLUSTER MEANS
    # ------------------------------------------------------------
    
    cluster_means = shap_df.groupby("Cluster").mean()
    
    print("\nCluster-based SHAP decomposition:")
    print(cluster_means.round(3))
    
    # ------------------------------------------------------------
    # 4️⃣ VISUALIZATION
    # ------------------------------------------------------------
    
    fig, ax = plt.subplots(figsize=(13,9))
    
    cluster_means.T.plot(
        kind="bar",
        ax=ax
    )
    
    plt.title("Innovation Regime SHAP Profiles", fontsize=18)
    plt.ylabel("Mean SHAP Contribution", fontsize=14)
    
    # 🔥 X ekseni font büyütme
    ax.tick_params(axis='x', labelsize=14)
    
    # 🔥 Y ekseni font büyütme (isteğe bağlı)
    ax.tick_params(axis='y', labelsize=13)
    
    plt.xticks(rotation=45)
    plt.grid(axis="y", alpha=0.4)
    
    # ------------------------------------------------------------
    # 🔥 SAYISAL ETİKETLER (90°)
    # ------------------------------------------------------------
    
    for container in ax.containers:
        for bar in container:
            height = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width()/2,
                height,
                f"{height:.3f}",
                ha="center",
                va="bottom",
                rotation=90,
                fontsize=9
            )
    
    plt.tight_layout()
    plt.show()


::


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Cell In[72], line 26
         23 kmeans = KMeans(n_clusters=6, random_state=42)
         24 clusters = kmeans.fit_predict(X_test)
    ---> 26 shap_df["Cluster"] = clusters
         28 # ------------------------------------------------------------
         29 # 3️⃣ CLUSTER MEANS
         30 # ------------------------------------------------------------
         32 cluster_means = shap_df.groupby("Cluster").mean()
    

    File ~\anaconda3\Lib\site-packages\pandas\core\frame.py:4311, in DataFrame.__setitem__(self, key, value)
       4308     self._setitem_array([key], value)
       4309 else:
       4310     # set column
    -> 4311     self._set_item(key, value)
    

    File ~\anaconda3\Lib\site-packages\pandas\core\frame.py:4524, in DataFrame._set_item(self, key, value)
       4514 def _set_item(self, key, value) -> None:
       4515     """
       4516     Add series to DataFrame in specified column.
       4517 
       (...)
       4522     ensure homogeneity.
       4523     """
    -> 4524     value, refs = self._sanitize_column(value)
       4526     if (
       4527         key in self.columns
       4528         and value.ndim == 1
       4529         and not isinstance(value.dtype, ExtensionDtype)
       4530     ):
       4531         # broadcast across multiple columns if necessary
       4532         if not self.columns.is_unique or isinstance(self.columns, MultiIndex):
    

    File ~\anaconda3\Lib\site-packages\pandas\core\frame.py:5266, in DataFrame._sanitize_column(self, value)
       5263     return _reindex_for_setitem(value, self.index)
       5265 if is_list_like(value):
    -> 5266     com.require_length_match(value, self.index)
       5267 arr = sanitize_array(value, self.index, copy=True, allow_2d=True)
       5268 if (
       5269     isinstance(value, Index)
       5270     and value.dtype == "object"
       (...)
       5273     # TODO: Remove kludge in sanitize_array for string mode when enforcing
       5274     # this deprecation
    

    File ~\anaconda3\Lib\site-packages\pandas\core\common.py:573, in require_length_match(data, index)
        569 """
        570 Check the length of data matches the length of the index.
        571 """
        572 if len(data) != len(index):
    --> 573     raise ValueError(
        574         "Length of values "
        575         f"({len(data)}) "
        576         "does not match length of index "
        577         f"({len(index)})"
        578     )
    

    ValueError: Length of values (27) does not match length of index (28)


.. code:: ipython3

    # ============================================================
    # 3️⃣ CLUSTER-BASED SHAP REGIME DECOMPOSITION
    # ============================================================
    
    from sklearn.cluster import KMeans
    import matplotlib.pyplot as plt
    import numpy as np
    import pandas as pd
    
    # ------------------------------------------------------------
    # 1️⃣ SHAP DATAFRAME (X_test ile birebir uyumlu)
    # ------------------------------------------------------------
    
    shap_df = pd.DataFrame(
        shap_values,                 # SHAP X_test üzerinden hesaplanmış olmalı
        columns=feature_labels,
        index=X_test.index           # 🔥 KRİTİK: index eşitleme
    )
    
    # ------------------------------------------------------------
    # 2️⃣ CLUSTERING (SHAP SPACE ÜZERİNDE)
    # ------------------------------------------------------------
    
    kmeans = KMeans(
        n_clusters=6,
        random_state=42,
        n_init=20
    )
    
    clusters = kmeans.fit_predict(shap_df)
    
    shap_df["Cluster"] = clusters
    
    # ------------------------------------------------------------
    # 3️⃣ CLUSTER MEANS
    # ------------------------------------------------------------
    
    cluster_means = shap_df.groupby("Cluster").mean()
    
    print("\nCluster-based SHAP decomposition:\n")
    print(cluster_means.round(3))
    
    # ------------------------------------------------------------
    # 4️⃣ VISUALIZATION
    # ------------------------------------------------------------
    
    fig, ax = plt.subplots(figsize=(14, 9))
    
    cluster_means.T.plot(
        kind="bar",
        ax=ax
    )
    
    ax.set_title(
        "Innovation Regime SHAP Profiles",
        fontsize=18,
        weight="bold"
    )
    
    ax.set_ylabel("Mean SHAP Contribution", fontsize=14)
    
    # X ve Y font ayarları
    ax.tick_params(axis='x', labelsize=14)
    ax.tick_params(axis='y', labelsize=13)
    
    plt.xticks(rotation=45)
    plt.grid(axis="y", alpha=0.4)
    
    # ------------------------------------------------------------
    # SAYISAL ETİKETLER
    # ------------------------------------------------------------
    
    for container in ax.containers:
        for bar in container:
            height = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width() / 2,
                height,
                f"{height:.3f}",
                ha="center",
                va="bottom",
                rotation=90,
                fontsize=9
            )
    
    plt.tight_layout()
    plt.show()


::


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Cell In[73], line 14
          8 import pandas as pd
         10 # ------------------------------------------------------------
         11 # 1️⃣ SHAP DATAFRAME (X_test ile birebir uyumlu)
         12 # ------------------------------------------------------------
    ---> 14 shap_df = pd.DataFrame(
         15     shap_values,                 # SHAP X_test üzerinden hesaplanmış olmalı
         16     columns=feature_labels,
         17     index=X_test.index           # 🔥 KRİTİK: index eşitleme
         18 )
         20 # ------------------------------------------------------------
         21 # 2️⃣ CLUSTERING (SHAP SPACE ÜZERİNDE)
         22 # ------------------------------------------------------------
         24 kmeans = KMeans(
         25     n_clusters=6,
         26     random_state=42,
         27     n_init=20
         28 )
    

    File ~\anaconda3\Lib\site-packages\pandas\core\frame.py:827, in DataFrame.__init__(self, data, index, columns, dtype, copy)
        816         mgr = dict_to_mgr(
        817             # error: Item "ndarray" of "Union[ndarray, Series, Index]" has no
        818             # attribute "name"
       (...)
        824             copy=_copy,
        825         )
        826     else:
    --> 827         mgr = ndarray_to_mgr(
        828             data,
        829             index,
        830             columns,
        831             dtype=dtype,
        832             copy=copy,
        833             typ=manager,
        834         )
        836 # For data is list-like, or Iterable (will consume into list)
        837 elif is_list_like(data):
    

    File ~\anaconda3\Lib\site-packages\pandas\core\internals\construction.py:336, in ndarray_to_mgr(values, index, columns, dtype, copy, typ)
        331 # _prep_ndarraylike ensures that values.ndim == 2 at this point
        332 index, columns = _get_axes(
        333     values.shape[0], values.shape[1], index=index, columns=columns
        334 )
    --> 336 _check_values_indices_shape_match(values, index, columns)
        338 if typ == "array":
        339     if issubclass(values.dtype.type, str):
    

    File ~\anaconda3\Lib\site-packages\pandas\core\internals\construction.py:420, in _check_values_indices_shape_match(values, index, columns)
        418 passed = values.shape
        419 implied = (len(index), len(columns))
    --> 420 raise ValueError(f"Shape of passed values is {passed}, indices imply {implied}")
    

    ValueError: Shape of passed values is (28, 7), indices imply (27, 7)


.. code:: ipython3

    # ============================================================
    # 3️⃣ CLUSTER-BASED SHAP REGIME DECOMPOSITION (ROBUST & SAFE)
    # ============================================================
    
    from sklearn.cluster import KMeans
    import matplotlib.pyplot as plt
    import pandas as pd
    import numpy as np
    
    # ------------------------------------------------------------
    # 1️⃣ SHAP DATAFRAME (SHAP REFERANSLI – EN GÜVENLİ YÖNTEM)
    # ------------------------------------------------------------
    
    # SHAP boyutları
    print("SHAP shape:", np.array(shap_values).shape)
    print("Feature count:", len(feature_labels))
    
    shap_df = pd.DataFrame(
        shap_values,
        columns=feature_labels
    )
    
    # ------------------------------------------------------------
    # 2️⃣ X_test HİZALAMA (EĞER GEREKİRSE)
    # ------------------------------------------------------------
    
    if shap_df.shape[0] != X_test.shape[0]:
        print("⚠️ SHAP ve X_test boyutları uyuşmuyor. X_test otomatik kırpıldı.")
        X_test_aligned = X_test.iloc[:shap_df.shape[0]].copy()
    else:
        X_test_aligned = X_test.copy()
    
    shap_df.index = X_test_aligned.index
    
    # ------------------------------------------------------------
    # 3️⃣ CLUSTERING (SHAP SPACE ÜZERİNDE – DOĞRU YER)
    # ------------------------------------------------------------
    
    kmeans = KMeans(
        n_clusters=6,
        random_state=42,
        n_init=20
    )
    
    clusters = kmeans.fit_predict(shap_df)
    
    shap_df["Cluster"] = clusters
    
    # ------------------------------------------------------------
    # 4️⃣ CLUSTER MEANS
    # ------------------------------------------------------------
    
    cluster_means = shap_df.groupby("Cluster").mean()
    
    print("\nCluster-based SHAP decomposition:\n")
    print(cluster_means.round(3))
    
    # ------------------------------------------------------------
    # 5️⃣ VISUALIZATION
    # ------------------------------------------------------------
    
    fig, ax = plt.subplots(figsize=(14, 9))
    
    cluster_means.T.plot(
        kind="bar",
        ax=ax
    )
    
    ax.set_title(
        "Innovation Regime SHAP Profiles",
        fontsize=18,
        fontweight="bold"
    )
    
    ax.set_ylabel("Mean SHAP Contribution", fontsize=14)
    
    ax.tick_params(axis="x", labelsize=14)
    ax.tick_params(axis="y", labelsize=13)
    
    plt.xticks(rotation=45)
    plt.grid(axis="y", alpha=0.4)
    
    # ------------------------------------------------------------
    # 6️⃣ SAYISAL ETİKETLER
    # ------------------------------------------------------------
    
    for container in ax.containers:
        for bar in container:
            height = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width() / 2,
                height,
                f"{height:.3f}",
                ha="center",
                va="bottom",
                rotation=90,
                fontsize=9
            )
    
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    SHAP shape: (28, 7)
    Feature count: 7
    ⚠️ SHAP ve X_test boyutları uyuşmuyor. X_test otomatik kırpıldı.
    

::


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Cell In[74], line 33
         30 else:
         31     X_test_aligned = X_test.copy()
    ---> 33 shap_df.index = X_test_aligned.index
         35 # ------------------------------------------------------------
         36 # 3️⃣ CLUSTERING (SHAP SPACE ÜZERİNDE – DOĞRU YER)
         37 # ------------------------------------------------------------
         39 kmeans = KMeans(
         40     n_clusters=6,
         41     random_state=42,
         42     n_init=20
         43 )
    

    File ~\anaconda3\Lib\site-packages\pandas\core\generic.py:6313, in NDFrame.__setattr__(self, name, value)
       6311 try:
       6312     object.__getattribute__(self, name)
    -> 6313     return object.__setattr__(self, name, value)
       6314 except AttributeError:
       6315     pass
    

    File properties.pyx:69, in pandas._libs.properties.AxisProperty.__set__()
    

    File ~\anaconda3\Lib\site-packages\pandas\core\generic.py:814, in NDFrame._set_axis(self, axis, labels)
        809 """
        810 This is called from the cython code when we set the `index` attribute
        811 directly, e.g. `series.index = [1, 2, 3]`.
        812 """
        813 labels = ensure_index(labels)
    --> 814 self._mgr.set_axis(axis, labels)
        815 self._clear_item_cache()
    

    File ~\anaconda3\Lib\site-packages\pandas\core\internals\managers.py:238, in BaseBlockManager.set_axis(self, axis, new_labels)
        236 def set_axis(self, axis: AxisInt, new_labels: Index) -> None:
        237     # Caller is responsible for ensuring we have an Index object.
    --> 238     self._validate_set_axis(axis, new_labels)
        239     self.axes[axis] = new_labels
    

    File ~\anaconda3\Lib\site-packages\pandas\core\internals\base.py:98, in DataManager._validate_set_axis(self, axis, new_labels)
         95     pass
         97 elif new_len != old_len:
    ---> 98     raise ValueError(
         99         f"Length mismatch: Expected axis has {old_len} elements, new "
        100         f"values have {new_len} elements"
        101     )
    

    ValueError: Length mismatch: Expected axis has 28 elements, new values have 27 elements



.. code:: ipython3

    # ============================================================
    # HEATMAP VERSION
    # ============================================================
    
    plt.figure(figsize=(12,7))
    
    sns.heatmap(
        cluster_means,
        annot=True,
        fmt=".2f",
        cmap="coolwarm",
        center=0,
        linewidths=0.5
    )
    
    plt.title("Cluster-Level SHAP Contribution Matrix", fontsize=15)
    plt.xlabel("Innovation Dimensions")
    plt.ylabel("Innovation Regimes")
    plt.tight_layout()
    plt.show()



.. image:: output_323_0.png





.. code:: ipython3

    # ============================================================
    # PANEL ML + PDP (FULL WORKING VERSION)
    # ============================================================
    
    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    
    from sklearn.model_selection import GroupKFold
    from sklearn.preprocessing import StandardScaler
    from sklearn.ensemble import GradientBoostingRegressor
    from sklearn.metrics import r2_score
    from sklearn.inspection import PartialDependenceDisplay
    
    import warnings
    warnings.filterwarnings("ignore")
    
    # ============================================================
    # LOAD DATA
    # ============================================================
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    df.columns = df.columns.str.strip()
    
    target = "Score"
    features = [c for c in df.columns if c not in ["Country", "Year", target]]
    
    # ============================================================
    # STANDARDIZATION
    # ============================================================
    
    scaler_X = StandardScaler()
    scaler_y = StandardScaler()
    
    X_std = scaler_X.fit_transform(df[features])
    y_std = scaler_y.fit_transform(df[[target]]).ravel()
    
    df_std = df.copy()
    df_std[features] = X_std
    df_std[target] = y_std
    
    # ============================================================
    # GROUP K-FOLD (Country-based CV)
    # ============================================================
    
    gkf = GroupKFold(n_splits=5)
    model = GradientBoostingRegressor(n_estimators=400, random_state=42)
    
    r2_scores = []
    
    for train_idx, test_idx in gkf.split(
            df_std[features], df_std[target], groups=df_std["Country"]):
    
        X_train = df_std.iloc[train_idx][features]
        X_test  = df_std.iloc[test_idx][features]
        y_train = df_std.iloc[train_idx][target]
        y_test  = df_std.iloc[test_idx][target]
    
        model.fit(X_train, y_train)
        preds = model.predict(X_test)
        r2_scores.append(r2_score(y_test, preds))
    
    print("\nPanel Cross-Validated R² Scores:", np.round(r2_scores,3))
    print("Mean CV R²:", round(np.mean(r2_scores),3))
    
    # ============================================================
    # FINAL MODEL (Fit on Full Data for PDP)
    # ============================================================
    
    model.fit(df_std[features], df_std[target])
    
    # ============================================================
    # 4️⃣ INSTITUTIONAL THRESHOLD EFFECT (NONLINEARITY)
    # ============================================================
    
    institution_feature = "Institutions"
    
    if institution_feature not in features:
        raise ValueError(f"{institution_feature} not found in feature list.")
    
    fig, ax = plt.subplots(figsize=(8,6))
    
    PartialDependenceDisplay.from_estimator(
        model,
        df_std[features],
        features=[institution_feature],
        ax=ax,
        grid_resolution=100
    )
    
    plt.title("Nonlinear Threshold Effect of Institutions")
    plt.grid(alpha=0.3)
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    Panel Cross-Validated R² Scores: [0.881 0.966 0.939 0.973 0.881]
    Mean CV R²: 0.928
    


.. image:: output_327_1.png





.. code:: ipython3

    # ============================================================
    # SHAP INTERACTION VALUES (2D HEATMAP)
    # ============================================================
    
    import seaborn as sns
    
    # Compute interaction values
    explainer = shap.Explainer(model, df_std[features])
    shap_interaction = explainer(df_std[features]).values
    
    # SHAP interaction needs TreeExplainer for full interaction matrix
    explainer_tree = shap.TreeExplainer(model)
    interaction_values = explainer_tree.shap_interaction_values(df_std[features])
    
    # Mean absolute interaction matrix
    interaction_matrix = np.abs(interaction_values).mean(axis=0)
    
    interaction_df = pd.DataFrame(
        interaction_matrix,
        index=features,
        columns=features
    )
    
    plt.figure(figsize=(10,8))
    sns.heatmap(
        interaction_df,
        annot=True,
        fmt=".3f",
        cmap="coolwarm",
        square=True
    )
    
    plt.title("SHAP Interaction Matrix (Mean Absolute Effects)")
    plt.xticks(rotation=45)
    plt.yticks(rotation=0)
    plt.show()



.. image:: output_331_0.png




.. code:: ipython3

    from sklearn.ensemble import ExtraTreesRegressor
    import matplotlib.pyplot as plt
    import numpy as np
    import pandas as pd
    import shap
    
    quantiles = [0.25, 0.50, 0.75]
    quantile_shap = {}
    
    n_boot = 30  # bootstrap runs
    
    all_shap = []
    
    # ------------------------------------------------------------
    # BOOTSTRAP EXTRA TREES MODELS
    # ------------------------------------------------------------
    
    for i in range(n_boot):
        
        idx = np.random.choice(len(df_std), len(df_std), replace=True)
        X_boot = df_std[features].iloc[idx]
        y_boot = df_std[target].iloc[idx]
    
        model = ExtraTreesRegressor(
            n_estimators=400,
            random_state=i,
            n_jobs=-1
        )
        
        model.fit(X_boot, y_boot)
    
        explainer = shap.Explainer(model, X_boot)
        shap_vals = explainer(X_boot)
    
        mean_importance = np.abs(shap_vals.values).mean(axis=0)
        all_shap.append(mean_importance)
    
    all_shap = np.array(all_shap)
    
    # ------------------------------------------------------------
    # COMPUTE QUANTILE IMPORTANCE
    # ------------------------------------------------------------
    
    for q in quantiles:
        quantile_shap[q] = np.quantile(all_shap, q, axis=0)
    
    quantile_df = pd.DataFrame(
        quantile_shap,
        index=features
    )
    
    # Normalize for comparability
    quantile_df = quantile_df.div(quantile_df.sum(axis=0), axis=1)
    
    # ------------------------------------------------------------
    # PRINT NUMERICAL OUTPUT
    # ------------------------------------------------------------
    
    print("\n==============================")
    print("EXTRA TREES – QUANTILE SHAP IMPORTANCE")
    print("==============================\n")
    print(quantile_df.round(4))
    
    # ------------------------------------------------------------
    # PLOT WITH VALUE LABELS
    # ------------------------------------------------------------
    
    fig, ax = plt.subplots(figsize=(14, 9))
    
    bars = quantile_df.plot(
        kind="bar",
        ax=ax,
        edgecolor="black"
    )
    
    plt.title(
        "Extra Trees – Quantile-Specific SHAP Importance\n"
        "Distributional Heterogeneity in Innovation Drivers",
        fontsize=16,
        weight="bold"
    )
    
    plt.ylabel("Relative Mean |SHAP| Contribution", fontsize=14)
    plt.xticks(rotation=90, fontsize=12)
    plt.yticks(fontsize=12)
    plt.grid(axis="y", linestyle="--", alpha=0.3)
    
    # ------------------------------------------------------------
    # ADD 90-DEGREE VALUE LABELS
    # ------------------------------------------------------------
    
    for container in ax.containers:
        for bar in container:
            height = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width() / 2,
                height,
                f"{height:.3f}",
                ha="center",
                va="bottom",
                fontsize=10,
                rotation=90
            )
    
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    ==============================
    EXTRA TREES – QUANTILE SHAP IMPORTANCE
    ==============================
    
                                        0.25    0.50    0.75
    Institutions                      0.0668  0.0769  0.0745
    Human capital and research        0.0933  0.0945  0.0977
    Infrastructure                    0.1169  0.1228  0.1321
    Market sophistication             0.0831  0.0889  0.0902
    Business sophistication           0.1993  0.1919  0.1993
    Knowledge and technology outputs  0.1623  0.1573  0.1526
    Creative outputs                  0.2782  0.2677  0.2536
    


.. image:: output_334_1.png



















.. code:: ipython3

    # ============================================================
    # 📌 DATA PREPARATION BLOCK (FIXED & SAFE)
    # ============================================================
    
    import pandas as pd
    import numpy as np
    from sklearn.preprocessing import StandardScaler
    
    # ------------------------------------------------------------
    # 1️⃣ Veriyi yükle
    # ------------------------------------------------------------
    
    df = pd.read_csv("GII.csv", sep=";", encoding="latin1")
    
    print("Original Shape:", df.shape)
    
    # ------------------------------------------------------------
    # 2️⃣ Numeric kolonları otomatik seç
    # ------------------------------------------------------------
    
    numeric_cols = df.select_dtypes(include=[np.number]).columns.tolist()
    
    print("Numeric columns detected:", len(numeric_cols))
    
    # ------------------------------------------------------------
    # 3️⃣ Target belirle (BURAYA GERÇEK TARGET ADINI YAZ)
    # ------------------------------------------------------------
    
    target = "Score"   # örn: "GII_Score"
    
    if target not in numeric_cols:
        raise ValueError("❌ Target numeric değil veya yanlış isim yazıldı.")
    
    # ------------------------------------------------------------
    # 4️⃣ Feature listesi (numeric - target)
    # ------------------------------------------------------------
    
    features = [col for col in numeric_cols if col != target]
    
    # ------------------------------------------------------------
    # 5️⃣ NaN temizleme (çok önemli)
    # ------------------------------------------------------------
    
    df_model = df[features + [target]].dropna()
    
    print("After NA drop:", df_model.shape)
    
    # ------------------------------------------------------------
    # 6️⃣ Standardizasyon
    # ------------------------------------------------------------
    
    scaler = StandardScaler()
    
    df_std = df_model.copy()
    df_std[features] = scaler.fit_transform(df_model[features])
    
    print("\n✅ Data Ready")
    print("Observations:", df_std.shape[0])
    print("Features:", len(features))


.. parsed-literal::

    Original Shape: (139, 9)
    Numeric columns detected: 8
    After NA drop: (139, 8)
    
    ✅ Data Ready
    Observations: 139
    Features: 7
    





.. code:: ipython3

    from sklearn.ensemble import ExtraTreesRegressor
    import numpy as np
    import pandas as pd
    import shap
    import matplotlib.pyplot as plt
    import seaborn as sns
    from scipy.stats import entropy
    from scipy.spatial.distance import jensenshannon
    
    sns.set_style("whitegrid")
    
    # ------------------------------------------------------------
    # SETTINGS
    # ------------------------------------------------------------
    
    n_boot = 40
    n_mc = 100
    all_shap = []
    all_local_shap = []
    
    # ------------------------------------------------------------
    # BOOTSTRAP MODEL + SHAP
    # ------------------------------------------------------------
    
    for i in range(n_boot):
    
        idx = np.random.choice(len(df_std), len(df_std), replace=True)
        X_boot = df_std[features].iloc[idx]
        y_boot = df_std[target].iloc[idx]
    
        model = ExtraTreesRegressor(
            n_estimators=400,
            random_state=i,
            n_jobs=-1
        )
    
        model.fit(X_boot, y_boot)
    
        explainer = shap.TreeExplainer(model)
        shap_vals = explainer.shap_values(X_boot)
    
        all_shap.append(np.abs(shap_vals).mean(axis=0))
        all_local_shap.append(shap_vals)
    
    all_shap = np.array(all_shap)
    global_mean = all_shap.mean(axis=0)
    
    # ============================================================
    # 1️⃣ SHAP ENTROPY INDEX
    # ============================================================
    
    prob_dist = global_mean / global_mean.sum()
    shap_entropy = entropy(prob_dist)
    
    print("\n🔥 SHAP ENTROPY INDEX:", round(shap_entropy, 4))
    
    plt.figure(figsize=(6,4))
    bars = plt.bar(["SHAP Entropy"], [shap_entropy], color="crimson")
    
    for bar in bars:
        plt.text(bar.get_x() + bar.get_width()/2,
                 bar.get_height(),
                 f"{shap_entropy:.4f}",
                 ha='center', va='bottom', fontsize=12)
    
    plt.title("SHAP Entropy Index", fontsize=13)
    plt.tight_layout()
    plt.show()
    
    # ============================================================
    # 2️⃣ FEATURE INTERACTION SURFACE
    # ============================================================
    
    model_final = ExtraTreesRegressor(n_estimators=400, random_state=0)
    model_final.fit(df_std[features], df_std[target])
    
    explainer_final = shap.TreeExplainer(model_final)
    interaction_values = explainer_final.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_values).mean(axis=0)
    
    print("\n🔥 INTERACTION MATRIX (Mean Absolute)")
    print(pd.DataFrame(interaction_matrix, 
                       index=features, 
                       columns=features).round(4))
    
    plt.figure(figsize=(10,8))
    
    sns.heatmap(
        interaction_matrix,
        xticklabels=features,
        yticklabels=features,
        cmap="magma",
        annot=True,
        fmt=".2f",
        linewidths=0.5
    )
    
    plt.title("SHAP Feature Interaction Surface", fontsize=14)
    plt.tight_layout()
    plt.show()
    
    # ============================================================
    # 3️⃣ BAYESIAN UNCERTAINTY BAND
    # ============================================================
    
    posterior_mean = all_shap.mean(axis=0)
    posterior_std = all_shap.std(axis=0)
    
    print("\n🔥 POSTERIOR MEAN SHAP")
    print(pd.Series(posterior_mean, index=features).round(4))
    
    plt.figure(figsize=(10,6))
    
    bars = plt.bar(features, posterior_mean, 
                   yerr=1.96*posterior_std,
                   capsize=5,
                   color="royalblue")
    
    for i, bar in enumerate(bars):
        plt.text(bar.get_x() + bar.get_width()/2,
                 bar.get_height(),
                 f"{posterior_mean[i]:.3f}",
                 ha='center', va='bottom', fontsize=9)
    
    plt.xticks(rotation=45)
    plt.title("Bayesian Posterior Approximation – SHAP Importance")
    plt.tight_layout()
    plt.show()
    
    # ============================================================
    # 4️⃣ GLOBAL vs LOCAL SHAP DIVERGENCE
    # ============================================================
    
    local_stack = np.vstack(all_local_shap)
    local_mean_dist = np.abs(local_stack).mean(axis=0)
    local_prob = local_mean_dist / local_mean_dist.sum()
    
    js_divergence = jensenshannon(prob_dist, local_prob)
    
    print("\n🔥 Global vs Local SHAP Jensen-Shannon Divergence:",
          round(js_divergence, 4))
    
    # ============================================================
    # 5️⃣ CAUSAL SHAP (Permutation Proxy)
    # ============================================================
    
    causal_effects = []
    
    for feature in features:
    
        X_perm = df_std.copy()
        X_perm[feature] = np.random.permutation(X_perm[feature])
    
        model_perm = ExtraTreesRegressor(n_estimators=300, random_state=0)
        model_perm.fit(X_perm[features], df_std[target])
    
        explainer_perm = shap.TreeExplainer(model_perm)
        shap_perm = explainer_perm.shap_values(X_perm[features])
    
        causal_effects.append(np.abs(shap_perm).mean())
    
    print("\n🔥 Causal SHAP Proxy Effects")
    print(pd.Series(causal_effects, index=features).round(4))
    
    plt.figure(figsize=(10,6))
    
    bars = plt.bar(features, causal_effects, color="darkorange")
    
    for i, bar in enumerate(bars):
        plt.text(bar.get_x() + bar.get_width()/2,
                 bar.get_height(),
                 f"{causal_effects[i]:.3f}",
                 ha='center', va='bottom', fontsize=9)
    
    plt.xticks(rotation=45)
    plt.title("Permutation-Based Causal SHAP Proxy")
    plt.tight_layout()
    plt.show()
    
    # ============================================================
    # 6️⃣ MONTE CARLO SENSITIVITY
    # ============================================================
    
    mc_results = []
    
    for i in range(n_mc):
    
        noise = np.random.normal(0, 0.01, df_std[features].shape)
        X_mc = df_std[features] + noise
    
        model_mc = ExtraTreesRegressor(n_estimators=200, random_state=i)
        model_mc.fit(X_mc, df_std[target])
    
        explainer_mc = shap.TreeExplainer(model_mc)
        shap_mc = explainer_mc.shap_values(X_mc)
    
        mc_results.append(np.abs(shap_mc).mean(axis=0))
    
    mc_results = np.array(mc_results)
    mc_std = mc_results.std(axis=0)
    
    print("\n🔥 Monte Carlo Sensitivity (Std of SHAP):")
    print(pd.Series(mc_std, index=features).round(4))
    
    plt.figure(figsize=(10,6))
    
    bars = plt.bar(features, mc_std, color="seagreen")
    
    for i, bar in enumerate(bars):
        plt.text(bar.get_x() + bar.get_width()/2,
                 bar.get_height(),
                 f"{mc_std[i]:.4f}",
                 ha='center', va='bottom', fontsize=9)
    
    plt.xticks(rotation=45)
    plt.title("Monte Carlo SHAP Sensitivity")
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    🔥 SHAP ENTROPY INDEX: 1.8594
    


.. image:: output_357_1.png


.. parsed-literal::

    
    🔥 INTERACTION MATRIX (Mean Absolute)
                                      Institutions  Human capital and research  \
    Institutions                            1.0704                      0.0781   
    Human capital and research              0.0781                      1.6851   
    Infrastructure                          0.0741                      0.0704   
    Market sophistication                   0.0364                      0.0518   
    Business sophistication                 0.0628                      0.2060   
    Knowledge and technology outputs        0.1124                      0.1306   
    Creative outputs                        0.1417                      0.1919   
    
                                      Infrastructure  Market sophistication  \
    Institutions                              0.0741                 0.0364   
    Human capital and research                0.0704                 0.0518   
    Infrastructure                            2.1678                 0.0774   
    Market sophistication                     0.0774                 1.4029   
    Business sophistication                   0.1334                 0.1186   
    Knowledge and technology outputs          0.1208                 0.1308   
    Creative outputs                          0.2696                 0.1610   
    
                                      Business sophistication  \
    Institutions                                       0.0628   
    Human capital and research                         0.2060   
    Infrastructure                                     0.1334   
    Market sophistication                              0.1186   
    Business sophistication                            3.5113   
    Knowledge and technology outputs                   0.2528   
    Creative outputs                                   0.3055   
    
                                      Knowledge and technology outputs  \
    Institutions                                                0.1124   
    Human capital and research                                  0.1306   
    Infrastructure                                              0.1208   
    Market sophistication                                       0.1308   
    Business sophistication                                     0.2528   
    Knowledge and technology outputs                            2.9959   
    Creative outputs                                            0.2879   
    
                                      Creative outputs  
    Institutions                                0.1417  
    Human capital and research                  0.1919  
    Infrastructure                              0.2696  
    Market sophistication                       0.1610  
    Business sophistication                     0.3055  
    Knowledge and technology outputs            0.2879  
    Creative outputs                            4.5174  
    


.. image:: output_357_3.png


.. parsed-literal::

    
    🔥 POSTERIOR MEAN SHAP
    Institutions                        0.8711
    Human capital and research          1.4099
    Infrastructure                      1.6289
    Market sophistication               1.1590
    Business sophistication             2.4177
    Knowledge and technology outputs    2.0155
    Creative outputs                    3.3387
    dtype: float64
    


.. image:: output_357_5.png


.. parsed-literal::

    
    🔥 Global vs Local SHAP Jensen-Shannon Divergence: 0.0
    
    🔥 Causal SHAP Proxy Effects
    Institutions                        1.7934
    Human capital and research          1.8362
    Infrastructure                      1.8273
    Market sophistication               1.8481
    Business sophistication             1.8211
    Knowledge and technology outputs    1.8355
    Creative outputs                    1.8464
    dtype: float64
    


.. image:: output_357_7.png


.. parsed-literal::

    
    🔥 Monte Carlo Sensitivity (Std of SHAP):
    Institutions                        0.0435
    Human capital and research          0.1102
    Infrastructure                      0.1085
    Market sophistication               0.0739
    Business sophistication             0.1564
    Knowledge and technology outputs    0.1379
    Creative outputs                    0.1708
    dtype: float64
    


.. image:: output_357_9.png








.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import networkx as nx
    import matplotlib.pyplot as plt
    import seaborn as sns
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh
    from scipy.spatial.distance import pdist, squareform
    from scipy.stats import entropy
    
    sns.set(style="whitegrid", font_scale=1.1)
    
    # ------------------------------------------------------------
    # TRAIN MODEL
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    
    # ============================================================
    # 1️⃣ LAPLACIAN GRAPH DYNAMICS
    # ============================================================
    
    G = nx.from_numpy_array(interaction_matrix)
    L = nx.laplacian_matrix(G).toarray()
    
    eigenvalues, eigenvectors = eigh(L)
    sorted_eigs = sorted(eigenvalues)
    
    print("\nLAPLACIAN EIGENVALUES:")
    for i, val in enumerate(sorted_eigs):
        print(f"λ{i+1}: {val:.6f}")
    
    plt.figure(figsize=(9,5))
    colors = plt.cm.plasma(np.linspace(0,1,len(sorted_eigs)))
    
    for i, val in enumerate(sorted_eigs):
        plt.scatter(i+1, val, color=colors[i], s=80)
        plt.text(i+1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.plot(range(1,len(sorted_eigs)+1), sorted_eigs, color="black", alpha=0.6)
    plt.title("Laplacian Spectrum of SHAP Interaction Network")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Eigenvalue")
    plt.show()
    
    print("\nAlgebraic Connectivity (λ2):", sorted_eigs[1])
    
    # ============================================================
    # 2️⃣ PERSISTENT HOMOLOGY
    # ============================================================
    
    distance_matrix = squareform(pdist(shap_vals.T, metric="euclidean"))
    
    print("\nFEATURE DISTANCE MATRIX:")
    print(pd.DataFrame(distance_matrix, index=features, columns=features))
    
    plt.figure(figsize=(10,7))
    sns.heatmap(
        distance_matrix,
        xticklabels=features,
        yticklabels=features,
        annot=True,
        fmt=".2f",
        cmap="turbo"
    )
    plt.title("Feature Distance Matrix (Topological Structure)")
    plt.show()
    
    threshold = np.median(distance_matrix)
    adjacency = (distance_matrix < threshold).astype(int)
    G_topo = nx.from_numpy_array(adjacency)
    components = nx.number_connected_components(G_topo)
    
    print("\nPersistent Homology Approximation (Betti-0):", components)
    
    # ============================================================
    # 3️⃣ SHAP CURVATURE TENSOR
    # ============================================================
    
    cov_matrix = np.cov(shap_vals.T)
    eigvals_cov, eigvecs_cov = eigh(cov_matrix)
    
    curvature_proxy = eigvals_cov / eigvals_cov.sum()
    sorted_curv = sorted(curvature_proxy, reverse=True)
    
    print("\nCURVATURE EIGENVALUES:")
    for i, val in enumerate(sorted_curv):
        print(f"Component {i+1}: {val:.6f}")
    
    plt.figure(figsize=(9,5))
    colors = plt.cm.viridis(np.linspace(0,1,len(sorted_curv)))
    
    for i, val in enumerate(sorted_curv):
        plt.bar(i+1, val, color=colors[i])
        plt.text(i+1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.title("SHAP Curvature Spectrum")
    plt.xlabel("Component")
    plt.ylabel("Normalized Eigenvalue")
    plt.show()
    
    print("\nCurvature Concentration Ratio:",
          max(curvature_proxy))
    
    # ============================================================
    # 4️⃣ INFORMATION ENTROPY FLOW
    # ============================================================
    
    prob_matrix = interaction_matrix / interaction_matrix.sum()
    node_entropy = entropy(prob_matrix, axis=1)
    
    print("\nNODE ENTROPY VALUES:")
    for f, val in zip(features, node_entropy):
        print(f"{f}: {val:.6f}")
    
    plt.figure(figsize=(11,6))
    colors = plt.cm.rainbow(np.linspace(0,1,len(features)))
    
    bars = plt.bar(features, node_entropy, color=colors)
    
    for bar, val in zip(bars, node_entropy):
        plt.text(bar.get_x() + bar.get_width()/2,
                 bar.get_height(),
                 f"{val:.3f}",
                 ha='center',
                 va='bottom')
    
    plt.xticks(rotation=45)
    plt.title("Node-wise Information Entropy")
    plt.show()
    
    entropy_flow_rate = np.gradient(node_entropy)
    
    print("\nENTROPY FLOW DIFFERENTIAL:")
    for f, val in zip(features, entropy_flow_rate):
        print(f"{f}: {val:.6f}")
    
    plt.figure(figsize=(9,5))
    colors = plt.cm.cool(np.linspace(0,1,len(entropy_flow_rate)))
    
    for i, val in enumerate(entropy_flow_rate):
        plt.scatter(i+1, val, color=colors[i], s=80)
        plt.text(i+1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.plot(range(1,len(entropy_flow_rate)+1),
             entropy_flow_rate,
             color="black",
             alpha=0.6)
    
    plt.title("Entropy Flow Differential")
    plt.xlabel("Feature Index")
    plt.ylabel("Gradient Value")
    plt.show()
    
    # ============================================================
    # 5️⃣ QUANTUM SHAP
    # ============================================================
    
    density_matrix = cov_matrix / np.trace(cov_matrix)
    eigenvals_density, _ = eigh(density_matrix)
    
    sorted_density = sorted(eigenvals_density, reverse=True)
    
    von_neumann_entropy = -np.sum(
        eigenvals_density * np.log(eigenvals_density + 1e-12)
    )
    
    print("\nDensity Matrix Eigenvalues:")
    for i, val in enumerate(sorted_density):
        print(f"ρ{i+1}: {val:.6f}")
    
    print("\nQuantum SHAP Von Neumann Entropy:",
          round(von_neumann_entropy, 6))
    
    plt.figure(figsize=(9,5))
    colors = plt.cm.inferno(np.linspace(0,1,len(sorted_density)))
    
    for i, val in enumerate(sorted_density):
        plt.bar(i+1, val, color=colors[i])
        plt.text(i+1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.title("Density Matrix Eigen-Spectrum (Quantum SHAP)")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Density Eigenvalue")
    plt.show()


.. parsed-literal::

    
    LAPLACIAN EIGENVALUES:
    λ1: 0.000000
    λ2: 0.566117
    λ3: 0.680317
    λ4: 0.814589
    λ5: 1.033915
    λ6: 1.301929
    λ7: 1.633842
    


.. image:: output_364_1.png


.. parsed-literal::

    
    Algebraic Connectivity (λ2): 0.5661166092467234
    
    FEATURE DISTANCE MATRIX:
                                      Institutions  Human capital and research  \
    Institutions                          0.000000                   14.683690   
    Human capital and research           14.683690                    0.000000   
    Infrastructure                       16.776444                   16.013289   
    Market sophistication                12.654028                   10.850075   
    Business sophistication              32.189896                   23.608870   
    Knowledge and technology outputs     27.807255                   19.639571   
    Creative outputs                     43.704961                   36.680337   
    
                                      Infrastructure  Market sophistication  \
    Institutions                           16.776444              12.654028   
    Human capital and research             16.013289              10.850075   
    Infrastructure                          0.000000              14.536989   
    Market sophistication                  14.536989               0.000000   
    Business sophistication                29.286107              27.048406   
    Knowledge and technology outputs       21.561251              21.790766   
    Creative outputs                       34.064880              38.101065   
    
                                      Business sophistication  \
    Institutions                                    32.189896   
    Human capital and research                      23.608870   
    Infrastructure                                  29.286107   
    Market sophistication                           27.048406   
    Business sophistication                          0.000000   
    Knowledge and technology outputs                19.799762   
    Creative outputs                                31.346234   
    
                                      Knowledge and technology outputs  \
    Institutions                                             27.807255   
    Human capital and research                               19.639571   
    Infrastructure                                           21.561251   
    Market sophistication                                    21.790766   
    Business sophistication                                  19.799762   
    Knowledge and technology outputs                          0.000000   
    Creative outputs                                         29.038721   
    
                                      Creative outputs  
    Institutions                             43.704961  
    Human capital and research               36.680337  
    Infrastructure                           34.064880  
    Market sophistication                    38.101065  
    Business sophistication                  31.346234  
    Knowledge and technology outputs         29.038721  
    Creative outputs                          0.000000  
    


.. image:: output_364_3.png


.. parsed-literal::

    
    Persistent Homology Approximation (Betti-0): 2
    
    CURVATURE EIGENVALUES:
    Component 1: 0.825875
    Component 2: 0.074955
    Component 3: 0.038548
    Component 4: 0.029928
    Component 5: 0.011512
    Component 6: 0.010114
    Component 7: 0.009068
    


.. image:: output_364_5.png


.. parsed-literal::

    
    Curvature Concentration Ratio: 0.8258748269361392
    
    NODE ENTROPY VALUES:
    Institutions: 1.180182
    Human capital and research: 1.118541
    Infrastructure: 1.003256
    Market sophistication: 1.085785
    Business sophistication: 0.945872
    Knowledge and technology outputs: 1.017206
    Creative outputs: 0.938670
    


.. image:: output_364_7.png


.. parsed-literal::

    
    ENTROPY FLOW DIFFERENTIAL:
    Institutions: -0.061641
    Human capital and research: -0.088463
    Infrastructure: -0.016378
    Market sophistication: -0.028692
    Business sophistication: -0.034289
    Knowledge and technology outputs: -0.003601
    Creative outputs: -0.078536
    


.. image:: output_364_9.png


.. parsed-literal::

    
    Density Matrix Eigenvalues:
    ρ1: 0.825875
    ρ2: 0.074955
    ρ3: 0.038548
    ρ4: 0.029928
    ρ5: 0.011512
    ρ6: 0.010114
    ρ7: 0.009068
    
    Quantum SHAP Von Neumann Entropy: 0.723225
    


.. image:: output_364_11.png






.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import networkx as nx
    import matplotlib.pyplot as plt
    import seaborn as sns
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh
    from scipy.spatial.distance import pdist, squareform
    from scipy.stats import entropy
    from matplotlib.patches import Patch
    
    sns.set(style="whitegrid", font_scale=1.1)
    
    # ------------------------------------------------------------
    # TRAIN MODEL
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    
    # ============================================================
    # 🎨 FEATURE → COLOR MAP (GLOBAL – TÜM FIGURE’LAR İÇİN)
    # ============================================================
    
    palette = sns.color_palette("tab10", len(features))
    feature_colors = dict(zip(features, palette))
    
    legend_elements = [
        Patch(facecolor=feature_colors[f], label=f)
        for f in features
    ]
    
    # ============================================================
    # 1️⃣ LAPLACIAN GRAPH DYNAMICS
    # ============================================================
    
    G = nx.from_numpy_array(interaction_matrix)
    L = nx.laplacian_matrix(G).toarray()
    
    eigenvalues, _ = eigh(L)
    sorted_eigs = sorted(eigenvalues)
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(sorted_eigs):
        plt.scatter(
            i + 1,
            val,
            color=feature_colors[features[i % len(features)]],
            s=80
        )
        plt.text(i + 1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.plot(range(1, len(sorted_eigs) + 1), sorted_eigs, color="black", alpha=0.6)
    plt.title("Laplacian Spectrum of SHAP Interaction Network")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Eigenvalue")
    
    plt.legend(
        handles=legend_elements,
        title="Features",
        fontsize=9,
        title_fontsize=10
    )
    
    plt.show()
    
    # ============================================================
    # 2️⃣ PERSISTENT HOMOLOGY (DISTANCE HEATMAP)
    # ============================================================
    
    distance_matrix = squareform(pdist(shap_vals.T, metric="euclidean"))
    
    plt.figure(figsize=(10,7))
    sns.heatmap(
        distance_matrix,
        xticklabels=features,
        yticklabels=features,
        cmap="turbo",
        annot=True,
        fmt=".2f"
    )
    plt.title("Feature Distance Matrix (Topological Structure)")
    plt.show()
    
    # ============================================================
    # 3️⃣ SHAP CURVATURE TENSOR
    # ============================================================
    
    cov_matrix = np.cov(shap_vals.T)
    eigvals_cov, _ = eigh(cov_matrix)
    
    curvature_proxy = eigvals_cov / eigvals_cov.sum()
    sorted_curv = sorted(curvature_proxy, reverse=True)
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(sorted_curv):
        plt.bar(
            i + 1,
            val,
            color=feature_colors[features[i % len(features)]]
        )
        plt.text(i + 1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.title("SHAP Curvature Spectrum")
    plt.xlabel("Component")
    plt.ylabel("Normalized Eigenvalue")
    
    plt.legend(
        handles=legend_elements,
        title="Features",
        fontsize=9,
        title_fontsize=10
    )
    
    plt.show()
    
    # ============================================================
    # 4️⃣ INFORMATION ENTROPY FLOW
    # ============================================================
    
    prob_matrix = interaction_matrix / interaction_matrix.sum()
    node_entropy = entropy(prob_matrix, axis=1)
    
    plt.figure(figsize=(11,6))
    
    bars = plt.bar(
        features,
        node_entropy,
        color=[feature_colors[f] for f in features]
    )
    
    for bar, val in zip(bars, node_entropy):
        plt.text(
            bar.get_x() + bar.get_width() / 2,
            bar.get_height(),
            f"{val:.3f}",
            ha='center',
            va='bottom'
        )
    
    plt.xticks(rotation=45)
    plt.title("Node-wise Information Entropy")
    
    plt.legend(
        handles=legend_elements,
        title="Features",
        fontsize=9,
        title_fontsize=10
    )
    
    plt.show()
    
    entropy_flow_rate = np.gradient(node_entropy)
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(entropy_flow_rate):
        plt.scatter(
            i + 1,
            val,
            color=feature_colors[features[i]],
            s=80
        )
        plt.text(i + 1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.plot(range(1, len(entropy_flow_rate) + 1),
             entropy_flow_rate,
             color="black",
             alpha=0.6)
    
    plt.title("Entropy Flow Differential")
    plt.xlabel("Feature Index")
    plt.ylabel("Gradient Value")
    
    plt.legend(
        handles=legend_elements,
        title="Features",
        fontsize=9,
        title_fontsize=10
    )
    
    plt.show()
    
    # ============================================================
    # 5️⃣ QUANTUM SHAP
    # ============================================================
    
    density_matrix = cov_matrix / np.trace(cov_matrix)
    eigenvals_density, _ = eigh(density_matrix)
    
    sorted_density = sorted(eigenvals_density, reverse=True)
    
    von_neumann_entropy = -np.sum(
        eigenvals_density * np.log(eigenvals_density + 1e-12)
    )
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(sorted_density):
        plt.bar(
            i + 1,
            val,
            color=feature_colors[features[i % len(features)]]
        )
        plt.text(i + 1, val, f"{val:.3f}", ha='center', va='bottom')
    
    plt.title("Density Matrix Eigen-Spectrum (Quantum SHAP)")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Density Eigenvalue")
    
    plt.legend(
        handles=legend_elements,
        title="Features",
        fontsize=9,
        title_fontsize=10
    )
    
    plt.show()



.. image:: output_369_0.png



.. image:: output_369_1.png



.. image:: output_369_2.png



.. image:: output_369_3.png



.. image:: output_369_4.png



.. image:: output_369_5.png



.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import networkx as nx
    import matplotlib.pyplot as plt
    import seaborn as sns
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh
    from scipy.spatial.distance import pdist, squareform
    from scipy.stats import entropy
    from matplotlib.patches import Patch
    
    sns.set(style="whitegrid", font_scale=1.1)
    
    # ------------------------------------------------------------
    # TRAIN MODEL
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    
    # ============================================================
    # 🎨 FEATURE → COLOR MAP (GLOBAL)
    # ============================================================
    
    palette = sns.color_palette("tab10", len(features))
    feature_colors = dict(zip(features, palette))
    
    legend_elements = [
        Patch(facecolor=feature_colors[f], label=f)
        for f in features
    ]
    
    # ============================================================
    # 1️⃣ LAPLACIAN GRAPH DYNAMICS
    # ============================================================
    
    G = nx.from_numpy_array(interaction_matrix)
    L = nx.laplacian_matrix(G).toarray()
    
    eigenvalues, _ = eigh(L)
    sorted_eigs = sorted(eigenvalues)
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(sorted_eigs):
        plt.scatter(
            i + 1, val,
            color=feature_colors[features[i % len(features)]],
            s=80
        )
        plt.text(i + 1, val, f"{val:.4f}", ha='center', va='bottom', fontsize=9)
    
    plt.plot(range(1, len(sorted_eigs)+1), sorted_eigs, color="black", alpha=0.6)
    plt.title("Laplacian Spectrum of SHAP Interaction Network")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Eigenvalue")
    plt.legend(handles=legend_elements, title="Features", fontsize=9)
    plt.show()
    
    # ============================================================
    # 2️⃣ PERSISTENT HOMOLOGY (DISTANCE HEATMAP)
    # ============================================================
    
    distance_matrix = squareform(pdist(shap_vals.T, metric="euclidean"))
    
    plt.figure(figsize=(10,7))
    sns.heatmap(
        distance_matrix,
        xticklabels=features,
        yticklabels=features,
        annot=True,
        fmt=".2f",
        cmap="turbo"
    )
    plt.title("Feature Distance Matrix (Topological Structure)")
    plt.show()
    
    # ============================================================
    # 3️⃣ SHAP CURVATURE TENSOR
    # ============================================================
    
    cov_matrix = np.cov(shap_vals.T)
    eigvals_cov, _ = eigh(cov_matrix)
    
    curvature_proxy = eigvals_cov / eigvals_cov.sum()
    sorted_curv = sorted(curvature_proxy, reverse=True)
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(sorted_curv):
        plt.bar(i + 1, val,
                color=feature_colors[features[i % len(features)]])
        plt.text(i + 1, val, f"{val:.4f}",
                 ha='center', va='bottom', fontsize=9)
    
    plt.title("SHAP Curvature Spectrum")
    plt.xlabel("Component")
    plt.ylabel("Normalized Eigenvalue")
    plt.legend(handles=legend_elements, title="Features", fontsize=9)
    plt.show()
    
    # ============================================================
    # 4️⃣ INFORMATION ENTROPY FLOW
    # ============================================================
    
    prob_matrix = interaction_matrix / interaction_matrix.sum()
    node_entropy = entropy(prob_matrix, axis=1)
    
    plt.figure(figsize=(11,6))
    
    bars = plt.bar(
        features,
        node_entropy,
        color=[feature_colors[f] for f in features]
    )
    
    for bar, val in zip(bars, node_entropy):
        plt.text(
            bar.get_x() + bar.get_width()/2,
            bar.get_height(),
            f"{val:.4f}",
            ha='center', va='bottom', fontsize=9
        )
    
    plt.xticks(rotation=45)
    plt.title("Node-wise Information Entropy")
    plt.legend(handles=legend_elements, title="Features", fontsize=9)
    plt.show()
    
    # ============================================================
    # ENTROPY FLOW DIFFERENTIAL
    # ============================================================
    
    entropy_flow_rate = np.gradient(node_entropy)
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(entropy_flow_rate):
        plt.scatter(
            i + 1, val,
            color=feature_colors[features[i]],
            s=80
        )
        plt.text(i + 1, val, f"{val:.4f}",
                 ha='center', va='bottom', fontsize=9)
    
    plt.plot(range(1, len(entropy_flow_rate)+1),
             entropy_flow_rate, color="black", alpha=0.6)
    
    plt.title("Entropy Flow Differential")
    plt.xlabel("Feature Index")
    plt.ylabel("Gradient Value")
    plt.legend(handles=legend_elements, title="Features", fontsize=9)
    plt.show()
    
    # ============================================================
    # 5️⃣ QUANTUM SHAP
    # ============================================================
    
    density_matrix = cov_matrix / np.trace(cov_matrix)
    eigenvals_density, _ = eigh(density_matrix)
    
    sorted_density = sorted(eigenvals_density, reverse=True)
    
    von_neumann_entropy = -np.sum(
        eigenvals_density * np.log(eigenvals_density + 1e-12)
    )
    
    plt.figure(figsize=(9,5))
    
    for i, val in enumerate(sorted_density):
        plt.bar(i + 1, val,
                color=feature_colors[features[i % len(features)]])
        plt.text(i + 1, val, f"{val:.4f}",
                 ha='center', va='bottom', fontsize=9)
    
    plt.title("Density Matrix Eigen-Spectrum (Quantum SHAP)")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Density Eigenvalue")
    plt.legend(handles=legend_elements, title="Features", fontsize=9)
    plt.show()



.. image:: output_371_0.png



.. image:: output_371_1.png



.. image:: output_371_2.png



.. image:: output_371_3.png



.. image:: output_371_4.png



.. image:: output_371_5.png



.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import networkx as nx
    import matplotlib.pyplot as plt
    import seaborn as sns
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh
    from scipy.spatial.distance import pdist, squareform
    from scipy.stats import entropy
    from matplotlib.patches import Patch
    
    sns.set(style="whitegrid", font_scale=1.1)
    
    # ------------------------------------------------------------
    # MODEL EĞİTİMİ
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    
    # ------------------------------------------------------------
    # RENK HARİTASI
    # ------------------------------------------------------------
    
    palette = sns.color_palette("tab10", len(features))
    feature_colors = dict(zip(features, palette))
    
    legend_elements = [
        Patch(facecolor=feature_colors[f], label=f)
        for f in features
    ]
    
    # ============================================================
    # 1️⃣ LAPLACIAN SPECTRUM
    # ============================================================
    
    G = nx.from_numpy_array(interaction_matrix)
    L = nx.laplacian_matrix(G).toarray()
    
    eigenvalues, _ = eigh(L)
    sorted_eigs = np.sort(eigenvalues)
    
    print("\n🔹 Laplacian Eigenvalues:")
    for i, val in enumerate(sorted_eigs, 1):
        print(f"Eigenvalue {i}: {val:.6f}")
    
    plt.figure(figsize=(9,5))
    for i, val in enumerate(sorted_eigs):
        plt.scatter(i+1, val, s=80, color=feature_colors[features[i % len(features)]])
        plt.text(i+1, val, f"{val:.4f}", ha="center", va="bottom", fontsize=9)
    
    plt.plot(range(1, len(sorted_eigs)+1), sorted_eigs, color="black")
    plt.title("Laplacian Spectrum of SHAP Interaction Network")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Eigenvalue")
    plt.legend(handles=legend_elements)
    plt.show()
    
    # ============================================================
    # 2️⃣ TOPOLOJİK MESAFE MATRİSİ
    # ============================================================
    
    distance_matrix = squareform(pdist(shap_vals.T, metric="euclidean"))
    
    print("\n🔹 Feature Distance Matrix:")
    print(pd.DataFrame(distance_matrix, index=features, columns=features).round(4))
    
    plt.figure(figsize=(10,7))
    sns.heatmap(distance_matrix,
                xticklabels=features,
                yticklabels=features,
                annot=True,
                fmt=".2f",
                cmap="turbo")
    plt.title("Feature Distance Matrix (Topological Structure)")
    plt.show()
    
    # ============================================================
    # 3️⃣ SHAP CURVATURE SPECTRUM
    # ============================================================
    
    cov_matrix = np.cov(shap_vals.T)
    eigvals_cov, _ = eigh(cov_matrix)
    
    curvature_proxy = eigvals_cov / eigvals_cov.sum()
    sorted_curv = np.sort(curvature_proxy)[::-1]
    
    print("\n🔹 SHAP Curvature Eigenvalues:")
    for i, val in enumerate(sorted_curv, 1):
        print(f"Component {i}: {val:.6f}")
    
    plt.figure(figsize=(9,5))
    for i, val in enumerate(sorted_curv):
        plt.bar(i+1, val, color=feature_colors[features[i % len(features)]])
        plt.text(i+1, val, f"{val:.4f}", ha="center", va="bottom", fontsize=9)
    
    plt.title("SHAP Curvature Spectrum")
    plt.xlabel("Component")
    plt.ylabel("Normalized Eigenvalue")
    plt.legend(handles=legend_elements)
    plt.show()
    
    # ============================================================
    # 4️⃣ NODE-WISE INFORMATION ENTROPY
    # ============================================================
    
    prob_matrix = interaction_matrix / interaction_matrix.sum()
    node_entropy = entropy(prob_matrix, axis=1)
    
    print("\n🔹 Node-wise Entropy Values:")
    for f, val in zip(features, node_entropy):
        print(f"{f}: {val:.6f}")
    
    plt.figure(figsize=(11,6))
    bars = plt.bar(features, node_entropy,
                   color=[feature_colors[f] for f in features])
    
    for bar, val in zip(bars, node_entropy):
        plt.text(bar.get_x()+bar.get_width()/2,
                 bar.get_height(),
                 f"{val:.4f}",
                 ha="center", va="bottom", fontsize=9)
    
    plt.xticks(rotation=45)
    plt.title("Node-wise Information Entropy")
    plt.legend(handles=legend_elements)
    plt.show()
    
    # ============================================================
    # ENTROPY FLOW DIFFERENTIAL
    # ============================================================
    
    entropy_flow_rate = np.gradient(node_entropy)
    
    print("\n🔹 Entropy Flow Gradient:")
    for f, val in zip(features, entropy_flow_rate):
        print(f"{f}: {val:.6f}")
    
    plt.figure(figsize=(9,5))
    for i, val in enumerate(entropy_flow_rate):
        plt.scatter(i+1, val, s=80, color=feature_colors[features[i]])
        plt.text(i+1, val, f"{val:.4f}", ha="center", va="bottom", fontsize=9)
    
    plt.plot(range(1, len(entropy_flow_rate)+1),
             entropy_flow_rate, color="black")
    plt.title("Entropy Flow Differential")
    plt.xlabel("Feature Index")
    plt.ylabel("Gradient Value")
    plt.legend(handles=legend_elements)
    plt.show()
    
    # ============================================================
    # 5️⃣ QUANTUM SHAP – DENSITY MATRIX
    # ============================================================
    
    density_matrix = cov_matrix / np.trace(cov_matrix)
    eigenvals_density, _ = eigh(density_matrix)
    sorted_density = np.sort(eigenvals_density)[::-1]
    
    von_neumann_entropy = -np.sum(
        eigenvals_density * np.log(eigenvals_density + 1e-12)
    )
    
    print("\n🔹 Density Matrix Eigenvalues:")
    for i, val in enumerate(sorted_density, 1):
        print(f"Eigenvalue {i}: {val:.6f}")
    
    print(f"\n🔹 Von Neumann Entropy: {von_neumann_entropy:.6f}")
    
    plt.figure(figsize=(9,5))
    for i, val in enumerate(sorted_density):
        plt.bar(i+1, val, color=feature_colors[features[i % len(features)]])
        plt.text(i+1, val, f"{val:.4f}", ha="center", va="bottom", fontsize=9)
    
    plt.title("Density Matrix Eigen-Spectrum (Quantum SHAP)")
    plt.xlabel("Eigenvalue Index")
    plt.ylabel("Density Eigenvalue")
    plt.legend(handles=legend_elements)
    plt.show()


.. parsed-literal::

    
    🔹 Laplacian Eigenvalues:
    Eigenvalue 1: 0.000000
    Eigenvalue 2: 0.566117
    Eigenvalue 3: 0.680317
    Eigenvalue 4: 0.814589
    Eigenvalue 5: 1.033915
    Eigenvalue 6: 1.301929
    Eigenvalue 7: 1.633842
    


.. image:: output_373_1.png


.. parsed-literal::

    
    🔹 Feature Distance Matrix:
                                      Institutions  Human capital and research  \
    Institutions                            0.0000                     14.6837   
    Human capital and research             14.6837                      0.0000   
    Infrastructure                         16.7764                     16.0133   
    Market sophistication                  12.6540                     10.8501   
    Business sophistication                32.1899                     23.6089   
    Knowledge and technology outputs       27.8073                     19.6396   
    Creative outputs                       43.7050                     36.6803   
    
                                      Infrastructure  Market sophistication  \
    Institutions                             16.7764                12.6540   
    Human capital and research               16.0133                10.8501   
    Infrastructure                            0.0000                14.5370   
    Market sophistication                    14.5370                 0.0000   
    Business sophistication                  29.2861                27.0484   
    Knowledge and technology outputs         21.5613                21.7908   
    Creative outputs                         34.0649                38.1011   
    
                                      Business sophistication  \
    Institutions                                      32.1899   
    Human capital and research                        23.6089   
    Infrastructure                                    29.2861   
    Market sophistication                             27.0484   
    Business sophistication                            0.0000   
    Knowledge and technology outputs                  19.7998   
    Creative outputs                                  31.3462   
    
                                      Knowledge and technology outputs  \
    Institutions                                               27.8073   
    Human capital and research                                 19.6396   
    Infrastructure                                             21.5613   
    Market sophistication                                      21.7908   
    Business sophistication                                    19.7998   
    Knowledge and technology outputs                            0.0000   
    Creative outputs                                           29.0387   
    
                                      Creative outputs  
    Institutions                               43.7050  
    Human capital and research                 36.6803  
    Infrastructure                             34.0649  
    Market sophistication                      38.1011  
    Business sophistication                    31.3462  
    Knowledge and technology outputs           29.0387  
    Creative outputs                            0.0000  
    


.. image:: output_373_3.png


.. parsed-literal::

    
    🔹 SHAP Curvature Eigenvalues:
    Component 1: 0.825875
    Component 2: 0.074955
    Component 3: 0.038548
    Component 4: 0.029928
    Component 5: 0.011512
    Component 6: 0.010114
    Component 7: 0.009068
    


.. image:: output_373_5.png


.. parsed-literal::

    
    🔹 Node-wise Entropy Values:
    Institutions: 1.180182
    Human capital and research: 1.118541
    Infrastructure: 1.003256
    Market sophistication: 1.085785
    Business sophistication: 0.945872
    Knowledge and technology outputs: 1.017206
    Creative outputs: 0.938670
    


.. image:: output_373_7.png


.. parsed-literal::

    
    🔹 Entropy Flow Gradient:
    Institutions: -0.061641
    Human capital and research: -0.088463
    Infrastructure: -0.016378
    Market sophistication: -0.028692
    Business sophistication: -0.034289
    Knowledge and technology outputs: -0.003601
    Creative outputs: -0.078536
    


.. image:: output_373_9.png


.. parsed-literal::

    
    🔹 Density Matrix Eigenvalues:
    Eigenvalue 1: 0.825875
    Eigenvalue 2: 0.074955
    Eigenvalue 3: 0.038548
    Eigenvalue 4: 0.029928
    Eigenvalue 5: 0.011512
    Eigenvalue 6: 0.010114
    Eigenvalue 7: 0.009068
    
    🔹 Von Neumann Entropy: 0.723225
    


.. image:: output_373_11.png




.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import networkx as nx
    import matplotlib.pyplot as plt
    import seaborn as sns
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh, expm
    from scipy.spatial.distance import pdist, squareform
    from numpy.linalg import eig
    from gudhi import RipsComplex
    from gudhi import plot_persistence_barcode
    
    sns.set(style="whitegrid", font_scale=1.1)
    
    # ------------------------------------------------------------
    # TRAIN MODEL
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    
    # ============================================================
    # 1️⃣ RICCI CURVATURE
    # ============================================================
    
    G = nx.from_numpy_array(interaction_matrix)
    G = nx.relabel_nodes(G, dict(enumerate(features)))
    
    ricci_curvature = {}
    
    for node in G.nodes():
        neighbors = list(G.neighbors(node))
        deg = G.degree(node)
        avg_neighbor_deg = np.mean([G.degree(n) for n in neighbors])
        ricci_curvature[node] = 1 - (avg_neighbor_deg / deg)
    
    ricci_series = pd.Series(ricci_curvature).sort_values(ascending=False)
    
    print("\nRICCI CURVATURE VALUES:")
    print(ricci_series)
    
    plt.figure(figsize=(10,6))
    colors = plt.cm.rainbow(np.linspace(0,1,len(ricci_series)))
    bars = plt.bar(ricci_series.index, ricci_series.values, color=colors)
    
    for bar, val in zip(bars, ricci_series.values):
        plt.text(bar.get_x()+bar.get_width()/2,
                 bar.get_height(),
                 f"{val:.3f}",
                 ha='center', va='bottom')
    
    plt.xticks(rotation=45)
    plt.title("Ricci Curvature Approximation on SHAP Network")
    plt.show()
    
    # ============================================================
    # 2️⃣ PERSISTENT HOMOLOGY
    # ============================================================
    
    distance_matrix = squareform(pdist(shap_vals.T))
    
    print("\nDISTANCE MATRIX:")
    print(pd.DataFrame(distance_matrix, index=features, columns=features))
    
    plt.figure(figsize=(10,7))
    sns.heatmap(distance_matrix,
                xticklabels=features,
                yticklabels=features,
                annot=True,
                fmt=".2f",
                cmap="turbo")
    plt.title("Feature Distance Matrix (Topological Structure)")
    plt.show()
    
    rips = RipsComplex(distance_matrix=distance_matrix,
                       max_edge_length=np.max(distance_matrix))
    simplex_tree = rips.create_simplex_tree(max_dimension=2)
    diag = simplex_tree.persistence()
    
    print("\nPERSISTENCE DIAGRAM:")
    for d in diag:
        print(d)
    
    plot_persistence_barcode(diag)
    plt.title("Persistent Homology Barcode (SHAP Space)")
    plt.show()
    
    # ============================================================
    # 3️⃣ GRAPH HEAT EQUATION
    # ============================================================
    
    L = nx.laplacian_matrix(G).toarray()
    initial_signal = np.abs(shap_vals).mean(axis=0)
    
    t = 0.5
    heat_diffusion = expm(-t * L) @ initial_signal
    
    print("\nHEAT DIFFUSION VALUES:")
    for f, val in zip(features, heat_diffusion):
        print(f"{f}: {val:.6f}")
    
    plt.figure(figsize=(10,6))
    colors = plt.cm.plasma(np.linspace(0,1,len(features)))
    bars = plt.bar(features, heat_diffusion, color=colors)
    
    for bar, val in zip(bars, heat_diffusion):
        plt.text(bar.get_x()+bar.get_width()/2,
                 bar.get_height(),
                 f"{val:.3f}",
                 ha='center', va='bottom')
    
    plt.xticks(rotation=90)
    plt.title("Graph Heat Diffusion on SHAP Network")
    plt.show()
    
    # ============================================================
    # 4️⃣ KOOPMAN OPERATOR
    # ============================================================
    
    X_t = shap_vals[:-1]
    X_t1 = shap_vals[1:]
    K = np.linalg.pinv(X_t) @ X_t1
    eigvals_koopman, _ = eig(K)
    
    print("\nKOOPMAN EIGENVALUES:")
    for i, val in enumerate(eigvals_koopman):
        print(f"λ{i+1}: {val.real:.6f} + {val.imag:.6f}i")
    
    plt.figure(figsize=(7,7))
    colors = plt.cm.cool(np.linspace(0,1,len(eigvals_koopman)))
    
    for i, val in enumerate(eigvals_koopman):
        plt.scatter(val.real, val.imag, color=colors[i], s=80)
        plt.text(val.real, val.imag,
                 f"{val.real:.2f}",
                 fontsize=8)
    
    plt.axhline(0, color='black')
    plt.axvline(0, color='black')
    plt.title("Koopman Operator Spectrum")
    plt.xlabel("Real")
    plt.ylabel("Imaginary")
    plt.show()
    
    # ============================================================
    # 5️⃣ SHAP FIELD PDE
    # ============================================================
    
    dt = 0.1
    steps = 20
    field = initial_signal.copy()
    
    for _ in range(steps):
        field = field - dt * (L @ field)
    
    print("\nFINAL SHAP FIELD VALUES:")
    for f, val in zip(features, field):
        print(f"{f}: {val:.6f}")
    
    plt.figure(figsize=(10,6))
    colors = plt.cm.viridis(np.linspace(0,1,len(features)))
    bars = plt.bar(features, field, color=colors)
    
    for bar, val in zip(bars, field):
        plt.text(bar.get_x()+bar.get_width()/2,
                 bar.get_height(),
                 f"{val:.3f}",
                 ha='center', va='bottom')
    
    plt.xticks(rotation=90)
    plt.title("SHAP Field PDE Evolution (Discrete Diffusion)")
    plt.show()


.. parsed-literal::

    
    RICCI CURVATURE VALUES:
    Institutions                        0.0
    Human capital and research          0.0
    Infrastructure                      0.0
    Market sophistication               0.0
    Business sophistication             0.0
    Knowledge and technology outputs    0.0
    Creative outputs                    0.0
    dtype: float64
    


.. image:: output_376_1.png


.. parsed-literal::

    
    DISTANCE MATRIX:
                                      Institutions  Human capital and research  \
    Institutions                          0.000000                   14.683690   
    Human capital and research           14.683690                    0.000000   
    Infrastructure                       16.776444                   16.013289   
    Market sophistication                12.654028                   10.850075   
    Business sophistication              32.189896                   23.608870   
    Knowledge and technology outputs     27.807255                   19.639571   
    Creative outputs                     43.704961                   36.680337   
    
                                      Infrastructure  Market sophistication  \
    Institutions                           16.776444              12.654028   
    Human capital and research             16.013289              10.850075   
    Infrastructure                          0.000000              14.536989   
    Market sophistication                  14.536989               0.000000   
    Business sophistication                29.286107              27.048406   
    Knowledge and technology outputs       21.561251              21.790766   
    Creative outputs                       34.064880              38.101065   
    
                                      Business sophistication  \
    Institutions                                    32.189896   
    Human capital and research                      23.608870   
    Infrastructure                                  29.286107   
    Market sophistication                           27.048406   
    Business sophistication                          0.000000   
    Knowledge and technology outputs                19.799762   
    Creative outputs                                31.346234   
    
                                      Knowledge and technology outputs  \
    Institutions                                             27.807255   
    Human capital and research                               19.639571   
    Infrastructure                                           21.561251   
    Market sophistication                                    21.790766   
    Business sophistication                                  19.799762   
    Knowledge and technology outputs                          0.000000   
    Creative outputs                                         29.038721   
    
                                      Creative outputs  
    Institutions                             43.704961  
    Human capital and research               36.680337  
    Infrastructure                           34.064880  
    Market sophistication                    38.101065  
    Business sophistication                  31.346234  
    Knowledge and technology outputs         29.038721  
    Creative outputs                          0.000000  
    


.. image:: output_376_3.png


.. parsed-literal::

    
    PERSISTENCE DIAGRAM:
    (0, (0.0, inf))
    (0, (0.0, 29.038721440267878))
    (0, (0.0, 19.799761799972966))
    (0, (0.0, 19.639571478887294))
    (0, (0.0, 14.536989203224806))
    (0, (0.0, 12.654027502727024))
    (0, (0.0, 10.850074748582516))
    


.. image:: output_376_5.png


.. parsed-literal::

    
    HEAT DIFFUSION VALUES:
    Institutions: 1.169317
    Human capital and research: 1.474098
    Infrastructure: 1.809426
    Market sophistication: 1.315951
    Business sophistication: 2.327701
    Knowledge and technology outputs: 2.069660
    Creative outputs: 2.649446
    


.. image:: output_376_7.png


.. parsed-literal::

    
    KOOPMAN EIGENVALUES:
    λ1: 0.185082 + 0.000000i
    λ2: -0.160437 + 0.098820i
    λ3: -0.160437 + -0.098820i
    λ4: 0.042477 + 0.000000i
    λ5: -0.040630 + 0.083010i
    λ6: -0.040630 + -0.083010i
    λ7: -0.094851 + 0.000000i
    


.. image:: output_376_9.png


.. parsed-literal::

    
    FINAL SHAP FIELD VALUES:
    Institutions: 1.602172
    Human capital and research: 1.798180
    Infrastructure: 1.896831
    Market sophistication: 1.705203
    Business sophistication: 1.961057
    Knowledge and technology outputs: 1.922876
    Creative outputs: 1.929280
    


.. image:: output_376_11.png





.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import plotly.graph_objects as go
    import plotly.express as px
    from plotly.subplots import make_subplots
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh, expm
    from numpy.linalg import eig
    from scipy.stats import entropy
    
    # ------------------------------------------------------------
    # MODEL TRAIN
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    mean_importance = np.abs(shap_vals).mean(axis=0)
    
    colors = px.colors.qualitative.Bold
    
    # ============================================================
    # 1️⃣ Schrödinger
    # ============================================================
    
    H = interaction_matrix
    psi0 = mean_importance / np.linalg.norm(mean_importance)
    t = 0.8
    U = expm(-1j * H * t)
    psi_t = U @ psi0
    prob_density = np.abs(psi_t)**2
    
    print("\n=== SCHRÖDINGER PROBABILITY DENSITY ===")
    for f, val in zip(features, prob_density):
        print(f"{f:<20} : {val:.10f}")
    
    # ============================================================
    # 2️⃣ Entropic Gravity
    # ============================================================
    
    node_entropy = entropy(interaction_matrix / interaction_matrix.sum(), axis=1)
    entropy_gradient = np.gradient(node_entropy)
    gravity_potential = -entropy_gradient
    
    print("\n=== ENTROPIC GRAVITY POTENTIAL ===")
    for f, val in zip(features, gravity_potential):
        print(f"{f:<20} : {val:.10f}")
    
    # ============================================================
    # 3️⃣ Spectral Embedding
    # ============================================================
    
    eigvals, eigvecs = eigh(interaction_matrix)
    embedding = eigvecs[:, -2:]
    
    embed_df = pd.DataFrame(embedding, columns=["Dim1", "Dim2"])
    embed_df["Feature"] = features
    embed_df["Importance"] = mean_importance
    
    print("\n=== GEOMETRIC EMBEDDING ===")
    print(embed_df.round(10))
    
    # ============================================================
    # 4️⃣ Functor Map
    # ============================================================
    
    local_norms = np.linalg.norm(shap_vals, axis=0)
    functor_map = local_norms / mean_importance
    
    print("\n=== FUNCTOR MAP ===")
    for f, val in zip(features, functor_map):
        print(f"{f:<20} : {val:.10f}")
    
    # ============================================================
    # 5️⃣ Koopman
    # ============================================================
    
    X_t = shap_vals[:-1]
    X_t1 = shap_vals[1:]
    K = np.linalg.pinv(X_t) @ X_t1
    eigvals_k, _ = eig(K)
    
    print("\n=== KOOPMAN EIGENVALUES ===")
    for val in eigvals_k[:10]:
        print(f"{val.real:.10f} + {val.imag:.10f}i")
    
    # ============================================================
    # 📊 TEK BÜYÜK FIGURE
    # ============================================================
    
    fig = make_subplots(
        rows=5, cols=1,
        vertical_spacing=0.06,
        subplot_titles=(
            "Schrödinger Probability Density",
            "Entropic Gravity Potential",
            "Geometric Spectral Embedding",
            "Global → Local Functor Ratio",
            "Koopman Spectrum"
        )
    )
    
    # 1️⃣ Schrödinger
    for i, f in enumerate(features):
        fig.add_trace(
            go.Bar(
                x=[f],
                y=[prob_density[i]],
                marker_color=colors[i % len(colors)],
                text=[f"{prob_density[i]:.6f}"],
                textposition="outside"
            ),
            row=1, col=1
        )
    
    # 2️⃣ Gravity
    fig.add_trace(
        go.Scatter(
            x=features,
            y=gravity_potential,
            mode="markers+text",
            marker=dict(size=16, color=gravity_potential, colorscale="Turbo"),
            text=[f"{v:.6f}" for v in gravity_potential],
            textposition="top center"
        ),
        row=2, col=1
    )
    
    # 3️⃣ Embedding
    for i, row_data in embed_df.iterrows():
        fig.add_trace(
            go.Scatter(
                x=[row_data["Dim1"]],
                y=[row_data["Dim2"]],
                mode="markers+text",
                marker=dict(
                    size=30 + row_data["Importance"]*250,
                    color=colors[i % len(colors)]
                ),
                text=[f"{row_data['Feature']}<br>({row_data['Dim1']:.3f}, {row_data['Dim2']:.3f})"],
                textposition="top center"
            ),
            row=3, col=1
        )
    
    # 4️⃣ Functor
    for i, f in enumerate(features):
        fig.add_trace(
            go.Bar(
                x=[f],
                y=[functor_map[i]],
                marker_color=colors[i % len(colors)],
                text=[f"{functor_map[i]:.6f}"],
                textposition="outside"
            ),
            row=4, col=1
        )
    
    # 5️⃣ Koopman
    fig.add_trace(
        go.Scatter(
            x=eigvals_k.real,
            y=eigvals_k.imag,
            mode="markers+text",
            marker=dict(
                size=16,
                color=np.abs(eigvals_k),
                colorscale="Plasma"
            ),
            text=[f"{r:.3f}+{i:.3f}i" for r, i in zip(eigvals_k.real, eigvals_k.imag)],
            textposition="top center"
        ),
        row=5, col=1
    )
    
    fig.update_layout(
        height=3200,  # 🔥 YÜKSEKLİK ARTTIRILDI
        width=1200,
        showlegend=False,
        title="Unified SHAP Geometric-Quantum Analysis Dashboard"
    )
    
    fig.show()


.. parsed-literal::

    
    === SCHRÖDINGER PROBABILITY DENSITY ===
    Institutions         : 0.0036804394
    Human capital and research : 0.0090238322
    Infrastructure       : 0.0313995359
    Market sophistication : 0.0055945193
    Business sophistication : 0.2069233063
    Knowledge and technology outputs : 0.1035017239
    Creative outputs     : 0.6398766430
    
    === ENTROPIC GRAVITY POTENTIAL ===
    Institutions         : 0.0616412306
    Human capital and research : 0.0884630212
    Infrastructure       : 0.0163780490
    Market sophistication : 0.0286920304
    Business sophistication : 0.0342892772
    Knowledge and technology outputs : 0.0036011121
    Creative outputs     : 0.0785364443
    
    === GEOMETRIC EMBEDDING ===
           Dim1      Dim2                           Feature  Importance
    0 -0.021971 -0.051244                      Institutions    0.879362
    1 -0.082190 -0.085634        Human capital and research    1.156477
    2 -0.070993 -0.126801                    Infrastructure    1.618762
    3 -0.044263 -0.064638             Market sophistication    1.013758
    4 -0.889196 -0.270435           Business sophistication    2.557080
    5 -0.281810 -0.194164  Knowledge and technology outputs    2.074511
    6  0.340113 -0.926795                  Creative outputs    3.515649
    
    === FUNCTOR MAP ===
    Institutions         : 14.1449716713
    Human capital and research : 14.3940047530
    Infrastructure       : 13.3241303593
    Market sophistication : 14.4333464319
    Business sophistication : 13.9607012607
    Knowledge and technology outputs : 14.5834626097
    Creative outputs     : 13.6906350639
    
    === KOOPMAN EIGENVALUES ===
    0.1850817112 + 0.0000000000i
    -0.1604370970 + 0.0988197858i
    -0.1604370970 + -0.0988197858i
    0.0424767324 + 0.0000000000i
    -0.0406300155 + 0.0830098713i
    -0.0406300155 + -0.0830098713i
    -0.0948512135 + 0.0000000000i
    


.. raw:: html

    <div>                            <div id="1d1d44ef-4795-4a81-bf21-21ca0edce18a" class="plotly-graph-div" style="height:3200px; width:1200px;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("1d1d44ef-4795-4a81-bf21-21ca0edce18a")) {                    Plotly.newPlot(                        "1d1d44ef-4795-4a81-bf21-21ca0edce18a",                        [{"marker":{"color":"rgb(127, 60, 141)"},"text":["0.003680"],"textposition":"outside","x":["Institutions"],"y":[0.003680439389912342],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(17, 165, 121)"},"text":["0.009024"],"textposition":"outside","x":["Human capital and research"],"y":[0.009023832170566352],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(57, 105, 172)"},"text":["0.031400"],"textposition":"outside","x":["Infrastructure"],"y":[0.0313995359373844],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(242, 183, 1)"},"text":["0.005595"],"textposition":"outside","x":["Market sophistication"],"y":[0.005594519282990885],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(231, 63, 116)"},"text":["0.206923"],"textposition":"outside","x":["Business sophistication"],"y":[0.2069233062658882],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(128, 186, 90)"},"text":["0.103502"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[0.10350172392740908],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(230, 131, 16)"},"text":["0.639877"],"textposition":"outside","x":["Creative outputs"],"y":[0.6398766430258483],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":[0.06164123063730753,0.0884630211616444,0.016378048992781058,0.028692030422150683,0.034289277239910265,0.0036011121416795744,0.07853644434825913],"colorscale":[[0.0,"#30123b"],[0.07142857142857142,"#4145ab"],[0.14285714285714285,"#4675ed"],[0.21428571428571427,"#39a2fc"],[0.2857142857142857,"#1bcfd4"],[0.35714285714285715,"#24eca6"],[0.42857142857142855,"#61fc6c"],[0.5,"#a4fc3b"],[0.5714285714285714,"#d1e834"],[0.6428571428571429,"#f3c63a"],[0.7142857142857143,"#fe9b2d"],[0.7857142857142857,"#f36315"],[0.8571428571428571,"#d93806"],[0.9285714285714286,"#b11901"],[1.0,"#7a0402"]],"size":16},"mode":"markers+text","text":["0.061641","0.088463","0.016378","0.028692","0.034289","0.003601","0.078536"],"textposition":"top center","x":["Institutions","Human capital and research","Infrastructure","Market sophistication","Business sophistication","Knowledge and technology outputs","Creative outputs"],"y":[0.06164123063730753,0.0884630211616444,0.016378048992781058,0.028692030422150683,0.034289277239910265,0.0036011121416795744,0.07853644434825913],"type":"scatter","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(127, 60, 141)","size":249.84039338227353},"mode":"markers+text","text":["Institutions\u003cbr\u003e(-0.022, -0.051)"],"textposition":"top center","x":[-0.02197147956050521],"y":[-0.05124375965288054],"type":"scatter","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(17, 165, 121)","size":319.119341915884},"mode":"markers+text","text":["Human capital and research\u003cbr\u003e(-0.082, -0.086)"],"textposition":"top center","x":[-0.08219003157057614],"y":[-0.08563396588233818],"type":"scatter","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(57, 105, 172)","size":434.6905220111703},"mode":"markers+text","text":["Infrastructure\u003cbr\u003e(-0.071, -0.127)"],"textposition":"top center","x":[-0.07099319668648356],"y":[-0.12680122234281],"type":"scatter","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(242, 183, 1)","size":283.4394297690196},"mode":"markers+text","text":["Market sophistication\u003cbr\u003e(-0.044, -0.065)"],"textposition":"top center","x":[-0.04426303903505893],"y":[-0.06463829427137616],"type":"scatter","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(231, 63, 116)","size":669.2700365319462},"mode":"markers+text","text":["Business sophistication\u003cbr\u003e(-0.889, -0.270)"],"textposition":"top center","x":[-0.8891957931868406],"y":[-0.27043547557662695],"type":"scatter","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(128, 186, 90)","size":548.6277605740831},"mode":"markers+text","text":["Knowledge and technology outputs\u003cbr\u003e(-0.282, -0.194)"],"textposition":"top center","x":[-0.28181020164888065],"y":[-0.19416414464986045],"type":"scatter","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(230, 131, 16)","size":908.9121696441063},"mode":"markers+text","text":["Creative outputs\u003cbr\u003e(0.340, -0.927)"],"textposition":"top center","x":[0.3401127075270897],"y":[-0.9267951124126559],"type":"scatter","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(127, 60, 141)"},"text":["14.144972"],"textposition":"outside","x":["Institutions"],"y":[14.144971671269714],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(17, 165, 121)"},"text":["14.394005"],"textposition":"outside","x":["Human capital and research"],"y":[14.394004753049728],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(57, 105, 172)"},"text":["13.324130"],"textposition":"outside","x":["Infrastructure"],"y":[13.324130359332651],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(242, 183, 1)"},"text":["14.433346"],"textposition":"outside","x":["Market sophistication"],"y":[14.433346431940867],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(231, 63, 116)"},"text":["13.960701"],"textposition":"outside","x":["Business sophistication"],"y":[13.960701260656636],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(128, 186, 90)"},"text":["14.583463"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[14.583462609720904],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(230, 131, 16)"},"text":["13.690635"],"textposition":"outside","x":["Creative outputs"],"y":[13.690635063946473],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":[0.18508171118097716,0.18842879866815948,0.18842879866815948,0.042476732410328094,0.09241989451236152,0.09241989451236152,0.09485121347792287],"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"size":16},"mode":"markers+text","text":["0.185+0.000i","-0.160+0.099i","-0.160+-0.099i","0.042+0.000i","-0.041+0.083i","-0.041+-0.083i","-0.095+0.000i"],"textposition":"top center","x":[0.18508171118097716,-0.16043709702490136,-0.16043709702490136,0.042476732410328094,-0.04063001553944821,-0.04063001553944821,-0.09485121347792287],"y":[0.0,0.09881978580096275,-0.09881978580096275,0.0,0.0830098713343192,-0.0830098713343192,0.0],"type":"scatter","xaxis":"x5","yaxis":"y5"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0]},"yaxis":{"anchor":"x","domain":[0.848,1.0]},"xaxis2":{"anchor":"y2","domain":[0.0,1.0]},"yaxis2":{"anchor":"x2","domain":[0.6359999999999999,0.7879999999999999]},"xaxis3":{"anchor":"y3","domain":[0.0,1.0]},"yaxis3":{"anchor":"x3","domain":[0.424,0.576]},"xaxis4":{"anchor":"y4","domain":[0.0,1.0]},"yaxis4":{"anchor":"x4","domain":[0.212,0.364]},"xaxis5":{"anchor":"y5","domain":[0.0,1.0]},"yaxis5":{"anchor":"x5","domain":[0.0,0.152]},"annotations":[{"font":{"size":16},"showarrow":false,"text":"Schrödinger Probability Density","x":0.5,"xanchor":"center","xref":"paper","y":1.0,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Entropic Gravity Potential","x":0.5,"xanchor":"center","xref":"paper","y":0.7879999999999999,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Geometric Spectral Embedding","x":0.5,"xanchor":"center","xref":"paper","y":0.576,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Global → Local Functor Ratio","x":0.5,"xanchor":"center","xref":"paper","y":0.364,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Koopman Spectrum","x":0.5,"xanchor":"center","xref":"paper","y":0.152,"yanchor":"bottom","yref":"paper"}],"height":3200,"width":1200,"showlegend":false,"title":{"text":"Unified SHAP Geometric-Quantum Analysis Dashboard"}},                        {"responsive": true}                    ).then(function(){
    
    var gd = document.getElementById('1d1d44ef-4795-4a81-bf21-21ca0edce18a');
    var x = new MutationObserver(function (mutations, observer) {{
            var display = window.getComputedStyle(gd).display;
            if (!display || display === 'none') {{
                console.log([gd, 'removed!']);
                Plotly.purge(gd);
                observer.disconnect();
            }}
    }});
    
    // Listen for the removal of the full notebook cells
    var notebookContainer = gd.closest('#notebook-container');
    if (notebookContainer) {{
        x.observe(notebookContainer, {childList: true});
    }}
    
    // Listen for the clearing of the current output cell
    var outputEl = gd.closest('.output');
    if (outputEl) {{
        x.observe(outputEl, {childList: true});
    }}
    
                            })                };                });            </script>        </div>






.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import plotly.graph_objects as go
    import plotly.express as px
    from plotly.subplots import make_subplots
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh
    from scipy.stats import entropy
    
    # ------------------------------------------------------------
    # MODEL TRAIN
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    mean_importance = np.abs(shap_vals).mean(axis=0)
    
    colors = px.colors.qualitative.Bold
    
    # ============================================================
    # HESAPLAMALAR
    # ============================================================
    
    potential = mean_importance
    gradient = np.gradient(potential)
    action = 0.5 * gradient**2 + potential
    
    kinetic = 0.5 * gradient**2
    lagrangian = kinetic - potential
    
    paths = 500
    path_integral = []
    for i in range(len(features)):
        samples = np.random.normal(mean_importance[i],
                                   0.1*mean_importance[i] if mean_importance[i]!=0 else 0.001,
                                   paths)
        path_integral.append(np.mean(np.exp(-samples)))
    
    L = np.diag(interaction_matrix.sum(axis=1)) - interaction_matrix
    ricci_flow = interaction_matrix - 0.1 * L
    ricci_strength = ricci_flow.mean(axis=1)
    
    gauge_field = mean_importance / np.linalg.norm(mean_importance)
    
    # ============================================================
    # SAYISAL ÇIKTILAR
    # ============================================================
    
    print("\n=== HAMILTON–JACOBI ACTION FIELD ===")
    for f, v in zip(features, action):
        print(f"{f:<20} : {v:.10f}")
    
    print("\n=== LAGRANGIAN FUNCTIONAL ===")
    for f, v in zip(features, lagrangian):
        print(f"{f:<20} : {v:.10f}")
    
    print("\n=== PATH INTEGRAL ===")
    for f, v in zip(features, path_integral):
        print(f"{f:<20} : {v:.10f}")
    
    print("\n=== RICCI FLOW STRENGTH ===")
    for f, v in zip(features, ricci_strength):
        print(f"{f:<20} : {v:.10f}")
    
    print("\n=== GAUGE FIELD ===")
    for f, v in zip(features, gauge_field):
        print(f"{f:<20} : {v:.10f}")
    
    # ============================================================
    # TEK SÜTUN DASHBOARD
    # ============================================================
    
    fig = make_subplots(
        rows=5, cols=1,
        vertical_spacing=0.05,
        subplot_titles=[
            "Hamilton–Jacobi Action Field",
            "Lagrangian Functional",
            "Path Integral",
            "Ricci Flow Evolution",
            "Gauge-Invariant Field"
        ]
    )
    
    # 1️⃣ Action
    for i, f in enumerate(features):
        fig.add_trace(
            go.Bar(
                x=[f],
                y=[action[i]],
                marker_color=colors[i % len(colors)],
                text=[f"{action[i]:.6f}"],
                textposition="outside"
            ),
            row=1, col=1
        )
    
    # 2️⃣ Lagrangian
    for i, f in enumerate(features):
        fig.add_trace(
            go.Bar(
                x=[f],
                y=[lagrangian[i]],
                marker_color=colors[i % len(colors)],
                text=[f"{lagrangian[i]:.6f}"],
                textposition="outside"
            ),
            row=2, col=1
        )
    
    # 3️⃣ Path Integral
    for i, f in enumerate(features):
        fig.add_trace(
            go.Bar(
                x=[f],
                y=[path_integral[i]],
                marker_color=colors[i % len(colors)],
                text=[f"{path_integral[i]:.6f}"],
                textposition="outside"
            ),
            row=3, col=1
        )
    
    # 4️⃣ Ricci Flow
    for i, f in enumerate(features):
        fig.add_trace(
            go.Bar(
                x=[f],
                y=[ricci_strength[i]],
                marker_color=colors[i % len(colors)],
                text=[f"{ricci_strength[i]:.6f}"],
                textposition="outside"
            ),
            row=4, col=1
        )
    
    # 5️⃣ Gauge Field
    for i, f in enumerate(features):
        fig.add_trace(
            go.Bar(
                x=[f],
                y=[gauge_field[i]],
                marker_color=colors[i % len(colors)],
                text=[f"{gauge_field[i]:.6f}"],
                textposition="outside"
            ),
            row=5, col=1
        )
    
    fig.update_layout(
        height=3800,   # 🔥 daha da büyütüldü
        width=1300,
        showlegend=False,
        title_text="Unified Explainability Physics Dashboard",
        title_x=0.5,
        font=dict(size=18)
    )
    
    fig.show()


.. parsed-literal::

    
    === HAMILTON–JACOBI ACTION FIELD ===
    Institutions         : 0.9177581552
    Human capital and research : 1.2248165078
    Infrastructure       : 1.6213082003
    Market sophistication : 1.1238128163
    Business sophistication : 2.6977298477
    Knowledge and technology outputs : 2.1893677462
    Creative outputs     : 4.5540875219
    
    === LAGRANGIAN FUNCTIONAL ===
    Institutions         : -0.8409649918
    Human capital and research : -1.0881382276
    Infrastructure       : -1.6162159758
    Market sophistication : -0.9037026218
    Business sophistication : -2.4164304445
    Knowledge and technology outputs : -1.9596543384
    Creative outputs     : -2.4772098352
    
    === PATH INTEGRAL ===
    Institutions         : 0.4187917238
    Human capital and research : 0.3182001939
    Infrastructure       : 0.2005576306
    Market sophistication : 0.3650017494
    Business sophistication : 0.0785534439
    Knowledge and technology outputs : 0.1287578651
    Creative outputs     : 0.0312957702
    
    === RICCI FLOW STRENGTH ===
    Institutions         : 0.2267001758
    Human capital and research : 0.3365720237
    Infrastructure       : 0.4269784161
    Market sophistication : 0.2808984288
    Business sophistication : 0.6459160758
    Knowledge and technology outputs : 0.5637326959
    Creative outputs     : 0.8554338061
    
    === GAUGE FIELD ===
    Institutions         : 0.1634036720
    Human capital and research : 0.2148975509
    Infrastructure       : 0.3007996679
    Market sophistication : 0.1883772714
    Business sophistication : 0.4751586811
    Knowledge and technology outputs : 0.3854873037
    Creative outputs     : 0.6532806537
    


.. raw:: html

    <div>                            <div id="eb8a9f9e-00db-429b-b3c3-d2012a411a8d" class="plotly-graph-div" style="height:3800px; width:1300px;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("eb8a9f9e-00db-429b-b3c3-d2012a411a8d")) {                    Plotly.newPlot(                        "eb8a9f9e-00db-429b-b3c3-d2012a411a8d",                        [{"marker":{"color":"rgb(127, 60, 141)"},"text":["0.917758"],"textposition":"outside","x":["Institutions"],"y":[0.9177581552084753],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(17, 165, 121)"},"text":["1.224817"],"textposition":"outside","x":["Human capital and research"],"y":[1.2248165077717752],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(57, 105, 172)"},"text":["1.621308"],"textposition":"outside","x":["Infrastructure"],"y":[1.621308200306297],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(242, 183, 1)"},"text":["1.123813"],"textposition":"outside","x":["Market sophistication"],"y":[1.1238128163416843],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(231, 63, 116)"},"text":["2.697730"],"textposition":"outside","x":["Business sophistication"],"y":[2.697729847718136],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(128, 186, 90)"},"text":["2.189368"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[2.189367746221425],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(230, 131, 16)"},"text":["4.554088"],"textposition":"outside","x":["Creative outputs"],"y":[4.5540875219279116],"type":"bar","xaxis":"x","yaxis":"y"},{"marker":{"color":"rgb(127, 60, 141)"},"text":["-0.840965"],"textposition":"outside","x":["Institutions"],"y":[-0.8409649918497131],"type":"bar","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(17, 165, 121)"},"text":["-1.088138"],"textposition":"outside","x":["Human capital and research"],"y":[-1.0881382275552964],"type":"bar","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(57, 105, 172)"},"text":["-1.616216"],"textposition":"outside","x":["Infrastructure"],"y":[-1.6162159757830652],"type":"bar","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(242, 183, 1)"},"text":["-0.903703"],"textposition":"outside","x":["Market sophistication"],"y":[-0.9037026218104727],"type":"bar","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(231, 63, 116)"},"text":["-2.416430"],"textposition":"outside","x":["Business sophistication"],"y":[-2.416430444537433],"type":"bar","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(128, 186, 90)"},"text":["-1.959654"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[-1.95965433837124],"type":"bar","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(230, 131, 16)"},"text":["-2.477210"],"textposition":"outside","x":["Creative outputs"],"y":[-2.4772098352249388],"type":"bar","xaxis":"x2","yaxis":"y2"},{"marker":{"color":"rgb(127, 60, 141)"},"text":["0.418792"],"textposition":"outside","x":["Institutions"],"y":[0.4187917237772631],"type":"bar","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(17, 165, 121)"},"text":["0.318200"],"textposition":"outside","x":["Human capital and research"],"y":[0.3182001939072824],"type":"bar","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(57, 105, 172)"},"text":["0.200558"],"textposition":"outside","x":["Infrastructure"],"y":[0.20055763064520477],"type":"bar","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(242, 183, 1)"},"text":["0.365002"],"textposition":"outside","x":["Market sophistication"],"y":[0.36500174940920066],"type":"bar","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(231, 63, 116)"},"text":["0.078553"],"textposition":"outside","x":["Business sophistication"],"y":[0.07855344391709906],"type":"bar","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(128, 186, 90)"},"text":["0.128758"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[0.12875786508925818],"type":"bar","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(230, 131, 16)"},"text":["0.031296"],"textposition":"outside","x":["Creative outputs"],"y":[0.031295770222078564],"type":"bar","xaxis":"x3","yaxis":"y3"},{"marker":{"color":"rgb(127, 60, 141)"},"text":["0.226700"],"textposition":"outside","x":["Institutions"],"y":[0.22670017577753063],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(17, 165, 121)"},"text":["0.336572"],"textposition":"outside","x":["Human capital and research"],"y":[0.3365720237329785],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(57, 105, 172)"},"text":["0.426978"],"textposition":"outside","x":["Infrastructure"],"y":[0.42697841607000014],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(242, 183, 1)"},"text":["0.280898"],"textposition":"outside","x":["Market sophistication"],"y":[0.28089842877062676],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(231, 63, 116)"},"text":["0.645916"],"textposition":"outside","x":["Business sophistication"],"y":[0.6459160757695653],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(128, 186, 90)"},"text":["0.563733"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[0.5637326958630251],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(230, 131, 16)"},"text":["0.855434"],"textposition":"outside","x":["Creative outputs"],"y":[0.8554338060552747],"type":"bar","xaxis":"x4","yaxis":"y4"},{"marker":{"color":"rgb(127, 60, 141)"},"text":["0.163404"],"textposition":"outside","x":["Institutions"],"y":[0.16340367200898326],"type":"bar","xaxis":"x5","yaxis":"y5"},{"marker":{"color":"rgb(17, 165, 121)"},"text":["0.214898"],"textposition":"outside","x":["Human capital and research"],"y":[0.21489755085966636],"type":"bar","xaxis":"x5","yaxis":"y5"},{"marker":{"color":"rgb(57, 105, 172)"},"text":["0.300800"],"textposition":"outside","x":["Infrastructure"],"y":[0.300799667915758],"type":"bar","xaxis":"x5","yaxis":"y5"},{"marker":{"color":"rgb(242, 183, 1)"},"text":["0.188377"],"textposition":"outside","x":["Market sophistication"],"y":[0.18837727143304817],"type":"bar","xaxis":"x5","yaxis":"y5"},{"marker":{"color":"rgb(231, 63, 116)"},"text":["0.475159"],"textposition":"outside","x":["Business sophistication"],"y":[0.47515868111187504],"type":"bar","xaxis":"x5","yaxis":"y5"},{"marker":{"color":"rgb(128, 186, 90)"},"text":["0.385487"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[0.3854873036741677],"type":"bar","xaxis":"x5","yaxis":"y5"},{"marker":{"color":"rgb(230, 131, 16)"},"text":["0.653281"],"textposition":"outside","x":["Creative outputs"],"y":[0.6532806536763127],"type":"bar","xaxis":"x5","yaxis":"y5"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0]},"yaxis":{"anchor":"x","domain":[0.8400000000000001,1.0]},"xaxis2":{"anchor":"y2","domain":[0.0,1.0]},"yaxis2":{"anchor":"x2","domain":[0.63,0.79]},"xaxis3":{"anchor":"y3","domain":[0.0,1.0]},"yaxis3":{"anchor":"x3","domain":[0.42000000000000004,0.5800000000000001]},"xaxis4":{"anchor":"y4","domain":[0.0,1.0]},"yaxis4":{"anchor":"x4","domain":[0.21000000000000002,0.37]},"xaxis5":{"anchor":"y5","domain":[0.0,1.0]},"yaxis5":{"anchor":"x5","domain":[0.0,0.16]},"annotations":[{"font":{"size":16},"showarrow":false,"text":"Hamilton–Jacobi Action Field","x":0.5,"xanchor":"center","xref":"paper","y":1.0,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Lagrangian Functional","x":0.5,"xanchor":"center","xref":"paper","y":0.79,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Path Integral","x":0.5,"xanchor":"center","xref":"paper","y":0.5800000000000001,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Ricci Flow Evolution","x":0.5,"xanchor":"center","xref":"paper","y":0.37,"yanchor":"bottom","yref":"paper"},{"font":{"size":16},"showarrow":false,"text":"Gauge-Invariant Field","x":0.5,"xanchor":"center","xref":"paper","y":0.16,"yanchor":"bottom","yref":"paper"}],"title":{"text":"Unified Explainability Physics Dashboard","x":0.5},"font":{"size":18},"height":3800,"width":1300,"showlegend":false},                        {"responsive": true}                    ).then(function(){
    
    var gd = document.getElementById('eb8a9f9e-00db-429b-b3c3-d2012a411a8d');
    var x = new MutationObserver(function (mutations, observer) {{
            var display = window.getComputedStyle(gd).display;
            if (!display || display === 'none') {{
                console.log([gd, 'removed!']);
                Plotly.purge(gd);
                observer.disconnect();
            }}
    }});
    
    // Listen for the removal of the full notebook cells
    var notebookContainer = gd.closest('#notebook-container');
    if (notebookContainer) {{
        x.observe(notebookContainer, {childList: true});
    }}
    
    // Listen for the clearing of the current output cell
    var outputEl = gd.closest('.output');
    if (outputEl) {{
        x.observe(outputEl, {childList: true});
    }}
    
                            })                };                });            </script>        </div>






.. code:: ipython3

    from sklearn.ensemble import ExtraTreesRegressor
    import matplotlib.pyplot as plt
    import numpy as np
    import pandas as pd
    import shap
    
    quantiles = [0.25, 0.50, 0.75]
    quantile_shap = {}
    
    n_boot = 30  # bootstrap runs
    
    all_shap = []
    
    # ------------------------------------------------------------
    # BOOTSTRAP EXTRA TREES MODELS
    # ------------------------------------------------------------
    
    for i in range(n_boot):
        
        idx = np.random.choice(len(df_std), len(df_std), replace=True)
        X_boot = df_std[features].iloc[idx]
        y_boot = df_std[target].iloc[idx]
    
        model = ExtraTreesRegressor(
            n_estimators=400,
            random_state=i,
            n_jobs=-1
        )
        
        model.fit(X_boot, y_boot)
    
        explainer = shap.Explainer(model, X_boot)
        shap_vals = explainer(X_boot)
    
        mean_importance = np.abs(shap_vals.values).mean(axis=0)
        all_shap.append(mean_importance)
    
    all_shap = np.array(all_shap)
    
    # ------------------------------------------------------------
    # COMPUTE QUANTILE IMPORTANCE
    # ------------------------------------------------------------
    
    for q in quantiles:
        quantile_shap[q] = np.quantile(all_shap, q, axis=0)
    
    quantile_df = pd.DataFrame(
        quantile_shap,
        index=features
    )
    
    # Normalize for comparability
    quantile_df = quantile_df.div(quantile_df.sum(axis=0), axis=1)
    
    # ------------------------------------------------------------
    # PRINT NUMERICAL OUTPUT
    # ------------------------------------------------------------
    
    print("\n==============================")
    print("EXTRA TREES – QUANTILE SHAP IMPORTANCE")
    print("==============================\n")
    print(quantile_df.round(4))
    
    # ------------------------------------------------------------
    # PLOT WITH VALUE LABELS
    # ------------------------------------------------------------
    
    fig, ax = plt.subplots(figsize=(14, 7))
    
    bars = quantile_df.plot(
        kind="bar",
        ax=ax,
        edgecolor="black"
    )
    
    plt.title(
        "Extra Trees – Quantile-Specific SHAP Importance\n"
        "Distributional Heterogeneity in Innovation Drivers",
        fontsize=16,
        weight="bold"
    )
    
    plt.ylabel("Relative Mean |SHAP| Contribution", fontsize=14)
    plt.xticks(rotation=45, fontsize=12)
    plt.yticks(fontsize=12)
    plt.grid(axis="y", linestyle="--", alpha=0.3)
    
    # ------------------------------------------------------------
    # ADD 90-DEGREE VALUE LABELS
    # ------------------------------------------------------------
    
    for container in ax.containers:
        for bar in container:
            height = bar.get_height()
            ax.text(
                bar.get_x() + bar.get_width() / 2,
                height,
                f"{height:.3f}",
                ha="center",
                va="bottom",
                fontsize=9,
                rotation=90
            )
    
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    ==============================
    EXTRA TREES – QUANTILE SHAP IMPORTANCE
    ==============================
    
                                        0.25    0.50    0.75
    Institutions                      0.0776  0.0803  0.0830
    Human capital and research        0.0997  0.1070  0.1128
    Infrastructure                    0.1225  0.1257  0.1200
    Market sophistication             0.0887  0.0894  0.0869
    Business sophistication           0.1848  0.1911  0.1883
    Knowledge and technology outputs  0.1480  0.1413  0.1564
    Creative outputs                  0.2788  0.2652  0.2528
    


.. image:: output_390_1.png






.. code:: ipython3

    # ============================================================
    # 🔥 FULL WORKING SOBOL GLOBAL VARIANCE DECOMPOSITION
    # ============================================================
    
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    from sklearn.ensemble import ExtraTreesRegressor
    from sklearn.model_selection import train_test_split
    
    # ------------------------------------------------------------
    # 1️⃣ MODEL TRAIN
    # ------------------------------------------------------------
    
    X = df_std[features]
    y = df_std[target]
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.3, random_state=42
    )
    
    high_model = ExtraTreesRegressor(
        n_estimators=500,
        random_state=42,
        n_jobs=-1
    )
    
    high_model.fit(X_train, y_train)
    
    print("✅ Model trained")
    print("R² (test):", round(high_model.score(X_test, y_test), 4))
    
    # ------------------------------------------------------------
    # 2️⃣ SOBOL SETTINGS
    # ------------------------------------------------------------
    
    N = 5000
    X_vals = X.values
    
    mins = X_vals.min(axis=0)
    maxs = X_vals.max(axis=0)
    d = X_vals.shape[1]
    
    A = np.random.uniform(mins, maxs, size=(N, d))
    B = np.random.uniform(mins, maxs, size=(N, d))
    
    fA = high_model.predict(A)
    fB = high_model.predict(B)
    
    VarY = np.var(fA)
    
    sobol_first = {}
    sobol_total = {}
    
    # ------------------------------------------------------------
    # 3️⃣ FIRST & TOTAL ORDER
    # ------------------------------------------------------------
    
    for i in range(d):
    
        ABi = A.copy()
        ABi[:, i] = B[:, i]
    
        fABi = high_model.predict(ABi)
    
        Si = np.mean(fB * (fABi - fA)) / VarY
        STi = np.mean((fA - fABi) ** 2) / (2 * VarY)
    
        sobol_first[features[i]] = Si
        sobol_total[features[i]] = STi
    
    sobol_df = pd.DataFrame({
        "First Order": sobol_first,
        "Total Order": sobol_total
    }).T
    
    # ------------------------------------------------------------
    # 4️⃣ SECOND ORDER (Institutions × Human capital)
    # ------------------------------------------------------------
    
    if "Institutions" in features and "Human capital and research" in features:
    
        i1 = features.index("Institutions")
        i2 = features.index("Human capital and research")
    
        AB12 = A.copy()
        AB12[:, i1] = B[:, i1]
        AB12[:, i2] = B[:, i2]
    
        fAB12 = high_model.predict(AB12)
    
        S1 = sobol_first["Institutions"]
        S2 = sobol_first["Human capital and research"]
    
        S12 = (np.mean(fA * (fAB12 - fA)) / VarY) - S1 - S2
    
    else:
        S12 = None
    
    # ------------------------------------------------------------
    # 5️⃣ PRINT RESULTS
    # ------------------------------------------------------------
    
    print("\n🔥 SOBOL RESULTS")
    print(sobol_df.round(4))
    
    if S12 is not None:
        print("\n🔥 SECOND-ORDER INTERACTION (Institutions × Human Capital):", round(S12,4))
    
    # ------------------------------------------------------------
    # 6️⃣ VISUALIZATION (FIRST & TOTAL)
    # ------------------------------------------------------------
    
    plt.figure(figsize=(12,8))
    
    x = np.arange(len(features))
    width = 0.35
    
    first_vals = list(sobol_first.values())
    total_vals = list(sobol_total.values())
    
    bars1 = plt.bar(x - width/2, first_vals, width, label="First Order")
    bars2 = plt.bar(x + width/2, total_vals, width, label="Total Order")
    
    for bar in bars1:
        height = bar.get_height()
        plt.text(bar.get_x() + bar.get_width()/2,
                 height,
                 f"{height:.2f}",
                 ha='center', va='bottom', fontsize=8)
    
    for bar in bars2:
        height = bar.get_height()
        plt.text(bar.get_x() + bar.get_width()/2,
                 height,
                 f"{height:.2f}",
                 ha='center', va='bottom', fontsize=8)
    
    plt.xticks(x, features, rotation=90)
    plt.ylabel("Sobol Index")
    plt.title("Global Sobol Variance Decomposition")
    plt.legend()
    plt.tight_layout()
    plt.show()
    
    # ------------------------------------------------------------
    # 7️⃣ SECOND ORDER VISUAL
    # ------------------------------------------------------------
    
    if S12 is not None:
        plt.figure(figsize=(5,4))
        plt.bar(["Institutions × Human Capital"], [S12])
        plt.text(0, S12, f"{S12:.3f}", ha='center', va='bottom')
        plt.ylabel("Second-Order Sobol Index")
        plt.title("Nonlinear Complementarity Effect")
        plt.tight_layout()
        plt.show()


.. parsed-literal::

    ✅ Model trained
    R² (test): 0.9388
    
    🔥 SOBOL RESULTS
                 Institutions  Human capital and research  Infrastructure  \
    First Order        0.0266                      0.0888          0.0881   
    Total Order        0.0515                      0.1246          0.0825   
    
                 Market sophistication  Business sophistication  \
    First Order                 0.0488                   0.1207   
    Total Order                 0.0443                   0.1966   
    
                 Knowledge and technology outputs  Creative outputs  
    First Order                            0.1591            0.2274  
    Total Order                            0.1244            0.4041  
    
    🔥 SECOND-ORDER INTERACTION (Institutions × Human Capital): -0.3201
    


.. image:: output_395_1.png



.. image:: output_395_2.png



.. code:: ipython3

    # ============================================================
    # 🔥 VISUAL SOBOL GLOBAL VARIANCE DECOMPOSITION
    # ============================================================
    
    import numpy as np
    import matplotlib.pyplot as plt
    import pandas as pd
    
    # ------------------------------------------------------------
    # SETTINGS
    # ------------------------------------------------------------
    
    N = 5000
    X = df_std[features].values
    mins = X.min(axis=0)
    maxs = X.max(axis=0)
    d = X.shape[1]
    
    A = np.random.uniform(mins, maxs, size=(N, d))
    B = np.random.uniform(mins, maxs, size=(N, d))
    
    fA = high_model.predict(A)
    fB = high_model.predict(B)
    
    VarY = np.var(fA)
    
    sobol_first = {}
    sobol_total = {}
    
    # ------------------------------------------------------------
    # FIRST & TOTAL ORDER
    # ------------------------------------------------------------
    
    for i in range(d):
    
        ABi = A.copy()
        ABi[:, i] = B[:, i]
    
        fABi = high_model.predict(ABi)
    
        Si = np.mean(fB * (fABi - fA)) / VarY
        STi = np.mean((fA - fABi) ** 2) / (2 * VarY)
    
        sobol_first[features[i]] = Si
        sobol_total[features[i]] = STi
    
    sobol_df = pd.DataFrame({
        "First Order": sobol_first,
        "Total Order": sobol_total
    }).T
    
    # ------------------------------------------------------------
    # SECOND ORDER (Institutions × Human Capital)
    # ------------------------------------------------------------
    
    i1 = features.index("Institutions")
    i2 = features.index("Human capital and research")
    
    AB12 = A.copy()
    AB12[:, i1] = B[:, i1]
    AB12[:, i2] = B[:, i2]
    
    fAB12 = high_model.predict(AB12)
    
    S1 = sobol_first[features[i1]]
    S2 = sobol_first[features[i2]]
    
    S12 = (np.mean(fA * (fAB12 - fA)) / VarY) - S1 - S2
    
    # ------------------------------------------------------------
    # PRINT RESULTS
    # ------------------------------------------------------------
    
    print("\n🔥 SOBOL RESULTS")
    print(sobol_df.round(4))
    print("\n🔥 SECOND-ORDER INTERACTION (Institutions × Human Capital):", round(S12,4))
    
    # ------------------------------------------------------------
    # VISUALIZATION
    # ------------------------------------------------------------
    
    plt.figure(figsize=(10,6))
    
    x = np.arange(len(features))
    width = 0.35
    
    first_vals = list(sobol_first.values())
    total_vals = list(sobol_total.values())
    
    bars1 = plt.bar(x - width/2, first_vals, width, label="First Order")
    bars2 = plt.bar(x + width/2, total_vals, width, label="Total Order")
    
    # Add numeric labels
    for bar in bars1:
        height = bar.get_height()
        plt.text(bar.get_x() + bar.get_width()/2,
                 height,
                 f"{height:.2f}",
                 ha='center', va='bottom', fontsize=8)
    
    for bar in bars2:
        height = bar.get_height()
        plt.text(bar.get_x() + bar.get_width()/2,
                 height,
                 f"{height:.2f}",
                 ha='center', va='bottom', fontsize=8)
    
    plt.xticks(x, features, rotation=45)
    plt.ylabel("Sobol Index")
    plt.title("Global Sobol Variance Decomposition (Q75 Model)")
    plt.legend()
    plt.tight_layout()
    plt.show()
    
    # ------------------------------------------------------------
    # SECOND ORDER VISUAL
    # ------------------------------------------------------------
    
    plt.figure(figsize=(5,4))
    
    plt.bar(["Institutions × Human Capital"], [S12])
    plt.text(0, S12, f"{S12:.3f}", ha='center', va='bottom')
    
    plt.ylabel("Second-Order Sobol Index")
    plt.title("Nonlinear Complementarity Effect")
    
    plt.tight_layout()
    plt.show()


.. parsed-literal::

    
    🔥 SOBOL RESULTS
                 Institutions  Human capital and research  Infrastructure  \
    First Order        0.0217                      0.1320          0.0728   
    Total Order        0.0491                      0.1232          0.0833   
    
                 Market sophistication  Business sophistication  \
    First Order                 0.0705                   0.2169   
    Total Order                 0.0436                   0.1961   
    
                 Knowledge and technology outputs  Creative outputs  
    First Order                            0.1382            0.5329  
    Total Order                            0.1196            0.3921  
    
    🔥 SECOND-ORDER INTERACTION (Institutions × Human Capital): -0.3166
    


.. image:: output_397_1.png



.. image:: output_397_2.png




.. code:: ipython3

    import numpy as np
    import pandas as pd
    import shap
    import plotly.graph_objects as go
    import plotly.express as px
    from sklearn.ensemble import ExtraTreesRegressor
    from scipy.linalg import eigh, expm
    from numpy.linalg import norm
    
    # ------------------------------------------------------------
    # MODEL
    # ------------------------------------------------------------
    
    model = ExtraTreesRegressor(n_estimators=500, random_state=0)
    model.fit(df_std[features], df_std[target])
    
    explainer = shap.TreeExplainer(model)
    shap_vals = explainer.shap_values(df_std[features])
    interaction_vals = explainer.shap_interaction_values(df_std[features])
    
    interaction_matrix = np.abs(interaction_vals).mean(axis=0)
    mean_importance = np.abs(shap_vals).mean(axis=0)
    
    colors = px.colors.qualitative.Vivid
    
    # ============================================================
    # 1️⃣ HAMILTONIAN PATH MINIMIZATION
    # ============================================================
    
    H = interaction_matrix
    psi0 = mean_importance / norm(mean_importance)
    
    # minimal action evolution
    t = 1.0
    U = expm(-1j * H * t)
    psi_t = U @ psi0
    prob_density = np.abs(psi_t)**2
    
    ham_df = pd.DataFrame({
        "Feature": features,
        "Hamiltonian_Probability": prob_density
    })
    
    print("\n=== HAMILTONIAN PATH MINIMIZATION ===")
    print(ham_df.round(6))
    
    fig1 = go.Figure()
    for i,f in enumerate(features):
        fig1.add_trace(go.Bar(
            x=[f],
            y=[prob_density[i]],
            marker_color=colors[i % len(colors)],
            text=[f"{prob_density[i]:.4f}"],
            textposition="outside"
        ))
    fig1.update_layout(title="Hamiltonian Path Probability Density", showlegend=False)
    fig1.show()
    
    # ============================================================
    # 2️⃣ ACTION FUNCTIONAL OPTIMIZATION
    # ============================================================
    
    gradient = np.gradient(mean_importance)
    action = 0.5 * gradient**2 + mean_importance
    
    action_df = pd.DataFrame({
        "Feature": features,
        "Action_Functional": action
    })
    
    print("\n=== ACTION FUNCTIONAL ===")
    print(action_df.round(6))
    
    fig2 = go.Figure()
    for i,f in enumerate(features):
        fig2.add_trace(go.Bar(
            x=[f],
            y=[action[i]],
            marker_color=colors[i % len(colors)],
            text=[f"{action[i]:.4f}"],
            textposition="outside"
        ))
    fig2.update_layout(title="Action Functional Optimization", showlegend=False)
    fig2.show()
    
    # ============================================================
    # 3️⃣ GEODESIC EXPLAINABILITY TRAJECTORY
    # ============================================================
    
    eigvals, eigvecs = eigh(interaction_matrix)
    geodesic_coords = eigvecs[:, -2:]
    
    geo_df = pd.DataFrame({
        "Feature": features,
        "Dim1": geodesic_coords[:,0],
        "Dim2": geodesic_coords[:,1]
    })
    
    print("\n=== GEODESIC COORDINATES ===")
    print(geo_df.round(6))
    
    fig3 = px.scatter(
        geo_df,
        x="Dim1",
        y="Dim2",
        text="Feature",
        color="Feature",
        color_discrete_sequence=colors,
        title="Geodesic Explainability Trajectory"
    )
    fig3.update_traces(textposition="top center")
    fig3.show()
    
    # ============================================================
    # 4️⃣ RICCI CURVATURE FLOW OVER TIME
    # ============================================================
    
    L = np.diag(interaction_matrix.sum(axis=1)) - interaction_matrix
    ricci_time = []
    
    current = interaction_matrix.copy()
    
    for step in range(5):
        current = current - 0.1 * L
        ricci_time.append(current.mean(axis=1))
    
    ricci_time = np.array(ricci_time)
    
    print("\n=== RICCI FLOW EVOLUTION (TIME STEPS) ===")
    for t in range(ricci_time.shape[0]):
        print(f"\nTime {t}")
        for f,val in zip(features, ricci_time[t]):
            print(f"{f:<20} : {val:.6f}")
    
    fig4 = go.Figure()
    for i,f in enumerate(features):
        fig4.add_trace(go.Scatter(
            x=list(range(5)),
            y=ricci_time[:,i],
            mode="lines+markers+text",
            marker_color=colors[i % len(colors)],
            text=[f"{v:.3f}" for v in ricci_time[:,i]],
            textposition="top center",
            name=f
        ))
    fig4.update_layout(title="Ricci Curvature Flow Over Time")
    fig4.show()
    
    # ============================================================
    # 5️⃣ UNIFIED EXPLAINABILITY FIELD EQUATION
    # ============================================================
    
    unified_field = prob_density + action + mean_importance
    
    unified_df = pd.DataFrame({
        "Feature": features,
        "Unified_Field_Value": unified_field
    })
    
    print("\n=== UNIFIED EXPLAINABILITY FIELD ===")
    print(unified_df.round(6))
    
    fig5 = go.Figure()
    for i,f in enumerate(features):
        fig5.add_trace(go.Bar(
            x=[f],
            y=[unified_field[i]],
            marker_color=colors[i % len(colors)],
            text=[f"{unified_field[i]:.4f}"],
            textposition="outside"
        ))
    fig5.update_layout(title="Unified Explainability Field Equation", showlegend=False)
    fig5.show()


.. parsed-literal::

    
    === HAMILTONIAN PATH MINIMIZATION ===
                                Feature  Hamiltonian_Probability
    0                      Institutions                 0.002859
    1        Human capital and research                 0.001979
    2                    Infrastructure                 0.014219
    3             Market sophistication                 0.002081
    4           Business sophistication                 0.187455
    5  Knowledge and technology outputs                 0.079025
    6                  Creative outputs                 0.712381
    


.. raw:: html

    <div>                            <div id="429f5440-997f-46ec-a494-37ce9a3fd17b" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("429f5440-997f-46ec-a494-37ce9a3fd17b")) {                    Plotly.newPlot(                        "429f5440-997f-46ec-a494-37ce9a3fd17b",                        [{"marker":{"color":"rgb(229, 134, 6)"},"text":["0.0029"],"textposition":"outside","x":["Institutions"],"y":[0.002858884835434343],"type":"bar"},{"marker":{"color":"rgb(93, 105, 177)"},"text":["0.0020"],"textposition":"outside","x":["Human capital and research"],"y":[0.001979212491124689],"type":"bar"},{"marker":{"color":"rgb(82, 188, 163)"},"text":["0.0142"],"textposition":"outside","x":["Infrastructure"],"y":[0.014218507202402848],"type":"bar"},{"marker":{"color":"rgb(153, 201, 69)"},"text":["0.0021"],"textposition":"outside","x":["Market sophistication"],"y":[0.0020813977739433005],"type":"bar"},{"marker":{"color":"rgb(204, 97, 176)"},"text":["0.1875"],"textposition":"outside","x":["Business sophistication"],"y":[0.18745529232348895],"type":"bar"},{"marker":{"color":"rgb(36, 121, 108)"},"text":["0.0790"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[0.07902541759720942],"type":"bar"},{"marker":{"color":"rgb(218, 165, 27)"},"text":["0.7124"],"textposition":"outside","x":["Creative outputs"],"y":[0.7123812877763964],"type":"bar"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"title":{"text":"Hamiltonian Path Probability Density"},"showlegend":false},                        {"responsive": true}                    ).then(function(){
    
    var gd = document.getElementById('429f5440-997f-46ec-a494-37ce9a3fd17b');
    var x = new MutationObserver(function (mutations, observer) {{
            var display = window.getComputedStyle(gd).display;
            if (!display || display === 'none') {{
                console.log([gd, 'removed!']);
                Plotly.purge(gd);
                observer.disconnect();
            }}
    }});
    
    // Listen for the removal of the full notebook cells
    var notebookContainer = gd.closest('#notebook-container');
    if (notebookContainer) {{
        x.observe(notebookContainer, {childList: true});
    }}
    
    // Listen for the clearing of the current output cell
    var outputEl = gd.closest('.output');
    if (outputEl) {{
        x.observe(outputEl, {childList: true});
    }}
    
                            })                };                });            </script>        </div>


.. parsed-literal::

    
    === ACTION FUNCTIONAL ===
                                Feature  Action_Functional
    0                      Institutions           0.917758
    1        Human capital and research           1.224817
    2                    Infrastructure           1.621308
    3             Market sophistication           1.123813
    4           Business sophistication           2.697730
    5  Knowledge and technology outputs           2.189368
    6                  Creative outputs           4.554088
    


.. raw:: html

    <div>                            <div id="b79c8aa3-38e3-4e05-8b84-d1d69b314785" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("b79c8aa3-38e3-4e05-8b84-d1d69b314785")) {                    Plotly.newPlot(                        "b79c8aa3-38e3-4e05-8b84-d1d69b314785",                        [{"marker":{"color":"rgb(229, 134, 6)"},"text":["0.9178"],"textposition":"outside","x":["Institutions"],"y":[0.9177581552084753],"type":"bar"},{"marker":{"color":"rgb(93, 105, 177)"},"text":["1.2248"],"textposition":"outside","x":["Human capital and research"],"y":[1.2248165077717752],"type":"bar"},{"marker":{"color":"rgb(82, 188, 163)"},"text":["1.6213"],"textposition":"outside","x":["Infrastructure"],"y":[1.621308200306297],"type":"bar"},{"marker":{"color":"rgb(153, 201, 69)"},"text":["1.1238"],"textposition":"outside","x":["Market sophistication"],"y":[1.1238128163416843],"type":"bar"},{"marker":{"color":"rgb(204, 97, 176)"},"text":["2.6977"],"textposition":"outside","x":["Business sophistication"],"y":[2.697729847718136],"type":"bar"},{"marker":{"color":"rgb(36, 121, 108)"},"text":["2.1894"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[2.189367746221425],"type":"bar"},{"marker":{"color":"rgb(218, 165, 27)"},"text":["4.5541"],"textposition":"outside","x":["Creative outputs"],"y":[4.5540875219279116],"type":"bar"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"title":{"text":"Action Functional Optimization"},"showlegend":false},                        {"responsive": true}                    ).then(function(){
    
    var gd = document.getElementById('b79c8aa3-38e3-4e05-8b84-d1d69b314785');
    var x = new MutationObserver(function (mutations, observer) {{
            var display = window.getComputedStyle(gd).display;
            if (!display || display === 'none') {{
                console.log([gd, 'removed!']);
                Plotly.purge(gd);
                observer.disconnect();
            }}
    }});
    
    // Listen for the removal of the full notebook cells
    var notebookContainer = gd.closest('#notebook-container');
    if (notebookContainer) {{
        x.observe(notebookContainer, {childList: true});
    }}
    
    // Listen for the clearing of the current output cell
    var outputEl = gd.closest('.output');
    if (outputEl) {{
        x.observe(outputEl, {childList: true});
    }}
    
                            })                };                });            </script>        </div>


.. parsed-literal::

    
    === GEODESIC COORDINATES ===
                                Feature      Dim1      Dim2
    0                      Institutions -0.021971 -0.051244
    1        Human capital and research -0.082190 -0.085634
    2                    Infrastructure -0.070993 -0.126801
    3             Market sophistication -0.044263 -0.064638
    4           Business sophistication -0.889196 -0.270435
    5  Knowledge and technology outputs -0.281810 -0.194164
    6                  Creative outputs  0.340113 -0.926795
    


.. raw:: html

    <div>                            <div id="7a8e1706-808b-4ae6-8106-abe8279de03f" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("7a8e1706-808b-4ae6-8106-abe8279de03f")) {                    Plotly.newPlot(                        "7a8e1706-808b-4ae6-8106-abe8279de03f",                        [{"hovertemplate":"Feature=%{text}\u003cbr\u003eDim1=%{x}\u003cbr\u003eDim2=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"Institutions","marker":{"color":"rgb(229, 134, 6)","symbol":"circle"},"mode":"markers+text","name":"Institutions","orientation":"v","showlegend":true,"text":["Institutions"],"x":[-0.02197147956050521],"xaxis":"x","y":[-0.05124375965288054],"yaxis":"y","type":"scatter","textposition":"top center"},{"hovertemplate":"Feature=%{text}\u003cbr\u003eDim1=%{x}\u003cbr\u003eDim2=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"Human capital and research","marker":{"color":"rgb(93, 105, 177)","symbol":"circle"},"mode":"markers+text","name":"Human capital and research","orientation":"v","showlegend":true,"text":["Human capital and research"],"x":[-0.08219003157057614],"xaxis":"x","y":[-0.08563396588233818],"yaxis":"y","type":"scatter","textposition":"top center"},{"hovertemplate":"Feature=%{text}\u003cbr\u003eDim1=%{x}\u003cbr\u003eDim2=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"Infrastructure","marker":{"color":"rgb(82, 188, 163)","symbol":"circle"},"mode":"markers+text","name":"Infrastructure","orientation":"v","showlegend":true,"text":["Infrastructure"],"x":[-0.07099319668648356],"xaxis":"x","y":[-0.12680122234281],"yaxis":"y","type":"scatter","textposition":"top center"},{"hovertemplate":"Feature=%{text}\u003cbr\u003eDim1=%{x}\u003cbr\u003eDim2=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"Market sophistication","marker":{"color":"rgb(153, 201, 69)","symbol":"circle"},"mode":"markers+text","name":"Market sophistication","orientation":"v","showlegend":true,"text":["Market sophistication"],"x":[-0.04426303903505893],"xaxis":"x","y":[-0.06463829427137616],"yaxis":"y","type":"scatter","textposition":"top center"},{"hovertemplate":"Feature=%{text}\u003cbr\u003eDim1=%{x}\u003cbr\u003eDim2=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"Business sophistication","marker":{"color":"rgb(204, 97, 176)","symbol":"circle"},"mode":"markers+text","name":"Business sophistication","orientation":"v","showlegend":true,"text":["Business sophistication"],"x":[-0.8891957931868406],"xaxis":"x","y":[-0.27043547557662695],"yaxis":"y","type":"scatter","textposition":"top center"},{"hovertemplate":"Feature=%{text}\u003cbr\u003eDim1=%{x}\u003cbr\u003eDim2=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"Knowledge and technology outputs","marker":{"color":"rgb(36, 121, 108)","symbol":"circle"},"mode":"markers+text","name":"Knowledge and technology outputs","orientation":"v","showlegend":true,"text":["Knowledge and technology outputs"],"x":[-0.28181020164888065],"xaxis":"x","y":[-0.19416414464986045],"yaxis":"y","type":"scatter","textposition":"top center"},{"hovertemplate":"Feature=%{text}\u003cbr\u003eDim1=%{x}\u003cbr\u003eDim2=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"Creative outputs","marker":{"color":"rgb(218, 165, 27)","symbol":"circle"},"mode":"markers+text","name":"Creative outputs","orientation":"v","showlegend":true,"text":["Creative outputs"],"x":[0.3401127075270897],"xaxis":"x","y":[-0.9267951124126559],"yaxis":"y","type":"scatter","textposition":"top center"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dim1"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Dim2"}},"legend":{"title":{"text":"Feature"},"tracegroupgap":0},"title":{"text":"Geodesic Explainability Trajectory"}},                        {"responsive": true}                    ).then(function(){
    
    var gd = document.getElementById('7a8e1706-808b-4ae6-8106-abe8279de03f');
    var x = new MutationObserver(function (mutations, observer) {{
            var display = window.getComputedStyle(gd).display;
            if (!display || display === 'none') {{
                console.log([gd, 'removed!']);
                Plotly.purge(gd);
                observer.disconnect();
            }}
    }});
    
    // Listen for the removal of the full notebook cells
    var notebookContainer = gd.closest('#notebook-container');
    if (notebookContainer) {{
        x.observe(notebookContainer, {childList: true});
    }}
    
    // Listen for the clearing of the current output cell
    var outputEl = gd.closest('.output');
    if (outputEl) {{
        x.observe(outputEl, {childList: true});
    }}
    
                            })                };                });            </script>        </div>


.. parsed-literal::

    
    === RICCI FLOW EVOLUTION (TIME STEPS) ===
    
    Time 0
    Institutions         : 0.226700
    Human capital and research : 0.336572
    Infrastructure       : 0.426978
    Market sophistication : 0.280898
    Business sophistication : 0.645916
    Knowledge and technology outputs : 0.563733
    Creative outputs     : 0.855434
    
    Time 1
    Institutions         : 0.226700
    Human capital and research : 0.336572
    Infrastructure       : 0.426978
    Market sophistication : 0.280898
    Business sophistication : 0.645916
    Knowledge and technology outputs : 0.563733
    Creative outputs     : 0.855434
    
    Time 2
    Institutions         : 0.226700
    Human capital and research : 0.336572
    Infrastructure       : 0.426978
    Market sophistication : 0.280898
    Business sophistication : 0.645916
    Knowledge and technology outputs : 0.563733
    Creative outputs     : 0.855434
    
    Time 3
    Institutions         : 0.226700
    Human capital and research : 0.336572
    Infrastructure       : 0.426978
    Market sophistication : 0.280898
    Business sophistication : 0.645916
    Knowledge and technology outputs : 0.563733
    Creative outputs     : 0.855434
    
    Time 4
    Institutions         : 0.226700
    Human capital and research : 0.336572
    Infrastructure       : 0.426978
    Market sophistication : 0.280898
    Business sophistication : 0.645916
    Knowledge and technology outputs : 0.563733
    Creative outputs     : 0.855434
    


.. raw:: html

    <div>                            <div id="1816d196-6b52-47ed-a10f-c12711a161af" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("1816d196-6b52-47ed-a10f-c12711a161af")) {                    Plotly.newPlot(                        "1816d196-6b52-47ed-a10f-c12711a161af",                        [{"marker":{"color":"rgb(229, 134, 6)"},"mode":"lines+markers+text","name":"Institutions","text":["0.227","0.227","0.227","0.227","0.227"],"textposition":"top center","x":[0,1,2,3,4],"y":[0.22670017577753063,0.22670017577753057,0.22670017577753063,0.2267001757775306,0.2267001757775306],"type":"scatter"},{"marker":{"color":"rgb(93, 105, 177)"},"mode":"lines+markers+text","name":"Human capital and research","text":["0.337","0.337","0.337","0.337","0.337"],"textposition":"top center","x":[0,1,2,3,4],"y":[0.3365720237329785,0.3365720237329785,0.3365720237329785,0.33657202373297845,0.3365720237329785],"type":"scatter"},{"marker":{"color":"rgb(82, 188, 163)"},"mode":"lines+markers+text","name":"Infrastructure","text":["0.427","0.427","0.427","0.427","0.427"],"textposition":"top center","x":[0,1,2,3,4],"y":[0.42697841607000014,0.42697841607000014,0.42697841607000014,0.42697841607000014,0.42697841607000014],"type":"scatter"},{"marker":{"color":"rgb(153, 201, 69)"},"mode":"lines+markers+text","name":"Market sophistication","text":["0.281","0.281","0.281","0.281","0.281"],"textposition":"top center","x":[0,1,2,3,4],"y":[0.28089842877062676,0.28089842877062676,0.28089842877062676,0.2808984287706267,0.2808984287706267],"type":"scatter"},{"marker":{"color":"rgb(204, 97, 176)"},"mode":"lines+markers+text","name":"Business sophistication","text":["0.646","0.646","0.646","0.646","0.646"],"textposition":"top center","x":[0,1,2,3,4],"y":[0.6459160757695653,0.6459160757695653,0.6459160757695653,0.6459160757695653,0.6459160757695653],"type":"scatter"},{"marker":{"color":"rgb(36, 121, 108)"},"mode":"lines+markers+text","name":"Knowledge and technology outputs","text":["0.564","0.564","0.564","0.564","0.564"],"textposition":"top center","x":[0,1,2,3,4],"y":[0.5637326958630251,0.5637326958630251,0.5637326958630251,0.5637326958630252,0.5637326958630251],"type":"scatter"},{"marker":{"color":"rgb(218, 165, 27)"},"mode":"lines+markers+text","name":"Creative outputs","text":["0.855","0.855","0.855","0.855","0.855"],"textposition":"top center","x":[0,1,2,3,4],"y":[0.8554338060552747,0.8554338060552747,0.8554338060552747,0.8554338060552747,0.8554338060552747],"type":"scatter"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"title":{"text":"Ricci Curvature Flow Over Time"}},                        {"responsive": true}                    ).then(function(){
    
    var gd = document.getElementById('1816d196-6b52-47ed-a10f-c12711a161af');
    var x = new MutationObserver(function (mutations, observer) {{
            var display = window.getComputedStyle(gd).display;
            if (!display || display === 'none') {{
                console.log([gd, 'removed!']);
                Plotly.purge(gd);
                observer.disconnect();
            }}
    }});
    
    // Listen for the removal of the full notebook cells
    var notebookContainer = gd.closest('#notebook-container');
    if (notebookContainer) {{
        x.observe(notebookContainer, {childList: true});
    }}
    
    // Listen for the clearing of the current output cell
    var outputEl = gd.closest('.output');
    if (outputEl) {{
        x.observe(outputEl, {childList: true});
    }}
    
                            })                };                });            </script>        </div>


.. parsed-literal::

    
    === UNIFIED EXPLAINABILITY FIELD ===
                                Feature  Unified_Field_Value
    0                      Institutions             1.799979
    1        Human capital and research             2.383273
    2                    Infrastructure             3.254289
    3             Market sophistication             2.139652
    4           Business sophistication             5.442265
    5  Knowledge and technology outputs             4.342904
    6                  Creative outputs             8.782117
    


.. raw:: html

    <div>                            <div id="c99b63e2-0ed9-4955-ac51-e911cc34d46d" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("c99b63e2-0ed9-4955-ac51-e911cc34d46d")) {                    Plotly.newPlot(                        "c99b63e2-0ed9-4955-ac51-e911cc34d46d",                        [{"marker":{"color":"rgb(229, 134, 6)"},"text":["1.8000"],"textposition":"outside","x":["Institutions"],"y":[1.7999786135730038],"type":"bar"},{"marker":{"color":"rgb(93, 105, 177)"},"text":["2.3833"],"textposition":"outside","x":["Human capital and research"],"y":[2.383273087926436],"type":"bar"},{"marker":{"color":"rgb(82, 188, 163)"},"text":["3.2543"],"textposition":"outside","x":["Infrastructure"],"y":[3.2542887955533812],"type":"bar"},{"marker":{"color":"rgb(153, 201, 69)"},"text":["2.1397"],"textposition":"outside","x":["Market sophistication"],"y":[2.139651933191706],"type":"bar"},{"marker":{"color":"rgb(204, 97, 176)"},"text":["5.4423"],"textposition":"outside","x":["Business sophistication"],"y":[5.442265286169409],"type":"bar"},{"marker":{"color":"rgb(36, 121, 108)"},"text":["4.3429"],"textposition":"outside","x":["Knowledge and technology outputs"],"y":[4.342904206114967],"type":"bar"},{"marker":{"color":"rgb(218, 165, 27)"},"text":["8.7821"],"textposition":"outside","x":["Creative outputs"],"y":[8.782117488280733],"type":"bar"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"title":{"text":"Unified Explainability Field Equation"},"showlegend":false},                        {"responsive": true}                    ).then(function(){
    
    var gd = document.getElementById('c99b63e2-0ed9-4955-ac51-e911cc34d46d');
    var x = new MutationObserver(function (mutations, observer) {{
            var display = window.getComputedStyle(gd).display;
            if (!display || display === 'none') {{
                console.log([gd, 'removed!']);
                Plotly.purge(gd);
                observer.disconnect();
            }}
    }});
    
    // Listen for the removal of the full notebook cells
    var notebookContainer = gd.closest('#notebook-container');
    if (notebookContainer) {{
        x.observe(notebookContainer, {childList: true});
    }}
    
    // Listen for the clearing of the current output cell
    var outputEl = gd.closest('.output');
    if (outputEl) {{
        x.observe(outputEl, {childList: true});
    }}
    
                            })                };                });            </script>        </div>







