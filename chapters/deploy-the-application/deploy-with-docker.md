## Docker ile Dağıtma

Docker gibi konteyner teknolojileri web uygulamalarının dağıtımını oldukça kolaylaştırmıştır. Sunucu ayarlama, gerekli eklentilerin yüklenmesi vs. dosyların kopyalanması, sunucunun her defasında yeniden başlatılması gibi işlemler tek bir seferde Docker imaji alınarak ortadan kaldırılabilir. Yapmanız gereken imajı Docker çalıştırabilecek bir sunucuya yerleştirmek. 

Projenizi ölçeklendirme açısından da Docker oldukça yararlıdır. Eğer imaj oluşturduysanız 1 konteyner ile 100 konteyner oluşturma sırasında aynı işlemi yaparsınız.

Başlamadan önce Docker CLI'ın bilgisayarına kurulu olması gerekmekte. Bunun için internetten işletim sisteminize göre Docker'ı indirip kurun, resmi web sitesi en güvenilir kaynaktır.

```
docker --version
```

> Eğer **Güvenlik ve Kimlik** bölümünde Facebook ile giriş bölümünü kodladıysanız Docker'a Facebook ID ve appSecret'i bildirmeniz gerekmekte. Docker ayarlarının bu değerler için nasıl yapılacağı bu kitabın konusu değildir. Eğer istiyorsanız `ConfigureServices` içerisindeki `AddFacebook` bölümünü komut satırı yaparak Facebook ile girişi şimdilik iptal edebilirsiniz.


### Docker dosyası oluşturma

İlk yapmanız gereken Docker dosyası oluşturma, Docker dosyası uygulanızın nelere ihtiyacı olduğunu belirtmenizi sağlar.

Eklentisiz bir şekilde `Dockerfile` adında bir dosya oluşturun. Bunu uygulama içerisinde yapmanız lazım yani `Program.cs` nin olduğu yerde.

bu dosyayı açın ve aşağıdaki satırı ekleyin

```dockerfile
FROM microsoft/dotnet:latest
```

Bu Docker'a başlangıç imajı olarak Microsoft'un yayınladığı imajı al demek. Bu ASP.NET Core uygulaması için gerekli herşeyin alınmasını sağlar.

```dockerfile
COPY . /app
```

`COPY` komutu bilgisayarınızda bulunan klasörün proje klasörünüzün `/app` adında bir klasörün içine kopyalanmasını sağlar. 


```dockerfile
WORKDIR /app
```
`WORKDIR` Windows komut satırında bulunan `cd`'ye denk gelir. Yani bundan sonraki yazacağımız komutlar `/app` klasörü içerisinde çalışacaktır.


```dockerfile
RUN ["dotnet", "restore"]
RUN ["dotnet", "build"]
```

