# pasos prueba "el tuercas"

## Estructura del proyecto:

* user
* vehicle
* appointment
* service

## Relaciones:

* user:vehicle 1:n
* vehicle:appointment 1:n
* vehicle:service n:n

## [[[[VEHICLES]]]]

## scaffold vehicle y relacion 1:n con user 

```bash
rails g scaffold vehicle brand:string model:string plate_number:string user:references
```

## agregar al modelo User:

```bash
# Relación uno a muchos con vehicles
has_many :vehicles, dependent: :destroy

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
    @users = User.all
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
              <th scope="col">Actions</th>
            </tr>
          </thead>
          <tbody>
            <% vehicles.each do |vehicle| %>
              <%= render partial: 'vehicle', locals: { vehicle: vehicle, show_link: true } %>
            <% end %>
          </tbody>
        </table>
      <% else %>
        <p>There is no vehicle registered for this user.</p>
      <% end %>
    <% end %>
  </div>
</div>
<!-- Si tiene instalado pagy habilitar el control -->
<!--
<div class=" d-flex justify-content-center align-items-center">
  <%== pagy_bootstrap_nav(@pagy) %>
</div>
-->
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
      <%= link_to vehicle, class:'btn' do%>
      <!--Agrego icono show en formato svg de fontawesome y convierto el link Show a un botón-->
        <svg xmlns="http://www.w3.org/2000/svg" height="1.5em" viewBox="0 0 576 512"><!--! Font Awesome Free 6.4.2 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2023 Fonticons, Inc. --><style>svg{fill:#006eff}</style><path d="M288 80c-65.2 0-118.8 29.6-159.9 67.7C89.6 183.5 63 226 49.4 256c13.6 30 40.2 72.5 78.6 108.3C169.2 402.4 222.8 432 288 432s118.8-29.6 159.9-67.7C486.4 328.5 513 286 526.6 256c-13.6-30-40.2-72.5-78.6-108.3C406.8 109.6 353.2 80 288 80zM95.4 112.6C142.5 68.8 207.2 32 288 32s145.5 36.8 192.6 80.6c46.8 43.5 78.1 95.4 93 131.1c3.3 7.9 3.3 16.7 0 24.6c-14.9 35.7-46.2 87.7-93 131.1C433.5 443.2 368.8 480 288 480s-145.5-36.8-192.6-80.6C48.6 356 17.3 304 2.5 268.3c-3.3-7.9-3.3-16.7 0-24.6C17.3 208 48.6 156 95.4 112.6zM288 336c44.2 0 80-35.8 80-80s-35.8-80-80-80c-.7 0-1.3 0-2 0c1.3 5.1 2 10.5 2 16c0 35.3-28.7 64-64 64c-5.5 0-10.9-.7-16-2c0 .7 0 1.3 0 2c0 44.2 35.8 80 80 80zm0-208a128 128 0 1 1 0 256 128 128 0 1 1 0-256z"/></svg>
      <% end %>
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

## [[[[[[[[[[[[[[APPOINMENT]]]]]]]]]]]]]]

## Creo el modelo appointment y defino relacion 1:n entre vehicle y appointment

```bash
rails g model appointment appointment_date:date vehicle:references
```

## Hago migración y ejecuto commit

```bash
rails db:migrate
git add .
git commit -m "Modelo appointment creado"
```

## Agrego al modelo appointment

```bash
  belongs_to :vehicle
```

## Agrego al modelo vehicle

```bash
    has_one :appointment, dependent: :destroy
    #Para ceptar atributos anidados
    accepts_nested_attributes_for :appointment
```

NOTA: cada vehicle solo puede tener un appointment y un appointment puede tener varios vehicles

## Sustituyo el código del vehicle_controller para pasar el nombre del user para una vista show personalizada

```bash
class VehiclesController < ApplicationController
  before_action :set_vehicle, only: %i[ show edit update destroy ]

  # GET /vehicles or /vehicles.json
  def index
    # Obtener todos los usuarios con sus vehículos asociados
    @users = User.includes(:vehicles)
    @users = User.all
    #HAbilitar para usar paginación
    #@pagy, @users = pagy(User.order(created_at: :desc), items: 5) # Paginación
    # Preparar un array de [user, Vehículos] para la vista
    @users_with_vehicles = @users.map { |user| [user, user.vehicles] }

  end

  # GET /vehicles/1 or /vehicles/1.json
  def show
    @vehicle = Vehicle.find(params[:id])
    @user = @vehicle.user
    @appointment = @vehicle.appointment
  end

  # GET /vehicles/new
  def new
    @vehicle = Vehicle.new
    @user = User.find(params[:user_id]) # Obtén el usuario correspondiente
    @vehicle.user_id = @user.id # Asigna el user_id del usuario al vehículo
    @vehicle.build_appointment # Esto crea un objeto Appointment asociado al vehículo
  end

  # GET /vehicles/1/edit
  def edit
    @vehicle = Vehicle.find(params[:id])
    @appointment = @vehicle.appointment || @vehicle.build_appointment
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
    @vehicle = Vehicle.find(params[:id])
    @appointment = @vehicle.appointment
    
    # Actualiza la fecha de cita en el Appointment, no en el Vehicle
    if @appointment.update(appointment_date: params[:vehicle][:appointment_date])
      respond_to do |format|
        if @vehicle.update(vehicle_params)
          format.html { redirect_to vehicle_url(@vehicle), notice: "Vehicle was successfully updated." }
          format.json { render :show, status: :ok, location: @vehicle }
        else
          format.html { render :edit, status: :unprocessable_entity }
          format.json { render json: @vehicle.errors, status: :unprocessable_entity }
        end
      end
    else
      # Maneja errores de validación del appointment si es necesario
      # ...
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
      params.require(:vehicle).permit(
        :brand, 
        :model, 
        :plate_number, 
        :user_id, 
        appointment_attributes: [:id, :appointment_date]  # Asegúrate de incluir los atributos del appointment
      )
    end
    
