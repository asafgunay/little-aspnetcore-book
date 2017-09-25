## Yeni Yapılacak Maddesi Ekleme

Kullanıcılar yeni yapılacak maddesini aşağıdaki gibi yapılacaklar listesinin altındaki form'dan ekleyecekler.

![Final form](final-form.png)

Bu özelliği eklemek için şu adımları tamamlamak gerekmektedir:

* Görüntü(View) dosyasını eğiştirerek form elementlerini ihtiyacımıza göre düzenlemek
* Javascript kodu ile butona tıklandığında bunu arka-uç'a göndermek.
* Kontrolörde yeni aksiyon oluşturarak gelecek taleplere cevap vermek.
* Servis katmanına veri tabanını güncelleyecek metodları yazmak.


### Görüntüye Form Eklemek.

`Views/Todo/Index.cshtml` in en altına aşağıdaki formu ekleyiniz.

```html
<div class="add-item-form">
    <form>
        <div id="add-item-error" class="text-danger"></div>
        <label for="add-item-title">Add a new item:</label>
        <input id="add-item-title">
        <button type="button" id="add-item-button">Add</button>
    </form>
</div>
```
Arka-uç'a POST talebiyle veri gönderebilmek için jQuery kullanacağız. Butona tıklandığında bu işlem gerçekleşecek.

### Add JavaScript code

`wwwroot/site.js` dosyasını açın ve aşağıdaki kodu ekleyin.

```javascript
$(document).ready(function() {

    // Add butonuna tıklanıldığında addItem fonksiyonu çalışacak.
    $('#add-item-button').on('click', addItem);

});
```
`addItem` fonksiyonunu dosyanın altına aşağıdaki gibi ekleyiniz.

```javascript
function addItem() {
    $('#add-item-error').hide();
    var newTitle = $('#add-item-title').val();

    $.post('/Todo/AddItem', { title: newTitle }, function() {
        window.location = '/Todo';
    })
    .fail(function(data) {
        if (data && data.responseJSON) {
            var firstError = data.responseJSON[Object.keys(data.responseJSON)[0]];
            $('#add-item-error').text(firstError);
            $('#add-item-error').show();
        }
    });
}
```

Bu fonksiyon `http://localhost:5000/Todo/AddItem`'a post talebi ile kullanıcının metin kutusuna girdiği değeri paslar. `$.post` fonksiyonundaki üçüncü parametre eğer işlem başarı ile gerçekleşirse çalışacak fonksiyondur. Eğer kontrolör başarı ile çalışır ve cevap dönerse bu cevapla birlikte `200 OK` döner. Başarılı olduğunda sayfa `windwos.location` ile yönlendirilerek yeniden yüklenir. Eğer sunucu `400 Bad Request` cevabı verseydi `fail` fonksiyonu çalışacaktı. Bu da hatayı ekrana yazaktı.

### Aksiyon Ekleme

Şu anda yukarıdaki Javascript kodu çalışmayacak çünkü `/Todo/AddItem` yolunda henüz bu talebi karşılacak bir aksiyonumuz bulunmamaktadır. Eğer şu anda denerseniz, ASP.NET Core `404 Not Found` hatası dönecektir.

`TodoController` içerisine `AddItem` adında yeni bir aksiyon oluşturun.

```csharp
public async Task<IActionResult> AddItem(NewTodoItem newItem)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    var successful = await _todoItemService.AddItemAsync(newItem);
    if (!successful)
    {
        return BadRequest(new { error = "Could not add item" });
    }

    return Ok();
}
```

Sizinde dikkat edeceğiniz üzere bu metod `NewTodoItem` isminde bir parametre beklemektedir. Şu anda bu model tanımlı değil. Öyleyse bunu tanımlayalım.


**`Models/NewTodoItem.cs`**

```csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace AspNetCoreTodo.Models
{
    public class NewTodoItem
    {
        [Required]
        public string Title { get; set; }
    }
}
```
Bu modelde sadece `Title` isminde yeni bir özelliklik tanımladık. Bu özelliğin ismi Jquery ile gönderdiğimiz talepte bulunan isimle aynıdır.

```javascript
$.post('/Todo/AddItem', { title: newTitle }  // ...

// Bir özellikli JSON Objesi:
// {
//   title: tipi
// }
```

