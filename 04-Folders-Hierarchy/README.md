# Proje Klasör Mimarisi ve Hiyerarşi Standartları

## 📁 Proje Kök Klasör Yapısı

Proje ana dizininde sadece temel yapılandırma dosyaları bulunur:

```
Project_Root/
├── Assets/              # Tüm game asset'leri
├── Packages/            # Unity package manager
├── ProjectSettings/     # Unity proje ayarları
├── .gitignore          # Git ignore kuralları
├── README.md           # Proje dokümantasyonu
└── Docs/               # Proje dokümantasyonu
```

---

## 🎨 Assets Klasör İç Yapısı

### 📂 Sprites Klasörü
Tüm 2D görsel asset'lerin organize şekilde saklandığı alan

```
Assets/
└── Sprites/
    ├── Player_Sprites/
    │   ├── Player_Idle.png
    │   ├── Player_Walk.png
    │   └── Player_Jump.png
    └── Environment_Sprites/
        ├── Background_Sprites/
        │   └── Level1_Background.png
        └── Tile_Sprites/
            └── Street_Tiles/
                └── Street_Tileset.png
```

### 🎭 Models Klasörü
3D model asset'leri ve ilişkili materyallerin düzenlendiği alan

```
Assets/
└── Models/
    └── Player_Models/
        ├── Player_Model.fbx
        ├── Player_Materials/
        │   ├── Player_Body_Material.mat
        │   └── Player_Clothing_Material.mat
        └── Player_Textures/
            ├── Player_Diffuse.png
            └── Player_Normal.png
```

**📋 Model Klasörü Kuralları:**
- Her model kendi klasöründe bulunur
- Materyaller model klasörü içinde `ModelName_Materials/` altında
- Texture'lar model klasörü içinde `ModelName_Textures/` altında

### 🧩 PreFabs Klasörü
Oyun objelerinin prefab versiyonlarının kategorilendiği alan

```
Assets/
└── PreFabs/
    ├── Player_PreFabs/
    │   └── Player.prefab
    ├── Enemy_PreFabs/
    │   ├── Level1_Enemy_Prefabs/
    │   │   ├── Enemy1.prefab
    │   │   └── Enemy2.prefab
    │   └── Boss_Enemy_Prefabs/
    │       └── Level1_Boss.prefab
    └── UI_PreFabs/
        ├── Menu_UI.prefab
        └── Game_UI.prefab
```

### 💻 Scripts Klasörü
Tüm kod dosyalarının modüler şekilde organize edildiği alan

```
Assets/
└── Scripts/
    ├── Player_Scripts/
    │   ├── Player_Movement/
    │   │   ├── Player_Horizontal_Move.cs
    │   │   └── Player_Jump_Move.cs
    │   └── Player_Attack_System/
    │       ├── Player_Melee_Attack.cs
    │       └── Player_Ranged_Attack.cs
    ├── Manager_Scripts/
    │   ├── Game_Manager.cs
    │   └── Audio_Manager.cs
    └── Editor/                    # Editor-only scripts
        └── Custom_Tools.cs
```

**📋 Scripts Klasörü Kuralları:**
- Her sistem kendi klasöründe (`System_Scripts/`)
- Alt sistemler kendi alt klasörlerinde organize edilir
- Editor scripts `Editor/` klasöründe ayrı tutulur
- Resources scripti farklı konumda olabilir

### 🎬 Scenes Klasörü
Oyun sahnelerinin kategorilere ayrılarak düzenlendiği alan

```
Assets/
└── Scenes/
    ├── Test_Scenes/
    │   ├── Mechanic_Tests/
    │   │   ├── Movement_Test.unity
    │   │   └── Combat_Test.unity
    │   └── Performance_Tests/
    │       └── Stress_Test.unity
    ├── Level_Scenes/
    │   ├── Level1_Scene.unity
    │   └── Level2_Scene.unity
    ├── Menu_Scenes/
    │   ├── Main_Menu.unity
    │   └── Settings_Menu.unity
    └── CutScenes/
        └── Level1_End_CutScene.unity
```

---

## 🏗️ Hiyerarşi Tab Standartları

### 📋 GameObject İsimlendirme Kuralları

#### Parent-Child İlişki Sistemi
```
In_Game_Managers/
├── Spawn_Managers/
│   ├── Enemy_Spawn_Manager
│   └── Coin_Spawn_Manager
├── UI_Managers/
│   ├── HUD_Manager
│   └── Menu_Manager
└── Audio_Managers/
    ├── Music_Manager
    └── SFX_Manager
```

#### İsimlendirme Formatı
- **Parent Objeler**: `Category_Name` (örn: `Game_Managers`)
- **Child Objeler**: Parent ile bağlantılı isim (örn: `Audio_Manager`)
- **Tutarlılık**: Aynı kategorideki objeler benzer isim yapısı

### 🎛️ GameObject Aktivasyon Standartları

#### Başlangıç Durumu Kuralları
```csharp
// ✅ Her GameObject başlangıçta kapalı
public class Object_Controller : MonoBehaviour
{
    private void Awake()
    {
        gameObject.SetActive(false);  // Başlangıçta kapalı
    }
    
    //*---------------------------------------------------------------------------//
    public void Enable_Object()
    {
        gameObject.SetActive(true);
        // Initialization logic
    }
    
    //*---------------------------------------------------------------------------//
    public void Disable_Object()
    {
        // Cleanup logic
        gameObject.SetActive(false);
    }
}
```

