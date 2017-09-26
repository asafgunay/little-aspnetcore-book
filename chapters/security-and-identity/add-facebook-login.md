## Facebook Login Ekleme

Projeyi oluşturduğumuzda kişisel izin şablonu doğrudan kullanıcın kayıt olmasına ve giriş yapmasına olanak sağlar. Bu özelliğe ek olarak farklı servisleri de kullanabiliriz. Bunlar Google, Facebook veya Twitter olabilir.

Harici servisleri kullanmak için aşağıdaki iki maddeyi uygulamalısınız. 
1. Harici servis sağlayıcıda App veya *client* oluşturmalısınız.

1. Oluşturduğunuz App sonucunda size sağlanacak **secret** ve **ID**'yi kodunuzda kullanmalısınız.

### Facebook'ta App Oluşturma

https://developers.facebook.com/apps adresinden yeni bir Facebook uygulaması oluşturabilirsiniz. **Add a New App**' tıklayarak yönergeleri uygularsanız uygulamayı oluşturur ve app ID sahibi olursunuz.

> Eğer Facebook hesabınız yoksa Google veya Twitter'da da bu işlemler aynıdır. Tabi basamaklar aynı olabilir ama sonucunda size bir kod verecektir.

Sonrasında Facebook Login'e tıklayarak sol tarafta bulunan Settings bölümüne girin.

![Settings button](facebook-login-settings.png)

Aşağıdaki URL'i **Valid OAuth redirect URIs** bölümüne yapıştırın.

```
http://localhost:5000/signin-facebook
```

**Save Changes** ile değişiklikleri kaydedip **Dashboard** sayfasına geçin. Burada sizin uygulamanız için oluşturulmuş App ID ve Secret'i göreceksiniz.

Facebook Login'i etkinleştirmek için Aşağıdaki kodu `Startup` sınıfı `ConfigureService` metodu içerisine yapıştırın

```csharp
services
    .AddAuthentication()
    .AddFacebook(options =>
    {
        options.AppId = Configuration["Facebook:AppId"];
        options.AppSecret = Configuration["Facebook:AppSecret"];
    });
```
Burada bulunan şifreleri doğrudan yazmak yerine bunların `appsettings.json` ayar dosyasından çekilmesi sağlanabilir. Fakat bu dosya kaynak kontrolü(Source Control) üzerinde tutulduğundan dolayı. Bu özel bilgilerin taşınması iyi değildir. Örneğin bunu Github üzerinde herkesin görebileceği bir proje olarak yayınlar isek bu durumda başka birisi sizin adınıza işlem yapabilir.

### Gizli bilgileri Gizlilik Yönetici(Secrets Manager) ile saklama

Hassas bilgilerin kayıt edilmesi için Gizlilik Yöneticisi Aracını kullanabilirsiniz. Bunun için komut satırını veya terminali açın ( ana klasörde olduğunuza emin olun ) ve aşağıdaki komutu yazın.

```
dotnet user-secrets --help
```
Facebook sayfasından ID ve secret'i kopyalayarak aşağıdaki gibi iki defa `set` komutunu çalıştırarak bu bilgileri sistmee girin.

```
dotnet user-secrets set Facebook:AppId <App Id'yi yapıştırın>
dotnet user-secrets set Facebook:AppSecret <App Secret'i yapıştırın>
```
Gizlilik yöneticisi bu bilgileri `Configuration` içerisine yazdığında ve uygulama ayağa kalktığında `ConfigureService` içerisinde doğrudan kullanılabilir hale gelir.

Uygulamanızı çalıştırın ve navigasyonda bulunan giriş butonuna tıklayın. `Facebook` butonunu göreceksiniz.

![Facebook login button](facebook-login-button.png)

Facebook bilgileri ile giriş yapmaya çalışın. Facebook'tan izinleri alacak ve sonrasında sisteminize kullanıcı girişi yapacaksınız.
