# vornex_bridge

Capa de compatibilidad multi-framework para los recursos **Vornex**. Abstrae el framework (qbox / qbcore / esx), el inventario (ox_inventory / qb-inventory) y el sistema de target (ox_target / qb-target) detrás de una sola API `Bridge.*`, detectando todo en runtime.

Es **gratis** y open-source. Los demás recursos Vornex dependen de él.

🔗 **Repositorio:** https://github.com/VornexV2/vornex_bridge

---

## Características

- Detección automática de framework, inventario y target (con override por convar).
- Jugador **normalizado**: misma estructura sin importar el framework.
- API de dinero, trabajos, gangs, metadata y permisos, online y offline.
- Wrappers de inventario (server y client) y de target (client).
- Capa de vehículos de jugador y consultas a base de datos (ciudadanos, vehículos, búsquedas).
- Sistema de zonas: persistencia en DB y creador de zonas con freecam.
- Chequeo de versiones contra GitHub Releases con changelog en consola.

---

## Instalación

**Opción A: Release**

1. Descargá la última [release](https://github.com/VornexV2/vornex_bridge/releases).
2. Poné la carpeta `vornex_bridge` en tu carpeta de recursos.

**Opción B: Clonar el repositorio**

```bash
git clone https://github.com/VornexV2/vornex_bridge.git
```

Copiá la carpeta interna `vornex_bridge/vornex_bridge` (la que contiene el `fxmanifest.lua`) a tu carpeta de recursos.

Después asegurate de iniciarlo **antes** de cualquier recurso Vornex:

```cfg
ensure vornex_bridge
```

---

## Dependencias

| Requerido | Detalle |
|-----------|---------|
| [ox_lib](https://github.com/overextended/ox_lib) | Siempre |
| [oxmysql](https://github.com/overextended/oxmysql) | Para DB, vehículos, zonas y jugadores offline |
| Framework | **qbox**, **qbcore** o **esx** |
| Inventario | **ox_inventory** o **qb-inventory** |
| Target | **ox_target** o **qb-target** |

### Soportados

| Capa | Opciones |
|------|----------|
| Framework | qbox · qbcore · esx |
| Inventario | ox_inventory · qb-inventory |
| Target | ox_target · qb-target |

---

## Uso en tu recurso

Agregá el bridge a los `shared_scripts` de tu `fxmanifest.lua`:

```lua
shared_scripts {
    '@ox_lib/init.lua',
    '@vornex_bridge/init.lua',
}
```

Después usá el global `Bridge` desde el server o el client:

```lua
-- server
local player = Bridge.GetPlayer(src)
Bridge.AddMoney(src, 'bank', 500, 'reason')
Bridge.Inventory.AddItem(src, 'water', 1)
Bridge.VersionCheck('VornexV2/mi_recurso')

-- client
Bridge.Target.addLocalEntity(entity, options)
local job = Bridge.GetJob()
```

Podés saber con qué está corriendo:

```lua
Bridge.framework            -- 'qbox' | 'qbcore' | 'esx'
Bridge.inventory            -- 'ox_inventory' | 'qb-inventory'
Bridge.target               -- 'ox_target' | 'qb-target'
Bridge.context              -- 'server' | 'client'

Bridge.is('esx')
Bridge.usesInventory('ox_inventory')
Bridge.usesTarget('qb-target')
```

---

## Configuración (convars)

Por defecto todo se detecta solo (`auto`). Si necesitás forzarlo, agregalo en tu `server.cfg`:

```cfg
setr vornex_bridge:framework   "qbox"           # qbox | qbcore | esx
setr vornex_bridge:inventory   "ox_inventory"   # ox_inventory | qb-inventory
setr vornex_bridge:target      "ox_target"      # ox_target | qb-target
```

Orden de detección del framework: `qbx_core` → `qb-core` → `es_extended`. Si no encuentra ninguno usa `qbox` como fallback y lo avisa en consola.

Convars opcionales para esquemas de base de datos personalizados:

| Convar | Default |
|--------|---------|
| `vornex_bridge:charTable` | `players` |
| `vornex_bridge:charIdColumn` | `citizenid` |
| `vornex_bridge:vehTable` | `player_vehicles` |
| `vornex_bridge:vehOwnerColumn` | `citizenid` |

---

## Objeto jugador normalizado

`Bridge.GetPlayer(src)` devuelve la misma estructura en todos los frameworks:

```lua
{
    source    = 1,
    citizenid = 'ABC12345',
    name      = '...',
    charinfo  = { ... },
    money     = { cash = 0, bank = 0, ... },
    job  = { name, label, grade, gradeLabel, isboss, onduty, type },
    gang = { name, label, grade, gradeLabel, isboss },
    metadata  = { ... },
    _raw      = <objeto original del framework>,
}
```

Si necesitás el objeto nativo del framework usá `Bridge.GetRawPlayer(src)`.

---

## API del servidor

### Jugadores
`GetPlayer(src)` · `GetRawPlayer(src)` · `GetPlayerByCitizenId(cid)` · `GetOfflinePlayer(cid)` · `GetPlayers()` · `GetSource(identifier)` · `GetIdentifier(src)`

### Dinero
`GetMoney(src, account)` · `AddMoney(src, account, amount, reason)` · `RemoveMoney(src, account, amount, reason)`
Offline: `GetMoneyOffline` · `AddMoneyOffline` · `RemoveMoneyOffline` (por identifier)

### Trabajos y gangs
`GetJob(src)` · `GetJobInfo(name)` · `GetJobs()` · `SetJob(src, name, grade)` · `SetDuty(src, onDuty)` · `GetDutyCount(jobType)` · `GetDutyCountJob(jobName)` · `GetGroupMembers(name, gtype)` · `AddPlayerToJob` · `RemovePlayerFromJob` · `SetPrimaryJob`
`GetGang(src)` · `SetGang(src, name, grade)` · `CreateGangs(gangs)` · `RemoveGang(name)` · `IsGradeBoss(gangName, grade)`

### Otros
`GetMetadata(src, key)` · `SetMetadata(src, key, value)` · `SetCharName(src, first, last)` · `HasPermission(src, perm)` · `IsAceAllowed(src, ace)` · `CreateUseableItem(name, cb)` · `Notify(src, message, type, duration)` · `DeleteVehicle(entity)` · `GetVehiclesByName(name)` · `GetVehiclesByHash(hash)`

### Eventos
`OnPlayerLoaded(cb)` · `OnPlayerUnload(cb)` · `OnPlayerDropped(cb)` · `OnJobUpdate(cb)` · `OnMoneyChange(cb)`

### Inventario (`Bridge.Inventory`)
`AddItem` · `RemoveItem` · `GetItemCount` · `Search` · `CanCarryItem` · `GetItem` · `GetSlot` · `SetMetadata` · `GetInventoryItems` · `GetInventory` · `ClearInventory` · `RegisterStash`

Solo ox_inventory: `openInventory` · `openNearbyInventory` · `forceOpenInventory` · `registerHook` · `displayMetadata`. En qb-inventory estas funciones son *no-op* y avisan por consola.

### Vehículos (`Bridge.Vehicles`)
`GetPlayerVehicle` · `GetPlayerVehicles` · `GetVehicleIdByPlate` · `SaveVehicle` · `CreatePlayerVehicle` · `SetPlayerVehicleOwner` · `DeletePlayerVehicles` · `SetState` · `SetStateGarage` · `SetStateDepotPrice` · `SetGarage` · `CountByGarage` · `MileageFor` · `AdminList`

Operaciones atómicas (solo cambian si el estado previo coincide): `ClaimOwner` · `ClaimState` · `ClaimStateGarage` · `ClaimGarage`

Tablas: `player_vehicles` en qbox/qbcore, `owned_vehicles` en ESX.

### Base de datos (`Bridge.DB`)
`GetCitizen` · `SearchCitizens` · `GetWanted` · `GetCitizenFlags` · `SetCitizenFlag` · `SetCitizenImage` · `GetCitizenImages` · `GetCitizenInfoBatch` · `GetVehiclesByOwner` · `SearchVehicles` · `GetVehicleByPlate` · `GetVehicleScan` · `SetVehicleField` · `ListPlayers` · `GetMoneyTotals` · `GetTopPlayers` · `GetJobMembers`

---

## API del cliente

`GetPlayerData()` · `IsLoggedIn()` · `GetJob()` · `GetJobInfo(name)` · `GetGang()` · `GetMetadata(key)`
Eventos: `OnPlayerLoaded` · `OnPlayerUnload` · `OnJobUpdate` · `OnGangUpdate`

Utilidades:

```lua
Bridge.Notify('Mensaje')                                   -- string o tabla de ox_lib
Bridge.Notify({ title = 'Hola', type = 'success' })
Bridge.GetVehicleLabel('adder')                            -- "Truffade Adder"
Bridge.GetVehicleClass('adder')                            -- clase GTA (0-22) o -1
```

### Target (`Bridge.Target`)
`addModel` · `removeModel` · `addLocalEntity` · `removeLocalEntity` · `addGlobalPlayer` · `removeGlobalPlayer` · `addBoxZone` · `removeZone`

```lua
local id = Bridge.Target.addBoxZone({
    coords   = vec3(0.0, 0.0, 0.0),
    size     = vec3(1.0, 1.0, 1.0),
    rotation = 0.0,
    options  = { ... },
})
Bridge.Target.removeZone(id)
```

### Inventario (cliente)
`useItem` · `Search` · `displayMetadata` · `GetItemCount` · `weaponWheel` · `openNearbyInventory`

---

## Zonas

Persistencia de zonas poligonales en una tabla propia (cada recurso elige su `tableName`). La tabla necesita las columnas `id`, `label`, `geometry`, `metadata`, `active` y `created_by`.

**Server**

```lua
local zones = Bridge.Zones.DB.Fetch('mi_tabla', true)   -- solo activas
local id    = Bridge.Zones.DB.Insert('mi_tabla', { label = 'Zona', geometry = geo, metadata = {}, created_by = cid })
Bridge.Zones.DB.Update('mi_tabla', id, { label = 'Nuevo nombre' })
Bridge.Zones.DB.SetActive('mi_tabla', id, false)
Bridge.Zones.DB.Delete('mi_tabla', id)
```

**Client**: creador interactivo con freecam:

```lua
Bridge.Zones.OpenCreator({}, function(geometry)
    if not geometry then return end   -- cancelado
    -- guardar la geometría
end)
```

Controles: `Mouse Izq` agrega punto · `Mouse Der` deshace · `Flechas/Scroll` ajusta altura · `Enter` guarda · `Backspace` cancela.

---

## Chequeo de versiones

```lua
Bridge.VersionCheck('VornexV2/mi_recurso', { changelog = true, maxLines = 10 })
```

Compara la `version` del `fxmanifest.lua` del recurso que llama con la última release de GitHub (el tag debe tener formato `vX.Y.Z`) e imprime un banner en consola (`MAJOR` / `MINOR` / `PATCH`) con las novedades y el link de descarga.

---

## Notas por framework

- **qbcore:** el multi-job no está soportado en el core base (`AddPlayerToJob` / `RemovePlayerFromJob` solo avisan); requiere `qb-multijob`.
- **esx:** `GetOfflinePlayer` requiere `oxmysql`.
- Las funciones no implementadas en un adaptador lanzan un error claro indicando la función y el adaptador.

---

## Estructura

```
vornex_bridge/
├── init.lua              # Punto de entrada, carga de módulos
├── fxmanifest.lua
├── shared/detect.lua     # Detección de framework / inventario / target
├── server/
│   ├── api.lua           # Contrato de la API
│   ├── adapters/         # qbox · qbcore · esx
│   ├── inventory/        # ox_inventory · qb-inventory
│   ├── vehicles/         # qbox · qbcore · esx
│   ├── db/               # qbox · qbcore · esx
│   ├── zones.lua
│   └── version.lua
└── client/
    ├── api.lua
    ├── adapters/         # qbox · qbcore · esx
    ├── inventory/        # ox_inventory · qb-inventory
    ├── target/           # ox_target · qb-target
    └── zones.lua
```

---

## Soporte

¿Encontraste un bug o querés proponer algo? Abrí un [issue](https://github.com/VornexV2/vornex_bridge/issues).

## Licencia

Open-source y gratuito. Consultá el repositorio para los términos de la licencia.
