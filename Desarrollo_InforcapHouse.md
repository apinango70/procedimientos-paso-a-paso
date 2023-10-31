# Pasos de reforzamiento InforcapHouse

## Creo el proyecto

```bash
rails new InforcapHouse -d postgresql && cd InforcapHouse && rails db:create
```

## Instalo las gemas que usaré en el proyecto

```bash
bundle add faker devise annotate pagy
```

## Ejecuto commit

```bash
git add .
git commit -m "Proyecto inicial creado y gemas instaladas"
```

## Creo las vistas estáticas

```bash
rails g controller pages home terms privacy
```

## Edito las rutas para hacerlas amigables app/config/routes.rb

```bash
  # Paginas estaticas
  get '/home', to: 'pages#home'
  get '/terms', to: 'pages#terms'
  get '/privacy', to: 'pages#privacy'

  root "pages#home"
```

## Hago commit

```bash
git add .
git commit -m "Páginas estáticas"
```

## Creo el formulario de contacto

```bash
rails g scaffold contact name email message:text
```

## Agrego validaciones al modelo contact con validates

```bash
class Contact < ApplicationRecord
  # Validaciones
  validates :name,    presence: true
  validates :email,   presence: true,
                      format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :message, presence: true
end
```

## Agrego validaciones a formulario _form.html.erb de contact agregando a cada campo "required: true"

```bash
<%= form_with(model: contact) do |form| %>
  <% if contact.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(contact.errors.count, "error") %> prohibited this contact from being saved:</h2>

      <ul>
        <% contact.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :name, style: "display: block" %>
    <%= form.text_field :name, required: true %>
  </div>

  <div>
    <%= form.label :email, style: "display: block" %>
    <%= form.text_field :email, required: true %>
  </div>

  <div>
    <%= form.label :message, style: "display: block" %>
    <%= form.text_area :message, required: true %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

## Creo un seed para crear contactos ficticios con faker, en app/db creo las carpeta /seeds/rb y dentro de rb creo el archivo contacts.rb

```bash
# rails runner 'load(File.join(Rails.root, "db", "seeds", "rb", "contacts.rb"))'
puts 'Importing contacts...'

20.times do
  Contact.create(
    name: Faker::Name.name_with_middle,
    email: Faker::Internet.email,
    message: Faker::Lorem.paragraph_by_chars(number: 500)
  )
end
```

## Ejecuto en la consola la dreación de contacts

```bash
rails runner 'load(File.join(Rails.root, "db", "seeds", "rb", "contacts.rb"))'
```

## Edito el contacts_controller para eliminar los métodos que no necesito (before_action, index, show, edit, update y destroy)

```bash
class ContactsController < ApplicationController

  # GET /contacts/new
  def new
    @contact = Contact.new
  end

  # POST /contacts or /contacts.json
  def create
    @contact = Contact.new(contact_params)

    respond_to do |format|
      if @contact.save
        format.html { redirect_to root_path, notice: "Contact was successfully created." }
        format.json { render :show, status: :created, location: @contact }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @contact.errors, status: :unprocessable_entity }
      end
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_contact
      @contact = Contact.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def contact_params
      params.require(:contact).permit(:name, :email, :message)
    end
end
```

## Elimino de las vistas de contact las que no necesito y solo dejaré

Eliminar los archivos:
- _form.html.erb
- new.html.erb

## Modifico las rutas del scaffold de contacts para que no de error

```bash
  # Contacts
  resources :contacts, only: %i[new create]
```

## Modifico la redirección luego de crear un mensaje en el contacts_controller del método create

```bash
  def create
    @contact = Contact.new(contact_params)

    respond_to do |format|
      if @contact.save
        #Luego de crear el mensaje, se redirecciona al enlace root_path
        format.html { redirect_to root_path, notice: "Contact was successfully created." }
        format.json { render :show, status: :created, location: @contact }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @contact.errors, status: :unprocessable_entity }
      end
    end
  end
```

## Creo un commit

```bash
git add .
git commit -m "Formulario de contacto listo"
```

## Configuro la gema devise para el control de acceso

- Ver pasos devise_1
- Agregar campo phone

## Creo un seed para crear users.rb

```bash
# rails runner 'load(File.join(Rails.root, "db", "seeds", "rb", "users.rb"))'
puts 'Importing users...'

20.times do
  User.create(
    email: Faker::Internet.email,
    password: '123456',
    password_confirmation: '123456',
    name: Faker::Name.name_with_middle,
    phone: Faker::PhoneNumber.cell_phone,
    role: rand(0..1)
  )
