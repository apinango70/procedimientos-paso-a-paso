# Configuración gema active_storage

## Instalaciones previas.

### En el termina ejecutar:

```hash
sudo apt update
sudo apt-get -y install libvips-dev
sudo apt update
sudo apt install ffmpeg
sudo apt update
sudo apt-get install libpoppler-dev
sudo apt update
sudo apt-get -y install mupdf
sudo apt update
```

## Proyecto de prueba:
```hash
rails new adjuntos -d postgresql
```

```hash
cd adjuntos
rails db:create
```

```hash
git add .
git commit -m "creacion inicial del proyecto"
```

```hash
rails g scaffold article title body:text
rails db:migrate
```

```hash
git add .
git commit -m "Scaffold creado"
```
## Habilitar la ruta principal en config>routes>routes.rb

	_root "articles#index"_

## Instalación gema active_storage:

```hash
rails active_storage:install
rails db:migrate
```

## Hago commit

```hash
git add .
git  commit -m "Active storage instalado."
```

## Descomentar en el Gemfile en la línea 49

```hash
gem "image_processing", ">= 1.2"
```

## Ejecutar bundle

```hash
bundle install
```

## Copiar en el modelo que estemos usando app>models>modelo.rb, modelo del ejemplo "article" app>model>article.rb y **para integrarlo al modelo user app>model>user.rb**.


class Article < ApplicationRecord

```hash
  # Defino una foto al usuario
  has_one_attached :photo
```

## Para agregarlo al modelo user: app>views>devise>registrations>new.html.erb

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
            <!--Selector de foto del perfil-->
            <div class="mb-4">
              <%= f.label :photo, style: "display: block" %>
              <%= f.file_field :photo %>
            </div>
            <div class="mb-4">                   
                <%= f.label :firstname, class:'form-label' %>
                <%= f.text_field :firstname, autofocus: true, autocomplete: "firstname", class:'form-control' %>
            </div>

            <div class="mb-4">                   
                <%= f.label :lastname, class:'form-label' %>
                <%= f.text_field :lastname, autofocus: true, autocomplete: "lastname", class:'form-control' %>
            </div>

              <% if User.where(role: User.roles[:admin]).exists? %>
                <!-- El administrador ya existe, ocultar el select de roles y asigna user por defecto. -->
                <%= f.hidden_field :role, value: User.roles[:user] %>
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

## Agregar al controller el strong parameter 

### Modelo User app>users>registration_controller.rb
```hash
  protected

  #If you have extra params to permit, append them to the sanitizer.
  def configure_sign_up_params
    devise_parameter_sanitizer.permit(:sign_up, keys: [:firstname, :lastname, :role, :photo])
  end

  #If you have extra params to permit, append them to the sanitizer.
  def configure_account_update_params
    devise_parameter_sanitizer.permit(:account_update, keys: [:firstname, :lastname, :role, :photo])
  end
```

### Mostrar foto en edit user, al modelo user app>views>devise>registrations>edit.html.erb

```hash
<%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
<%= render "devise/shared/error_messages", resource: resource %>

<!-- Edit Form with photo and username added-->
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
              <% if current_user.photo.attached? %>
                <%= f.label :photo %><br />
                <%= image_tag current_user.photo, style: "width: 150px", class: "foto_circular",class:'form-control' %>
              <% else %>
                <p> No Photo </p>
              <% end %>
            </div>
            <!--Permite cambiar la foto actual del perfil-->
            <div class="mb-4">
              <%= f.label :photo, style: "display: block" %>
              <%= f.file_field :photo %>
            </div>

            <div class="mb-4">                   
                <%= f.label :firstname, class:'form-label' %>
                <%= f.text_field :firstname, autofocus: true, autocomplete: "firstname", class:'form-control' %>
            </div>

            <div class="mb-4">                   
                <%= f.label :lastname, class:'form-label' %>
                <%= f.text_field :lastname, autofocus: true, autocomplete: "lastname", class:'form-control' %>
            </div>

            <div class="mb-4">
              <%= f.label :email %><br />
              <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
            </div>

            <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
              <div>Currently waiting confirmation for: <%= resource.unconfirmed_email,class:'form-control' %></div>
            <% end %>

            <div class="mb-4">
              <%= f.label :password %> <i>(leave blank if you don't want to change it)</i><br />
              <%= f.password_field :password, autocomplete: "new-password",class:'form-control' %>
              <% if @minimum_password_length %>
                <br />
                <em><%= @minimum_password_length %> characters minimum</em>
              <% end %>
            </div>

            <div class="mb-4">
              <%= f.label :password_confirmation %><br />
              <%= f.password_field :password_confirmation, autocomplete: "new-password",class:'form-control' %>
            </div>

            <div class="mb-4">
              <%= f.label :current_password %> <i>(we need your current password to confirm your changes)</i><br />
              <%= f.password_field :current_password, autocomplete: "current-password",class:'form-control' %>
            </div>

            <div class="actions">
              <%= f.submit "Update", class:"btn btn-success"  %>
            </div>
            <% end %>

            <h3>Cancel my account</h3>

            <div>Unhappy? <%= button_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?", turbo_confirm: "Are you sure?" }, method: :delete, class:"btn btn-danger"  %></div>
            <%= link_to "Back", :back %>.
          </form>
        </div>
      </div>
    </div>
  </div>
</div> 
```

## Agregar al css para mostrar la foto de forma circular:

```hash
 .foto_circular {
    border-radius: 50%;
  }
```

## Hacer commit

```hash
git add .
git commit -m "Se configuró active_storage y se modificaron las vistas del modelo user para poder agregar una foto al perfil."
```
