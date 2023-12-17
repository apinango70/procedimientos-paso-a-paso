# Crear una relacion muchos a muchos n:n con sistema de tags en el modelo User

```bash
https://www.youtube.com/watch?v=03enr4NNgLI
```

## Creo elproyecto many_to_many

```bash
rails new many_to_many -d postgresql && cd many_to_many && rails db:create
```

## crear scaffold posts

```bash
rails g scaffold post title body:text
```

## crear scaffold tags

```bash
rails g scaffold tag name
```

## creo la tabla intermedia entre post y tag para establecer la relacion n:n

```bash
rails g model taggable post:references tag:references
```

## Ejecuto migracion

```bash
rails db:migrate
```

## Defino post como root en routes.rb

```bash
root "posts#index"
```

## Defino en el modelo post la relación n:n con tag a través de taggables

```bash
class Post < ApplicationRecord
  has_many :taggables, dependent: :destroy
  has_many :tags, through: :taggables
end
```

## Defino en el modelo tag la relación n:n con user a través de taggables

```bash
class Tag < ApplicationRecord
  has_many :taggables, dependent: :destroy
  has_many :posts, through: :taggables
end
```

## En el form de post defino el ingreso de los tags

```bash
  <div>
    <%= form.label :tag, style: "display: block" %>
    <%= form.text_field :tags, value: post.tags.map(&:name).join(",") %>
  </div>
```

## Agrego a los strong params en el controlador de post el campo tag

:tags

## En el metodo create del posts_controller sustituyo

```bash
  def create
    @post = Post.new(post_params.except(:tags))

    create_or_delete_posts_tags(@post, params[:post][:tags],)

    respond_to do |format|
      if @post.save
        format.html { redirect_to post_url(@post), notice: "Post was successfully created." }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end
```

## Agregamos la misma línea en el método update

```bash
  def update
    create_or_delete_posts_tags(@post, params[:post][:tags],)

    respond_to do |format|
      if @post.update(post_params.except(:tags))
        format.html { redirect_to post_url(@post), notice: "Post was successfully updated." }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end
```

## Creo en private el método _create_or_delete_posts_tags_

```bash
    def create_or_delete_posts_tags(post, tags)
      post.taggables.destroy_all
        tags = tags.strip.split(',')
        tags.each do |tag|
        post.tags << Tag.find_or_create_by(name: tag)
      end
    end
```

## En el partial de post agrego

```bash
<strong>Tags:</strong>
  <% post.tags.each do |tag| %>
  <%= link_to tag, style: "text-decoration:none" do %>
    <span class="tag"><%= tag.name.capitalize %></span>
    <%end%>
  <%end%>
```

## Aregar en la hoja de estilos

```bash
.tag {
  background: #0045c6;
  color: #fff;
  paddig: 0.5em;
  margin: 0.5em;
  border-radius: 0.25em;
  display: inline-block;
}
```

## Agregar en el partial de tag

```bash
  <% tag.taggables.each do |taggable| %>
    <h2>Post:</h2>
    <%= render 'posts/post', post: taggable.post %>
    <%= link_to "Read More", taggable.post %>
    <hr />
  <% end %>
```
