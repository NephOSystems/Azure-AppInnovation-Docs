---
Baslik: App Service
Tanim: Azure App Service, web uygulamalarını, REST API'lerini ve mobil arka uçları barındırmak için HTTP tabanlı bir hizmettir. .NET, .NET Core, Java, Ruby, Node.js, PHP veya Python gibi en sevdiğiniz dilde geliştirebilirsiniz. Uygulamalar hem Windows hem de Linux tabanlı ortamlarda kolaylıkla çalışır ve ölçeklenir.
Yazar: ifyucel

ms.assetid: f3359464-fa44-4f4a-9ea6-7821060e8d0d
ms.topic: article
ms.date: 1/28/2022
ms.author: ifyucel

---
# Azure App Service en iyi pratik
Bu makale, [Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/) kullanımına yönelik en iyi uygulamaları özetler. 

## <a name="colocation"></a>Colocation
Web uygulaması ve veritabanı gibi bir çözüm oluşturan Azure kaynakları farklı bölgelerde bulunduğunda, aşağıdaki etkilere sahip olabilir:

* Kaynaklar arasındaki iletişimde artan gecikme
* Belgede belirtildiği gibi bölgeler arası giden veri aktarımı için parasal ücretler [Azure pricing page](https://azure.microsoft.com/pricing/details/data-transfers).

Aynı bölgedeki ortak yerleşim, bir web uygulaması ve içerik veya verileri tutmak için kullanılan bir veritabanı veya depolama hesabı gibi bir çözüm oluşturan Azure kaynakları için en iyisidir. Kaynakları oluştururken, olmamaları için belirli bir iş veya tasarım nedeniniz yoksa, bunların aynı Azure bölgesinde olduklarından emin olun. kullanarak bir App Service uygulamasını veritabanınızla aynı bölgeye taşıyabilirsiniz. [App Service cloning feature](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-app-cloning) Şu anda Premium Uygulama Hizmet Planı uygulamaları için kullanılabilir.   

## <a name="memoryresources"></a>Uygulamalar beklenenden daha fazla bellek tükettiğinde
Bir uygulamanın, izleme veya hizmet önerileri aracılığıyla belirtilenden daha fazla bellek tükettiğini fark ettiğinizde, [App Service Otomatik İyileştirme özelliğini](https://azure.microsoft.com/blog/auto-healing-windows-azure-web) göz önünde bulundurun. -Siteler). Otomatik İyileştirme özelliğinin seçeneklerinden biri, bir bellek eşiğine dayalı olarak özel eylemler gerçekleştirmektir. Eylemler, e-posta bildirimlerinden, bellek dökümü yoluyla araştırmaya ve çalışan sürecini geri dönüştürerek yerinde hafifletmeye kadar geniş bir yelpazeyi kapsar. Otomatik iyileştirme, [App Service Destek Sitesi Uzantısı](https://azure.microsoft.com/blog/additional-updates-to) için bu blog gönderisinde açıklandığı gibi web.config aracılığıyla ve kullanıcı dostu bir arabirim aracılığıyla yapılandırılabilir. -azure-app-service-web-apps-site-uzantısı-destek).

## <a name="CPUresources"></a>Uygulamalar beklenenden daha fazla CPU tükettiğinde
Bir uygulamanın beklenenden daha fazla CPU tükettiğini fark ettiğinizde veya izleme veya hizmet önerilerinde belirtildiği gibi tekrarlayan CPU artışları yaşadığınızda, Uygulama Hizmeti planını büyütmeyi veya küçültmeyi düşünün. Uygulamanız durum bilgisine sahipse ölçeği büyütmek tek seçenektir, uygulamanız durumsuz ise ölçeği genişletme size daha fazla esneklik ve daha yüksek ölçeklendirme potansiyeli sağlar.

App Service ölçeklendirme ve otomatik ölçeklendirme seçenekleri hakkında daha fazla bilgi için bkz. [Azure App Service'te bir Web Uygulamasını Ölçeklendirme](https://docs.microsoft.com/en-us/azure/app-service/manage-scale-up).  

## <a name="socketresources"></a>Soket kaynakları tükendiğinde
Giden TCP bağlantılarını tüketmenin yaygın bir nedeni, TCP bağlantılarını yeniden kullanmak için uygulanmayan istemci kitaplıklarının kullanılması veya HTTP - Canlı Tutma gibi daha yüksek düzeyli bir protokolün kullanılmamasıdır. Giden bağlantıların verimli bir şekilde yeniden kullanımı için kodunuzda yapılandırıldıklarından veya bunlara erişildiklerinden emin olmak için App Service Plan'ınızdaki uygulamalar tarafından başvurulan kitaplıkların her birinin belgelerini inceleyin. Bağlantıların sızmasını önlemek için düzgün oluşturma ve serbest bırakma veya temizleme için kitaplık dokümantasyon kılavuzunu da takip edin. Bu tür istemci kitaplıkları araştırmaları sürerken, birden çok örneğe ölçeklenerek etki azaltılabilir.

### Node.js ve giden http istekleri
Node.js ve birçok giden http istekleriyle çalışırken HTTP - Canlı Tut ile uğraşmak önemlidir. Kodunuzu kolaylaştırmak için [agentkeepalive](https://www.npmjs.com/package/agentkeepalive) `npm` paketini kullanabilirsiniz.

İşleyicide hiçbir şey yapmasanız bile her zaman "http" yanıtını kullanın. Yanıtı düzgün bir şekilde işlemezseniz, daha fazla yuva olmadığı için uygulamanız sonunda takılır.

Örneğin, "http" veya "https" paketiyle çalışırken:

```javascript
const request = https.request(options, function(response) {
    response.on('data', function() { /* hicbirsey yapma */ });
});
```

Birden çok çekirdeğe sahip bir makinede Linux üzerinde App Service çalıştırıyorsanız, uygulamanızı yürütmek üzere birden çok Node.js işlemini başlatmak için PM2'yi kullanmak başka bir en iyi uygulamadır. Bunu, kapsayıcınıza bir başlangıç komutu belirterek yapabilirsiniz.

Örneğin, dört örneği başlatmak için:

```
pm2 start /home/site/wwwroot/app.js --no-daemon -i 4
```

## <a name="appbackup"></a>Uygulama yedeklemeniz başarısız olmaya başladığında
Uygulama yedeklemesinin başarısız olmasının en yaygın iki nedeni şunlardır: geçersiz depolama ayarları ve geçersiz veritabanı yapılandırması. Bu hatalar genellikle, depolama veya veritabanı kaynaklarında veya bu kaynaklara nasıl erişileceğine ilişkin değişiklikler (örneğin, yedekleme ayarlarında seçilen veritabanı için güncellenen kimlik bilgileri) olduğunda meydana gelir. Yedeklemeler tipik olarak bir zamanlamaya göre çalışır ve depolamaya (yedeklenen dosyaların çıktısını almak için) ve veritabanlarına (yedeğe dahil edilecek içerikleri kopyalamak ve okumak için) erişim gerektirir. Bu kaynaklardan herhangi birine erişememenin sonucu, tutarlı yedekleme hatası olacaktır.

Yedekleme hataları meydana geldiğinde, hangi tür hatanın meydana geldiğini anlamak için en son sonuçları inceleyin. Depolama erişim hataları için, yedekleme yapılandırmasında kullanılan depolama ayarlarını gözden geçirin ve güncelleyin. Veritabanı erişim hataları için, uygulama ayarlarının bir parçası olarak bağlantı dizelerinizi gözden geçirin ve güncelleyin; ardından, gerekli veritabanlarını uygun şekilde dahil etmek için yedekleme yapılandırmanızı güncellemeye devam edin. Uygulama yedeklemeleri hakkında daha fazla bilgi için bkz. [Azure App Service'te bir web uygulamasını yedekleme](https://docs.microsoft.com/en-us/azure/app-service/manage-backup).

## <a name="nodejs"></a>Azure App Service'e yeni Node.js uygulamaları dağıtıldığında
Node.js uygulamaları için Azure App Service varsayılan yapılandırması, en yaygın uygulamaların gereksinimlerine en iyi şekilde uyacak şekilde tasarlanmıştır. Node.js uygulamanızın yapılandırması, performansı artırmak veya CPU/bellek/ağ kaynakları için kaynak kullanımını optimize etmek için kişiselleştirilmiş ayardan faydalanacaksa, bkz. [Azure App Service'teki Node uygulamaları için en iyi uygulamalar ve sorun giderme kılavuzu](https://docs.microsoft.com/tr-tr/azure/app-service/app-service-web-nodejs-best-practices-and-troubleshoot-guide). Bu makale, Node.js uygulamanız için yapılandırmanız gerekebilecek iisnode ayarlarını açıklar, uygulamanızın karşılaşabileceği çeşitli senaryoları veya sorunları açıklar ve bu sorunların nasıl çözüleceğini gösterir.
