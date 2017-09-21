## Görüntü(View) Oluşturma

ASP.NET Core'da görüntü Razor Şablon Dili üzerine kuruludur. Bu HTML ile C# kodunu birleştirebilmemizi sağlar. ( Eğer daha öncesinde Javascript'de Jade/Pug veya Handlebars, Javada Thymeleaf kullandıysanız ana hatlarıyla ne yaptığımızı anladınız demektir)

Çoğunlukla görüntü kodları sadece HTML'dir. Görüntü Modelinden gelen değerleri içerisinden alıp HTML içerisine yerleştirmek için C# kullanırız. C# komutları `@` sembolü ile başlar.

`Index` aksiyonu tarafından gösterilecek sayfada liste halinde görüntü modelinde taşınan `TodoItem`'ı göstermemiz gerekmektedir. İsmine münhasır Görüntü dosyalarını View klasörünün içerisinde oluşturmamız gerekmektedir. Bu klasörün içerisine kontrolörün ismine göre bir başka klasör açıp aksiyon ismiyle dosya oluşturmamız gerekmektedir.

Örneğin Index aksiyonunu kullanacağız ve kontrolörümüzün ismi `TodoController`, yapmamız gereken Views içerisine Todo adında bir klasör oluşturmak ve bunun içerisine `Index.cshtml` dosyası oluşturmak. Razor şema dili `.cshtml` uzantısını kullanmaktadır.

**`Views/Todo/Index.cshtml`**

```html
@model TodoViewModel;

@{
    ViewData["Title"] = "Yapılacaklar Listesini Yönet";
}

<div class="panel panel-default todo-panel">
  <div class="panel-heading">@ViewData["Title"]</div>

  <table class="table table-hover">
      <thead>
          <tr>
              <td>&#x2714;</td>
              <td>Item</td>
              <td>Due</td>
          </tr>
      </thead>
      
      @foreach (var item in Model.Items)
      {
          <tr>
              <td>
                <input type="checkbox" name="@item.Id" value="true" class="done-checkbox">
              </td>
              <td>@item.Title</td>
              <td>@item.DueAt</td>
          </tr>
      }
  </table>

  <div class="panel-footer add-item-form">
    <form>
        <div id="add-item-error" class="text-danger"></div>
        <label for="add-item-title">Yeni yapılacak ekle:</label>
        <input id="add-item-title">
        <button type="button" id="add-item-button">Ekle</button>
    </form>
  </div>
</div>
```


Sayfanın başındaki `@model` tanımı Razor'a hangi modelin beklendiğini bildiriyor. Bu modele daha sonra `Model` özelliği üzerinden ulaşılır.

`Model.Items`da veri olduğunu varsayarsak bu verilere `foreach` cümlesiyle döngü içerisine alabilir ve bu döngünün içerisinde `<tr>`ile satırı oluşturup değerlerimizi yazdırabiliriz. Ayrıca gördüğünüz gibi her satır için onay kutusunu da başta yerleştirdik. Daha sonra bunu işaretlediğimizde görevin tamamlandığını bildiren kodu yazacağız.

### Dosya düzeni

Dosyanın içerisine baktığınızda `<body>` veya `<footer>` nerede diyebilirsiniz. Görüntü dosyaları daha kolay çalışabilmemiz için `Views/Shared/_Layout.cshtml` içerisinde tanımlanır. Bizim oluşturduğumuz yeni sayfa dinamik olarak değişen içerik bölümüne gelir. 

ASP.NET Core'un varsayılan teması Bootstrap ve JQuery'i içerir. Böylece çok hızlı bir şekilde web uygulaması geliştirebilirsiniz. Tabiki kendi CSS ve Javascript kütüphanelerinizi de kullanabilirsiniz.

### Stil üzerinde değişiklikler

Şimdilik CSS üzerinde değişiklik yapmak için `site.css` dosyasının en altına aşağıdaki stilleri yapıştırın.

**`wwwroot/css/site.css`**

```css
div.todo-panel {
  margin-top: 15px;
}

table tr.done {
  text-decoration: line-through;
  color: #888;
}
```

Kodda da gördüğünüz gibi CSS kuralları ile sayfanıza istediğiniz stili verebilirsiniz.

ASP.NET Core elbette bundan çok daha fazlasını yapabilir. Örneğin kısmi görüntüler(partial views) veya sunucu tarafından yorumlanmış görüntü bileşenleri vs. Fakat basit bir web sitesi için bunlara gerek yoktur. Bu konuyu daha derinine incelemek isterseniz `https://docs.asp.net` adresini kullanabilirsiniz.