end
```

## Agrego un commit

```bash
git add .
git commit -m "Devise y seed users listo"
```

## Genero el modelo typeProperty

```bash
rails g model typeProperty name && rails db:migrate && annotate --models
```

## Defino las validaciones en el modelo typeProperty

```bash
class TypeProperty < ApplicationRecord
  # Validaciones
  validates :name, presence: true, uniqueness: true
end
```

##  Creo el seed para typeProperty para importar sus datos, debo crear una carpeta "csv" y dentro de ella el archivo en app/db/seeds/csv/typeProperty.csv

```bash
type_property_id,name
1,Apartamento
2,Casa
3,Finca
4,Habitacion
5,Cabaña
```

## Agrego en el seed el enlace al csv

```bash
require 'csv'

puts 'Importing typeProperties...'
CSV.foreach(Rails.root.join('db/seeds/csv/typeProperties.csv'), headers: true) do |row|
  TypeProperty.create! do |type_property|
    type_property.id = row[0]
    type_property.name = row[1]
  end
end
```

## Agrego un commit

```bash
git add .
git commit -m "Modelo typeProperty y seed creado" 
```

## Genero el modelo de typeOffer

```bash
rails g model typeOffer name && rails db:migrate && annotate --models
```

## Defino las validaciones en el modelo typeOffer

```bash
class TypeOffer < ApplicationRecord
  # Validaciones
  validates :name, presence: true, uniqueness: true
end
```

##  Creo el seed para typeOffer para importar un csv, en app/db/seeds/csv/typeOffer.csv

```bash
type_offer_id,name
1,Venta
2,Alquiler
```

## Agrego en el seed el enlace al csv, para ejecutar este seed, se debe comentar el bloque typePrpperty para evitar errores de duplicado

```bash
puts 'Importing typeOffers...'
CSV.foreach(Rails.root.join('db/seeds/csv/typeOffers.csv'), headers: true) do |row|
  TypeOffer.create! do |type_offer|
    type_offer.id = row[0]
    type_offer.name = row[1]
   end
end
```

## Agrego un commit

```bash
git add .
git commit -m "Modelo typeOffer y seed creado"
```

## Creo el modelo feature

```bash
rails g model feature name && rails db:migrate && annotate --models
```

## Defino las validaciones en el modelo feature

```bash
class Feature < ApplicationRecord
  # Validaciones
  validates :name, presence: true, uniqueness: true
end
```

##  Creo el seed para feature para importar un csv, en app/db/seeds/csv/reatures.csv

```bash
feature_id,name
1,Sala
2,Comedor
3,Cocina
4,Vestier
5,Closets
6,Despensa
7,Lavanderia
8,Oficina
9,Bodega
10,Atico
```

## Agrego en el seed el enlace al csv, para ejecutar este seed, se debe comentar el bloque typeProperty y typeOffer para evitar errores de duplicado

```bash
puts 'Importing features...'
CSV.foreach(Rails.root.join('db/seeds/csv/features.csv'), headers: true) do |row|
  Feature.create! do |feature|
    feature.id = row[0]
    feature.name = row[1]
  end
end
```

## Agrego un commit

```bash
git add .
git commit -m "Modelo feature y seed creado"
```

## Genero el scaffold para property relación n:n 

```bash
rails g scaffold property user:references type_offer:references type_property:references description:text price:decimal area:integer
```

## Ejecuto migración y agrego un commit

```bash
rails db:migrate
git add .
git commit -m "Scaffold property"
```

## Defino validaciones en el modelo property.rb

```bash
  # Validaciones
  validates :description, presence: true
  validates :area, presence: true
  validates :price, presence: true
```

## Defino validaciones en la vista _formhtml.erb de property

```bash
<%= form_with(model: property) do |form| %>
  <% if property.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(property.errors.count, "error") %> prohibited this property from being saved:</h2>

      <ul>
        <% property.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :user_id, style: "display: block" %>
    <%= form.collection_select :user_id, User.all, :id, :name %>
  </div>

  <div>
    <%= form.label :type_offer_id, style: "display: block" %>
    <%= form.collection_select :type_offer_id, TypeOffer.all, :id, :name %>
  </div>

  <div>
    <%= form.label :type_property_id, style: "display: block" %>
    <%= form.collection_select :type_property_id, TypeProperty.all, :id, :name %>
  </div>

  <div>
    <%= form.label :description, style: "display: block" %>
    <%= form.text_area :description, required: true %>
  </div>

  <div>
    <%= form.label :area, style: "display: block" %>
    <%= form.number_field :area, required: true %>
  </div>

  <div>
    <%= form.label :price, style: "display: block" %>
    <%= form.text_field :price, required: true %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

