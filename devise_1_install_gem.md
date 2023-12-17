# Instrucciones para instalar y configurar la gema devise

## Agregar las gemas devise, annotate y faker al proyecto.

```hash
bundle add devise annotate faker
```

## Instalar la gema Devise y seguir las instrucciones que salen al final.

```hash
rails g devise:install
```

### Usar estos alerts con bootstrap

```hash
    <% if notice %>
      <p class="alert alert-success"><%= notice %></p>
    <% end %>
    <% if alert %>
      <p class="alert alert-danger"><%= alert %></p>
    <% end %>
```

### Buscar en config>initializers>devise.rb

*descomentar en la linea 247 y cambiar a true*
config.scoped_views = true
   
*descomentar en la linea 266*
config.navigational_formats = ['*/*', :html, :turbo_stream]

## Generar el modelo para User

```hash
rails generate devise User first_name:string last_name:string role:integer
```

## Agrego el valor de default 0 al role en la migración

```hash
    t.integer :role, default: 0 # 0: user, 1: admin
```

## Migrar la base de datos y hacer commit

```hash
rails db:migrate
git add .
git commit -m "feat: crear modelo user"
```

## Crear vistas para poder personalizarlas

```hash
rails g devise:views
```

## Importamos los controladores de devise para personalizarlos

```hash
rails g devise:controllers users
```

## Modificar las rutas para evitar errores app>config>routes.rb

_reemplazar_

```hash
Rails.application.routes.draw do

  devise_for :users, controllers: {
    sessions: 'users/sessions',
    registrations: 'users/registrations'
  }
  # root "articles#index"

end
```

## Ejecutar la migración y acer commit:

```hash
rails db:migrate
git add .
git commit -m "feat: crear vistas y campos first_name, last_name y role modelo User"
```

## Modificar app/controllers/application_controllers.rb para agregar los strong parameters

```hash
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [
                                                        :first_name,
                                                        :last_name,
                                                        :role
                                                       ])

    devise_parameter_sanitizer.permit(:account_update, keys: [
                                                                :first_name,
                                                                :last_name,
                                                                :role
                                                              ])

  end

  def after_sign_in_path_for(resource)
    root_path  # Redirige al usuario a la ruta root
  end
end
```

## Agrego el enum de los tipos de usuarios a app/models/user.rb

```hash
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  # Enum de roles
  enum role: [:user, :admin]

end
```

## Hacer commit

```hash
git add .
git commit -m "feat: modificar registration, application controller y modelo user"
```

## Agregar el CDN de bootstrap al header del layout app/views/layout/application.html.erb.

### En el head

```hash
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
```

## Agregar navbar en un partial para devise y cargarlo en el layout

Se debe crear el partial en la ruta app/assets/shared/_navbar.html.erb y agregar el siguiente código de Bootstrap

```hash
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

## Renderizar el navbar en el layout app/views/lalyout.application.html.erb.

```hash
 <header>
    <%= render "shared/navbar" %>
  </header>
```

## Hacer commit

```hash
git add .
git commit -m "style: aregar CDN de bootstrap en layout y crear partial navbar"
```

## Para probar el navbar, crear un controlador con una vista.

```hash
rails g controller pages index
```

## Modificar app>config>routes.rb para configurar la vista root

```hash
root "pages#index"
```

```hash
git add .
git commit -m "feat: crear controlador pages y la accion index"
```

## Agregar a la vista "Sign in" un formulario bootstrap, reemplazar todo el código de: app/views/devise/session/new.html.erb por:

```hash
<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
<!-- Sign in Form -->
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-4 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">Log in</h2>
        </div>
        <div class="card-body">
          <form>
            <div class="mb-4">                   
                <%= f.label :email, class:'form-label' %><br />
                <%= f.email_field :email, autocomplete: "email", class:'form-control' %>
            </div>
            <div class="mb-4">
              <%= f.label :password, class:'form-label' %><br />
              <%= f.password_field :password, autocomplete: "current-password", class:'form-control' %>
            </div>
              <% if devise_mapping.rememberable? %>
                <div class="field">
                  <%= f.check_box :remember_me %>
                  <%= f.label :remember_me %>
                </div>
              <% end %>
                <div class="d-grid">
                  <%= f.submit "Log in", class:"btn btn-success" %>
                </div>
                <% end %>
              <%= render "devise/shared/links" %>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
