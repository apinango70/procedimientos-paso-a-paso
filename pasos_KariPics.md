# Pasos para crear el proyecto KariPics

## Creo el proyecto con postgresql activo el servicio y creo la base de datos.

```hash
rails new KariPics-2.0 -d postgresql
sudo service postgresql start
rails db:create
```

## Primer commit.

```hash
git add .
git commit -m "Proyecto inicial y base de datos creada."
```

## Creo el controlador Pages y la vista home.

```hash
rails g controller Pages home
```

## Hago commit.

```hash
git add .
git commit -m "Se agregó el controlador Pages y la vista home."
```

## Instalo devise siguiendo la guía

VER INSTRUCCIONES

## Creo el Scaffold de Posts

```hash
rails generate scaffold Post title:string legend:text user:references
```

```hash
rails db:migrate
```

## Hago commit.

```hash
git add .
git commit -m "Scaffold Post creado."
```

## Creo el modelo y el controlador Comment.

```hash
rails g controller Comments content:text user:references post:references
```

## Agrego las relaciones 1:n de user con post y comment app>models>user.rb.

```hash
  has_many :posts
  has_many :comments
```

## Ejecuto la migración

```hash
rails db:migrate
```

## Hago commit.

```hash
git add .
git commit -m "Modelo comment creado y se definieron las relaciones 1:n en cada modelo."
```

## Cambio el incio root a posts

```hash
root "posts#index"
```

## Defino que solo el admin (Karina) pueda crear, editar o borrar posts.

```hash
# Llamo a check_admin para las acciones "new y  edit"
before_action :check_admin, only: [:new, :edit] 

    def check_admin
      if current_user.nil? || !current_user.admin?
        redirect_to root_url, alert: "Lo sentimos, no tienes permisos para realizar esta acción"
      end
    end
```

## sustituir el codigo de la acción create en app>controllers>posts_controller.rb para que el autor del post siempre sea el admin logeado.

Cambiar:

    @post = Post.new(post_params)

Por:

```hash   
    @post = current_user.posts.build(post_params)
```

### Elimino el campo user de la vista new

  <div>
    <%= form.label :user_id, style: "display: block" %>
    <%= form.text_field :user_id %>
  </div>

## Mostrar unsername en vez de user_id en los posts app>views>posts>_post.html.erb

Cambiar:

  <p>
    <strong>User:</strong>
    <%= post.user_id %>
  </p>

Por:

```hash 
<p>
  <strong>User:</strong>
  <%= post.user.username %>
</p>
```

## Ocultar las opciones Edit this post y destroy this post a los user normal sustituyendo en app>views>posts>show.html.erb

Cambiar:

<div>
  <%= link_to "Edit this post", edit_post_path(@post) %> |
  <%= link_to "Back to posts", posts_path %>

  <%= button_to "Destroy this post", @post, method: :delete %>
</div>

Por:

```hash 
<div>
  <% if current_user && current_user.admin? %>
    <%= link_to "Edit this post", edit_post_path(@post) %> |
    <%= button_to "Destroy this post", @post, method: :delete %>
  <% end %>
    <%= link_to "Back to posts", posts_path %>
</div>
```

## Ocultar el enlace new post del index al role normal app>vies>posts_index.html.erb

```hash 
<div>

  <% if current_user && current_user.admin? %>
    <%= link_to "New post", new_post_path %>
  <% end %>

</div>
```

## Agregar los comments a los posts


### Agregar en app>views>posts>show.html.erb

```hash
<div>
  <h2>Comentarios:</h2>
  <% @post.comments.each do |comment| %>
    <div>
      <p><strong>Usuario:</strong> <%= comment.user.username %></p>
      <p><%= comment.content %></p>
    </div>
  <% end %>
</div>

<div>
  <h2>Agregar Comentario:</h2>
  <%= form_with(model: [post, Comment.new]) do |form| %>
    <div>
      <%= form.label :content, style: "display: block" %>
      <%= form.text_area :content %>
    </div>

    <div>
      <%= form.submit "Agregar Comentario" %>
    </div>
  <% end %>
</div>
```

## Agregar en la acción show en app>controllers>posts_controller.rb el código:

```hash
@comment = Comment.new # Esto es para el formulario de comentario
```

## Agregar en app>controllers>comments_controller.rb el código:

```hash
class CommentsController < ApplicationController
  before_action :authenticate_user!, only: [:create] # Asegura que solo los usuarios autenticados puedan crear comentarios

  def create
    @post = Post.find(params[:post_id])
    @comment = @post.comments.build(comment_params)
    @comment.user = current_user

    if @comment.save
      redirect_to @post, notice: "Comentario agregado exitosamente."
    else
      # Manejo de errores si la validación del comentario falla
      render "posts/show"
    end
  end

  private

  def comment_params
    params.require(:comment).permit(:content)
  end
end

```

## Hago commit.

```hash
git add .
git commit -m "Se editaron las vistas show post para agregar el campo comment y los controladores necesarios."
```

## Instalo y configuro la gema active_storage para agregar imágenes al proyecto.








