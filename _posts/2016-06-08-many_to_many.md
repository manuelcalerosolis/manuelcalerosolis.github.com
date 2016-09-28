---
layout: post
title: ManyToMany
date: 2016-01-01
---

Siguiendo con las relaciones el **laravel** os mostrare una solución práctica de [Many-to-many](https://laravel.com/docs/5.2/eloquent-relationships#many-to-many). 

Many-to-many se usa para crear relaciones de varios-a-varios, usando una tabla intermedia denominada generalmente tabla pívot, donde contienen las referencias a los identificadores de las tablas que se relacionan. Many-to-many (data model)
]( https://en.wikipedia.org/wiki/Many-to-many_(data_model))

En mi caso trato de relacionar una tabla de entidades (Entities) de una empresa (Clientes, Proveedores, Agentes, Personal), con una tabla de roles (Roles), en la que especifico el role que mantiene con nuestra empresa.

Os muestro a continuación los modelos.

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

    /**
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function roles()
    {
        return $this->belongsToMany('App\Models\Role')->withTimestamps();
    }

    /**
     * get list of roles id associate to an entity
     *
     * @return array
     */
    public function getRoleListAttribute()
    {
        return $this->roles->lists('id')->all();
    }

}
```

Modelo Roles

```php
<?php

namespace App\models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Role extends Model
{
    use SoftDeletes;

    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'roles';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];

    public function entities()
    {
        return $this->belongsToMany('App\Models\Entity');
    }
}
```

Modelo EntityRole
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

/**
 * Class Address
 * @package App\Models
 */
class EntityRole extends Model
{
    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'entity_role';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['entity_id', 'role_id'];

 }
```

La relación entre entidades y roles la determinamos con la función Roles() que usa la llamada belongsToMany, con el modelo como único parámetro.
También introduzco el método getRoleListArttribute() que posteriormente me servirá para obtener todos los roles de nuestra entidad.

```php
/**
 * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
 */
public function roles()
{
    return $this->belongsToMany('App\Models\Role')->withTimestamps();
}

/**
 * get list of roles id associate to an entity
 *
 * @return array
 */
public function getRoleListAttribute()
{i
    return $this->roles->lists('id')->all();
}
```


Desde el modelo roles  describimos la relación con el modelo entidades desde el método entities que contiene una llamada a belongsToMany, pasando como parámetro Entity.

```php
public function entities()
{
    return $this->belongsToMany('App\Models\Entity');
}
```

Ya tenemos la relación establecida entre ambos modelos, vamos a trabjar con ellos.

Vamos a crear nuestro controlador para Entity llamado EntityController, y como cosas a destacar, tengo que nombrar el uso de inyección de dependencias y desde el método __construct() le paso los parámetros de los modelos Entity $entity, Role $role, para utilizarlos ya en todo el controlador y no estar continuamente instanciándolos.

Volveremos mas tarde para detenernos en la creación de una nueva entidad pero antes os muestro la vista, contienen detalle importentes.

```php
<?php

namespace App\Http\Controllers;

use App\Models\Entity;
use App\Models\Role;
use App\Http\Requests;
use App\Http\Requests\Entity\Create;
use App\Http\Requests\Entity\Update;
use Illuminate\Support\Facades\Redirect;

class EntityController extends Controller
{

    protected $entity;
    protected $role;

    public function __construct(Entity $entity, Role $role)
    {
        $this->entity   = $entity;
        $this->role     = $role;
    }

    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $entities = $this->entity->all();

        return view('entity.index', ['entities' => $entities]);
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        $roles  = $this->role->lists('name', 'id');

        return view('entity.create', ['roles' => $roles]);
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Create $request)
    {
        $entity = $this->entity->create($request->all());

        $entity->roles()->attach( $request->input('role_list') );

        return Redirect::to('entity');
    }

    /**
     * Display the specified resource.
     *
     * @param  Entity  $entity
     * @return \Illuminate\Http\Response
     */
    public function show(Entity $entity)
    {
        return view('entity.edit', ['entity' => $entity]);
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  Entity  $entity
     * @return \Illuminate\Http\Response
     */
    public function edit(Entity $entity)
    {
        $roles  = $this->role->lists('name', 'id');

        return view('entity.edit', ['entity' => $entity, 'roles' => $roles]);
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  Entity  $entity
     * @return \Illuminate\Http\Response
     */
    public function update(Update $request, Entity $entity)
    {
        $entity->update($request->all());

        $entity->roles()->sync( $request->input('role_list') );

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
        $this->entity->destroy($id);

        return Redirect::to('entity');
    }
}
```

A continuación os muestro la vista para crear nuevas vistas

```php
<!DOCTYPE html>
<html>

<body>

    <div class="container">

        @include('partials.errors')

        {!! Form::open(['route' => 'entity.store', 'method' => 'post']) !!}

        @include('entity.fields')

        <p>
            {!! Form::submit(trans('forms.new')) !!}
        </p>

        {!! Form::close() !!}

    </div>

</body>

</html>
```

Yo mantengo en un fichero fields los campos porque son comunes a la creación y modificación de los registros.

```php
<p>
    {!! Form::label('company', trans('forms.company') ) !!}
    {!! Form::text('company', ( isset( $entity ) ? $entity->company : null ) ) !!}
</p>

<p>
    {!! Form::label('first_name', trans('forms.first_name') ) !!}
    {!! Form::text('first_name', ( isset( $entity ) ? $entity->first_name : null ) ) !!}
</p>

<p>
    {!! Form::label('last_name', trans('forms.last_name') ) !!}
    {!! Form::text('last_name', ( isset( $entity ) ? $entity->last_name : null ) ) !!}
</p>

<p>
    {!! Form::label('role_list', trans('forms.roles') ) !!}
    {!! Form::select('role_list[]', ( isset( $roles ) ? $roles : [''] ), ( isset($entity) ? $entity->getRoleListAttribute() : null ), ['multiple'] ) !!}
</p>
```
Parémonos en el control **select** que contiene un array role_list[] donde almacenaremos la selección que nuestro usuario ha hecho de los roles que desea que mantenga nuestra entidad, si observáis incluimos una llamada a getRoleLisAttribute(), para que el control select contenga la lista de Roles que tiene nuestra aplicación.
