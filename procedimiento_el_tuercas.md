# pasos prueba "el tuercas"

## user creado por devise con campos adicionales:
*role
*firstname
*lastname

## scaffold vehicle


rails g scaffold vehicle brand:string model:string plate_number:string user_id:integer



Modelo User:
class User < ApplicationRecord
  has_many :vehiculos
end



## Modelo Vehicle:


class Vehicle < ApplicationRecord
  belongs_to :user
end


## scaffold service:


rails g scaffold service service_name:string spare_parts:string deadline:datetime status:string vehicle_id:integer


## Tabla intermedia vehicle_services relación n:n


rails g migration CreateVehicleServices vehicle:references service:references

## Modelo vehicle:

class Vehicle < ApplicationRecord
  belongs_to :service
  has_and_belongs_to_many :services

  validates :brand, :model, :year, :plate_number, presence: true
end

### En este modelo, un vehículo pertenece a un usuario (belongs_to :user), tiene muchas citas (has_many :appointments), y tiene y pertenece a muchos servicios (has_and_belongs_to_many :services).

## Modelo service:

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

## Un servicio tiene y pertenece a muchos vehículos (has_and_belongs_to_many :vehicles).

modelo appointment y la relacion 1:n a vehicle
rails g model appointment appointment_date:datetime vehicle_id:integer vehicle:references

class Appointment < ApplicationRecord
  belongs_to :vehicle

  validates :appointment_date , presence: true
end





Defino relacion 1:n entre vehicle y appointment



modelo tabla intermedia vehicle_service
rails generate migration CreateJoinTableVehicleService s genders
