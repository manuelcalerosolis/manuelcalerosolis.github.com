---
layout: post
title: OneToMany
---

Una de las cosas que más me fascina de Laravel es Eloquent, y una de las cosas que más me gusto dentro de Eloquent, fue la posibilidad de crear relaciones de una forma muy simple, usando el método [One To Many](https://laravel.com/docs/5.2/eloquent-relationships#one-to-many).

HasMany se usa para crear relaciones de uno-a-varios, usando tan solo una referencia desde uno de los modelos, al modelo con el que se relaciona [One-to-many (data model)
](https://en.wikipedia.org/wiki/One-to-many_(data_model))

En mi caso trato de relacionar una tabla de entidades (Entities) de una empresa (Clientes, Proveedores, Agentes, Personal), con una tabla de direcciones (Addressess), y una entidad puede tener múltiples direcciones.

Os muestro a continuación los modelos que he usado.

Modelo de entidades.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Entity extends Model
{
    use SoftDeletes;

    /**
     * @var string
     */
    protected $table = 'entities';

    /**
     * @var array
     */
    protected $fillable = ['company', 'first_name', 'last_name'];

    /**
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function addresses()
    {
        return $this->hasMany('App\Models\Address');
    }

}

```

Modelo de direcciones.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

/**
 * Class Address
 * @package App\Models
 */
class Address extends Model
{
    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'addresses';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['entity_id', 'name', 'first_name', 'last_name', 'address1', 'address2', 'postal_code', 'city', 'other', 'phone', 'phone_mobile'];

    /**
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function entities()
    {
        return $this->belongsTo('App\Models\Entity');
    }
}

```

La relación entre las tablas como podréis fácilmente observar se realizan a través de dos métodos

Desde el modelo entidad describimos la relación con el modelo Address mediante la llamada a hasMany, pasando como parámetro dicho modelo
```php
    public function addresses()
    {
        return $this->hasMany('App\Models\Address');
    }
```
Desde el modelo direcciones describimos la relación con el modelo enitdades llamando a  método belongsTo, pasando como parámetro Entity.
```php
    public function entities()
    {
        return $this->belongsTo('App\Models\Entity');
    }
```
Ya tenemos la relación establecida entre ambos modelos y para obtener la lista de direcciones que posee una entidad tan solo necesitamos llamar a la variable addresses, y cuidado con esto porque no se llama a método addresses(), sino a una variable que se instancia una ver creada esta relación, lo explica muy bien como siempre Jeffrey Way en este video [laracast]( https://laracasts.com/series/laravel-5-fundamentals/episodes/14), veamos por ejemplo como recorrerlo
```php
@foreach($entity->addresses as $address )
    <p>
        {!! Form::label('addresses', $address->name ) !!}
    </p>
@endforeach
```


Saludos.
