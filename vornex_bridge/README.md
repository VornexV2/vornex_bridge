# vornex_bridge

Capa de compatibilidad multi-framework para los recursos **Vornex**. Abstrae el framework (qbox / qbcore / esx), el inventario (ox_inventory / qb-inventory) y el sistema de target (ox_target / qb-target) detrás de una sola API `Bridge.*`, detectando todo en runtime.

Es **gratis** y open-source. Los demás recursos Vornex dependen de él.

## Instalación

1. Descargá la última [release](https://github.com/Vornexv2/vornex_bridge/releases).
2. Poné `vornex_bridge` en tu carpeta de recursos.
3. `ensure vornex_bridge` **antes** de cualquier recurso Vornex.

```cfg
ensure vornex_bridge
```

## Dependencias

- [ox_lib](https://github.com/overextended/ox_lib)
- Un framework: **qbox**, **qbcore** o **esx**
- Inventario: **ox_inventory** o **qb-inventory**
- Target: **ox_target** o **qb-target**

## Uso

```lua
shared_scripts {
    '@ox_lib/init.lua',
    '@vornex_bridge/init.lua',
}
```

```lua
local player = Bridge.GetPlayer(src)
Bridge.AddMoney(src, 'bank', 500, 'reason')
Bridge.Target.addLocalEntity(entity, options)
Bridge.VersionCheck('owner/repo')
```

## Soportados

| Capa | Opciones |
|------|----------|
| Framework | qbox · qbcore · esx |
| Inventario | ox_inventory · qb-inventory |
| Target | ox_target · qb-target |
