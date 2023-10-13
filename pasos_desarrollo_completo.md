#Pasos modelo 1:n y n:n

##Se debe instalar devise y active storage antes de seguir con estas instrucciones

##Paso 1: Crear scaffolds y modelos relacionales

```bash
rails g scaffold Viviendas superficie:integer direccion:string user:references
rails g scaffold TipoViviendas nombre:string vivienda:references
rails g scaffold Espacios nombre:string
rails g model ViviendasEspacios vivienda:references espacio:references
rails db:migrate
```

##Paso 2: Definir las asociaciones en los modelos

###User Model app/models/user.rb:

```bash
class User < ApplicationRecord
  #Relación 1:n con viviendas y borrado en cascada
  has_many :viviendas, dependent: :destroy
end
```

###app/model/vivienda.rb:

```bash
class Vivienda < ApplicationRecord
  belongs_to :user
  belongs_to :tipo_vivienda
  has_many :viviendas_espacios, dependent: :destroy
  has_many :espacios, through: :viviendas_espacios
end
```

###app/models/tipo_vivienda.rb:

```bash
class TipoVivienda < ApplicationRecord
  has_many :viviendas, dependent: :destroy
end
```

###app/model/espacio.rb:

```bash
class Espacio < ApplicationRecord
  has_many :viviendas_espacios, dependent: :destroy
  has_many :viviendas, through: :viviendas_espacios
end
```

###app/models/viviendas_espacios.rb:

```bash
class ViviendasEspacio < ApplicationRecord
  belongs_to :vivienda
  belongs_to :espacio
end
```

##Paso 3: Crear checkboxes de espacios y listbox de tipo_vivienda en la vista de Vivienda

###Agregar el el formulario en app/views/viviendas/_form.html.erb

```bash
  <div>
    <%= form.label :tipo_vivienda_id, style: "display: block" %>
    <%= form.collection_select :tipo_vivienda_id, TipoVivienda.all, :id, :nombre %>
  </div>

  <div>
    <%= form.label :espacios, style: "display: block" %>
    <% Espacio.all.each do |espacio| %>
      <label>
        <%= check_box_tag "vivienda[espacio_ids][]", espacio.id, @vivienda.espacios.include?(espacio) %>
        <%= espacio.nombre %>
      </label>
    <% end %>
  </div>
```

##Agregar strong parameters app/controllers/viviendas_controllers.rb

```bash
    def vivienda_params
      params.require(:vivienda).permit(:user_id, :superficie, :direccion, :tipo_vivienda_id, espacio_ids: [])
    end
```

##Paso 4:

##Mostrar en la vista viviendas los campos de tipo_vivienda y espacios app/views/viviendas/_vivienda.html.erb

```bash
  <p>
    <strong>Tipo de Vivienda:</strong>
    <%= vivienda.tipo_vivienda.nombre %>
  </p>

  <p>
    <strong>Espacios:</strong>
    <%= vivienda.espacios.map(&:nombre).join(', ') %>
  </p>
```

##Definir por defecto el user_id del user logeado cambiar en app/controllers/viviendas_controllers.rb

```bash
  def create
    @vivienda = current_user.viviendas.build(vivienda_params)
```

##Para crear una nueva vivienda obligar a logearse

```bash
  before_action :authenticate_user!, only: [:new, :create]
```

##El user solo puedo hacer CRUD con sus registros, el admin puede hacer CRUD con todos los registros

_app/controllers/viviendas_controllers.rb_

```bash
before_action :authorize_user!, only: [:edit, :update, :destroy]
```

```bash
    def authorize_user!
      # Solo permite que el administrador o el creador de la vivienda realicen acciones de edición y eliminación
      unless current_user.admin? || current_user == @vivienda.user
        redirect_to root_path, alert: "No tienes permisos para realizar esta acción."
      end
    end
```

##Muestra la opción new en la vista index si el user esta logeado

###app/views/viviendas/index.html.erb

```bash
 <div class="container">
  <h1>Viviendas</h1>
  <div id="viviendas">
    <% @viviendas.each do |vivienda| %>
      <%= render vivienda %>
      <p>
        <%= link_to "Show this vivienda", vivienda %>
      </p>
    <% end %>
  </div>
  <!--Muestra la opción crear solo si el user está logeado-->
  <% if user_signed_in? && current_user.user? || current_user.admin?  %>
    <%= link_to 'New vivienda', new_vivienda_path %>
  <% end %>
 </div>
 ```

##Muestra las opciones edit y destroy en la vista show si el user es el creador o es el admin

###app/views/viviendas/index.html.erb

```bash
<div class="container">
  <%= render @vivienda %>

    <%= link_to "Back to viviendas", viviendas_path %>

    <% if user_signed_in? && current_user.user? || current_user.admin?  %>
      <%= link_to "Edit this vivienda", edit_vivienda_path(@vivienda) %>
      <%= button_to "Destroy this vivienda", @vivienda, method: :delete %>
    <% end %>

</div>
```
