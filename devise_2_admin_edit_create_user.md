# Configurar la creación de usuarios por parte del administrador

## Pasos previos

+ Tener instalado devise en el proyecto
+ Tener instalado active_storage en el proyecto

## Generar el controlador admin con las vistas edit y create user

```bash
rails g controller admin create_user edit_user list_user
```

## Verificar que exista el enum role en el modelo user app>models>user.rb

```ruby
  # Enum de roles
  enum role: [:user, :admin] 
```

## Configurar las rutas para el controlador admin app>config>routes.rb

```ruby
  # Permisos del admin
  resources :admin, only: [:index, :new, :create, :update]
  # Ruta para crear usuarios
  post 'admin/create', to: 'admin#create', as: 'admin_create'
```

## Defino los métodos en el admin_controller app>controllers>admin_controller.rb

```ruby
class AdminController < ApplicationController
  before_action :set_user, only: [ :update]
  before_action :authenticate_user!
  before_action :authorize_admin!

  def list_user
    @users = User.all
  end

  def edit_user
    @users = User.order(:id)
  end
  
  def create
    @user = User.new(user_params)
    puts "Received parameters: #{params.inspect}"

    if @user.save
      redirect_to admin_create_user_path, notice: 'User was successfully created.'
    else
      #Muestro los mensajes personalizados de error definidos en las validaciones
      error_messages = @user.errors.full_messages.join(', ')
      redirect_to admin_create_user_path, alert: "User was not created. #{error_messages}"
    end
  end
  
  def new
    @user = User.new
  end
  
  def update
    if @user.update(user_params)
      redirect_to admin_edit_user_path, notice: 'User was successfully updated.'
    else
      redirect_to admin_edit_user_path, alert: 'User was not updated.'
    end
  end
  
  private

  def authorize_admin!
    redirect_to root_path, alert: 'Access Denied' unless current_user.admin?
  end

  # Método para pasar parámetros anidados bajo un hash con la clave :user en la solicitud por seguridad
  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation, :role, :first_name, :last_name)
  end
  
  def set_user
    @user = User.find(params[:id])
  end
  
end
```

## Diseño de la vista create_user con bootstrap app>views>admin>create_user.html.erb

```ruby
<%= form_with(model: @user, url: admin_create_path, method: :post) do |f| %>
  <%= f.fields_for :user do |user_fields| %>
    <div class="container">
      <div class="row justify-content-center mt-5">
        <div class="col-lg-4 col-md-6 col-sm-6">
          <div class="card shadow">
            <div class="card-title text-center border-bottom">
              <h2 class="p-3">Create new user</h2>
            </div>
            <div class="card-body">
              <form>

                <div class="mb-4">                  
                  <%= user_fields.label :first_name %><br />
                  <%= user_fields.text_field :first_name, autofocus: true, autocomplete: "first_name", class:'form-control'  %>
                </div>

                <div class="mb-4">                  
                  <%= user_fields.label :last_name %><br />
                  <%= user_fields.text_field :last_name, autocomplete: "last_name", class:'form-control'  %>
                </div>

                <div class="mb-4">                   
                  <%= user_fields.label :email %><br />
                  <%= user_fields.email_field :email, autocomplete: "email", class:'form-control'  %>
                </div>
                <div class="mb-4">     
                  <%= user_fields.label :role %><br />
                  <%= user_fields.select :role, User.roles.keys.map { |w| [w.humanize, w] }, include_blank: "Select a role", class:'form-select' %>
                </div>

                <div class="mb-4">
                  <%= user_fields.label :password %>
                  <% if @minimum_password_length %>
                    <em>(<%= @minimum_password_length %> characters minimum)</em>
                  <% end %><br />
                  <%= user_fields.password_field :password, autocomplete: "new-password", class:'form-control' %>
                </div>
        
                <div class="mb-4">
                  <%= user_fields.label :password_confirmation %><br />
                  <%= user_fields.password_field :password_confirmation, autocomplete: "new-password", class:'form-control' %>
                </div>

                <% end %>
                <div>
                  <%= f.submit "Create user", class:"btn btn-success" %>
                </div>
              <% end %>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
<br >
```

## Diseño de la vista edit_user con bootstrap app>views>admin>edit_user.html.erb