```

## Agregar a la vista "Sign up" un formulario bootstrap, sustituir todo el código de: app/views/devise/registration/new.html.erb por:

```hash
<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= render "devise/shared/error_messages", resource: resource %>
<!-- Sign up Form -->
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-4 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">Sign up</h2>
        </div>
        <div class="card-body">
          <form>
            <div class="mb-4">                   
                <%= f.label :first_name, class:'form-label' %>
                <%= f.text_field :first_name, autofocus: true, autocomplete: "first_name", class:'form-control' %>
            </div>
            <div class="mb-4">                   
                <%= f.label :last_name, class:'form-label' %>
                <%= f.text_field :last_name, autofocus: true, autocomplete: "last_name", class:'form-control' %>
            </div>
              <% if User.where(role: User.roles[:admin]).exists? %>
                <!-- El administrador ya existe, ocultar el select de role y define user por defecto-->
                <%= f.hidden_field :role, value: User.roles.key(User.roles[:user]) %>
              <% else %>
                <!-- El administrador no existe, mostrar el select de roles -->
                <div class="mb-4">     
                  <%= f.label :role, class:'form-label' %>
                  <%= f.select :role, User.roles.keys %>
                </div>
              <% end %>
            <div class="mb-4">                   
                <%= f.label :email, class:'form-label' %><br />
                <%= f.email_field :email, autocomplete: "email", class:'form-control' %>
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
                  <%= f.submit "Sign up", class:"btn btn-success" %>
              <% end %>
            <%= render "devise/shared/links" %>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
```
## Agregar a la vista "User edit" un formulario bootstrap, sustituir todo el código de: app/views/devise/registrations/edit.html.erb por:

```hash
<%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
<%= render "devise/shared/error_messages", resource: resource %>

<!-- Edit Form with username added-->
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-4 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">Edit <%= resource_name.to_s.humanize %></h2>
        </div>
        <div class="card-body">
          <form>

            <div class="mb-4">                   
                <%= f.label :first_name, class:'form-label' %>
                <%= f.text_field :first_name, autofocus: true, autocomplete: "first_name", class:'form-control' %>
            </div>

            <div class="mb-4">                   
                <%= f.label :last_name, class:'form-label' %>
                <%= f.text_field :last_name, autofocus: true, autocomplete: "last_name", class:'form-control' %>
            </div>

            <div class="mb-4">
              <%= f.label :email %><br />
              <%= f.email_field :email, autofocus: true, autocomplete: "email", class:'form-control' %>
            </div>

            <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
              <div>Currently waiting confirmation for: <%= resource.unconfirmed_email %></div>
            <% end %>

            <div class="mb-4">
              <%= f.label :password %> <i>(leave blank if you don't want to change it)</i><br />
              <%= f.password_field :password, autocomplete: "new-password", class:'form-control' %>
              <% if @minimum_password_length %>
                <br />
                <em><%= @minimum_password_length %> characters minimum</em>
              <% end %>
            </div>

            <div class="mb-4">
              <%= f.label :password_confirmation %><br />
              <%= f.password_field :password_confirmation, autocomplete: "new-password", class:'form-control' %>
            </div>

            <div class="mb-4">
              <%= f.label :current_password %> <i>(we need your current password to confirm your changes)</i><br />
              <%= f.password_field :current_password, autocomplete: "current-password", class:'form-control' %>
            </div>

            <div class="mb-4">
              <%= f.submit "Update", class:"btn btn-success" %>
            </div>
            <% end %>
            <hr>
            <h3>Cancel my account</h3>

            <div>Unhappy? <%= button_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?", turbo_confirm: "Are you sure?" }, method: :delete, class:"btn btn-danger" %></div>
            <%= link_to "Back", :back %>.
          </form>
        </div>
      </div>
    </div>
  </div>
</div> 
```

> [!IMPORTANT]
> El enum que muestra los tipos de user para elegir el admin, solo aparecerá en el registro mientras no exista ningún admin, luego de asignar el role admin a un user, el enum desaparecerá de la vista new user..

## Agregar a la vista "Forgot password" un formulario bootstrap, sustituir todo el código de: app/views/devise/password/new.html.erb por:

```hash
<%= form_for(resource, as: resource_name, url: password_path(resource_name), html: { method: :post }) do |f| %>
  <%= render "devise/shared/error_messages", resource: resource %>
<!-- Forgot your password Form -->
<div class="container">
  <div class="row justify-content-center mt-5">
    <div class="col-lg-4 col-md-6 col-sm-6">
      <div class="card shadow">
        <div class="card-title text-center border-bottom">
          <h2 class="p-3">Forgot your password?</h2>
        </div>
        <div class="card-body">
          <form>
            <div class="mb-4">                   
              <%= f.label :email, class:'form-label' %><br />
              <%= f.email_field :email, autofocus: true, autocomplete: "email", class:'form-control' %>
            </div>
            <div class="d-grid">
              <%= f.submit "Send me reset password instructions", class:"btn btn-success" %>
              </div>
            <% end %>
            <%= render "devise/shared/links" %>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>
```

```hash
git add .
git commit -m "style: agregar bootstrap a las vistas sign_in sign_up, forgot_password y edit_user"
```

> [!NOTE]
> devise ya está instalado y configurado para utilizarse