end
```

## Modifico la vista index para aregar la columna appointment app/views/vehicles/index.html.erb

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
              <th scope="col">Appointment</th>
              <th scope="col">Actions</th>
            </tr>
          </thead>
          <tbody>
            <% vehicles.each do |vehicle| %>
              <%= render partial: 'vehicle', locals: { vehicle: vehicle, show_link: true } %>
            <% end %>
          </tbody>
        </table>
      <% else %>
        <p>There is no vehicle registered for this user.</p>
      <% end %>
    <% end %>
  </div>
</div>
```

## Modifico elpartial de _vehicle para agregar el campo appointment al listado app/views/vehicles/_vehicle.html.erb

```bash
<div id="<%= dom_id vehicle %>">
<tbody>
  <tr>
    <td><%= vehicle.brand %></td>
    <td><%= vehicle.model %></td>
    <td><%= vehicle.plate_number %></td>
    <td>
        <% if vehicle.appointment && vehicle.appointment.appointment_date %>
          <p><%= vehicle.appointment.appointment_date.strftime('%d-%m-%Y') %></p>
        <% else %>
          <p><strong>There is no appointment scheduled for this vehicle.</strong></p>
        <% end %>  
    </td>
    <td>  <% if local_assigns[:show_link] %>
    <p>
      <%= link_to vehicle, class:'btn' do%>
      <!--Agrego icono show en formato svg de fontawesome-->
        <svg xmlns="http://www.w3.org/2000/svg" height="1.5em" viewBox="0 0 576 512"><!--! Font Awesome Free 6.4.2 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2023 Fonticons, Inc. --><style>svg{fill:#006eff}</style><path d="M288 80c-65.2 0-118.8 29.6-159.9 67.7C89.6 183.5 63 226 49.4 256c13.6 30 40.2 72.5 78.6 108.3C169.2 402.4 222.8 432 288 432s118.8-29.6 159.9-67.7C486.4 328.5 513 286 526.6 256c-13.6-30-40.2-72.5-78.6-108.3C406.8 109.6 353.2 80 288 80zM95.4 112.6C142.5 68.8 207.2 32 288 32s145.5 36.8 192.6 80.6c46.8 43.5 78.1 95.4 93 131.1c3.3 7.9 3.3 16.7 0 24.6c-14.9 35.7-46.2 87.7-93 131.1C433.5 443.2 368.8 480 288 480s-145.5-36.8-192.6-80.6C48.6 356 17.3 304 2.5 268.3c-3.3-7.9-3.3-16.7 0-24.6C17.3 208 48.6 156 95.4 112.6zM288 336c44.2 0 80-35.8 80-80s-35.8-80-80-80c-.7 0-1.3 0-2 0c1.3 5.1 2 10.5 2 16c0 35.3-28.7 64-64 64c-5.5 0-10.9-.7-16-2c0 .7 0 1.3 0 2c0 44.2 35.8 80 80 80zm0-208a128 128 0 1 1 0 256 128 128 0 1 1 0-256z"/></svg>
      <% end %>
    </p>
  <% end %></td>
  </tr>
</tbody>
``` 

## Creo un partial personalizado para renderizarlo en la vista show de vehicle en app/views/vehicles/_vehicle_details.html.erb

```bash
<div id="<%= dom_id vehicle %>">
  <div class="card-body">
    <h5 class="card-title"><%= label_tag "Client: " %> <%= @user.full_name %></h5>
    <p class="card-text"><%= label_tag "Brand: " %> <%= vehicle.brand %></p>
    <p class="card-text"><%= label_tag "Model: " %> <%= vehicle.model %></p>
    <p class="card-text"><%= label_tag "Plate number: " %> <%= vehicle.plate_number %></p>

    <% if @appointment && @appointment.appointment_date %>
      <p><strong>Appointment date:</strong> <%= @appointment.appointment_date.to_date.strftime('%d-%m-%Y') %></p>
    <% else %>
      <p><strong>There is no appointment scheduled for this vehicle.</strong></p>
    <% end %>
    
    <%= link_to 'Edit', edit_vehicle_path(vehicle), class: "btn btn-primary" %>
    <%= link_to "Back to vehicles", vehicles_path, class: "btn btn-primary" %>
  </div>
</div>
```

