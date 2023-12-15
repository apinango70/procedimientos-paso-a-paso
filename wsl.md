# Instalacion WSL en windows

## Listar distribuciones instaladas

```bash
wsl -l
```

## borrar distribucion

```bash
wsl --unregister <distroName>
```

## Listar distribuciones disponibles en linea para instalar

```bash
wsl -l -o
```

## Instalar distribucion

```bash
wsl --install -d <Distribution Name>
```

## Agregar permisos de escritura sin ser root

```bash
sudo chown -R apinango:apinango /home/apinango/rails
```

