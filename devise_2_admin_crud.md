# Crear el MVC de devise para que el role admin pueda hacer CRUD con los users

> [!IMPORTANT]
> Para realizar este procedimiento, se debe tener instaldo y configurado:

> Devise gem https://github.com/apinango70/procedimientos-paso-a-paso/blob/main/devise_1_install_gem.md

> Active storage gem https://github.com/apinango70/procedimientos-paso-a-paso/blob/main/active_storage_gem_Install.md

> Pagy gem https://github.com/apinango70/procedimientos-paso-a-paso/blob/main/pagy_gem_install.md

## Generar el controlador para administrar a los usuarios

Se debe generar un controlador para administrar a los usuarios

```ruby
rails generate controller Admin::Users
```

## Hago commit

```bash
git add .
git commit -m "feat: crear controller admin/user"
```

## Configuración de rutas

En el archivo routes.rb se deben definir las rutas para la administración de los usuarios bajo un namespace admin, así podemos diferenciarlas de las rutas que usa devise por defecto. Para ello se debe agregar: 

```ruby
devise_for :users
namespace :admin do
  resources :users
end
```

## Definir las acciones del controlador

En el admin::users_controller se deben implementar las acciones CRUD y definir los strong parameters.

```ruby
class Admin::UsersController < ApplicationController
  # Autenticación de usuario administrador
  before_action :authenticate_admin!
  before_action :set_user, only: [:show, :edit, :update, :destroy]

  # Ejemplo de acciones CRUD
  def index
    #Orden de usuarios por rol, nombre y 5 usuarios por página
    @pagy, @users = pagy(User.order("role DESC, first_name"), items: 5)

  end

  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
  end

  def update
    @user = User.find(params[:id])
  
    if @user.update(user_params)
      redirect_to admin_user_path(@user), notice: 'User was successfully updated.'
    else
      redirect_to admin_users_path, alert: 'User was not updated.'
    end
  end

  def destroy
    @user = User.find(params[:id])
    @user.destroy
    redirect_to admin_users_path, notice: 'User was successfully deleted.'
  end

  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
  
    if @user.save
      redirect_to admin_user_path(@user), notice: 'User was successfully created.'
    else
      render :new
    end
  end

  private
 
  def authenticate_admin!
    unless current_user && current_user.admin?
      redirect_to root_path, alert: 'Access Denied.!'
    end
  end

  # Utilidad para encontrar y asignar el usuario correcto según el ID proporcionado
  def set_user
    @user = User.find(params[:id])
  end

  def user_params
    params.require(:user).permit(
                                  :first_name,
                                  :last_name,
                                  :email,
                                  :role,
                                  :photo,
                                  :password,
                                  :password_confirmation
                                  )
  end

end
```

## Hago commit

```bash
git add .
git commit -m "feat: Definir acciones del admin::users_controller"
```

## Vista index

En app/views/admin/users/, se debe crear el archivo index.html.erb para la acción index del controlador Admin::UsersController.

```bash
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-10 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h1 class="p-3">Users list</h1>
        </div>

        <table class="table">
          <thead>
            <tr>
              <th>ID</th>
              <th>First name</th>
              <th>Last name</th>
              <th>Email</th>
              <th>Role</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>

            <% @users.each do |user| %>
              <tr>
                <td><%= user.id %></td>
                <td><%= user.first_name %></td>
                <td><%= user.last_name %></td>
                <td><%= user.email %></td>
                <td><%= user.role %></td>
                <td>
                  <div class="user-actions">
                    <span class="action-link">
                      <%= link_to 'Show', admin_user_path(user), class: 'btn btn-info btn-sm' %>
                    </span>
                    <span class="action-link">
                      <%= link_to 'Edit', edit_admin_user_path(user), class: 'btn btn-primary btn-sm' %>
                    </span>
                    <span class="action-link">
                      <%= button_to 'Delete', admin_user_path(user), method: :delete, data: { confirm: 'Are you sure?' }, class: 'btn btn-danger btn-sm', form_class: 'd-inline' %>
                    </span>
                  </div>
                </td>
              </tr>
            <% end %>

          </tbody>
        </table>

      </div>
    </div>
  </div>
</div>
```

Esta vista generará una tabla que muestra todos los usuarios registrados en la aplicación web con las opciones para ver, editar y eliminar cada uno.

## Hago commit

```bash
git add .
git commit -m "feat: Crear vista index de admin::user"
```

## Vista show

```bash
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-4 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">User details</h2>
        </div>
        <div class="card-body">

          <p><strong>ID:</strong> <%= @user.id %></p>

          <p><strong>Photo:</strong></p>
          <% if @user.photo.attached? %>
            <%= image_tag @user.photo, style: "width: 150px", class: "foto_circular" %>
          <% else %>
            <p> No Photo added</p>
          <% end %>

          <p><strong>First name:</strong> <%= @user.first_name %></p>
          <p><strong>Last name:</strong> <%= @user.last_name %></p>
          <p><strong>Email:</strong> <%= @user.email %></p>
          <p><strong>Role:</strong> <%= @user.role %></p>

          <%= link_to 'Edit', edit_admin_user_path(@user), class: 'btn btn-primary' %>
          <%= link_to 'Return users list', admin_users_path, class: 'btn btn-secondary' %>
        </div>
      </div>
    </div>
  </div>
</div>
```

