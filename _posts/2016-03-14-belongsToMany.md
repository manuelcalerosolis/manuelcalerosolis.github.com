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

Desde el modelo entidad describimos la relación con el modelo Address mediante la llamada a hasMany, pasando como parámetro dicho modelo.

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

Ya tenemos la relación establecida entre ambos modelos y para obtener la lista de direcciones que posee una entidad tan solo necesitamos llamar a la variable addresses, y ¡cuidado con esto!, porque no se llama a método addresses(), sino a una variable que se instancia una vez creada esta relación, lo explica muy bien como siempre Jeffrey Way en este video [laracast]( https://laracasts.com/series/laravel-5-fundamentals/episodes/14), veamos por ejemplo como recorrerlo.

```php
@foreach($entity->addresses as $address )
    <p>
        {!! Form::label('addresses', $address->name ) !!}
    </p>
@endforeach
```
Ok, vale, todo esto está muy bien, y hasta aquí no aporto nada nuevo, hay muchos manuales que puedes consultar en internet sobre lo que os cuento.
Pero mi intención es realizar un blog eminentemente práctico y para eso,  queda como realizar un mantenimiento completo de las Entidades, y por tanto de las Direcciones, vamos a ello.

Vamos con las Etidades, lo primero es crear el conjunto de rutas para realizar el CRUD de esta tabla.

```php
    Route::resource('entity', 'EntityController');
```

Esto nos crea un conjunto de rutas necesarias para el mantenimiento de la tabla, veamos esas rutas

php artisan route:list
+--------+-----------+-------------------------------------------+------------------------+---------------------------------------------------------------+------------+
| Domain | Method    | URI                                       | Name                   | Action                                                        | Middleware |
+--------+-----------+-------------------------------------------+------------------------+---------------------------------------------------------------+------------+
|        | GET|HEAD  | entity                                    | entity.index           | App\Http\Controllers\EntityController@index                   | auth       |
|        | POST      | entity                                    | entity.store           | App\Http\Controllers\EntityController@store                   | auth       |
|        | GET|HEAD  | entity/create                             | entity.create          | App\Http\Controllers\EntityController@create                  | auth       |
|        | DELETE    | entity/{entity}                           | entity.destroy         | App\Http\Controllers\EntityController@destroy                 | auth       |
|        | PUT|PATCH | entity/{entity}                           | entity.update          | App\Http\Controllers\EntityController@update                  | auth       |
|        | GET|HEAD  | entity/{entity}                           | entity.show            | App\Http\Controllers\EntityController@show                    | auth       |
|        | GET|HEAD  | entity/{entity}/edit                      | entity.edit            | App\Http\Controllers\EntityController@edit                    | auth       |
+--------+-----------+-------------------------------------------+------------------------+---------------------------------------------------------------+------------+

Nescesitamos escribir el controlador EntityController 
 
```php
<?php

namespace App\Http\Controllers;

use App\Models\Address;
use App\Models\Entity;
use App\Http\Requests;
use App\Http\Requests\Entity\Create;
use App\Http\Requests\Entity\Update;
use Illuminate\Support\Facades\Redirect;

class EntityController extends Controller
{

    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        return view('entity.index');
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('entity.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Create $request)
    {
        Entity::create($request->all());

        return Redirect::to('entity');
    }

    /**
     * Display the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function show($id)
    {
        $entity = Entity::findOrFail($id);

        return view('entity.edit', ['entity' => $entity]);
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function edit($id)
    {
        $entity = Entity::findOrFail($id);

        return view('entity.edit', ['entity' => $entity]);
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Update $request, $id)
    {
        $entity = Entity::findOrFail($id);

        $entity->update($request->all());

        return Redirect::to('entity');
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function destroy($id)
    {
        Entity::destroy($id);

        return Redirect::to('entity');
    }
}
```

Nada raro un controlador de lo mas simple que necesita de una vista para solicitar al usuario los datos que contendra la tabla.
Yo uso tres vistas una para mostrar la releacion de Entidades [index.blade.php] otra para crearlas [create.blade.php] y otra para editarlas [edit.blade.php].

\entity\create.blade.php
```php
<!DOCTYPE html>
<html>

<body>

    <div class="container">

        @include('partials.errors')

        {!! Form::open( ['route' => 'entity.store', 'method' => 'post'] ) !!}

        @include('entity.fields')

        <p>
            {!! Form::submit(trans('forms.new')) !!}
        </p>

        {!! Form::close() !!}

    </div>

</body>

</html>
```

\entity\create.blade.php
```php
<!DOCTYPE html>
<html>

<body>

<div class="container">

    @include('partials.errors')

    {!! Form::open( ['route' => [ 'entity.update', $entity->id ], 'method' => 'put' ] ) !!}

    @include('entity.fields')

    <p>
        @foreach($entity->addresses as $address )
            <p>
                {!! Form::label('addresses', $address->name ) !!}
                {!! link_to_route('entity.address.edit', trans('forms.update'), [$address->id, $entity]) !!}
                {!! link_to_route('entity.address.destroy', trans('forms.delete'), [$address->id, $entity]) !!}
            </p>
        @endforeach
    </p>

    <p>
        {!! link_to_route('entity.address.create', trans('forms.new_address'), [$entity]) !!}
    </p>

    <p>
        {!! Form::submit(trans('forms.update')) !!}
    </p>

    {!! Form::close() !!}

</div>

</body>

</html>
```


Continuara...