## Defino las relaciones de user con property 1:n en elmodelo user

```bash
#relaciones
  has_many :properties, dependent: :destroy
```

## creo un seed properties.rb para generar 100 properties 

```bash
# rails runner 'load(File.join(Rails.root, "db", "seeds", "rb", "properties.rb"))'
puts 'Importing properties...'

100.times do
  Property.create(
    user_id: User.all.sample.id,
    type_offer_id: TypeOffer.all.sample.id,
    type_property_id: rand(1..5),
    description: Faker::Lorem.paragraph(sentence_count: 2),
    area: rand(50..200),
    price: rand(100_000..1_000_000)
  )
end
```

## Creo la tabla intermedia entre property y feature n:n

```bash
rails g model propertyFeature property:references feature:references
```

## Ejecuto migración y agrego un commit

```bash
rails db:migrate
git add .
git commit -m "modelo propertyFeature creado"
```

## Agrego validacion en el modelo propertyFeature.rb

```bash
  validates :property, :feature, presence: true
```

## Agrego relaccion n:n en el modelo feature.rb

```bash
  #relaciones
  has_many :property_features, dependent: :destroy
  has_many :properties, through: :property_features 
```

## Agrego relaccion n:n en el modelo property.rb

```bash
  #relaciones
  has_many :property_features, dependent: :destroy
  has_many :features, through: :property_features 
```

## Hago commit

```bash
git add .
git commit -m "modelo relacion n:n property_feature"
```

## agrego feature en la vista _form de property

```bash
<%= form_with(model: property) do |form| %>
  <% if property.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(property.errors.count, "error") %> prohibited this property from being saved:</h2>

      <ul>
        <% property.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :user_id, style: "display: block" %>
    <%= form.collection_select :user_id, User.all, :id, :name %>
  </div>

  <div>
    <%= form.label :type_offer_id, style: "display: block" %>
    <%= form.collection_select :type_offer_id, TypeOffer.all, :id, :name %>
  </div>

  <div>
    <%= form.label :type_property_id, style: "display: block" %>
    <%= form.collection_select :type_property_id, TypeProperty.all, :id, :name %>
  </div>

  <div>
    <%= form.label :features, style: "display: block" %>
    <%= form.collection_check_boxes :feature_ids, Feature.all, :id, :name %>
  </div>

  <div>
    <%= form.label :description, style: "display: block" %>
    <%= form.text_area :description, required: true %>
  </div>

  <div>
    <%= form.label :area, style: "display: block" %>
    <%= form.number_field :area, required: true %>
  </div>

  <div>
    <%= form.label :price, style: "display: block" %>
    <%= form.text_field :price, required: true %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

## Agrego los strong parameters de feature en el properties_controller.rb

```bash
  def property_params
    params.require(:property).permit(:user_id, :type_offer_id, :type_property_id, :description, :area, :price, feature_ids: [])
  end
```

## Muestro las opciones seleccionadas en features en la vista de _property.html.erb 

```bash
<div id="<%= dom_id property %>">
<div class="card">
  <%# <img src="..." class="card-img-top" alt="..."> %>
  <div class="card-body">
    <h5 class="card-title">
      <%= property.type_property.name %> en <%= property.type_offer.name %>
    </h5>
    <p class="card-text"><%= property.description %></p>

    <p>
      <strong>Contacto:</strong>
      <%= property.user.name %><br>
      <a href="mailto:<%= property.user.email %>"><%= property.user.email %></a><br>
      <a href="tel:+<%= property.user.phone %>"><%= property.user.phone %></a>
    </p>

    <p>
      <strong>Features:</strong>
      <% property.features.each do |item| %>
        <%= content_tag :span, item.name, class: "tag"  %>
      <% end %>
    </p>

    <p>
      <strong>Area:</strong>
      <%= property.area %>
    </p>

    <p>
      <strong>Price:</strong>
      <%= property.price %>
    </p>
  </div>
</div>
</div>
```

## Hago commit

```bash
git add .
git commit -m "campo features agregado al form y a la vista de property"
```

## Creo el seed para poblar property_features.rb

```bash
# rails runner 'load(File.join(Rails.root, "db", "seeds", "rb", "propertyFeatures.rb"))'
puts 'Importing propertyFeatures...'

1000.times do
  PropertyFeature.create(
    property_id: Property.all.sample.id,
    feature_id: Feature.all.sample.id,
  )
end
```

## Hago commit

```bash
git add .
git commit -m "Seed property_feaures creado"
```

## Defino que solo los user logeados pueden crear, editar y borrar registros, coloco en el properties_controller.rb

```bash
before_action :authenticate_user!, except: %i[ index show ]
```
