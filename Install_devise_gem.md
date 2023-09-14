# Instrucciones para instalar y configurar la gema devise

## Agregar las gemas devise y annotate al proyecto.

```hash
bundle add devise annotate
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

## Para que el navbar que está a continuación funcione hay que modificar el modelo User agregando el los campos username y role

```hash
rails g migration AddDetailsToUsers username:string role:string
```

## Importamos los controladores de devise para personalizarlos

```hash
rails generate devise:controllers users
```

## Modificar las rutas para evitar errores app>config>routes.rb

_Se debe reemplazar devise_for :users, por el texto:_

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

### Descomentamos y editamos métodos protegidos a patir de la línea 41 y agrego los nuevos campos:

### Descomentar
  protected

### Reemplazar el texto de la línea 43 a la 51

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

## Agrego el enum de los tipos de usuarios a app>models>user.rb
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



## Agregar el CDN de bootstrap al header del layout app>views>layout>application.html.erb.

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

## Renderizar el navbar en el layout app>views>lalyout.application.html.erb.

```hash
 <header>
    <%= render "shared/navbar" %>
  </header>
```

## Hacer commit

```hash
git add .
git commit -m "Se agregó el CDN de bootstrap y se creó el partial navbar."
```

## Para probar el navbar, crear un controlador con una vista.

```hash
rails g controller pages index
```

## Modificar app>config>routes.rb para configurar la vista root

```hash
root "pages#index"
```

## Agregar a la vista "Sign in" un formulario bootstrap, reemplazar todo el código de: app>views>devise>session>new.html.erb por:

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

## Agregar a la vista "Sign up" un formulario bootstrap, sustituir todo el código de: app>views>devise>registration>new.html.erb por:

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

OJO: este dropdown list solo lo debería ver el admin, NUNCA debe estar visible para los user normal, lo defino solo como práctica y para el ambiente de desarrollo.

```hash
<%# Defino el listado de los tipo de roles %>
<div class="mb-4">     
  <%= f.label :role, class:'form-label' %>
  <%= f.select :role, User.roles.keys %>
</div>
```

Cómo cambiar el rol del admin de normal a admin por cónsola.

Ejecutar:

```hash
rails c
```

## En la consola ejecutar:

```hash
user = User.find_by(email: 'nombreemail@gmail.com')
user.update(role: 'admin')
user.save
```

## Verificamos el cambio ejecutando:

```hash
Users.all
```

_salir de la cónsola_

## Agregar a la vista "Forgot password" un formulario bootstrap, sustituir todo el código de: app>views>devise>password>new.html.erb por:

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
git commit -m "Se agregó formato bootstrap a las vistas sign_in sign_up y forgot_password y se creó el controlador Pages con la vista index."
```

## -== devise ya está instalado y configurado para utilizarse ==-

## NOTAS DE SEGURIDAD

## Verificar en app>config>routes.rb que tenga definida la ruta para usar los controladores personalizados:

```hash
  devise_for :users, controllers: {
    sessions: 'users/sessions',
    registrations: 'users/registrations'
    }
```

## Agregar en app>controller>users>registration_controller.rb los strong parameters

_class RegistrationsController < Devise::RegistrationsController_

```hash
  private

  def sign_up_params
    params.require(:user).permit(:username, :email, :password, :password_confirmation)
  end

  def account_update_params
    params.require(:user).permit(:username, :email, :password, :password_confirmation, :current_password)
  end
```

### Si quiero obligar que en una vista se logee el usuario, debo colocar al inicio del controlador de donde quiero obligar el logeo el siguiente código: 

```hash
before_action :authenticated_user!, except: [:index, :show]
```

_en el arreglo de except se colocan las vistas permitidas sin logeo._

### Para asegurarme que solo el user que creo un registro lo pueda modificar agregar en el método new y create

```hash
@xxxx = current_user.xxxx.xxxx
``` 