```ruby
<!--Listar todos los user y poder editar sus campos-->

<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-10 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">Edit user</h2>
          </div>
          <div class="card-body">
            <form>
              <% @users.each do |user| %>
                <%= form_for(user, url: admin_path(user), remote: true, method: :patch) do |f| %>
                  <p><%= f.text_field :email %> - <%= f.text_field :first_name %> - <%= f.text_field :last_name %> - <%= f.select(:role, User.roles.keys.map { |w| [w.humanize, w] }) %> - <%= f.submit "Update", class: "btn btn-success" %></p>
                <% end %>
              <% end %>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
<br>
```

## Vista list_user app/views/admin/list_user.html.erb

```ruby
<div class="container">
  <h1 class="mt-5 mb-4">Lista de Usuarios</h1>

  <table class="table table-striped">
    <thead>
      <tr>
        <th scope="col">Email</th>
        <th scope="col">Nombre</th>
        <th scope="col">Apellido</th>
        <th scope="col">Rol</th>
        <th scope="col">Actions</th>

      </tr>
    </thead>
    <tbody>
      <% @users.each do |user| %>
        <tr>
          <td><%= user.email %></td>
          <td><%= user.first_name %></td>
          <td><%= user.last_name %></td>
          <td><%= user.role %></td>
          <td><%= link_to "Edit", edit_user_path(user) %></td>
        </tr>
      <% end %>
    </tbody>
  </table>
</div>
```


## Definir rutas personalizadas en app/config/routes.rb

  get 'create_user', to: 'admin#user', as: 'create'
  
  get 'edit_user', to: 'admin#user', as: 'edit'
  
  get 'list_user', to: 'admin#user', as: 'show'

## Agregar al partial del navbar la opción de administrar usuers en app>views>shared>_navbar.html.erb

```ruby
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <%= link_to 'Logo', root_path, class: 'navbar-brand' %>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarSupportedContent">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <%= link_to "Home", root_path, class: 'nav-link' %>
        </li>
          <!--Si el user es admin mustra este menu-->
          <% if user_signed_in? && current_user.admin? %>
            <li class="nav-item">
                <%= link_to 'Edit profile', edit_user_registration_path, class: 'nav-link' %>
            </li>
            <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                    Users
                </a>
                <ul class="dropdown-menu">
                    <li><%= link_to 'Create user', admin_create_user_path, class: 'nav-link' %></li>
                    <li><%= link_to 'Edit Users', admin_edit_user_path, class: 'nav-link' %></li>
                    <li><%= link_to 'List all users', admin_list_user_path, class: 'nav-link' %></li>
                </ul>
            </li>
          <% end %>
          <!--Fin opciones del admin-->
      </ul>
      <!--identificación en el nabvar del user y tipo de role-->
      <ul class="navbar-nav ">
        <% if user_signed_in? %>
          <li class="nav-item">
            <%= content_tag :span, "Hi: #{current_user.first_name} #{current_user.last_name} | Role: #{current_user.role}", class: 'nav-link margen' %>
          </li>
      <!--Fin identificación user-->
          <!--Opciones para cualquier tipo de user-->
          <li class="nav-item">
            <%= button_to 'Cerrar sesión', destroy_user_session_path, class: 'btn btn-outline-success', method: :delete %>
          </li>
        <% else %>
          <li class="nav-item">
            <%= link_to 'Iniciar sesión', new_user_session_path, class: 'nav-link margen' %>
          </li>
          <li class="nav-item">
            <%= link_to 'Registro', new_user_registration_path, class: 'btn btn-outline-success' %>
          </li>
        <% end %>
        <!--Fin opciones cualquier tipo de user-->
      </ul>
    </div>
  </div>
</nav>
```

## Crear 10 users de prueba con faker app/db/seed.rb

```ruby
# Crea 10 usuarios con datos ficticios
10.times do
    User.create(
      email: Faker::Internet.email,
      password: Faker::Internet.password,
      first_name: Faker::Name.first_name,
      last_name: Faker::Name.last_name,
      role: "user"
    )
  end

  puts "Seed data generated successfully!"
```

## Hago commit

```ruby
git add .
git commit -m "Se creó el controlador y la vista admin view y los formularios para que el admin pueda crear, editar y listar users."
```
