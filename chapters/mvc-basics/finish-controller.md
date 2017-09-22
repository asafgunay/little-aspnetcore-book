## Kontrolörü Tamamlama

Artık kontrolörümüzü tamamlayabiliriz. Artık kontrolörümüzün servis katmanına bağlandığını varsayabiliriz. Bu servisten aldığımız veriler ile daha önce oluşturduğumuz`TodoViewModel` oluşturup bunu önyüze gönderebiliriz.


**`Controllers/TodoController.cs`**

```csharp
public async Task<IActionResult> Index()
{
    var todoItems = await _todoItemService.GetIncompleteItemsAsync();

    var model = new TodoViewModel()
    {
        Items = todoItems
    };

    return View(model);
}
```

## Test it out
Eğer Visual Studio Code veya Visual Studio kullanıyorsanız F5 Tuşu ile projenizi çalıştırabilirsiniz. Bunun yerine komut satırı veya terminalden `dotnet run` komutunu da kullanabilirsiniz elbette. Eğer kodunuzda hata yoksa sunucunuz varsayılan port olan `5000` portundan çalışmaya başlayacaktır.

Eğer tarayıcınız otomatik olarak açılmadıysa `http://localhost:5000/todo' adresine giderek, hazırladığımız sahte(fake) veri tabanı katmanından verileri çektiğini ve ekranda gösterdiğini göreceksiniz.
 
 Tebrikler! Sırada yaptığımız bu projeye 3. parti paketler ekleme ve gerçek veri tabanına bağlama var.

