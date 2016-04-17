---
layout: post
title: OneToMany
---

Una de las cosas que más me fascina de Laravel es Eloquent, y una de las cosas que más me gusto dentro de Eloquent, fue la posibilidad de crear relaciones de una forma muy simple, usando el método [One To Many](https://laravel.com/docs/5.2/eloquent-relationships#one-to-many).

HasMany se usa para crear relaciones de uno-a-varios, usando tan solo una referencia desde uno de los modelos, al modelo con el que se relaciona [One-to-many (data model)
](https://en.wikipedia.org/wiki/One-to-many_(data_model))

En mi caso trato de relacionar una tabla de entidades (Entities) de una empresa (Clientes, Proveedores, Agentes, Personal), con una tabla de direcciones (Addresses), y una entidad puede tener múltiples direcciones.

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

Esto nos crea un conjunto de rutas necesarias para el mantenimiento de la tabla, veamos esas rutas.

```php
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
```

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

Mostrare aquí tan solo la de edición por ser la más compleja

\entity\edit.blade.php
```php
<!DOCTYPE html>
<html>

<body>

<div class="container">

    @include('partials.errors')

    {!! Form::open( ['route' => [ 'entity.update', $entity->id ], 'method' => 'put' ] ) !!}

    @include('entity.fields')

    <p>
        {!! link_to_route('entity.address.create', trans('forms.new_address'), [$entity]) !!}
    </p>

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
        {!! Form::submit(trans('forms.update')) !!}
    </p>

    {!! Form::close() !!}

</div>

</body>

</html>
```

Desglosare las partes que son objeto de este artículo, lo primero es mostrar el bucle @foreach donde se muestran todas las direcciones perteneceientes a una Entidad.

```php
@foreach($entity->addresses as $address)
    <p>
        {!! Form::label('addresses', $address->name ) !!}
        {!! link_to_route('entity.address.edit', trans('forms.update'), [$address->id, $entity]) !!}
        {!! link_to_route('entity.address.destroy', trans('forms.delete'), [$address->id, $entity]) !!}
    </p>
@endforeach
```

Fijaros como se utiliza la variable $entity->addresses y no el metodo $entity->addresses() para sacar la relación de direcciones, sé que estoy siendo muy pesado con este tema, quizás porque a mí me trajo más de un quebradero de cabeza.
Mostramos el nombre de la dirección $address->name con el comando, e incluimos dos enlaces para actualizar y borrar la dirección que están es este párrafo. 

¿Como creamos direcciones [Addresses] que se relacionen con nuestras entidades [Entities]?
Esta era una de mis principlaes dudas, entendiendo que para crear una nueva dirección necesitaba tener una referencia a la entidad a la que perteneceria.

La solución es crear una rutas donde llegen información de la entidad a la que va a pertenecer.
 
Para eso escribo unas rutas que recogen tanto el id de la entidad como la de la dirección, asi puedo establecer facilmente la relación.

Siguiendo el consejo de mi amigo Jaime (@jaimesares) me recomendo que creara direcciones de rutas subordinadas enteponiendo el texto 'entity' a 'address', asi claramente se puede observar la relación de pertenencia entre ambos modelos, observando tan solo la ruta.

```php
    Route::get('entity/address/{entity}',
        ['uses' => 'AddressController@create', 'as' => 'entity.address.create']);
    Route::post('entity/address/{entity}',
        ['uses' => 'AddressController@store', 'as' => 'entity.address.store']);
    Route::put('entity/address/{address}/{entity}',
        ['uses' => 'AddressController@update', 'as' => 'entity.address.update']);
    Route::get('entity/address/{address}/edit/{entity}',
        ['uses' => 'AddressController@edit', 'as' => 'entity.address.edit']);
    Route::get('entity/address/{address}/destroy/{entity}',
        ['uses' => 'AddressController@destroy', 'as' => 'entity.address.destroy']);
```

Para crear una nueva dirección utilizo este link.

```php
    {!! link_to_route('entity.address.create', trans('forms.new_address'), [$entity]) !!}
```

Como veis llamo a la direccion 'entity.address.create' pasandole $entity, asi sabre la entidad a la que pertenece la direccion que vamos a crear, 

```php
    public function create(Entity $entity)
    {
        return view('address.create', ['entity' => $entity]);
    }
```

Este llama a la vista 'address.create' con el parametro $entity

```php
<!DOCTYPE html>
<html>

<body>

    <div class="container">

        @include('partials.errors')

        {!! Form::open( ['route' => ['entity.address.store', $entity], 'method' => 'post'] ) !!}

        @include('address.fields')

        <p>
            {!! Form::submit(trans('forms.new')) !!}
        </p>

        {!! Form::close() !!}

    </div>

</body>

</html>
```

El formulario llama a 'entity.address.store' con $entity, y en controlador.

```php
    public function store(Create $request, Entity $entity)
    {
        $entity->addresses()->create( $request->all() );

        return redirect()->route('entity.edit', [$entity->id]);
    }
```

Este método recibe dos parámetros el $request de donde obtenemos todos los datos de la dirección, y un modelo de entidad, fijaros bien porque nuestro segundo parámetro es un modelo de tipo Entity, aunque nosotros lo que realmente hemos ido pasando entre nuestras vistas y controladores es tan solo el id de la entidad. 

¿Cómo se consigue esto?

Mediante algo novedoso en Laravel como es el [Route Model Binding](https://laravel.com/docs/5.2/routing#route-model-binding), Laravel se encarga de convertir nuestro id de la entidad en un modelo entidad, una vez tenemos esa entidad, tan solo hacemos una llamada al método create() de la relación que addresses() mantiene con la entidad, pasándole como parámetro el $request, quedaría así.

```php
    $entity->addresses()->create( $request->all() );
```

El resultado de esta operación sería la creación de una dirección que pertenece a nuestra entidad, en definitiva solo mantiene una referencia al id del modelo entidades en el campo 'entity_id'.
 

Continuara...

