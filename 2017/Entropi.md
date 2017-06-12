---
Title: Veriyi Anlamak - Entropi
PublishDate: 11/06/2017
IsActive: True
Tags: C#, Veri Madenciliği
---
Entropi deyince ilk akla gelen her ne kadar termodinomi olsa da veriyi anlama ve veriyi sıkıştırma gibi konularda da oldukça önemli bir yere sahiptir.

Önce basitlerle başlayalım. Elimizde bazı renkler olsun, `"yeşil, kırmızı, mavi, sarı, siyah"` bu renkleri en az _bit_kullanarak nasıl ifade edebiliriz? Bir _bit_  1 ve 0 olmak üzere iki farklı değer alabildiğini biliyoruz. Elimizdeki elemanlar 2’den fazla olduğu için tek bir _bit_ yeterli olmayacaktır. Elimizde 2 bit olursa bu durumda `00,01,10,11` olmak üzere 4 farklı değer elde edebiliriz. _Bit_ sayısını bir arttırdığımızda ise `“000,001, 010, 011, 100, 101, 110, 111”` şeklinde 8 farklı değer oluşturabiliriz. Fark ettiğiniz üzere 2,4,8 şeklinde sırayla 2’nin kuvvetleri olarak ilerleme var. Bu durumda kuvvet almanın tersi olan logaritma kullanarak gerekli _bit_ sayısını bulabiliriz. Örneğimiz için `log(5,2)` hesabını yaptığımızda `~2.32` elde ederiz. _Bit_’i daha küçük parçalara bölemeyeceğimiz için bu sayıyı yukarıya yuvarladığımızda 3 tam sayısını elde ederiz. Bu işlemi C# ile bir method haline getirmek istersek:

```csharp
    public static intGerekliBitSayisi<T>( ISet<T> dizi)
   {
	    return(int)Math.Ceiling(Math.Log(dizi.Count, 2));
    }
```



Bu methodu kullanmak istersek, kodumuz aşağıdaki gibi olacaktır:

```csharp
void Main()
{
	var ornek = new[] { "yeşil", "kırmızı", "mavi","sarı", "siyah" };
	var sonuc = GerekliBitSayisi(newSystem.Collections.Generic.HashSet<string>(ornek));
}

```



Fonksiyonda `ISet` arayüzünü kullanmamın amacı koleksiyon içerisindeki tüm elemanların tekrarsız olmasını garanti etmek içindir. .net içerisinde ISet uygulanmış koleksiyonlar aynı değeri birden fazla içeremezler -eğer rasgele sayılar ile tekrarsız sonuçlar üretmek istiyorsanız _Hashset_ gibi `ISet` uygulanmış koleksiyon tiplerini tercih etmeniz hem daha performanslı hem de daha kısa kod yazmanızı sağlayacaktır. 

Peki aynı elemanlar tekrarlı şekilde koleksiyon içerisinde bulunsalardı bu benim sonucumu etkiler miydi? İlk bakışta etkilememesi gerekiyor gibi gözükür çünkü  `“kırımızı = 001”` gibi eşleştirme yapmış olsaydım,kırmızı ile tekrar karşılaştığımda `001` yazar geçerdim. Öyleyse şöyle bir diziyidüşünelim:

```csharp
              var a =new[] { "kırmızı" ,"mavi","sarı", "mor", "siyah", "gri" ,"beyaz","yeşil","yeşil", "yeşil", "yeşil", "yeşil",
"yeşil", "yeşil" , "yeşil", "yeşil", "yeşil", "yeşil", "yeşil", "yeşil"};

```



Bu dizide 8 farklı eleman olduğu için ilk fonksiyonumuzu çalıştırdığımızda eleman başına gereken en az bit sayısı 3 olarak karşımıza çıkacaktır. Gerçekten de `000 = yeşil, 001 = kırmızı` gibi tek tek eşleştirme yaparsam sonuç böyle çıkacaktır.

Fakat koleksiyonun dağılımına baktığımızda `“yeşil”` elemanının 13 defa tekrarlandığı görülmektedir. Bu durumda, ben yeşil yerine tek _bit_’lik bir ifade ile 0 desem, herhangi rasgele bir renge 111 desem ve diğer kalan renklere 10 ve 11 ile başlayacak ama 111 ile başlamayacak şekilde eşleştirme yapsam aşağıdaki gibi bir eşleştirme tablosu çıkacaktır:

```
  Yeşil		=	0
  Beyaz		=	111
  Kırmızı	=	1000
  Mavi		=	1001
  Sarı		=	1010
  Mor		=	1011
  Siyah		=	1100
  Gri		=	1101
  
```



Bu durumda eleman başına 3.5bit’lik bir harcama yapmış olurum fakat ilk durumda toplam boyut `3 * 20 = 60 bit` ilen ikinci durumda `(13 *1) +(1 * 3) + (6 * 4) = 40 bit` yer kaplayacaktır. Bu durumda aslında eleman başına `40 / 20 = 2 bit` yeterli olmuştur. Bu da kayıpsız veri sıkıştırmanın ilk adımlarındandır. Bu eşleştirme işlemini _Shannon-Fano_ algoritması ile siz de yapabilirsiniz. Bu algoritmanın bazı sıkıntılarının giderilmesi ile `Huffman`   algoritması geliştirilmiştir.