## Sustituyo el codigo de app/views/vehicles/show.html.erb para que renderize el partial personalizado y aplique estilo bootstrap:

```bash
    <div class="container">
      <div class="row justify-content-center mt-5">
        <div class="col-lg-4 col-md-6 col-sm-6">
          <div class="card shadow">
            <div class="card-title text-center border-bottom">
              <h2 class="p-3">Vehicle info</h2>
            </div>
            <div class="card-body">
              <div id="<%= dom_id @vehicle %>">
                <%= render partial: 'vehicles/vehicle_details', locals: { vehicle: @vehicle, show_link: false } %>

              </div>
              
              <%= button_to "Destroy this vehicle", @vehicle, method: :delete, class: "btn btn-danger", style: "text-decoration:none;color:white;" %>

            </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

## Agrego el control selector de fecha en el form para signar un appointment app/views/vehicles/_form.html.erb

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

  <!--Aplico método do por si el vehiculo tiene cita agendada la muestre en el control de fecha -->
  <%= form.fields_for :appointment do |appointment_form| %>
    <%= appointment_form.hidden_field :id %>
    <div>
      <%= appointment_form.label :appointment_date, "Appointment date", style: "display: block" %>
      <%= appointment_form.date_field :appointment_date, class: 'form-control' %>
    </div>
  <% end %>

  <!--Toma el user_id del user seleccionado y lo pasa al form para la relación-->
  <%= form.hidden_field :user_id, value: @vehicle.user_id %>
  
  <div>
    <%= form.submit "Create", class:"btn btn-success" %>
  </div>
<% end %>
```

## Modifico la vista edit app/views/vehicles/edit.html.erb

```bash
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-4 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">Edit vehicle </h2>
        </div>
        <div class="card-body">
          <div class="mb-4">
            <%= render "form", vehicle: @vehicle %>
            <div>
              <%= link_to "Show this vehicle", @vehicle %> |
              <%= link_to "Back to vehicles", vehicles_path %>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

## Ejecuto commit

```bash
git add .
git commit -m "Vistas y métodos modificados para anidar appointment a vehicle"
```

## Sustituyo el seed.rb para crear vehículos con citas asginadas y ejecuto rails rb:seed

```bash
# Se deben borrar todos los registros existentes antes de crear nuevos registros con el campo appointment para que no nos de error al editarlos
User.destroy_all
Vehicle.destroy_all
Appointment.destroy_all

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

# Crea 30 vehículos asociados a usuarios existentes y establece fechas de cita
users = User.all

30.times do
  user = users.sample
  vehicle = Vehicle.create(
    brand: Faker::Vehicle.make,
    model: Faker::Vehicle.model,
    plate_number: Faker::Vehicle.license_plate,
    user: user
  )

  # Establece una fecha de cita para el vehículo
  vehicle.create_appointment(appointment_date: Faker::Date.forward(days: 30))
end

puts "Seed data generated successfully!"
```

## Agregar pagy a la vista index de vehicle. Seguir los pasos de instalación de la gema pagy. En app/controllers/vehicle_controller.rb sustituir en el método index:

* @users = User.all

```bash
@pagy, @users = pagy(User.order(created_at: :desc), items: 5) # Paginación
```

## Agregar el control de pagy con bootstrap al final de la vista index donde se desea paginar. En app/views/vehicles/index.html.erb:

```bash
<div class=" d-flex justify-content-center align-items-center">
  <%== pagy_bootstrap_nav(@pagy) %>
</div>
```

## [[[[[[[[[[[[[[SERVICE]]]]]]]]]]]]]]

## Crear scaffold service:

```bash
rails g scaffold service service_name:string spare_parts:string deadline:date status:string vehicle_id:integer
```

## Agregar en el modelo vehicle:

```bash
class Vehicle < ApplicationRecord
  belongs_to :service
  has_and_belongs_to_many :services

  validates :brand, :model, :plate_number, presence: true
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

## Creo la tabla intermedia vehicle_services relación n:n

```bash
rails g migration CreateVehicleServices vehicle:references service:references
```

 NOTA:Un servicio tiene y pertenece a muchos vehículos (has_and_belongs_to_many :vehicles).

## Muestro en un collection los datos de los enum definidos en el modelo service.rb, sustituyo en app/views/services/_form.html.erb

```bash
<%= form_with(model: service) do |form| %>
  <% if service.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(service.errors.count, "error") %> prohibited this service from being saved:</h2>

      <ul>
        <% service.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :service_name, style: "display: block" %>
    <%= form.select :service_name, Service.service_names.keys, { prompt: 'Select a status' } %>
  </div>

  <div>
    <%= form.label :spare_parts, style: "display: block" %>
    <%= form.text_field :spare_parts %>
  </div>

  <div>
    <%= form.label :deadline, style: "display: block" %>
    <%= form.date_field :deadline %>
  </div>

  <div>
    <%= form.label :status, style: "display: block" %>
    <%= form.select :status, Service.statuses.keys, { prompt: 'Select a status' } %>
  </div>

  <div>
    <%= form.label :vehicle_id, style: "display: block" %>
    <%= form.number_field :vehicle_id %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

