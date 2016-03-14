---
layout: post
title: BelongsToMany
---

Una de las cosas que más me fascina de laravel es eloquent, y una de las cosas que más me gusto dentro de eloquent, fue la posibilidad de crear relaciones many to many de una forma muy simple usando el metodo [belongsToMany]( https://laravel.com/docs/5.2/eloquent-relationships#many-to-many).

BelongsToMany se usa para crear relaciones de varios-a-varios usando una tabla [pívot](https://en.wikipedia.org/wiki/Pivot_table), las tablas pívots mantienen las referencias de las id´s de al menos dos tablas.
En mi caso trato de relacionar una tabla de entidades (Entities) de una empresa (Clientes, Proveedores, Agentes, Personal), con una tabla de direcciones (Addressess), y una entidad puede tener múltiples direcciones.


Saludos.
