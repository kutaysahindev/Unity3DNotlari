# Unity 3D Notları
## Giriş
* Herhangi bir projede kullanılabilecek planlama aracı: hacknplan
* Prefabler bir objenin farklı scene'lerde baştan oluşturulmadan aynı konfigürasyonla tekrar kullanılmasını sağlar. Bir nesnenin prefabini oluşturmak için:
    - Assets'in altına bir Game Folders klasörü aç
    - Game Folders'ın içine Prefabs klasörü aç
    - Hiyerarşi'de oluşturduğun objeyi sürükleyip Prefabs klasörünün içine bırak
    - Prefab oluştuysa obje mavi renk olacak
* Hiyerarşideki birçok objeyi temsil eden(*örneğin 4 adet duvar prefabini temsil eden 1 adet duvarlar objesi*) boş bir empty object oluşturulabillir. Bu objenin prefabini almak gerekmez.
* Hiyerarşiyi olabilidiğince düzenli tut. Gerekirse empty game object'ler yapıp child entity'leri buna uygun sınıflandır.
``` 
örneğin Game Area:
            -Walls
            -Floors
                -Start Floor
                    -Point Light
                -Finish Floor
                    -Point Light
```
* Game Area'ların normal şartlarda Prefab yapılması gerekmez, eğer bölümün seviye zorluğu, farklı bölümler oluşturma gibi durumlar varsa o zaman Prefab yapılabilir.
* Concretes - Somut class'lar, new'lenebilir.
* Abstracts - Soyut class'lar, new'lenemez.
* Namespace'ler class'ları birbirinden ayırmamızı sağlar.
* *Bir Script'i yazmaya başladığın ilk anda direkt component olarak nesne atamasını yap, ileride unutma ihtimali yüksek.*
* Bir Script'i direkt olarak hiyerarşideki Prefab'e component olarak verirsen, o Prefab'in kendisini değiştirmez, sadece mevcutta bulunan Prefab'i değiştirir. Tekrar kullanmak istediğinde o Script yok olur. Genel olarak player objesine ait bir özellik eklemek istiyorsan hiyerarşideki değil; proje dosyalarındaki Prefab'i değiştirmen gerekir. Onda yaptığın değişiklik kalıcı olur.
* Hiyerarşi'deki Prefab'in yanındaki minik oka tıklarsan da ana Prefab'e gitmiş olursun.
## Rigidbody ve Fizik İşlemleri
* rigidbody'yi dahil etmek:
```cs
public class PlayerController : MonoBehaviour{
    private Rigidbody _rigidbody;

    private void Awake(){
        _rigidbody = GetComponent<Rigidbody>();
    }
}
```
* Update methodu ile klavye/mouse vs ile ilgili input girişleri alınır.
* Update methodu her frame'de 1 çalışır. 30/60/120
```cs
public class PlayerController : MonoBehaviour{
    private void Update(){

    }
}
```
* FixedUpdate methodu ile fizik işlemleri yapılır.
* FixedUpdate methodu hiç işlem yoksa 0.2 sn'de bir çalışır, **AYRICA fizik motoruyla senkronize şekilde çalışır. (örneğin her frame'de 2 kere)**
```cs
public class PlayerController : MonoBehaviour{
    private void FixedUpdate(){

    }
}
```
* Vector3. yapısıyla uzaydaki yönlerle alakalı kuvvet ekleme işlemleri vs yapılabilir.
```cs
private void FixedUpdate(){
    _rigidbody.AddForce(Vector3.up * Time.deltaTime);
}
```
* Input almak için eskisi gibi tek tek script yazmaktansa yeni çıkan unity inputsystem kullanılır.
* Input sistemini kullanmak için Concretes'in altına "Inputs" klasörü açıp create-input actions'a tıkla.
    - Açılan ekranda "Action Maps" ile farklı kontrol inputları almak için action map'ler oluşturulabilir.
* Örneğin yukarı gitmek için "ForceUp" Action'ı ekleriz, sonra Binding kısmından tuş basımı ekleriz.
* Pass through aksiyonuyla bas-çek tuş aksiyonu sağlarız.
* Oluşturulan kontrolör'ün namespace'i şu şekilde ayarlanılabilir:
```cs
UdemyProject1.inputs
```
* New input system kullanıp class oluşturulduktan sonra yeni bir cs belgesi açıp ilgili component'ler newlenmelidir.
* **İvmeli hızlanmak için:**
```cs
_rigidbody.AddForce(Vector3.up * Time.deltaTime);
```
* SerializeField bir nesneyi public yapmadan inspector'da görülebilmesini sağlayan Attribute.
* Input alıp yönlendirmek
```cs
namespace UdemyProject1.inputs
{
    public class DefaultInput
    {
        private DefaultAction _input;

        public bool IsForceUp { get; private set; } 
        public DefaultInput()
        {
            _input = new DefaultAction();

            _input.Rocket.ForceUp.performed += context => IsForceUp = context.ReadValueAsButton();

            _input.Enable();
        }
    }

}
```
* Rotasyon ve hareket ekseni sınırlamaları Rigidbody componenti ile yapılabilir.
* SOLID prensiplerine göre her class'ın görevi sadece kendini ilgilendirir. Örneğin hareket işlemleri için input class'ına kod yazmaktansa mover adında yeni bir class açılması gerekir.
* **_rigidbody.AddForce fonksiyonu unity düzlemindeki ortak eksenlere göre kuvvet uygularken, _rigidbody.AddRelativeForce ilgili objenin eksenlerine göre kuvvet uygular.**
* *ctrl + .* ile otomatik olarak gerekli directory scripte dahil edilir.
* *örnek bir hareket class'ının cache'leme işlemi:*
```cs
namespace UdemyProject1.Movements
{
    public class Rotator
    {

        Rigidbody _rigidbody;
        PlayerController _playerController;

        public Rotator(PlayerController playerController)
        {
            _playerController = playerController;
            _rigidbody = playerController.GetComponent<Rigidbody>();
        }

        public void FixedTick(float direction)
        {

        }

    }

}
```
* Unity Input System'de tek eksenli bir hareket inputu(+/- z gibi) alınacaksa Actions kısmından *1D Axis Component*, iki eksenli bir hareket inputu alınacaksa (hem düşey hem yatay hareket gibi) *2D Vector Composite* eklenmelidir.
* Rotation işlemi için alınan inputta **Pass through** opsiyonu ve Axis control tipi seçilmelidir.
* Direction inputundan bir değer gelmiyorsa rotasyonu dondurmak için script:
```cs
 public void  FixedTick(float direction)
        {
            if (direction == 0)
            {
                if (_rigidbody.freezeRotation) _rigidbody.freezeRotation = false;
                // buradaki if yazımı tek satırda yazılabilen kısa yazım
                return;
            }
        }
```
