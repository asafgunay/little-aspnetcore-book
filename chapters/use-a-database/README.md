# Veri Tabanı Kullanma

Database kodu yazma alengirli olabilir. Ne yaptığınızı bilmiyorsanız doğrudan SQL sorgusunu kodlarınız arasına yazmak iyi bir firir değildir. **object-relational mapper** (ORM) bu olayı yeni bir katman ekleyerek daha kolay hale getirir. Bunun karşılığında  Java'da Hibernate veya Ruby'de ActiveRecord bilinen ORM'dir.

.NET için de bir çok ORM bulunmaktadır, Microsoft bunlardan biri olan Entity Framework'ü doğrudan ASP.NET Core içerisine entegre etmiştir. Entity Framework diğer veri tabanlarına bağlanmayı kulaylaştırdığı gibi veri tabanı sorgularını oluşturma veya tekrar obje haline çevirme işini yapar. (POCOs)

> Hatırlarsanız servis arayüzünü oluşturduğumuzda kontrolörden gerçek servisi ayırabilmiştik. Entity Framework'ü veri tabanı üzerine konulmuş büyük bir arayüz olarak görebiliriz. Bundan dolayı çalışan veri tabanı teknolojisinin hangisi olduğu önemsizdir.

Entity Framework SQL veri tabanlarına bağlanabilir. Örneğin MySQL veya SQL Server. Ayrıca Mongo gibi NoSQL veri tabanları ile de çalışabilir. Bu projede SQLite kullanacağız. Sonrasında siz isterseniz farklı veri tabanlarına da bağlayabilirsiniz.