ASP.NET Core model ile bu gelen parametreyi eşlemek için **model bağlama** işlemini kullanır. Eğer isimleri aynı ise veri modelin içerisine alınır.

**Model Bağlama**nın yanı sıra ASP.NET Core ayrıca **model doğrulama** işlemini de yapar. Modelde yazdığımız `[Required]` özelliği `Title`'ın boş olamayacağı bilgisini verir. Bu doğrulama doğrudan hata vermyecektir fakat durumu kaydedilecektir. Ardından kontrolör içerisinden bu kontrol edilebilir.

> Aslında `TodoItem`'ı tekrar kullanabilirdik. En nihayetinde bu modelde de `Title` özelliği mevcuttu. Fakat bu modelde ön yüzden hiç bir zaman gönderilmeyecek(ID ve done) gibi özellikleri içermektedir. Dolayısıyla sadece gelecek değerleri tanımlayan bir model yaratmak daha yararlı olacaktır.

`AddItem` aksiyonuna bakacak olursanız ilk yaptığımız iş model doğrulamadır. Eğer bu parametre doğrulandıysa işleme devam edecek aksi halde `BadRequest(ModelState)` döndürecektir.


```csharp
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}
```
Burada ModelState tam olarak neyin yanlış olduğunu açıklamasıyla birlikte dönmeye yarar. BadRequest ise `400 Bad Request` dönürür. Böylece hatanın tam olarak neyden kaynaklandığını anlayabilirsiniz.

```csharp
var successful = await _todoItemService.AddItemAsync(newItem);
if (!successful)
{
    return BadRequest(new { error = "Yeni madde eklenemedi" });
}
```
`AddItemAsync` metodu sadece `true` veya `false` döndürür. Herhangi bir nedenden dolayı hata aldıysa `400 Bad Request` ve bununla `error` özelliğini döner.

Eğer hiç bir hata almadıysa sonuçta `Ok()` yani `200 OK` döner.


### Servis Metodu Ekleme

Eğer C# anlayan bir kod editörü kullanıyorsanız `AddItemAsync` altında çizgi göreceksiniz. Daha bu servis metodunu yazmadığımızdan bu gayet normal.

Önce arayüze(`ITodoItemService`) bu tanımı ekleyin.


```csharp
public interface ITodoItemService
{
    Task<IEnumerable<TodoItem>> GetIncompleteItemsAsync();

    Task<bool> AddItem(NewTodoItem newItem);
}
```

Ardından gerçek uygulamasını `TodoItemService` içerisinde yapın:


```csharp
public async Task<bool> AddItem(NewTodoItem newItem)
{
    var entity = new TodoItem
    {
        Id = Guid.NewGuid(),
        IsDone = false,
        Title = newItem.Title,
        DueAt = DateTimeOffset.Now.AddDays(3)
    };

    _context.Items.Add(entity);

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1;
}
```
Bu metod yeni bir `TodoItem` ( veri tabanı modeli ) oluşturup `NewTodoItem` modelinden gelen `Title` alanını kopyalamaktadır. Sonrasında `SaveChangeAsync` ile bunu kalıcı olarak veri tabanına yazar.

> Yukarıdaki kod istediğimizi yapan bir yoldur. Bu şekilde farklı yollar da bulunmaktadır. Örneğin daha kompleks bir yapıyı düşünün, modelinizde birçok alan var ve bunu ekrana gösterip talep ediliğinde kaydetmek istiyorsunuz. Bunun için ASP.NET Core **tag helpers** adında modelden görüntü yaratabilecek özellikler barındırır. Bu konu ile ilgili örneklere https://docs.asp.net üzerinden erişebilirsiniz.

### Test Edin

Uygulamayı çalıştırın ve birkaç yeni yapılacak ekleyin. Girdiğiniz değerler veri tabanına kaydedildiğinden sayfa tekrardan yüklense de, tarayıcınızı kapatıp açsanız da verilerin aynı kaldığını göreceksiniz.

> Buna ek olarak tarih seçici kullanarak HTML ve Javascript ile kullanıcının `DueAt` özelliğini girmesini sağlayabilirsiniz. Eğer tarih boş ise son tarihi 3 gün sonraya atayabilirsiniz.