#### Activation Pattern Örneği
```csharp
public class Game_Manager : MonoBehaviour
{
    [Header("Scene Objects -----------------------------------------------------------")]
    [Space]
    [SerializeField] private GameObject player_Object;
    [SerializeField] private GameObject ui_Object;
    [SerializeField] private GameObject enemy_spawner;
    
    //*-----------------------------------------------------------------------------//
    private void Start()
    {
        Initialize_Game_Objects();
    }
    
    //*-----------------------------------------------------------------------------//
    private void Initialize_Game_Objects()
    {
        player_Object.GetComponent<Player_Controller>().Enable_Object();
        ui_Object.GetComponent<UI_Controller>().Enable_Object();
        enemy_spawner.GetComponent<Enemy_Spawner>().Enable_Object();
    }
}
```

---

## 🔄 Object Pooling Standartları

### Unity Object Pool + Custom Implementation

```csharp
using UnityEngine;
using UnityEngine.Pool;

public class Enemy_Pool_Manager : MonoBehaviour
{
    [Header("Pool Configuration ------------------------------------------------------")]
    [Space]
    [SerializeField] private GameObject enemy_Prefab;
    [SerializeField] private int pool_Size = 10;
    
    private ObjectPool<GameObject> enemy_Pool;
    
    //*-----------------------------------------------------------------------------//
    private void Awake()
    {
        Initialize_Object_Pool();
    }
    
    //*-----------------------------------------------------------------------------//
    private void Initialize_Object_Pool()
    {
        enemy_Pool = new ObjectPool<GameObject>(
            Create_Enemy,           // Create function
            On_Take_Enemy,          // Take function  
            On_Return_Enemy,        // Return function
            On_Destroy_Enemy,       // Destroy function
            maxSize: pool_Size
        );
    }
    
    //*-----------------------------------------------------------------------------//
    private GameObject Create_Enemy()
    {
        GameObject in_new_enemy = Instantiate(enemy_Prefab);
        in_new_enemy.SetActive(false);
        return in_new_enemy;
    }
    
    //*-----------------------------------------------------------------------------//
    private void On_Take_Enemy(GameObject enemy)
    {
        enemy.SetActive(true);
        enemy.GetComponent<Enemy_Controller>().Enable_Object();
    }
    
    //*-----------------------------------------------------------------------------//
    private void On_Return_Enemy(GameObject enemy)
    {
        enemy.GetComponent<Enemy_Controller>().Disable_Object();
        enemy.SetActive(false);
    }
    
    //*-----------------------------------------------------------------------------//
    public GameObject Get_Enemy()
    {
        return enemy_Pool.Get();
    }
    
    //*-----------------------------------------------------------------------------//
    public void Return_Enemy(GameObject enemy)
    {
        enemy_Pool.Release(enemy);
    }
}
```

### Custom Pool Implementation
```csharp
public class Custom_Object_Pool<T> where T : MonoBehaviour
{
    private Queue<T> available_Objects = new Queue<T>();
    private T prefab_Reference;
    
    //*-----------------------------------------------------------------------------//
    public Custom_Object_Pool(T prefab, int initial_Size)
    {
        prefab_Reference = prefab;
        
        for (int i = 0; i < initial_Size; i++)
        {
            T in_new_object = Object.Instantiate(prefab_Reference);
            in_new_object.gameObject.SetActive(false);
            available_Objects.Enqueue(in_new_object);
        }
    }
    
    //*-----------------------------------------------------------------------------//
    public T Get_Object()
    {
        if (available_Objects.Count > 0)
        {
            T in_reused_object = available_Objects.Dequeue();
            in_reused_object.gameObject.SetActive(true);
            return in_reused_object;
        }
        else
        {
            return Object.Instantiate(prefab_Reference);
        }
    }
    
    //*-----------------------------------------------------------------------------//
    public void Return_Object(T returned_Object)
    {
        returned_Object.gameObject.SetActive(false);
        available_Objects.Enqueue(returned_Object);
    }
}
```

---

## 🎯 Klasör Mimarisi Kuralları Özeti

### ✅ Yapılması Gerekenler

| Kategori | Kural | Örnek |
|----------|--------|--------|
| **Klasör İsimlendirme** | `Category_Name` formatı | `Player_Scripts/` |
| **Asset Organizasyonu** | Tip bazlı kategorizasyon | `Sprites/Environment_Sprites/` |
| **Hiyerarşi Düzeni** | Parent-Child mantıklı gruplandırma | `Managers/Audio_Managers/` |
| **GameObject Aktivasyonu** | Başlangıçta kapalı, kod ile aç | `SetActive(false)` at start |
| **Object Pooling** | Unity Pool + Custom implementation | Dual pooling system |
| **Prefab Yönetimi** | Değişiklikler prefab üzerinden | Override prefab values |

### ❌ Yapılmaması Gerekenler

- Kök dizinde asset dosyaları bırakmak
- Dağınık klasör yapısı oluşturmak  
- GameObject'leri başlangıçta aktif bırakmak
- Object pooling olmadan spawn işlemleri
- Scene'de prefab override'ları yapmak
- Materyalleri model klasörü dışında tutmak

---

## 🏛️ Mimari Felsefe

> **Branch Mantığı**: Her klasör bir `feature branch` gibi düşünülmelidir. Modüler, bağımsız ve organize.

Bu yaklaşım:
- **Scalability** sağlar
- **Team collaboration**'ı kolaylaştırır  
- **Asset management**'i optimize eder
- **Performance** artışı sağlar
- **Maintenance** süreçlerini basitleştirir

---

> **Not**: Bu standartlar Ethem Emre Özkan tarafından bireysel ve ufak ekip çalışmaları için bir temel oluşturulması amacıyla oluşturulmuştur.