# 🏦 Bank Müştərilərinin Kredit Ödəmə Qabiliyyətinə Görə Klasterləşdirilməsi

Bu repozitoriya bank müştərilərinin kredit ödəmə qabiliyyətini və maliyyə davranışlarını təhlil edərək onları **Nəzarətsiz Maşın Öyrənməsi (*Unsupervised Machine Learning*)** metodları vasitəsilə seqmentlərə (klasterlərə) ayıran və hər seqment üçün kredit siyasəti tövsiyələri çıxaran tam işlək Jupyter Notebook ([`bank_customer_clustering.ipynb`](./bank_customer_clustering.ipynb)) layihəsini əhatə edir.

---

## 📂 Layihənin Strukturu və Metodoloji Mərhələlər

1. **Məlumatın Hazırlanması (*Data Simulation & Preprocessing*):**
   - Bank domeni üçün 1,600 müştəridən ibarət realist sintetik datasetin (`Musteri_ID`, `Ayliq_Gelir`, `Cari_Borc`, `Borc_Gelir_Nisbeti` (DTI), `Gecikme_Gun_Sayi`, `Kredit_Xetti_Sayi`, `Kredit_Skoru_FICO`, `Ayliq_Xerc`) yaradılması.
   - Çatışmayan dəyərlərin (*Missing Values*) median ilə bərpası və kənar qiymətlərin (*Outliers*) IQR (`[Q1 - 1.5*IQR, Q3 + 1.5*IQR]`) metodu ilə təhlili/təmizlənməsi.
   - Məsafəyə əsaslanan alqoritmlər üçün `StandardScaler` ilə Z-score normallaşdırılması ($\mu = 0, \sigma = 1$).
2. **Ölçülərin Azaldılması (*Dimensionality Reduction — PCA & TruncatedSVD*):**
   - `PCA` vasitəsilə izah edilən dispersiya nisbətinin (`explained_variance_ratio_`) fərdi və kümülatif qrafiki (ilk 2 komponent məlumatın **92.73%**-ni saxlayır).
   - Müştərilərin `PC1` və `PC2` oxları üzrə 2D proyeksiya qrafiki və komponent yükləmələrinin (*loadings*) iqtisadi şərhi.
   - `TruncatedSVD` tətbiqi və PCA ilə riyazi/praktiki müqayisəsi.
3. **K-Means Klasterləşdirmə:**
   - $k = 2..10$ aralığında **Elbow Method (WCSS / Inertia)** və **Silhouette Score** təhlili ilə optimal klaster sayının ($k = 4$, Silhouette = `0.7822`) seçilməsi.
   - 2D PCA fəzasında klasterlərin və mərkəz nöqtələrinin (*centroids*) vizuallaşdırılması.
4. **İerarxik Klasterləşdirmə (*Hierarchical / Agglomerative Clustering*):**
   - `scipy.cluster.hierarchy` ilə **Ward Linkage** Dendroqramının qurulması və optimal kəsim həddinin (*horizontal threshold*) qırmızı qırıq xətlə göstərilməsi.
   - `AgglomerativeClustering` nəticələrinin K-Means ilə çarpaz cədvəl, Silhouette əmsalı (`0.7811`) və **Adjusted Rand Index (`ARI = 0.9950`, `99.81%` üst-üstə düşmə)** ilə müqayisəsi.
5. **Bank Riski və Biznes Klaster Profilləri (*Cluster Profiling*):**
   - `df.groupby('Cluster_KMeans').mean()` əsasında klasterlərin orta maliyyə göstəriciləri və Z-score istilik xəritəsi (*heatmap*).
   - Hər seqment üçün rəsmi **Bank Risk Statusu** və **Kredit Komitəsi üçün Siyasət Tövsiyələri** (faiz dərəcəsi, kredit limiti, girov/zamin tələbi, restrukturizasiya).

---

## 📊 Klasterlərin Yekun Biznes Profili

| Klaster | Bank Risk Statusu | Orta Gəlir (AZN) | Orta Cari Borc (AZN) | DTI (%) | Orta Gecikmə (Gün) | Orta FICO | Kredit Siyasəti |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **0** | 🟢 **Aşağı Riskli / Etibarlı (VIP Prime)** | 7,023.89 | 2,494.12 | 12% | 0.19 | 807 | İmtiyazlı faiz (-3%), yüksək pre-approved limit, girovsuz |
| **1** | 🔵 **Orta-Aşağı Riskli / Sabit (Standard)** | 3,198.26 | 9,534.66 | 38% | 5.04 | 689 | Standart faiz dərəcəsi, DTI ≤ 45% şərti ilə limit |
| **2** | 🟠 **Orta Riskli / Borc Yüklü (Watchlist)** | 2,402.90 | 26,043.46 | 68% | 21.63 | 585 | Risk əlavəli faiz (+4%), borcların konsolidasiyası, zamin/girov |
| **3** | 🔴 **Yüksək Riskli / Gecikməli (Subprime)** | 1,094.81 | 14,754.22 | 95% | 85.19 | 430 | Yeni kreditə qadağa (Stop), restrukturizasiya və təminatlı yığım |
