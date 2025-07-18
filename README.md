# sistech-take-home-2

## 🎯 Tujuan & Pendekatan

### 🎯 Tujuan

Tujuan sistem rekomendasi adalah merekomendasikan course yang mirip dengan course 6.867: Machine Learning. Berikut adalah atribut course tersebut:

![Atribut Course 6.867](img/atribut_course.png)
*Atribut course 6.867 yang akan dicari kemiripannya dengan course lain* 

### 🧩 Pendekatan

Digunakan beberapa metode vektorisasi dan similarity, yaitu:

#### Vektorisasi:
- Bag of Words
- TF-IDF
- Word2Vec
- FastText
- Sentence Transformers

#### Similarity:
- Cosine similarity (untuk BoW, TF-IDF, Word2Vec, dan Sentence Transformers)
- Word Mover Distance (untuk Word2Vec dan FastText)

## 📊 Vektorisasi & Similarity


### Vektorisasi

Digunakan metode vektorisasi Bag of Words (BoW), TF-IDF, Word2Vec, FastText, dan Sentence Transformers.

Teks yang divektorisasi adalah penggabungan `title`, `description`, `topics`, dan `department_name` yang sudah di-preprocess. Contoh teks yang akan divektorisasi:
![Teks Yang Akan Divektorisasi](img/to_be_vectorized.png)
*Teks yang akan divektorisasi* 

#### Sumber Data dan Metode

| Metode               | Sumber Data Training                             | Keterangan                                                                 |
|----------------------|--------------------------------------------------|-----------------------------------------------------------------------------|
| **BoW**              | Teks dari semua course                           | -                                                                           |
| **TF-IDF**           | Teks dari semua course                           | -                                                                           |
| **Word2Vec**         | Pre-trained WikiNews 300d                        | Awalnya ingin pakai Google News, tapi ukurannya terlalu besar               |
| **FastText**         | Teks dari semua course                           | Tidak memakai pre-trained Facebook model karena keterbatasan RAM            |
| **SentenceTransformers** | Pre-trained all-MiniLM-L6-v2               | Model ringan dan efisien, cocok untuk semantic similarity                   |


### Similarity

#### Cosine Similarity

Untuk metode BoW, TF-IDF, dan Sentence Transformers digunakan cosine similarity karena 1 kalimat dapat direpresentasikan dalam 1 vektor.

Saya menggunakan cosine similarity karena tidak memperhitungkan panjang dokumen. Ada beberapa teks dalam dokumen yang memiliki panjang cukup berbeda. Meskipun cosine similarity tidak memperhitungkan struktur kata, saya rasa tidak apa-apa karena course matching ini lebih ke lexical similarity, bukan semantic similarity, jadi kita cukup melihat apakah ada keywords yang mirip atau sama dengan course lain atau tidak dan tidak terlalu mempedulikan urutan kata dan strukturnya.


#### Word Mover Distance

Untuk Word2Vec dan FastText, digunakan Word Mover Distance karena 1 kalimat dipresentasikan dengan beberapa vektor.

#### Word2Vec x Cosine Similarity Workaround
Saya juga mencoba menggunakan cosine similarity untuk Word2Vec, namun 1 kalimat harus direpresentasikan dengan 1 vektor saja, sehingga saya membuat customized function untuk merata-rata sekumpulan vektor menjadi satu vektor.

``` python
def avg_vector(vectors:list, n_words:int) -> list:
    # Calculate the average vector from a list of vectors
    vector_sum = np.sum(vectors, axis=0)
    if n_words != 0:
        return np.divide(vector_sum, n_words)
```
*Fungsi untuk menghitung rata-rata dari sekumpulan vektor* 

## 📈 Hasil Rekomendasi

Berikut merupakan hasil rekomendasi dari beberapa metode

### Metode Tradisional (BoW & TF-IDF)
BoW x Cosine:
![Top 10 Similar Courses BoW X Cosine Method](img/bow_cos_result.png)
*Top 10 Similar Courses BoW X Cosine Method*

TF-IDF x Cosine:
![Top 10 Similar Courses TF-IDF X Cosine Method](img/tfidf_cos_result.png)
*Top 10 Similar Courses TF-IDF X Cosine Method*

BoW dan TF-IDF dengan cosine similarity menghasilkan hasil yang baik. Sebagian besar course yang direkomendasikan memiliki kemiripan dengan course asli ("Machine Learning"), ditandai dengan kemunculan kata kunci seperti "Statistics", "Artificial Intelligence", "Data Analysis", dan "Mathematics".

