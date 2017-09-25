## Maddeleri onay kutusu ile tamamlama

Yapılacaklar listesine ekleme işini yaptınız çokta güzel oldu. Fakat bir de bu işin tamamlanması var. `Views/Todo/Index.cshtml` sayfasına göre her yapılacak için onay kutusunu da gösterilecek.

```html
<input type="checkbox" name="@item.Id" value="true" class="done-checkbox">
```
Her maddenin bir benzersiz ID( guid )'si olduğundan bunu bullanabiliriz. Bunu name alanında belirterek kontrolöre bunu paslayarak bu maddeyi güncelleyebiliriz.

Akışın tamamı aşağıdaki gibi olacak

* Kullanıcı onay kutusunu işaretler ve Javascript fonksiyonu tetiklenir.

* Javascript kontrolöre talep gönderir.

* Kontrolörde bulunan aksiyon servis katmanına çağrıda bulunarak bu maddenin güncellenmesini sağlar.
* Güncelleme sonunda cevap Javascript fonksiyonuna geri gönderilir.
* HTML kodu güncellenir.

### Javascript Kodu Ekleme
Önce `site.js`'yi açık ve `$(document).ready` bloğunun içerisini aşağıdaki gibi düzenleyin.

**wwwroot/js/site.js**

```javascript
$(document).ready(function() {

    // ...

    // .done-checkbox sınıfına ait tüm onay kutuları aşağıdaki kodu çalıştırır.
    $('.done-checkbox').on('click', function(e) {
        markCompleted(e.target);
    });

});
```
Sonrasında sayfanın sonuna aşağıdaki kodu ekleyin:

```javascript
function markCompleted(checkbox) {
    checkbox.disabled = true;

    $.post('/Todo/MarkDone', { id: checkbox.name }, function() {
        var row = checkbox.parentElement.parentElement;
        $(row).addClass('done');
    });
}
```
Bu kod daha önceki yapılacak oluşturma fonksiyonu ile aynı yapıya sahip. Fakat bu defa HTTP POST ile  `http://localhost:5000/Todo/MarkDone` adresine talep gönderip bunun içerisinde id olarak veri tabanından gelen ve onay kutusunun name kısmında belirttiğimiz `id`yi paslıyoruz.

Herhangi bir onay kutusunu işaretleyip tarayıcınızın Network Araçlarına bakacak olursanız talebi aşağıdaki gibi görebilirsiniz.


```
POST http://localhost:5000/Todo/MarkDone
Content-Type: application/x-www-form-urlencoded

id=<some guid>
```

`$.post` başarılı bir şekilde döndüğünde onay kutusunun bulunduğu satır'a `done` sınıfı eklenir. Bu sınıfa has stil değişikliği uygulanmış olur.

### Kontrolöre aksiyon ekleme

Sizin de tahmin edeceğiniz gibi, `TodoController` içerisine `MarkDone` aksiyonunu eklemeliyiz.

```csharp
public async Task<IActionResult> MarkDone(Guid id)
{
    if (id == Guid.Empty) return BadRequest();

    var successful = await _todoItemService.MarkDoneAsync(id);

    if (!successful) return BadRequest();

    return Ok();
}
```
Şimdi bu adımların üzerinden geçelim. Öncelikle `id` isminde bir `Guid` argümanı ekledik. Yani bu metodun Guid beklediğini bildirdik. Daha önce yazdığımız `AddItem` aksiyonunda `NewTodoItem` modeli kullanmıştık. Fakat bu defa beklentimiz sadece `id` olduğundan böyle bir sınıf yapmaya gerek yok. ASP.NET Core gelen değeri **guid** olarak ayrıştırmaya çalışacak.

Herhangi bir model oluşturmadığımızdan ve doğruluk tanımını ` [Required]` yapmadığımızdan dolayı metodumuz içerisinde `ModelState` kelimesini kullanamıyoruz. Fakat bunun yerine `Guid.Empty` gibi bir kontrol ile boş olup olmadığını kontrol edebiliriz. Eğer boş ise BadRequest yani `400 Bad Request` dönderebiliriz.
```csharp
if (id == Guid.Empty) return BadRequest();
```

Sırada kontrolörün servisi veri tabanı güncellemesi için çağırması kaldı. Bu yeni bir metod olan `MarkDoneAsync` ile `ITodoItemService` üzerinde yapılmakta. Sonuç olarak eğer veri tabanı güncellemesi başarılı olursa true aksi halde false değeri döndürecek.

```csharp
var successful = await _todoItemService.MarkDoneAsync(id);
if (!successful) return BadRequest();
```

Sonuç olarak herşey düzgün bir şekilde çalıştıysa javascript'e `Ok()` yani `200 OK` değerini döndüreceğiz. Daha kompleks bir yapı ile JSON yapısında farklı veriler göndermekte mümkün fakat şimdilik buna ihtiyacımız yok. 

### Servis Metodu Ekleme

İlk olarak `MarkDoneAsync`'i arayüze ekleyin:


**`Services/ITodoItemService.cs`**

```csharp
Task<bool> MarkDoneAsync(Guid id);
```
Sonra bunun uygulamasını `TodoItemService` üzerinde şu şekilde tamamlayın:

**`Services/TodoItemService.cs`**

```csharp
public async Task<bool> MarkDoneAsync(Guid id)
{
    var item = await _context.Items
        .Where(x => x.Id == id)
        .SingleOrDefaultAsync();

    if (item == null) return false;

    item.IsDone = true;

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1; // One entity should have been updated
}
```

Bu metod Entity Framework Core'un `Where` komutunu kullanarak ID kolonuna göre arama yapmaktadır. `SingleOrDefaultAsync` metodu bulduğu satırı item'a atar eğer bulamaz ise bu durumda item `null` olur. Bunun kontrolünü yaptıktan sonra doğrudan false dönebiliriz. 
Eğer `item` boş değilse bunun sadece `IsDone` özelliğini ayarlayarak güncelleyebiliriz.

```csharp
item.IsDone = true;
```

`SaveChangesAsync` metodu uygulanana kadar yaptığımız değişiklikler sadece yerelde değişti.`SaveChangesAsync` çalıştıktan sonra kaç satırı güncellediyse onu dönderir. Bu durumda ya 1 tane gönderecek veya 0, ama 0 olursa bir yanlışlık olduğunu söyleyebileceğiz. Eğer 1 ise `true` aksi halde `false` döndereceğiz.

### Test Edin

Uygulamayı çalıştırıp bazı onay kutularını işaretleyin. Sayfayı yenilediğinizde bu maddelerin yapılacaklar listesinden silindiğini göreceksiniz. Bunun nedeni `GetIncompleteItemsAsync` te bulunan `Where` filtresidir.

Şu anda, uygulama herkesçe görülebilen yapılacaklar listesi durumundadır. Eğer kişiye özel bir liste olsa daha kullanışlı olabilir. Bir sonraki bölümde, ASP.NET Core Identity kullanarak güvenlik ve kimlik denetleme özelliklerini entegre edeceğiz.

