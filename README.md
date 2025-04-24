# Ruby Design Patterns

## 1. Design Pattern Nedir ve Ne İçin Kullanılır?

Design pattern'ler:
- Yazılımda sık karşılaşılan problemlere yönelik, test edilmiş, tekrar kullanılabilir çözüm şablonlarıdır.
- Uygulamanın ölçeklenmesini ve bakımını kolaylaştırır.

---

## 2. Design Pattern Türleri

- **Creational:** Nesne oluşturmayı yönetir (Singleton, Factory)
- **Structural:** Nesneler arası ilişkileri düzenler (Decorator, Adapter)
- **Behavioral:** Nesneler arası etkileşimleri yönetir (Observer, Strategy, Command, Template)

---

## 3. Ruby’de Yaygın Kullanılan Pattern'ler

### Singleton Pattern

**🧠 Problem:**  
Uygulamada yalnızca bir tane olması gereken bir nesne vardır (örneğin: konfigürasyon yöneticisi, log yöneticisi). Ancak her `new` çağrısında yeni bir örnek oluşturulur.

**✅ Çözüm:**  
Singleton, bir sınıftan sadece bir örneğin yaratılmasını garanti eder ve bu örneğe global erişim sağlar.

```ruby
require 'singleton'

class AppConfig
  include Singleton

  def initialize
    @settings = { theme: "dark", lang: "tr" }
  end

  def get(key)
    @settings[key]
  end
end

puts AppConfig.instance.get(:theme)
```

---

### Prototype Pattern
**🧠 Problem:**  
Bir nesnenin oluşturulmasında ortak özellikler kullanılacaksa, bu ortak özellikleri 'new' ile tekrar vererek oluşturmamız gerekir.

**✅ Çözüm:**  
Bu nesnede bir sürü ortak özellik varsa ve yeni nesnede sadece isim değişecek ise new ile nesne oluşturmak yerine mevcut bir nesnenin clone oluşturup bu yeni nesnede daha az değişiklik yaparak kullanabiliriz.

```ruby
class Robot
  attr_accessor :name, :type

  def initialize(name, type)
    @name = name
    @type = type
  end

  def clone
    self.dup
  end
end

prototype = Robot.new("RX-0", "Worker")

robot1 = prototype.clone
robot1.name = "RX-1"
```

---

### Observer Pattern

**🧠 Problem:**  
Bir nesnede (subject) değişiklik olduğunda, bu değişiklikten haberdar olması gereken başka nesneler (observers) olabilir. Ancak doğrudan çağrı yapılması, bileşenleri birbirine sıkı sıkıya bağlar.

**✅ Çözüm:**  
Observer pattern, "yayınla-abone ol" (publish-subscribe) ilişkisi kurar. Subject değiştiğinde, tüm subscriber'lar otomatik olarak bilgilendirilir.

```ruby
require 'observer'

class NewsAgency
  include Observable

  def publish(news)
    changed
    notify_observers(news)
  end
end

class Subscriber
  def update(news)
    puts "Yeni haber: #{news}"
  end
end

agency = NewsAgency.new
user = Subscriber.new
agency.add_observer(user)
agency.publish("Ruby 4.0 duyuruldu!")
```

---

### Factory Pattern

**🧠 Problem:**  
Kodda hangi sınıftan nesne oluşturulacağını bilmek istemiyoruz. Örneğin, kullanıcıdan gelen bir tür bilgisine göre farklı sınıflar kullanılabilir.

**✅ Çözüm:**  
Factory pattern, nesne oluşturma sürecini merkezi bir yerde soyutlar. Bu sayede yeni türler eklemek daha kolay hale gelir.

```ruby
class EmailNotification
  def send; puts "E-posta gönderildi"; end
end

class SMSNotification
  def send; puts "SMS gönderildi"; end
end

class NotificationFactory
  def self.build(type)
    case type
    when :email then EmailNotification.new
    when :sms then SMSNotification.new
    else raise "Geçersiz bildirim türü"
    end
  end
end

n = NotificationFactory.build(:sms)
n.send
```

---

### Strategy Pattern

**🧠 Problem:**  
Bir işlemin (örneğin ödeme) birden fazla yolu olabilir ve bu yollar koşullarla belirleniyor olabilir. Koşul içeren if/else yapıları kodu karmaşıklaştırır.

