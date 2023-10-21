# pasos prueba "el tuercas"

## user creado por devise con campos adicionales:
* role
* firstname
* lastname

## [[[[VEHICLES]]]]

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

## Para mostrar todos los users con su vehicles asociados y poder editar y crear vehicles, se debe sustituir en el vehicle_controller por este codigo app/controllers/vehicle_controller.rb

```bash
class VehiclesController < ApplicationController
  before_action :set_vehicle, only: %i[ show edit update destroy ]

  # GET /vehicles or /vehicles.json
  def index
    # Obtener todos los usuarios con sus vehículos asociados
    @users = User.includes(:vehicles)
    @pagy, @users = pagy(User.order(created_at: :desc), items: 5) # Paginación
    # Preparar un array de [user, Vehículos] para la vista
    @users_with_vehicles = @users.map { |user| [user, user.vehicles] }

  end

  # GET /vehicles/1 or /vehicles/1.json
  def show
  end

  # GET /vehicles/new
  def new
    @vehicle = Vehicle.new
    @user = User.find(params[:user_id]) # Obtén el usuario correspondiente
    @vehicle.user_id = @user.id # Asigna el user_id del usuario al vehículo
  end

  # GET /vehicles/1/edit
  def edit
  end

  # POST /vehicles or /vehicles.json
  def create
    @vehicle = Vehicle.new(vehicle_params)
    @vehicle.user_id = params[:vehicle][:user_id] # Asigna el user_id del formulario al vehículo
    @user = User.find(params[:vehicle][:user_id]) # Obtén el usuario correspondiente
    respond_to do |format|
      if @vehicle.save
        format.html { redirect_to vehicle_url(@vehicle), notice: "Vehicle was successfully created." }
        format.json { render :show, status: :created, location: @vehicle }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @vehicle.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /vehicles/1 or /vehicles/1.json
  def update
    # @vehicle = Vehicle.find(params[:id])
  
    respond_to do |format|
      if @vehicle.update(vehicle_params)
        format.html { redirect_to vehicle_url(@vehicle), notice: "Vehicle was successfully updated." }
        format.json { render :show, status: :ok, location: @vehicle }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @vehicle.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /vehicles/1 or /vehicles/1.json
  def destroy
    @vehicle.destroy

    respond_to do |format|
      format.html { redirect_to vehicles_url, notice: "Vehicle was successfully destroyed." }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_vehicle
      @vehicle = Vehicle.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def vehicle_params
      params.require(:vehicle).permit(:brand, :model, :plate_number, :user_id)
    end
end

```

## Ejecutar migracion y hacer commit

```bash
rails db:migrate
git add .
git commit -m "Scaffold vehicle creado, relacion con user definida"
```

**NOTA**: En este modelo, un vehículo pertenece a un usuario (belongs_to :user) y el user puede tener muchos vehicles (has_many :vehicles).

## app/views/vehicles/index.html.erb sustituir:

```bash
<div class='container'>
<div id="vehicles">
  <% @users_with_vehicles.each do |user, vehicles| %>
<h5><%= user.full_name %> <%= link_to 'Create New Vehicle', new_vehicle_path(user_id: user.id) %></h5>
    <% if vehicles.present? %>
      <table class="table">
        <thead>
          <tr>
            <th scope="col">Brand</th>
            <th scope="col">Model</th>
            <th scope="col">Plate number</th>
            <th scope="col">Action</th>
          </tr>
        </thead>
        <tbody>
          <% vehicles.each do |vehicle| %>
            <%= render partial: 'vehicle', locals: { vehicle: vehicle, show_link: true } %>
          <% end %>
        </tbody>
      </table>
    <% else %>
      <p>No hay vehículo registrado para este usuario.</p>
    <% end %>
  <% end %>
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

## Sustituir el formulario vehicle app/views/vehicles/_form.html.erb

```bash
<%= form_with(model: vehicle) do |form| %>
  <% if vehicle.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(vehicle.errors.count, "error") %> prohibited this vehicle from being saved:</h2>
      <ul>
        <% vehicle.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :brand, style: "display: block" %>
    <%= form.text_field :brand, class:'form-control', required: true %>
  </div>

  <div>
    <%= form.label :model, style: "display: block" %>
    <%= form.text_field :model, class:'form-control', required: true %>
  </div>

  <div>
    <%= form.label :plate_number, style: "display: block" %>
    <%= form.text_field :plate_number, class:'form-control', required: true %>
  </div>

  <!--Toma el user_id del user seleccionado y lo pasa al form para la relación-->
  <%= form.hidden_field :user_id, value: @vehicle.user_id %>
  
  <div>
    <%= form.submit "Create", class:"btn btn-success" %>
  </div>
<% end %>
```

## Agregar bootstrap al form new app/views/vehicles/new.html.erb

```bash
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-4 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">New vehicle for <%= @user.full_name %></h2>
        </div>
        <div class="card-body">
          <div class="mb-4">
            <%= render "form", vehicle: @vehicle %>
            <div>
              <%= link_to "Back to vehicles", vehicles_path %>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

## Agregar al navbar la opción list all vehicles dentro de las opciones del admin

```bash
<li class="nav-item">
    <%= link_to 'List vehicles', vehicles_path, class: 'nav-link' %>  
</li>
```

## Para evitar errores por las relaciones, se debe poblar la base de datos con varios vehicles y asociarlos a los users, copiar el app/db/see.rb

```bash
# Crea 10 usuarios con datos ficticios
10.times do
    User.create(
      email: Faker::Internet.email,
      password: Faker::Internet.password,
      firstname: Faker::Name.first_name,
      lastname: Faker::Name.last_name,
      role: "user"
    )
  end
  
  # Crea 30 vehículos asociados a usuarios existentes
  users = User.all
  
  30.times do
    Vehicle.create(
      brand: Faker::Vehicle.make,
      model: Faker::Vehicle.model,
      plate_number: Faker::Vehicle.license_plate,
      user: users.sample
    )
  end
  
  puts "Seed data generated successfully!"
  ```

## Hago commit

```bash
git add .
git commit -m "Se modificaron las vistas index y _vehicle, se agregó al seed la creacion de 30 vehicles asociados a users"
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