Kedua metode ini menghasilkan hasil yang hampir identik, dengan perbedaan utama hanya pada urutan tingkat kemiripan. Namun, course "Algorithmic Aspects of Machine Learning" muncul dalam metode TF-IDF tetapi tidak dalam BoW. Sebaliknya, course "Techniques in Artificial Intelligence (SMA 5504)" muncul dalam metode BoW tetapi tidak dalam TF-IDF.

### Metode Modern (Word2Vec, FastText, SentenceTransformers)
Word2Vec (Pre-Trained) x WMD:
![Top 10 Similar Courses Word2Vec x WMD](img/w2v_wmd_result.png)
*Top 10 Similar Courses Word2Vec x WMD*

Word2Vec (Pre-Trained) x Cosine:
![Top 10 Similar Courses Word2Vec x Cosine](img/w2v_cos_result.png)
*Top 10 Similar Courses Word2Vec x Cosine*

FastText (Not Pre-Trained) x WMD:
![Top 10 Similar Courses FastText x WMD](img/ft_wmd_result.png)
*Top 10 Similar Courses FastText x WMD*

Metode FastText, bahkan ketika dilatih pada korpus yang relatif kecil, juga menunjukkan performa yang baik dan sebanding dengan model pre-trained seperti W2V dan ST.

SentenceTransformers (Pre-Trained) x Cosine:
![Top 10 Similar Courses ST x Cosine](img/st_cos_result.png)
*Top 10 Similar Courses ST x Cosine*

Semua metode modern menghasilkan rekomendasi yang baik, dengan merekomendasikan course yang serupa dengan course asli ("Machine Learning"). Course-course yang dihasilkan memiliki kata kunci seperti "Statistics", "Artificial Intelligence", "Data Analysis", dan "Mathematics".

Namun, terdapat beberapa course yang direkomendasikan oleh metode modern tetapi tidak muncul dalam metode tradisional, seperti:

* "Pattern Recognition and Analysis" dari ST x Cosine

* "Probabilistic Systems Analysis and Applied Probability" dari W2V x Cosine

* "Algorithms for Inference" dari W2V x WMD

Hal ini menunjukkan bahwa metode modern mampu menangkap konteks yang lebih kaya secara semantik dan tidak hanya bergantung pada kemunculan kata (lexical similarity) seperti pada metode tradisional.

Selain itu, metode Word2Vec dengan similarity methods berbeda (WMD and Cosine) juga menghasilkan rekomendasi yang berbeda. Metode cosine dengan mengambil rata-rata vektor dari beberapa kata untuk satu kalimat tidak mempedulikan posisi kata, sehingga metode WMD lebih baik dalam mempertahankan konteks semantik.

### 🧾 Konklusi

Untuk permasalahan ini, saya percaya bahwa metode tradisional lebih disukai karena menghasilkan performa yang baik dan kecepatan yang tinggi. Karena tugasnya hanya melibatkan kesamaan teks berdasarkan kemunculan kata, serta deskripsi course bersifat deskriptif, maka metode tradisional sudah cukup.

## 🛠️ Kendala & Solusi

### 1. Text similarity Word2Vec & FastText

**Kendala**: Awalnya saya mengira Word2Vec dan FastText dapat menggunakan cosine similarity seperti metode lainnya, namun setelah investigasi lebih lanjut, ternyata dalam kedua metode tersebut, yang divektorisasi adalah kata, bukan kalimat. 

**Solusi**: 
1. Menggunakan Word Mover Distance
2. Membuat vektor kalimat dari rata-rata vektor kata dalam kalimat tersebut, sehingga dapat menggunakan cosine similarity setelahnya

### 2. Memory tidak cukup untuk menggunakan pre-trained Google News dan Facebook Model

**Kendala**: Google News digunakan dengan Word2Vec, sedangkan Facebook digunakan dalam FastText. Namun ketika mencoba run, terjadi memory error.

**Solusi**:
1. Mengganti Google News dengan Wiki News yang lebih kecil
2. Training FastText sendiri dengan data course yang dimiliki: Ternyata hasil FastText yang tidak menggunakan pre-trained model pun menghasilkan hasil rekomendasi yang cukup baik (lihat poin Hasil Rekomendasi)


