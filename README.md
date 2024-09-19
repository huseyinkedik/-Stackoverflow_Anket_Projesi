# StackOverflow 2024 Anket Verileri Analizi

## Proje Açıklaması

Bu proje, 2024 StackOverflow anket sonuçlarını Jupyter Notebook kullanarak analiz etmektedir. Çalışma, yazılım geliştiricilerin programlama dilleri tercihleri, çalışma alışkanlıkları, tazminatları ve eğitim seviyeleri gibi çeşitli konularda derinlemesine bir inceleme sunmaktadır. Analiz sırasında Pandas ve NumPy gibi yaygın veri analizi kütüphaneleri kullanılmıştır. Bu proje, sektördeki güncel trendleri anlamaya ve yazılım dünyasındaki önemli istatistikleri göz önüne sermeye odaklanmaktadır.

## İçindekiler

- [Proje İçeriği](#Proje-İçeriği)
- [Kullanılan Yöntemler ve Açıklamalar](#kullanılan-yöntemler-ve-açıklamalar)
- [Proje Çıktıları](#proje-çıktıları)
- [Gereksinimler](#Gereksinimler)

## Proje İçeriği

### Sorular

1. Ankete kaç kişi katıldı?
2. Kaç kişi tüm soruları yanıtladı?
3. Katılımcıların deneyimi için merkezi eğilim ölçüleri nelerdir?
4. Kaç kişi uzaktan çalışıyor?
5. Katılımcıların yüzde kaçı Python ile programlama yapıyor?
6. Kaç kişi online kurslar ile programlama öğrendi?
7. Ülkelere göre gruplanmış ve Python ile programlama yapan katılımcılar arasında, her ülkede ortalama ve medyan yıllık tazminat ne kadardır?
8. En yüksek tazminata sahip 5 katılımcının eğitim seviyeleri nelerdir?

### Bonus Sorular

- Her yaş grubunda Python ile programlama yapan katılımcıların yüzdesi nedir?
- Ortalama tazminatta 75. yüzdelik dilimde yer alan ve uzaktan çalışan katılımcılar arasında en yaygın endüstriler nelerdir?

## Kullanılan Yöntemler ve Açıklamalar

1. **Ankete kaç kişi katıldı**   
   Verilerle çalışırken ilk adım, veri setini belleğe aktarmaktır. Pandas kütüphanesi, bu tür işlemler için `read_csv()` fonksiyonunu sağlar. Bu fonksiyon bir CSV dosyasını okuyup Pandas'ın DataFrame formatına çevirir. DataFrame, verileri tablosal bir biçimde (satır ve sütunlar) saklar ve veri analiz işlemlerini oldukça kolaylaştırır. Örneğin, bir tabloyu SQL'de sorgular gibi kullanabiliriz.

   ```python
   df = pd.read_csv(r"C:\Users\husey\Downloads\survey_results_public.csv")
   schema = pd.read_csv(r"C:\Users\husey\Downloads\survey_results_schema.csv")
   print(f"the_number_of_survey_respondents: {df.shape[0]}")
      
   #  Ankete katılan kişi sayısı: 65437
   ```

   Bu kod parçası, CSV dosyasını okur ve anket verilerini bir Pandas DataFrame'e aktarır. `shape` fonksiyonu, verinin kaç satır ve sütundan oluştuğunu gösterir. `shape[0]` satır sayısını, yani ankete katılan kişi sayısını döndürür. Bu basit adım, veri setiyle çalışmaya başlamadan önce kaç kişinin anketi yanıtladığını anlamamızı sağlar.

2. **Kaç Kişi Tüm Soruları Yanıtladı?**  
   Bir veri setinde her zaman eksik veriler olabilir. Özellikle anketlerde bazı katılımcılar tüm soruları yanıtlamayabilir. Eksik verileri tespit edip onlarla nasıl başa çıkacağımıza karar vermek önemlidir. Pandas'ın `dropna()` fonksiyonu, eksik değer (NaN) içeren satırları veri setinden çıkarır. Burada tüm soruları yanıtlamış olan katılımcıları bulmak için bu fonksiyonu kullanıyoruz.
   İlk olarak, `schema.qname.unique()` komutu, anket formunda bulunan tüm soruların listesini alır. Daha sonra bu liste, veri setindeki mevcut sütun isimleriyle karşılaştırılır `(set(df.columns))`. Bu, hangi soruların veri setinde bulunduğunu ve eksik olanların hangileri olduğunu belirler.
   Ardından, `df.dropna(subset=questions)` komutu kullanılarak, yanıtları eksik olan satırlar veri setinden çıkarılır. Burada `subset=questions` ifadesi, sadece belirli soruları kontrol etmek için kullanılır. Son olarak, `.shape[0]` ile, eksiksiz yanıt veren katılımcıların toplam sayısı hesaplanır. Tüm verilerin eksiksiz olduğu bir alt küme elde etmek, daha güvenilir analizler yapmamıza olanak tanır. 
   ```python
   questions = set(schema.qname.unique()) & set(df.columns)
   df.dropna(subset=questions).shape[0]

   #Tüm Soruları Yanıtlayan Kişi Sayısı: 6306
   ```

3. **Merkezi Eğilim Ölçüleri: Ortalama (mean) ve Medyan (median)**  
   Veri setindeki tipik değeri bulmak için merkezi eğilim ölçüleri (ortalama ve medyan) kullanılır. `mean()` fonksiyonu, aritmetik ortalamayı, yani tüm değerlerin toplamının veri sayısına bölünmesiyle elde edilen sonucu verir. `median()` ise ortanca değeri, yani veri setini sıraladığımızda ortada kalan değeri gösterir. Aritmetik ortalama aşırı uç değerlerden etkilenebilirken, medyan bu tür etkilerden kaçınmamızı sağlar.

   ```python
   print(df["WorkExp"].median())
   print(round(df["WorkExp"].mean(),1))

   #Medyan: 9.0
   #Ortalama: 11.5
   ```

   `WorkExp` sütunundaki ortalama ve medyan deneyim değerlerini hesaplayarak, yazılım geliştiriciler arasında ne kadar deneyime sahip olunduğunu belirleriz. Bu ölçümler, ortalama deneyim süresini ve deneyim dağılımının ortanca noktasını anlamamıza yardımcı olur.
   
4. **Uzaktan Çalışan Kişi Sayısı**   
   Uzaktan çalışanların sayısını bulmak için, ilgili sütunu filtreleyip sayıyoruz. Bu analiz, iş dünyasındaki uzaktan çalışma trendini anlamamıza yardımcı olur.
   
   ```python
   remote_workers = (df["RemoteWork"] =="Remote").sum()
   print(f"remote_workers: {remote_workers}")

   #Uzaktan Çalışan Kişi Sayısı: 20831
   ```

5. **Python Kullanıcılarının Yüzdesi**  
   İlk olarak, `df['LanguageHaveWorkedWith'].str.contains('Python', case=False, na=False)` komutu, "LanguageHaveWorkedWith" sütununda Python kelimesini arar. `case=False` parametresi, büyük/küçük harf ayrımını yapmadan arama yapılmasını sağlar; yani "Python", "python" ya da "PYTHON" hepsini kapsar. `na=False` ise eksik (NaN) değerlerin yanlış olarak değerlendirilmesini sağlar, böylece eksik değerler arama sonucuna dahil edilmez.
   Bu işlem sonucunda, Python kullanan katılımcıları belirten bir Series (bir tür veri serisi) elde edilir. `python_users.sum()` komutu bu serideki True değerlerinin toplamını hesaplar, yani Python kullanan katılımcıların sayısını verir.
   Sonra, `total_count = len(df)` komutu veri setindeki toplam katılımcı sayısını hesaplar. `python_percentage = (python_count / total_count) * 100` hesaplamasıyla, Python kullanan katılımcıların yüzdesi bulunur. Bu oran, Python kullananların toplam katılımcılara oranını yüzdelik olarak ifade eder.
   Son olarak, `print(f"{python_percentage:.2f} percent of participants use Python")` komutu, hesaplanan yüzdelik değeri iki ondalık basamakla ekrana yazdırır ve böylece Python kullanan katılımcıların oranını net bir şekilde görebiliriz. 

   ```python
   python_users = df['LanguageHaveWorkedWith'].str.contains('Python', case=False, na=False)
   python_count = python_users.sum()
   total_count = len(df)
   python_percentage = (python_count / total_count) * 100
   print(f"{python_percentage:.2f} percent of participants use Python")

   #Python Kullanıcılarının Yüzdesi: 47.06
   ```

6. **Kaç kişi online kurslar ile programlama öğrendi**   
   İlk olarak, `df.LearnCode.apply(lambda x: 1 if 'online courses' in str(x).lower() else 0)` komutu, "LearnCode" sütunundaki her bir değeri değerlendirir. Burada `apply` fonksiyonu, sütundaki her bir hücre için belirli bir işlevi uygulamak amacıyla kullanılır. Bu işlev, lambda fonksiyonu tarafından sağlanır ve her hücrede "online courses" ifadesinin olup olmadığını kontrol eder. `str(x).lower()` ifadesi, hücredeki değeri küçük harflere dönüştürür ve `in 'online courses'` ifadesi bu değerin içinde "online courses" ifadesinin bulunup bulunmadığını kontrol eder. Eğer bu ifade varsa, lambda fonksiyonu 1 döndürür; aksi takdirde 0 döndürür. Sonuç olarak, bu işlemin sonucunda "learned_with_online_courses" adında yeni bir sütun oluşturulur ve her hücreye 1 veya 0 değerleri atanır. 1, online kurslar kullanarak programlama öğrendiğini belirten bir değeri ifade ederken, 0 bu durumu ifade etmez. Son olarak, `df.learned_with_online_courses.sum()` komutu bu yeni sütundaki 1'lerin toplamını hesaplar. Bu, veri setindeki toplam katılımcının online kurslar aracılığıyla programlama öğrendiği sayısını verir. Bu kod, online kursların veri setindeki etkisini ölçmek için kullanılır ve eğitim yöntemlerinin yaygınlığını anlamamıza yardımcı olur.
   
   ```python
   df['learned_with_online_courses'] = df.LearnCode.apply(lambda x: 1 if 'online courses' in str(x).lower() else 0)
   df.learned_with_online_courses.sum()

   #Online kurslar ile programlama öğrenen kişi sayısı: 30271
   ```
7. **Ülkelere göre gruplanmış ve Python ile programlama yapan katılımcılar arasında, her ülkede ortalama ve medyan yıllık tazminat ne kadardır**    
   Python programlama dilini kullanan katılımcılar için ülkeler bazında yıllık tazminatların ortalama ve medyan değerlerini hesaplamak amacıyla kullanılır. İşlemler şu şekilde yapılır:

   İlk olarak, `python_users = df[df['LanguageHaveWorkedWith'].str.contains('python', case=False, na=False)]` kodu, `df` veri çerçevesinde Python dilini kullanan katılımcıları filtreler. Burada, `str.contains('python', case=False, na=False)` ifadesi, `LanguageHaveWorkedWith` sütunundaki değerlerin içinde "python" ifadesinin geçip geçmediğini kontrol eder. `case=False` parametresi büyük-küçük harf duyarsızlığı sağlar, yani "Python" veya "python" fark etmeksizin eşleşir. `na=False` parametresi ise eksik verileri göz ardı eder. Bu filtreleme işlemi sonucunda sadece Python kullanan katılımcılar `python_users` adında yeni bir veri çerçevesine atanır.
   Sonraki adımda, `country_grouped = python_users.groupby("Country")` kodu ile Python kullanan katılımcılar, ülkelerine göre gruplandırılır. `groupby("Country")` fonksiyonu, `Country` sütununa göre veri çerçevesini gruplar, böylece her ülke için ayrı bir grup oluşturulur.   
   Daha sonra, `average = country_grouped["ConvertedCompYearly"].mean()` kodu, her ülke grubundaki katılımcıların yıllık tazminatlarının ortalamasını hesaplar. `mean()` fonksiyonu, veri kümesindeki her ülkenin yıllık tazminat ortalamasını verir.   
   `median = country_grouped["ConvertedCompYearly"].median()` kodu ise her ülke grubundaki katılımcıların yıllık tazminatlarının medyan değerlerini hesaplar. `median()` fonksiyonu, tazminat verilerinin orta noktasını belirler ve genellikle aşırı uç değerlerin etkisini azaltmak için kullanılır.
   Son olarak, `print(f"average_annual_compensation_by_country: \n{average}"'\n')` ve `print(f"median_annual_compensation_by_country: \n{median}")` kodları, her ülke için hesaplanan ortalama ve medyan yıllık tazminat değerlerini ekrana yazdırır. Bu çıktılar, Python kullanan katılımcıların farklı ülkelerdeki tazminat trendlerini anlamamıza yardımcı olur.

   ```python
   python_users = df[df['LanguageHaveWorkedWith'].str.contains('python', case=False, na=False)]
   country_grouped = python_users.groupby("Country")
   average = country_grouped["ConvertedCompYearly"].mean()
   median = country_grouped["ConvertedCompYearly"].median()
   print(f"average_annual_compensation_by_country: \n{average}"'\n')
   print(f"median_annual_compensation_by_country: \n{median}")

   #average_annual_compensation_by_country: 
   Country
   Afghanistan                               4543.000000
   Albania                                  56295.000000
   Algeria                                   9053.285714
   Andorra                                 193331.000000
   Angola                                       6.000000
                                               ...      
   Venezuela, Bolivarian Republic of...     21500.000000
   Viet Nam                                 14014.562500
   Yemen                                    10297.333333
   Zambia                                   28123.666667
   Zimbabwe                                 37500.000000
   Name: ConvertedCompYearly, Length: 173, dtype: float64
   
   #median_annual_compensation_by_country: 
   Country
   Afghanistan                               4768.5
   Albania                                  56295.0
   Algeria                                   6230.0
   Andorra                                 193331.0
   Angola                                       6.0
                                             ...   
   Venezuela, Bolivarian Republic of...      7100.0
   Viet Nam                                 10180.0
   Yemen                                     5333.0
   Zambia                                   22803.0
   Zimbabwe                                 18000.0
   Name: ConvertedCompYearly, Length: 173, dtype: float64
   ```

8. **En Yüksek Tazminata Sahip Katılımcılar**  
   En yüksek tazminata sahip katılımcıları bulmak için `nlargest()` fonksiyonunu kullanıyoruz. Bu fonksiyon, belirli bir sütundaki en yüksek N değeri bulur. Bu adımda, en yüksek yıllık tazminata sahip 5 katılımcıyı bulup, onların eğitim seviyelerine göz atıyoruz.

   ```python
   top_5_highest_compensation = df.nlargest(5, 'ConvertedCompYearly')
   top_5_education_levels = top_5_highest_compensation[['EdLevel','ConvertedCompYearly']]
   print(top_5_education_levels)

   #                                             EdLevel  ConvertedCompYearly
   15837    Bachelor’s degree (B.A., B.S., B.Eng., etc.)           16256603.0
   12723  Professional degree (JD, MD, Ph.D, Ed.D, etc.)           13818022.0
   28379  Professional degree (JD, MD, Ph.D, Ed.D, etc.)            9000000.0
   17593    Bachelor’s degree (B.A., B.S., B.Eng., etc.)            6340564.0
   17672  Professional degree (JD, MD, Ph.D, Ed.D, etc.)            4936778.0
   ```

   Bu analiz, en yüksek maaşa sahip yazılım geliştiricilerin eğitim geçmişlerini incelememizi sağlar. Bu veriler, yüksek tazminatla eğitim seviyesi arasında nasıl bir ilişki olduğunu anlamamıza yardımcı olabilir.

9. **Bonus: Yaş Gruplarına Göre Python Kullanımı**  
   Her yaş grubundaki Python kullanıcılarının yüzdesini hesaplamak için `groupby()` ve `apply()` fonksiyonlarını birlikte kullanıyoruz. `apply()` fonksiyonu, her bir grup için bir fonksiyonu uygulamamıza olanak tanır. Burada `lambda` fonksiyonu ile Python kullananların yüzdesini hesaplıyoruz.

   ```python
   python_users = df[df['LanguageHaveWorkedWith'].str.contains('Python', case=False, na=False)]
   python_users_by_age= python_users.groupby('Age').size()
   total_users_by_age = pd.DataFrame(df).groupby('Age').size()
   percentage_by_age = (python_users_by_age / total_users_by_age) * 100
   print(percentage_by_age)

   #Age
   18-24 years old       55.922826
   25-34 years old       45.773912
   35-44 years old       41.520546
   45-54 years old       41.910706
   55-64 years old       40.427184
   65 years or older     37.564767
   Prefer not to say     45.341615
   Under 18 years old    64.875389
   dtype: float64
   ```

   `groupby('Age')` ile yaş gruplarına göre veriyi böldükten sonra, her yaş grubunda Python kullananların yüzdesini hesaplıyoruz. Bu işlem, yaş gruplarına göre Python kullanım oranının nasıl değiştiğini gösterir.

10. **Bonus: Uzaktan Çalışan ve Yüksek Tazminatlı Katılımcıların Endüstrileri**   
   Öncelikle, `df[(df.ConvertedCompYearly > df.ConvertedCompYearly.quantile(0.75)) & (df.RemoteWork == 'Remote')]` ifadesi, veri çerçevesindeki iki koşulu aynı anda karşılayan katılımcıları filtreler. İlk koşul, `df.ConvertedCompYearly > df.ConvertedCompYearly.quantile(0.75)` ifadesi, yıllık tazminatı veri setindeki 75. yüzdelik dilimin üzerinde olanları seçer. Bu, en yüksek yıllık tazminat diliminde yer alanları işaret eder. İkinci koşul, `df.RemoteWork == 'Remote'` ifadesi ise, sadece uzaktan çalışan katılımcıları seçer. Bu iki koşulun `&` (ve) operatörü ile birleştirilmesi, hem yüksek tazminatlı hem de uzaktan çalışan katılımcıları filtreler.
Sonrasında, `.Industry.value_counts()` kodu, bu filtrelenmiş veri setinde yer alan endüstrilerin sıklığını hesaplar. `value_counts()` fonksiyonu, her endüstrinin ne kadar yaygın olduğunu belirler ve sonuçları bir seri olarak döndürür. Bu, her endüstrinin kaç katılımcıya sahip olduğunu gösterir.
Son olarak, `.reset_index()` kodu, `value_counts()` sonucunda oluşan seriyi bir veri çerçevesine dönüştürür ve endeks sütununu sıfırlar. Bu, endüstri sıklıklarının daha düzenli bir formatta görüntülenmesini sağlar.
Bu kod parçasının çıktısı, uzaktan çalışan ve en yüksek yıllık tazminat diliminde bulunan katılımcıların hangi endüstrilerde çalıştığını ve bu endüstrilerin ne kadar yaygın olduğunu gösterir. Bu tür bir analiz, tazminat ve çalışma biçimi gibi faktörlere göre endüstri dağılımlarını anlamak için faydalıdır. 

   ```python
   df[(df.ConvertedCompYearly > df.ConvertedCompYearly.quantile(0.75)) & (df.RemoteWork == 'Remote')].Industry.value_counts().reset_index()

   # 	Industry	                                    count
   0	Software Development	                        768
   1	Other:	                                    239
   2	Healthcare	                                 156
   3	Fintech	                                    156
   4	Internet, Telecomm or Information Services	145
   5	Retail and Consumer Services	               106
   6	Media & Advertising Services	               103
   7	Banking/Financial Services                  	69
   8	Government	                                 69
   9	Computer Systems Design and Services	      69
   10	Transportation, or Supply Chain	            67
   11	Insurance	                                 50
   12	Manufacturing	                              48
   13	Higher Education	                           42
   14	Energy	                                    36
   ```
        
## Proje Çıktıları

Bu analiz sonucunda, StackOverflow anketi ile ilgili çeşitli çıkarımlar yapılmıştır:

1. Anketi dolduran yazılım geliştiricilerin büyük bir kısmı Python kullanmaktadır.
2. Deneyim seviyelerine ve ülkelerine göre tazminatlar önemli farklılıklar göstermektedir.
3. Yüksek tazminat kazanan katılımcılar arasında farklı eğitim seviyeleri gözlemlenmiştir.
4. Python kullanımı, yaş gruplarına göre çeşitlilik göstermektedir.

Bu proje, StackOverflow Anketi'ndeki katılımcıların deneyimleri, çalışma şekilleri, programlama dilleri ve tazminatları üzerine kapsamlı bir analiz gerçekleştirmiştir.

## Gereksinimler

- Python 3.x
- Pandas
- NumPy
- Jupyter Notebook

Proje dosyasına ve Jupyter Notebook içeriğine [buradan](link_to_your_notebook) ulaşabilirsiniz.