Peki bu 2 değerini bilmenin bir yolu var mıdır? Bunun için gereken formül _Shannon_'ın entropi formülü olacaktır:

 $-\sum_{i=1}^n{P(x_i)log_2P(x_i)}$

Formülde ki P ifadesi probability yani ilgili değerin olasılığını belirtiyor. Yani aslında yaptığımız ilk formüldeki hesabı her bir değerin gelme olasılığına göre ağırlıklandırmak.  Bunu hesaplamak için C# kullanacak olursak kodumuz aşağıdaki gibi olacaktır:

```csharp
 public static double Entropi<T>(IEnumerable<T> data)
 {
     var sayac = new Dictionary<T, int>();
     int i;
     using (var sayici = data.GetEnumerator())
     {
         i = 0;
         while (sayici.MoveNext())
         {
             if (sayac.ContainsKey(sayici.Current))
             {
                 sayac[sayici.Current]++;
             }
             else
             {
                 sayac.Add(sayici.Current, 1);
             }
             i++;
         }
     }
     var sonuc = 0D;
     using (var sayici = sayac.GetEnumerator())
     {
         while (sayici.MoveNext())
         {
             var oran = (sayici.Current.Value / (double)i);
             sonuc += oran * Math.Log(oran, 2);
         }
     }
     return -sonuc;
 }
 
```



Bu fonksiyonu örnek koleksiyonumuz için çalıştıracak olursak bize `~1.98bit` sonucunu verecektir. Pratik kullanımda bir _bit_ parçalanamayacağı için yukarı yuvarladığımızda  ulaşacağımız sonuç  `2bit` olacaktır. Bu da bir dizinin her bir elemanını kodlamak için gereken optimal bit miktarını verecektir. Örneğin, bir metin dosyasında a harfi çok geçerken ğ harfi çok daha az geçebilir. Bu durumda a için daha az bit kullanılması dosyanın sıkıştırılabilmesine neden olmaktadır. Benzer şekilde metin kelimelerine ayrılıp numaralandırılırsa çok daha yüksek sıkıştırma oranları elde edilebilir. Logaritma değeri olarak istenilen bir değer verilebilmektedir. Sık olarak 2 değeri verilmekle birlikte, doğal logaritma ile hesaplandığı da görülmektedir.

Peki, bir koleksiyon ile başka bir koleksiyonu düzensizliklerine göre karşılaştırmak istersem. Örneğin, X ve Y müşterilerini harcama yaptıkları farklı mağazalara göre düzensizliklerini merak edebilirim. Mağazaları 1,2,3,4… olarak tam sayılar olarak gösterelim -ki reklam olmasın- ve harcama dağılımları aşağıda gösterildiği şekilde olsun:

```csharp
var x =new[] { 1, 1, 4, 2, 5, 1, 1, 1, 1, 3, 5, 2, 7, 2, 5, 9, 9, 7, 4, 3, 4, 1, 3, 5, 7, 3 };
var y =new[] { 1, 2, 4, 1, 1, 2, 1, 2, 2, 5, 2, 2, 4, 7, 3, 1, 2, 7, 2, 8, 2, 2, 9, 2,1 };

```



Burada her iki dizi de rasgele dağılmış gibi gözükmektedir.Ama en çok hangisi rasgele dağılıma sahiptir? Öncelikle bu iki durumu karşılaştırmak için ne kullanacağız? Amacımız en yüksek ihtimale 1 en düşük ihtimale 0 vermek olsun. Bu durumda eleman sayısı kadar farklı değere sahip olan bir koleksiyon 1 değerini alırken tüm elemanları aynı olan bir koleksiyon 0 değerini alacaktır. 

Bunu hesaplamak için bir önceki formülümüze geri dönüyoruz. Burada bu formüle sadık kalarak yapabileceğimiz birkaç şey var.Birincisi sadece logaritma tabanını 2 yerine farklı eleman sayısı olarak değiştirmek. Bu durumda 0 ile 1 arasında bir değer alırsınız. Bu yöntemde xiçin 0,96 y için 0.78 değerleri çıkacaktır. Diğeri ise yine logaritma 2 tabanında alınmakta fakat sonuç eleman sayısına bölünerek normalize edilmektedir. Bu durumda değerlerimiz 0.10 ve 0.09 olacaklardır. Son olarak `entropi / log(eleman sayisi, entropde kullanılan log tabanı)` kullanılarak ilk yöntemle aynı sonuç elde edilebilir.

Bunlardan çok daha karmaşık entropy hesapları olmakla birlikte, veriyi analiz ederken ve basit sıkıştırma çözümlerinde bu algoritma çoğunlukla yeterli olmaktadır. Pratik kullanımları ise sahtekarlık yakalamak, parola güvenliği ölçmek gibi farklı kullanımları olmaktadır.

 