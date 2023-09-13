# Instrucciones para instalar y configurar la gema devise

## Agregar Devise al proyecto

```hash
bundle add devise
```

## Instalar la gema Devise y seguir las instrucciones que salen al final.

```hash
rails generate devise:install
```

### Buscar en config>initializers>devise.rb
_descomentar en la linea 266_
_config.navigational_formats = ['*/*', :html, :turbo_stream]_

## Generar el modelo para User

```hash
rails generate devise User
```

## Migrar la base de datos

```hash
rails db:migrate
```

```hash
git add .
git commit -m "Se creó el modelo User."
```

## Crear vistas para poder personalizarlas

```hash
rails generate devise:views
```

## Para que el navbar que está a continuación funcione hay que modificar el modelo User agregando el campo username y admin

```hash
rails g migration AddDetailsToUsers username:string role:string
```

## Importamos los controladores de devise para personalizarlos

```hash
rails generate devise:controllers users
```

## Modificar las rutas para evitar errores app>config>routes.rb

```hash
devise_for :users, controllers: {
  sessions: 'users/sessions',
  registrations: 'users/registrations'
}
```

## Ejecutar la migración:

```hash
rails db:migrate
```

## Hacer commit

```hash
git add .
git commit -m "Se crearon las vistas y los campos username y role del modelo User."
```

## Modificar app>controllers>users>registration_controllers.rb para agregar los strong parameters

### Descomentamos las lineas 1 y 2

  _before_action :configure_sign_up_params, only: [:create]_
  
  _before_action :configure_account_update_params, only: [:update]_

### Descomentamos y editamos métodos protegidos a patir de la línea 41 y agrago los nuevos campos

### Descomentar
  protected

### Copiar

```hash
  #If you have extra params to permit, append them to the sanitizer.
  def configure_sign_up_params
    devise_parameter_sanitizer.permit(:sign_up, keys: [:username, :role])
  end

  #If you have extra params to permit, append them to the sanitizer.
  def configure_account_update_params
    devise_parameter_sanitizer.permit(:account_update, keys: [:username, :role])
  end
```

## Modificar app>controllers>application_controllers.rb para agregar los strong parameters

```hash
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:username, :role])
    devise_parameter_sanitizer.permit(:account_update, keys: [:username, :role])
  end

  def after_sign_in_path_for(resource)
    root_path  # Redirige al usuario a la ruta root
  end
end
```

## Agrego el enum de los tipos de usuarios a user.rb
```hash
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  # Enum de roles
  enum role: { normal: 'normal',
                admin: 'admin',
  }, _default: 'normal'

end
```

## Hacer commit

```hash
git add .
git commit -m "Se modificaron registration y application controller y se agregó el enum de los roles."
```



## Agregar el CDN de bootstrap al header del layout

```hash
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
```

## Agregar navbar en un partial para devise y cargarlo en el layout

_Se debe crear el partial en la ruta app>assets>shared>_navbar.html.erb y agregar el siguiente código de Bootstrap_

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
        <li class="nav-item">
          <% if user_signed_in? && current_user.admin? %>
            <%= link_to '#', root_path, class: 'nav-link' %>  
          <% end %>
        </li>
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

## Agregar a la vista "Sign in" un formulario bootstrap

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
                <%= f.label :username, class:'form-label' %><br />
                <%= f.text_field :username, autofocus: true, autocomplete: "username", class:'form-control' %>
            </div>
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

## Agregar a la vista "Sign up" un formulario bootstrap

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
                <%= f.label :username, class:'form-label' %>
                <%= f.text_field :username, autofocus: true, autocomplete: "username", class:'form-control' %>
            </div>
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
NOTA: Si quiero agregar el listado de los roles para elegir en la vista new de sign_up, debo agregar:

```hash
<%# Defino el listado de los tipo de roles %>
<div class="mb-4">     
  <%= f.label :role, class:'form-label' %>
  <%= f.select :role, User.roles.keys %>
</div>
```

## Agregar a la vista "Forgot password" un formulario bootstrap

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

## Modifico app>models>user.rb para verificar si el user es admin:

```hash
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

 # Método para verificar si un usuario es administrador
 def admin?
   admin
 end
end
```

## NOTAS DE SEGURIDAD

### Si quiero obligar que en una vista se logee el usuario, debo colocar en el controller, en except se colocan las vistas permitidas sin logeo.

```hash
before_action :authenticated_user!, except: [:index, :show]
```

### Para asegurarme que solo el user que creo un registro lopueda modificar agregar en el método new y create

```hash
@xxxx = current_user.xxxx.xxxx
``` 

## Asegúrate de que en tu archivo routes.rb esté configurado para utilizar el controlador personalizado de Devise. Debería verse algo como esto:

```hash
devise_for :users, controllers: { registrations: 'registrations' }
```

## class RegistrationsController < Devise::RegistrationsController

```hash
  private
  def sign_up_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation)
  end

  def account_update_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation, :current_password)
  end
end
```

## Actualiza las rutas de Devise:

```hash
devise_for :users, controllers: { registrations: 'registrations' }
```