**✅ Çözüm:**  
Strategy pattern, davranışları sınıflar haline getirir ve çalışma zamanında bunları değiştirebilir hale getirir.

```ruby
class PaymentContext
  def initialize(strategy)
    @strategy = strategy
  end

  def pay(amount)
    @strategy.pay(amount)
  end
end

class CreditCard
  def pay(amount)
    puts "#{amount} TL kredi kartıyla ödendi"
  end
end

class PayPal
  def pay(amount)
    puts "#{amount} TL PayPal ile ödendi"
  end
end

context = PaymentContext.new(PayPal.new)
context.pay(150)
```

---

### Decorator Pattern

**🧠 Problem:**  
Bir sınıfa yeni özellikler eklemek istiyoruz, ama miras almak hem karmaşık hem de esnek değil.

**✅ Çözüm:**  
Decorator, nesnenin davranışını değiştirmeden üzerine yeni yetenekler sarar.

```ruby
class Logger
  def log(message)
    puts message
  end
end

class LoggerDecorator
  def initialize(logger)
    @logger = logger
  end

  def log(message)
    @logger.log(message)
  end
end

class TimestampLogger < LoggerDecorator
  def log(message)
    super("[#{Time.now}] #{message}")
  end
end

class FileLogger < LoggerDecorator
  def log(message)
    super(message)
    File.open("log.txt", "a") { |f| f.puts(message) }
  end
end

base_logger = Logger.new
logger_with_time = TimestampLogger.new(base_logger)
logger_full = FileLogger.new(logger_with_time)

logger_full.log("Bir hata oluştu!")
```

---

### Command Pattern

**🧠 Problem:**  
Bir nesne, bir işlemi tetiklemek istiyor ama işlemin nasıl yapılacağını bilmek istemiyor (örneğin, uzaktan kumanda → lamba).

**✅ Çözüm:**  
Command pattern, işlemleri nesneler olarak soyutlar. Böylece işlemler sıraya alınabilir, geri alınabilir, loglanabilir.

```ruby
class UserService
  def create(name) = puts "Kullanıcı oluşturuldu: #{name}"
end

class CreateUserCommand
  def initialize(service, name)
    @service, @name = service, name
  end

  def execute
    @service.create(@name)
  end
end

class CommandInvoker
  def initialize = @commands = []
  def add(cmd) = @commands << cmd
  def run
    @commands.each(&:execute)
    @commands.clear
  end
end

service = UserService.new
cmd = CreateUserCommand.new(service, "Zeynep")
invoker = CommandInvoker.new
invoker.add(cmd)
invoker.run
```

---

### Template Method Pattern

**🧠 Problem:**  
Bir algoritmanın iskeleti sabit, ama bazı adımlar özelleştirilebilir olmalı.

**✅ Çözüm:**  
Template Method, üst sınıfta algoritma adımlarını tanımlar, alt sınıflar bu adımları özelleştirir.

```ruby
class Report
  def generate
    header
    body
    footer
  end

  def header; puts "Başlık"; end
  def footer; puts "Alt Bilgi"; end
  def body; raise NotImplementedError; end
end

class SalesReport < Report
  def body; puts "Satış verileri"; end
end

SalesReport.new.generate
```

---

## Ne zaman kullanılmalı
- Kod tekrar etmeye başladığında ve soyutlama ihtiyacı doğduğunda
- Farklı davranışları çalışma zamanında değiştirebilir hale getirmek gerektiğinde
- Sistemde büyüme/karmaşıklık arttığında mimariyi düzenli tutmak için
## Ne zaman kullanılmamalı
- Proje küçük ve basitse, pattern eklemek gereksiz karmaşıklık yaratabilir
- Problem tam tanımlanmamışsa ve çözüm ihtiyacı net değilse
- Yalnızca "teorik olarak doğru" olduğu için pattern ekleniyorsa 

## 4. Sonuç
Pattern'ler önlem değil, çözüm olarak düşünülmelidir. Gerçek bir ihtiyaç doğmadan pattern kullanımı kodu ağırlaştırabilir. Doğru zamanda, doğru yerde kullanılan design pattern, sürdürülebilirliği büyük ölçüde artırır.


- https://refactoring.guru/design-patterns
- https://medium.com/design-patterns/design-patterns-tasar%C4%B1m-%C3%B6r%C3%BCnt%C3%BCleri-3b0f67ae6488
