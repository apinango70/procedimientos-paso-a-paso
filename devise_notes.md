# Instrucciones para instalar y configurar la gema devise

## Paso 1: Agregar Devise al proyecto

```hash
bundle add devise
```

## Paso 2: Instalar la gema Devise

```hash
rails generate devise:install
```

## Paso 3: Generar el modelo para User

```hash
rails generate devise User
```

## Paso 4: Migrar la base de datos

```hash
rails db:migrate
```

## Paso 5: Crear vistas para poder personalizarlas (opcional)

```hash
rails generate devise:views
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
          Hi! <%= content_tag :span, current_user.email, class: 'margen' %>
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

## Para que este navbar funcione hay que modificar el modelo User agregando el campo name y admin

```hash
rails g migration AddAdminToUsers admin:boolean
rails g migration AddNameToUsers username:string
```

## Ejecutar la migración:

```hash
rails db:migrate
```
## Agregar al 



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
                <%= f.label :email, class:'form-label' %><br />
                <%= f.email_field :email, autofocus: true, autocomplete: "email", class:'form-control' %>
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
                <%= f.label :email, class:'form-label' %><br />
                <%= f.email_field :email, autofocus: true, autocomplete: "email", class:'form-control' %>
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

## Modelo estilo neumorfismo

```hash
<div class="wrapper">
        <div class="logo">
            <img src="https://www.freepnglogos.com/uploads/twitter-logo-png/twitter-bird-symbols-png-logo-0.png" alt="">
        </div>
        <div class="text-center mt-4 name">
            Twitter
        </div>
        <form class="p-3 mt-3">
            <div class="form-field d-flex align-items-center">
                <span class="far fa-user"></span>
                <input type="text" name="userName" id="userName" placeholder="Username">
            </div>
            <div class="form-field d-flex align-items-center">
                <span class="fas fa-key"></span>
                <input type="password" name="password" id="pwd" placeholder="Password">
            </div>
            <button class="btn mt-3">Login</button>
        </form>
        <div class="text-center fs-6">
            <a href="#">Forget password?</a> or <a href="#">Sign up</a>
        </div>
    </div>
```





## Actualiza las vistas y controladores de Devise (opcional):


## crear el controlador de registration

```hash
rails generate controller Registrations
```

## Editar las vistas de registro y edición (registrations/new.html.erb y registrations/edit.html.erb) para agregar un campo de entrada para el nombre.

## En el controlador registrations_controller.rb, asegúrate de que el controlador esté configurado para permitir los parámetros name durante la creación y actualización de usuarios.

## En este archivo, hereda de Devise::RegistrationsController y personaliza los métodos que desees modificar. Por ejemplo, puedes agregar o modificar los métodos new, create, edit, update, destroy, etc., según tus necesidades.

## Aquí tienes un ejemplo de cómo podría lucir un registrations_controller.rb personalizado:

class RegistrationsController < Devise::RegistrationsController
  
  private
  # Define métodos personalizados si es necesario

end

## Asegúrate de que en tu archivo routes.rb esté configurado para utilizar el controlador personalizado de Devise. Debería verse algo como esto:

```hash
devise_for :users, controllers: { registrations: 'registrations' }
```

class RegistrationsController < Devise::RegistrationsController

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

## Finalmente, actualiza las vistas de Devise para mostrar el campo name donde sea necesario.

 






















