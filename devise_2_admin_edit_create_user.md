# Configurar la creación de usuarios por parte del administrador

## Pasos previos

+ Tener instalado devise en el proyecto
+ Tener instalado active_storage en el proyecto

## Generar el controlador admin con las vistas edit y create user

```bash
rails g controller admin create_user edit_user
```

## Verificar que exista el enum role en el modelo user app>models>user.rb

```ruby
  # Enum de roles
  enum role: {  user: 'user',
                moderator: 'moderator',
                creator: 'creator',
                admin: 'admin',
  }, _default: 'user' 
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

  def edit_user
    @users = User.all
    @user = User.new
  end

  def create_user
    @user = User.new(user_params)
    if @user.save
      redirect_to admin_index_path, notice: 'User was successfully created.'
    else
     #Muestro los mensajes personalizados de error definidos en las validaciones
      error_messages = @user.errors.full_messages.join(', ')
      redirect_to admin_index_path, alert: "User was not created. #{error_messages}"
    end
  end

  def new
    @user = User.new
  end

  def update
    if @user.update(user_params)
      redirect_to admin_index_path, notice: 'User was successfully updated.'
    else
      redirect_to admin_index_path, alert: 'User was not updated.'
    end
  end

  private

  def authorize_admin!
    redirect_to root_path, alert: 'Access Denied' unless current_user.admin?
  end

  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation, :role, :username)
  end
  
  def set_user
    @user = User.find(params[:id])
  end
end
```

## Diseño de la vista create_user con bootstrap app>views>admin>create_user.html.erb

```ruby
<!--Crear nuevo user, Debo crear un nuevo controlador solo para crear users, por ahora lo dejo aquí-->
<%= form_with(model: @user, url: admin_create_path, method: :post) do |f| %>
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
                <%= f.label :username, class:'form-label' %><br />
                <%= f.text_field :username, autofocus: true, autocomplete: "username", class:'form-control' %>
            </div>
            <div class="mb-4">                   
                <%= f.label :email, class:'form-label' %><br />
                <%= f.email_field :email, autocomplete: "email", class:'form-control' %>
            </div>
            <div class="mb-4">     
              <%= f.label :role, class:'form-label' %><br />
              <%= f.select :role, User.roles.keys.map { |w| [w.humanize, w] }, include_blank: "Select a role" %>
            </div>
              <div class="mb-4">
                <%= f.label :password, class:'form-label' %>
                <% if @minimum_password_length %>
                <em>(<%= @minimum_password_length %> characters minimum)</em>
                <% end %><br />
                <%= f.password_field :password, autocomplete: "new-password",class:'form-control' %>
            </div>             
            <div class="mb-4">
                <%= f.label :password_confirmation, class:'form-label' %><br />
                <%= f.password_field :password_confirmation, autocomplete: "new-password",class:'form-control' %>
            </div> 
            <div class="d-grid">
                  <%= f.submit "Create user", class:"btn btn-success" %>
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
    <div class="col-lg-8 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">Edit user</h2>
        </div>
        <div class="card-body">
          <form>
            <% @users.each do |user| %>
              <%= form_for(user, url: admin_path(user), remote: true, metohd: :patch) do |f| %>

                <p><%= f.text_field :email %> - <%= f.text_field :username %> - <%= f.select(:role, User.roles.keys.map { |w| [w.humanize, w] }) %> - <%= f.submit "Update", class:"btn btn-success" %></p>

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

## Agregar la opcíon de crear usuarios para el user admin en el partial app>views>shared>_navbar.html.erb

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
        <!--Si es admin muestra estas opciones en el navbar-->
        <% if user_signed_in? && current_user.admin? %>
          <li class="nav-item">
              <%= link_to 'Edit profile', edit_user_registration_path, class: 'nav-link' %>  
          </li>
          <!--Opción para ir a la vista edit users del admin-->
          <li class="nav-item">
              <%= link_to 'Edit Users', admin_edit_user_path, class: 'nav-link' %>  
          </li>
          <li class="nav-item">
              <%= link_to 'Create user', admin_create_user_path, class: 'nav-link' %>  
          </li>
        <% end %>
        <!--Fin condicional admin-->
      </ul>
      <ul class="navbar-nav ">
      <% if user_signed_in? %>
        <li class="nav-item">
          <%= content_tag :span, "Hi: #{current_user.username} | Role: #{current_user.role}", class: 'nav-link margen' %>
        </li>
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
      </ul>
    </div>
  </div>
</nav>
```

## Hago commit

```ruby
git add .
git commit -m "Se crrearon el controlador y la vista admin view y los formularios para que el admin pueda crear y editar users."
```
