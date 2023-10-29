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
rails g scaffold contac name email message:text
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

## Agrego validaciones a la vista contact agregando a cada campo  required: true

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

## Modifico la redirección luego de crear un mensaje en el contacs_controller del método create

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

## 

```bash

```
