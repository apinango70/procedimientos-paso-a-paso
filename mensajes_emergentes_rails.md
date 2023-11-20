# Agregar mensajes popup usando SeetAlert2 [https://sweetalert2.github.io/]

png captura de pantalla

https://miro.medium.com/v2/resize:fit:720/format:webp/1*0ibVtn0a9PZ_4soCyUCZ8g.png

## Agregar SweetAlert2 a import map ejecutando

```bash
bin/importmap pin sweetalert2
```

## Agregar el CDN de SweetAlarm2 a app/views/layouts/application.html.erb

```bash
<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
```

## sustituir los mensajes success y alert de devise por:

```bash
    <% if notice %>
      <script>
        function sweetNotice() {
          Swal.fire({
            icon: 'success',
            title: '<%= notice %>',
          });
        }
        // Muestra el mensaje de exito
        sweetNotice();
      </script>
    <% end %>

    <% if alert %>
        <script>
        function sweetAlert() {
          Swal.fire({
            icon: 'error',
            title: '<%= alert %>',
          });
        }
        // Muestra el mensaje de error
        sweetAlert();
      </script>
    <% end %>
```