La vista show muestra los detalles específicos del usuario seleccionado en la vista index y mostrará un enlace que lo lleve a la vista edit o para poder regresar a la vista index

```bash
git add .
git commit -m "feat: Crear vista show de admin::user"
```

## Formulario edit

```bash
<%= form_with(model: [@user], url: admin_user_path(@user), local: true) do |form| %>
  <% if @user.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@user.errors.count, "error") %> han ocurrido:</h2>
      <ul>
        <% @user.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="container">
    <div class="row justify-content-center mt-5">
      <div class="col-lg-4 col-md-6 col-sm-6">
        <div class="card shadow">
          <div class="card-title text-center border-bottom">
            <h2 class="p-3">Edit user</h2>
          </div>
          <div class="card-body">
            <form>
              <% if @user.photo.attached? %>
                <%= form.label :photo %><br />
                <%= image_tag @user.photo, style: "width: 150px" %>
              <% else %>
                <p> No Photo added</p>
              <% end %>
              <!-- Permite cambiar la foto actual del perfil -->
              <div class="mb-4">
                <%= form.label :photo, style: "display: block" %>
                <%= form.file_field :photo, class:'form-control' %>
              </div>

              <div class="mb-4">
                <%= form.label :first_name %><br />
                <%= form.text_field :first_name, autofocus: true, autocomplete: "first_name", class:'form-control'  %>
              </div>

              <div class="mb-4">
                <%= form.label :last_name %><br />
                <%= form.text_field :last_name, autocomplete: "last_name", class:'form-control'  %>
              </div>

              <div class="mb-4">
                <%= form.label :email %><br />
                <%= form.email_field :email, autocomplete: "email", class:'form-control'  %>
              </div>

              <div class="mb-4">
                <%= form.label :role %><br />
                <%= form.select :role, User.roles.keys.map { |w| [w.humanize, w] }, include_blank: "Select a role", class:'form-select' %>
              </div>

              <div class="actions">
                <%= form.submit 'Save changes', class: 'btn btn-primary' %>
                <%= link_to 'Cancel', admin_user_path(@user), class: 'btn btn-secondary' %>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

El formulario edit usa form_with para crear un formulario que estará vinculado al modelo @user y muestra todos sus campos para que puedan ser modificados

```bash
git add .
git commit -m "feat: Crear form edit de admin::user"
```

## Formulario new

```bash
<%= form_with(model: @user, url: admin_users_path, local: true) do |form| %>

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
                  <%= form.label :first_name %><br />
                  <%= form.text_field :first_name, autofocus: true, autocomplete: "first_name", class:'form-control' %>
                </div>

                <div class="mb-4">
                  <%= form.label :last_name %><br />
                  <%= form.text_field :last_name, autocomplete: "last_name", class:'form-control'  %>
                </div>

                <div class="mb-4">
                  <%= form.label :email %><br />
                  <%= form.email_field :email, autocomplete: "email", class:'form-control'  %>
                </div>
                <div class="mb-4">
                  <%= form.label :role %><br />
                  <%= form.select :role, User.roles.keys.map { |w| [w.humanize, w] }, include_blank: "Select a role", class:'form-select' %>
                </div>

                 <div class="mb-4">
                  <%= form.label :password %>
                  <% if @minimum_password_length %>
                    <em>(<%= @minimum_password_length %> characters minimum)</em>
                  <% end %><br />
                  <%= form.password_field :password, autocomplete: "new-password", class:'form-control' %>
                </div>

                <div class="mb-4">
                  <%= form.label :password_confirmation %><br />
                  <%= form.password_field :password_confirmation, autocomplete: "new-password", class:'form-control' %>
                </div>

                <%= form.submit 'Create user', class: 'btn btn-primary' %>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

```bash
git add .
git commit -m "feat: Crear form new de admin::user"
```

## Agregar al partial del _navbar una opción para acceder a las vistas solo cuando el user logeado tiene role admin

```bash
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
                <%= link_to 'Edit my profile', edit_user_registration_path, class: 'nav-link' %>
            </li>

                    <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
            Users
          </a>
          <ul class="dropdown-menu">
            <li class="nav-item">
                <%= link_to 'Show Users', admin_users_path, class: 'nav-link' %>
            </li>
            <li class="nav-item">
                <%= link_to 'Create User', new_admin_user_path, class: 'nav-link' %>
            </li>

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

```bash
git add .
git commit -m "feat: Editar _navbar para agregar opciones de admin::user"
```

## Crear 10 users de prueba con foto app/db/seed.rb

> [!WARNING]
>  Se debe tener instalado y configurado active_storage para usar este seed.

```bash
# rails runner 'load(File.join(Rails.root, "db", "seeds", "rb", "users.rb"))'

require 'open-uri'

puts 'Creating 10 users with photos, please wait, this process may take a while...'


10.times do
  user = User.create(
    first_name: Faker::Name.first_name,
    last_name: Faker::Name.last_name,
    role: 0,
    email: Faker::Internet.email,
    password: '123456' # needs to be 6 digits
    # add any additional attributes you have on your model
  )

  # NOTA: debe tener instalado y configurado activestorage para usar esta opción

  file = URI.open('https://thispersondoesnotexist.com/')
  user.photo.attach(io: file, filename: 'photo.jpg', content_type: 'image/jpg')
end

puts '10 users successfully created!'
```

## Hago commit

```bash
git add .
git commit -m "feat: agregar seed para crear 10 users con foto"
```
