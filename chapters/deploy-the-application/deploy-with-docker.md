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



At the beginning of this chapter, I mentioned that you should use a reverse proxy like Nginx to proxy requests to Kestrel. You can use Docker for this, too.

The overall architecture will consist of two containers: an Nginx container listening on port 80, forwarding requests to a separate container running Kestrel and listening on port 5000.

The Nginx container needs its own Dockerfile. To keep it from colliding with the Dockerfile you just created, make a new directory in the web application root:

```
mkdir nginx
```

Create a new Dockerfile and add these lines:

**`nginx/Dockerfile`**

```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

Next, create an `nginx.conf` file:

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

This configuration file tells Nginx to proxy incoming requests to `http://kestrel:5000`. (You'll see why `kestrel:5000` works in a moment.)

### Set up Docker Compose

There's one more file to create. Up in the web application root directory, create `docker-compose.yml`:

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

Docker Compose is a tool that helps you create and run multi-container applications. This configuration file defines two containers: `nginx` from the `./nginx/Dockerfile` recipe, and `kestrel` from the `./Dockerfile` recipe. The containers are explicitly linked together so they can communicate.

You can try spinning up the entire multi-container application by running:

```
docker-compose up
```

Try opening a browser and navigating to `http://localhost` (not 5000!). Nginx is listening on port 80 (the default HTTP port) and proxying requests to your ASP.NET Core application hosted by Kestrel.

### Set up a Docker server

Specific setup instructions are outside the scope of this Little book, but any modern Linux distro (like Ubuntu) can be set up as a Docker host. For example, you could create a virtual machine with Amazon EC2, and install the Docker service. You can search for "amazon ec2 set up docker" (for example) for instructions.

I prefer DigitalOcean because they've made it really easy to get started. DigitalOcean has both a pre-built Docker virtual machine, and in-depth tutorials for getting Docker up and running (search for "digitalocean docker").
