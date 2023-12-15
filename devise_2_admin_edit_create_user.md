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

## Configuración de rutas

En el archivo routes.rb se deben definir las rutas para la administración de los usuarios bajo un namespace admin, así podemos diferenciarlas de las rutas que usa devise por defecto. Para ello se debe agregar: 

```ruby
devise_for :users
namespace :admin do
  resources :users
end
```

## Definir las acciones del controlador

En el _admin::users_controller_ se deben implementar las acciones CRUD y definir los strong parameters.

```ruby
   class Admin::UsersController < ApplicationController
     before_action :authenticate_admin! # Asegura que solo los user con rol de admin puedan acceder al CRUD de los usuarios
     before_action :set_user, only: [:show, :edit, :update, :destroy] # Permite tener el usuario seleccionado disponible en @user
   
  def index
    @users = User.all
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
      redirect_to admin_users_path(@user), notice: 'User was successfully updated.'
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
      redirect_to admin_users_path, notice: 'User was successfully created..'
    else
      render :new
    end
  end
     
  private
 
  def authenticate_admin! # Método que verifica si el usuario que ingresó a admin/users/ tiene el rol de admin, de lo contrario muestra el alert y lo devuleve al root_path
    unless current_user && current_user.admin?
      redirect_to root_path, alert: 'Access Denied.!'
    end
  end

  def set_user
    @user = User.find(params[:id]) # Método que permite encontrar y asignar el usuario correcto según el ID proporcionado
  end

  def user_params # Strong Parameters permitidos
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

El formulario edit usa form_with para crear un formulario que estará vinculado al modelo @user. Este formulario muestra todos los campos asociados al modelo user

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

## Crear 10 users de prueba con faker app/db/seed.rb

```bash
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

```bash
git add .
git commit -m "style: agregar bootstrap a las vistas y form del admin CRUD"
```
