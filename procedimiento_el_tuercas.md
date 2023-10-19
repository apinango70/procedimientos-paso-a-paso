# pasos prueba "el tuercas"

## user creado por devise con campos adicionales:
* role
* firstname
* lastname

## scaffold vehicle y relacion 1:n con user 

```bash
rails g scaffold vehicle brand:string model:string plate_number:string user:references
```

## agregar al modelo User:

```bash
has_many :vehicles
```

## Agregar al modelo Vehicle:

```bash
belongs_to :user
```
## Ejecutar migracion y hacer commit

```bash
rails db:migrate
git add .
git commit -m "Scaffold vehicle creado, relacion con user definida"
```

**NOTA**: En este modelo, un vehículo pertenece a un usuario (belongs_to :user) y el user puede tener muchos vehicles (has_many :vehicles).

## Crear scaffold service:

```bash
rails g scaffold service service_name:string spare_parts:string deadline:datetime status:string vehicle_id:integer
```

## Agregar en el modelo vehicle:

```bash
class Vehicle < ApplicationRecord
  belongs_to :service
  has_and_belongs_to_many :services

  validates :brand, :model, :year, :plate_number, presence: true
end
```


## Agregar al modelo service y agregar los enum de tipo de servicio y status:

```bash
class Service < ApplicationRecord
    has_and_belongs_to_many :vehicles
  
    enum service_name: { 
                        Mantenimiento: 'Mantenimiento', 
                        Afinacion: 'Afinación', 
                        Cambio_neumaticos: 'Cambio de neumáticos', 
                        Alineado_balance: 'Alineado y balance', 
                        Cambio_aceite_filtro: 'Cambio de aceite y filtro', 
                        Reparacion_frenos: 'Reparación de frenos', 
                        Latoneria_pintura: 'Latonería y pintura' }

    enum status: { 
                  en_progreso: 'en progreso', 
                  finalizado: 'finalizado' }
  
    validates :service_name, :status, presence: true
  end
```

## Ejecutar migracion y hacer commit

```bash
rails db:migrate
git add .
git commit -m "Scaffold service y enumeradores creados"
```

## Tabla intermedia vehicle_services relación n:n

```bash
rails g migration CreateVehicleServices vehicle:references service:references
```

 NOTA:Un servicio tiene y pertenece a muchos vehículos (has_and_belongs_to_many :vehicles).

## Creo el modelo  appointment y efino relacion 1:n entre vehicle y appointment

```bash
rails g model appointment appointment_date:datetime vehicle_id:integer vehicle:references
```

## Agregar al modelo appointment

```bash
  belongs_to :vehicle

  validates :appointment_date , presence: true
end
```

## Agregar al modelo vehicle

```bash
  has_one :appointment
```

NOTA: cada vehicle solo puede tener un appointment y un appointment puede tener vairos vehicles
