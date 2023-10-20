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
# Relación uno a muchos con vehicles
has_many :vehicles

# Método para obtener el nombre completo del usuario en la vista index de vehicles
def full_name
  "#{firstname} #{lastname}"
end
```

## Agregar al modelo Vehicle:

```bash
# Relación con user
belongs_to :user
```

## Para mostrar todos los users con su vehicles asociados se debe sustituir en el vehicle_controller el método de acción index en app/controllers/vehicle_controller.rb

```bash
  def index
    # Obtener todos los usuarios con sus vehículos asociados
    @users = User.includes(:vehicles)
  
    # Preparar un array de [Nombre Completo, Vehículos] para la vista
    @users_with_vehicles = @users.map { |user| [user.full_name, user.vehicles] }
  end
```

## Ejecutar migracion y hacer commit

```bash
rails db:migrate
git add .
git commit -m "Scaffold vehicle creado, relacion con user definida"
```

**NOTA**: En este modelo, un vehículo pertenece a un usuario (belongs_to :user) y el user puede tener muchos vehicles (has_many :vehicles).

## Mostrar en el index de vehicle los vehicles asociados a cada user: 

## app/views/vehicles.html.erb sustituir:

```bash
<div class="container">
  <div id="vehicles">
    <% @users_with_vehicles.each do |user_name, vehicles| %>
      <h2><%= user_name %></h2>
      <table class="table">
        <thead>
          <tr>
            <th scope="col">Brand</th>
            <th scope="col">Model</th>
            <th scope="col">Plate number</th>
             
            <th scope="col">Action</th>
          </tr>
        </thead>
  
      <% if vehicles.present? %>
        <% vehicles.each do |vehicle| %>
          <%= render partial: 'vehicle', locals: { vehicle: vehicle, show_link: true } %>
  
        <% end %>
      <% else %>
        <p>No hay vehículo registrado para este usuario.</p>
      <% end %>
    </table>
    <% end %>
  </div>
</div>
```

## en app/views/vehicles/_vehicle.html.erb sustituir:

```bash
<div id="<%= dom_id vehicle %>">

<tbody>
  <tr>
    <td><%= vehicle.brand %></td>
    <td><%= vehicle.model %></td>
    <td><%= vehicle.plate_number %></td>
    <td>  <% if local_assigns[:show_link] %>
    <p>
      <%= link_to 'Show this vehicle', vehicle %>
    </p>
  <% end %></td>
  </tr>
</tbody>
```

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

## Ocultar en el show el enlace [show this vehicle]

## Sustituir en app/views/vehicles/show.html.erb

```bash
<div class="container">
  <%= render @vehicle %>
  <h2>Appointment Details</h2>
  
    <% if @vehicle.appointment %>
      <p><strong>Appointment Date:</strong> <%= @vehicle.appointment.appointment_date %></p>
    <% else %>
      <p>No appointment scheduled for this vehicle.</p>
    <% end %>

  <%= form_with(model: [@vehicle, Appointment.new], url: vehicle_path(@vehicle), method: :patch, local: true) do |form| %>
    <div class="field">
      <%= form.label :appointment_date, 'Appointment Date' %>
      <%= form.datetime_local_field :appointment_date %>
    </div>

    <div class="actions">
      <%= form.submit "Schedule Appointment", class: "btn btn-primary" %>
    </div>
  <% end %>

  <%= link_to "Edit this vehicle", edit_vehicle_path(@vehicle) %> |
  <%= link_to "Back to vehicles", vehicles_path %>
  <%= button_to "Destroy this vehicle", @vehicle, method: :delete %>
</div>
```

# =============================================

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


