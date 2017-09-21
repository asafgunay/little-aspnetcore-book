## Model Oluşturma

İki türlü model sınıfı bulunmaktadır: Yapılacaklar maddesini veritabanında temsil eden sınıf. Buna **entity** de denilmektedir. İkincisi ise bu modeli önyüze göndermeye yaran modeldir. Her ikiside model olarak tanımlanabileceğinden dolayı biz ikincisine Görüntü Modeli diyeceğiz.

Öncelikle Models klasörünün içerisine `TodoItem` adında bir sınıf yaratalım.

**`Models/TodoItem.cs`**

```csharp
using System;

namespace AspNetCoreTodo.Models
{
    public class TodoItem
    {
        public Guid Id { get; set; }
        
        public bool IsDone { get; set; }

        public string Title { get; set; }

        public DateTimeOffset? DueAt { get; set; }
    }
}
```
Buradaki sınıf her bir yapılacak için gerekli olan alanları tanımlar. Bunlar ID, title, isDone, DueAt gibi değerlerdir. Dikkat ettiyseniz DateTimeOffset? biraz farklıdır. Buradaki soru işareti null olabileceğini belirtir. Böylece bu alanın kontrolü yapılmadan bir işlem yapılırsa Örneğin DueAt.Day dediğimizde eğer değer null ise hata atması gerekmektedir. Fakat soru işareti koyarak önce null kontrolü yapmış oluyoruz.

>  **g**lobally **u**nique **id**entifier çok uzun bir yazı ve rasgele seçildiğinden dolayı aynı değerin üstüste gelmesi neredeyse imkansızdır. Bunun için ID olarak GUID kullanacağız. Guid örneği : `43ec09f2-7f70-4f4b-9559-65011d5781bb`.

> Tabi bu değer yerinde veri tabanınızdan sayısal bir değer döndürmesini isteyebilirsiniz. Bunun için tablonudaki o alanı otomatik artım olarak işaretlemelisiniz. Böylece her yeni veri girişde bu kolon yeni bir değer alacaktır.

Bu noktada hangi veri tabanı kullandığınız önemli değil. Örneğin bu SQL Server, MySql, MongoDb, Redis veya başka aklınıza gelen herhangi bir veri tabanı olabilir. Oluşturduğunuz model veri tabanındaki bir satırın C# ta nasıl görüneceğini tanımlar. Bundan dolayı sizin veri tabanı hakkında endişelenmenize gerek yok. 

Genellikle veri tabanında oluşturduğunuz tablo modelinize çok benzemektedir. Fakat birebir aynı olmayabilir. Örneğin şu anda yaptığımız örnekte `TodoItem`tek bir satırı ifade etmektedir. Fakat biz önyüzde bunların listesini görmek istiyoruz. Bundan dolayı bizim Görüntü Modeli oluşturmalıyız.

İstediğimiz `TodoItem` Listesidir.

**`Models/TodoViewModel.cs`**

```csharp
using System.Collections.Generic;

namespace AspNetCoreTodo.Models
{
    public class TodoViewModel
    {
        public IEnumerable<TodoItem> Items { get; set; }
    }
}
```

`IEnumerable<>` C# ta `Itemlar` demenin yeni şeklidir. Bu özellik boş olabilir, bir tane veya daha fazla `TodoItem` içerebilir.( Teknik olarak array'a benzesede tam olarak array değildirler. IEnumerable bizim ihtiyacımız olduğunda veri tabanına istek gönderir. bkz: Lazy Loading. Ayrıca Linq sorgusunun sonunda çalıştırıldığını varsayarsak öncesinde veritabanına sorgu göndermezsek performanslı bir işlem yapmış oluruz)    

> `IEnumerable<>` arayüzü `System.Collections.Generic` isim uzayında ( namespace ) yer almaktadır, `using` kullanarak bunu dosyanın başında belirmeniz gerekmektedir.

Artık yapılacaklar için modelleri yaptığımıza göre `TodoViewModel` i alacak bir Görüntü(view) oluçturmanın zamanı geldi.