# KUBERNETES TEMELLERİ 


### Kubernetes Nedir ?
büyüyen veya büyük mikroservise uygun projelerin konteynırlar haline getirilmesi ve yönetilmesini sağlayan bir teknolojidir

Kubernetes kümesi node adı verilen bir dizi işçi makineden oluşur. Her kümede en az bir node bulunur ve bu node'lar konteynırları çalıştırır.

### Kubernetes Küme Komponentleri
![img](components-of-kubernetes.svg)
Küme'yi kontrol tarafı ve node tarafı olarak 2 parçaya bölebiliriz.

<br>


| Control Plane Komponentleri |
| ----------- |
|__kube-apiserver__ |
| Node'lar ile Control paneli arasındaki iletişimi sağlar |
|__etcd__ |
Kümenin kullandığı anahtar ve değerleri tuttar
|__kube-scheduler__ |
Yeni oluşturulan Podları izler ve eğer boş/uygun olan node varsa o noda iletir
|__kube-controller-manager__ |
Kümenin durumunu izleyen ve duruma göre değişiklik yapar
|__cloud-controller-manager__ |
Bulut sağlayaıcısının API'ına bağlanmaya yarar lokalde çalışan projelerde(own-premises) bu komponente ihtiyaç olmaz


| Node Plane Komponentleri |
| -------- |
|__kube-proxy__|
|Her bir node'da ağ vekil sunucusu(network proxy) olarak node'un ağ iletişimini sağlar |
|__kubelet__|
|Her bir node'da bulunan ajandır ve konteynır(lar)ın sağlıklı çalıştığına emin olur |

<br>
<br>

### Node ve Pod nedir?
Kubernetsde uygulamalrımızın/konteynır(lar)ımızın içinde bulunduğu ve yönetebildiğimiz en ufak küme __Pod__'dur.
Bu podları taşıyan fiziksel veya sanal bir makine ise __Node__'dur 

<br>
<br>

### Pod'un yaşam döngüsü  ☀️ -> 🌙
Pod'un oluşturulduktan itibaren 4 faz'ı mevcuttur;
| Faz ismi | Açıklama |
| -- | -- |
|Pending|Kubernets kümesi tarafından kabul edilmiş fakat çalıştırmaya henüz hazır olmadığını anlamıdır|
| Running | Pod içindeki tüm konteynırlar başarılı şekilde sonlanmıştır ve yeniden başlatılmayacaktır |
| Succeeded | Pod içindeki tüm konteynırlar veya herhangi biri başarısız şekilde sonlanmıştır __varsayılan ayarlarda konteynırlar başarılı olana kadar olarak yeniden başlatılır__
| Unknown | Pod'un durumu algılanammıştır genellikle pod ile node arasındaki iletişim sorunundan kaynaklıdır__

Pod'umuz başarısıza düşme seneryasounda Podumuzun __restartPolicy__ özelliğini kullanbiliriz
>restartPolicy: Always      #Pod'un herhangi bir şekilde kapanmasında yenidne başlatır
>restartPolicy: OnFailure    #Sadece başarısız şekilde kapanmasında yenidne başlatır
>restartPolicy: Never    #Pod kapansa yeniden başlatmaz yüzüne bile bakmaz 😁

Pod'da karşılaşabilceğimiz önemli bir problem:  __CrashLoopBackOff__ ;
Podun kapanmasına sebebiyet uygulamamızın hatalarında, yanlış/eksik konfigrasyonda,kaynak eksikliği veya kısıtlamalarında bu hata dönebilir kontrollerimizi sağlayıp podumuzu hayata döndürebiliriz.



<br>
<br>

### Minikube ile Docker-Desktop farkı nedir, ne işe yarar ?
__Minikube__ kullandığınız makinenizde sanal bir makine oluşturarak tek bir node'da kubernets kümeleri yönetmeye olanak sağlayan bir kubernetes uygulamasıdır. 

__Docker-Desktop__ ise temel görevi docker konteynırlarını yönetebidiğiniz arayüze sahip olan bir uygulamadır. Ayrıyeten kubernets context'ini aktif ederek kubernets kümlerinide burdan yönetebilirsiniz.

hangi platformu/context'i kullandığınız öğrenmek ve değiştirmek için; 
```
kubectl config get-contexts     #mevcut context'leri listeler 
kubectl config set-context minikube     #minikube context'ini seçer
```
<br>  

### Kubectl Nedir?
Kubectl kubernets'in CLI' versiyonudur tabiri caizse; İsviçre çakısıdır 😊
Kubernetsimizdeki pod'ları node'ları listeyebilir, inceleyebiir, durudurup çalıştırmak gibi yönetimleri yapabiliriz. <a href="https://kubernetes.io/docs/reference/kubectl/introduction/"> Bknz</a>

```
kubectl get pods -o wide    #mevcut podları detaylı şekilde listeler 
```
<br>
<br>
<br>
<br>

### Podlar için YAML dosyası oluşturmak
Kubernets podların yönetimini kolaylaştırmak için yaml dosyasından faylanmak bizim için uygun olacaktır.
```
# nginx_pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
``` 

### Workload Yönetimi
Nihayetinde uygulamlarımız Pod'larda çalışıyor ve ve bu Pod'ların yönetilmesi eforlu bir iş haline gelebilir burda devreye Kubernets'in kendi içinde tanımlı workloadları giriyor ve Pod'larımızı önceden tanımladığımız şekilde yönetmeyi kolaylaştırıyor  

__Deployment ve ReplicaSet__ durum güncellmesi olmayan(stateless) uygulamaların yönetimi için tavsiye edilir. Herhangi bir Pod aynı özelliklerle istenilen sayıda üretilebilir(horizantal scale up) veya değiştirilebilir(verison upgrade). için kullanılabilir

__StatefulSet__ bir veya birden fazla özellikle state yönetimi olan(statefull) Pod'ların yönetimi için tavsiye edilir.
çünkü deployment workload'ı yerine statefulset workladı;
her bir pod için benzersiz kimlik ve depolama alanı sağlar, x podu çöktüğünde bile x podunun depoloma alanı korunduğundan veri kaybı olamadan hayata kaldığı yerden devam edebilir

__DaemonSet__ Her node'da belirli bir pod'u çalıştırmaya yaramaktadır, örneğin devlopment node'u ile preprod node'unuz var ve bu iki node'danda log almak istiyorsanız DaomonSet ile log alabilen bir pod'u iki ortamada kurabilirsiniz

__Job__ bir görev veya işin tek seferlik veya tamamlanana kadar yürütülmesi veya tamamlanması adına kullanılır. bu işe örnek olarak veri tabanı yedeklemek veya belirli süre içinde cache silme işlemi olabilir.


Devam edecek... sevgiyle kalın