Bu komutlar `dotnet restore` ( Nuget paketlerinin indirilmesine yarar ) ve `dotnet build` ( uygulamayı derler


```dockerfile
EXPOSE 5000/tcp
```
Normalde Docker konteynırı hiç bir porttan dışarıya veri alıp vermeye izin vermez bunun için `EXPOSE` ile hangi portu açması gerektiğini söylediniz. Kestrel sunucunun varsayılan portu 5000 olduğundan 5000 kullandınız.

```dockerfile
ENV ASPNETCORE_URLS http://*:5000
```
`ENV` komutu çevre değişkenlerini tanımlar. Uygulamanın `ASPNETCORE_URLS` ile hangi porttan çalışacağını ayarlamanız lazım



```dockerfile
ENTRYPOINT ["dotnet", "run"]
```
Son olarak uygulamanın `dotnet run` komutu ile çalışması için yukarıdaki satırı eklediniz.

Bu işlemler sonunda docker dosyası aşağıdaki gibi görünecektir.

**`Dockerfile`**

```dockerfile
FROM microsoft/dotnet:latest
COPY . /app
WORKDIR /app
RUN ["dotnet", "restore"]
RUN ["dotnet", "build"]
EXPOSE 5000/tcp
ENV ASPNETCORE_URLS http://*:5000
ENTRYPOINT ["dotnet", "run"]
```

### İmaj Oluşturma

`Dockerfile`'i kaydettiğinize emin olun. Sonra `docker build` ile imajı şu şekilde oluşturabilirsiniz : 


```
docker build -t aspnetcoretodo .
```
En sondaki `.`'yı unutmayın bu `dockerfile`'in aynı klasörde olduğunu belirtiyor.

İmaj oluştuktan sonra `docker images` ile tüm oluşturulmuş imajları görebilirsiniz. Artık bu imajı test edebilirsiniz. Bunun için aşağıdaki satırı çalıştırın:

```
docker run -it -p 5000:5000 aspnetcoretodo
```

`-it` bayrağı Docker'a konteynerın interaktif modda çalıştırılacağını söyler.


### Nginx'in kurulması

Bu bölümün başında reverse proxy ile Nginx veya apache kullanarak Kerstel'e gelen talepleri reverse proxy yapmamız gerektiğini söylemiştim.

Genel mimarimiz iki tane konteynerdan oluşacak: Birincisi 80 portinu dinleyen ve 5000 portuna yönlendiren Nginx ve 5000 portundan bu taleplere cevap veren Kerstel.

Nginx kendi Dockerfile'ına ihtiyaç duyar. Daha önce yazdığınız Dockerfile ile karışmasın diye web uygulamasının ana dizininde bir tane klasör oluşturun.
```
mkdir nginx
```

içerisine yeni bir `Dockerfile` oluşturup şu satırları ekleyin.

**`nginx/Dockerfile`**

```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```
Sonra bu klasör içinde `nginx.conf` dosyasını oluşturun ve aşağıdaki satırları yapıştırın

**`nginx/nginx.conf`**

```
events { worker_connections 1024; }

http {

        server {
              listen 80;

              location / {
                proxy_pass http://kestrel:5000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'keep-alive';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
              }
        }
}
```
Bu ayar dosyası Nginx'e gelen taleplerin `http://kestel:5000 adresine yönlendirilmesi gerektiğini söyler. (Neden `kerstel:5000` kullandığımızı az sonra göreceksiniz.)


### Docker düzenleyici oluşturma

Web uygulamasının ana klasörüne `docker-compose.yml` adında bir dosya oluşturun.

**`docker-compose.yml`**

```yaml
nginx:
    build: ./nginx
    links:
        - kestrel:kestrel
    ports:
        - "80:80"
kestrel:
    build: .
    ports:
        - "5000"
```

Docker Compose birden fazla containerin birlikte oluşturulmasını ve çalışmasını sağlar. Yukarıda 2 tane konteyner tanımlanır. Bunlar `./nginx/Dockerfile`'da bulunan `nginx` ve `./Dockerfile` da bulunan `kestrel`. Doğrudan ikisiyle bağlantılı olarak çalıştırıldığından iletişimleri sağlanmış olur.

Birden fazla konteyner içeren uygulamalar aşağıdaki gibi çalıştırılır


```
docker-compose up
```

Tarayıcıyı açıp `http://localhost`'a gidin(dikkat ederseniz 5000 portunu yazmadık). Nginx 80 portunu dinlediğinden doğrudan bu telebi proxy ile ASP.NET Core uygulamasına yönlendirir.

### Docker sunucusu kurma

Docker sunucusu kurma adımları bu kitabın kapsamı içeriisnde değildir. Fakat modern bir linux sürümüne Docker servisi kurulabilir. Örneğin Amazon EC2 sanal makinesine Docker kurmal mümkündür. Bunun için internete "amazon ec2 set up docker" diye aratırsanız birçok yönerge bulabilirsiniz.

Dijital Ocean dökümanları ve sistemi EC2'ye göre daha kolay olduğundan öncelikle DigitalOcean tercih edilebilir.
