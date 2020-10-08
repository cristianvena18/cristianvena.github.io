---
layout: post
title: Programming Tricks
---

¿Cuántas veces tuvieron que mantener sistemas legacy o sistemas en cambios recurrentes? sistemas complejos que un día vienen y nos piden que agreguemos atributos o que quitemos en donde es un dolor de cabeza mantener interfaces limpias o claras?

Esto es una serie de tips que según mi experiencia me ayudaron a manejar mejor estos cambios recurrentes.

##Comandos o Commands

hace un tiempo leí Clean Code y me encontré con algo bastante interesante:

_“el número ideal de argumentos para una función es cero. Después uno (monádico) y dos (diádico). Siempre que sea posible evite la presencia de tres argumentos (triádico). Más de cuatro argumentos (polidiádico) requiere una justificación especial y no es muy habitual”._

normalmente cuando exponemos una API de tipo web esto se complica, son muchos los parámetros que necesitamos para, supongamos, crear un envío en nuestro sistema de logística, es complejo mantener una interfaz de nuestra capa de servicios o de aplicación cuando tenemos más de 2 argumentos y normalmente estos argumentos cambian constantemente por las reglas del negocio.

una forma fácil y sencilla de resolver esto (a mi criterio) es utilizar el patrón adaptador (adapter) con una estructura de datos simple creada por nosotros.

esto creo que es una de las principales ventajas con respecto a mantener interfaces limpias en sistemas con repentinos cambios, donde simplificamos todos los parámetros que deberíamos pasarle a un servicio a un solo parámetro, con un simple DTO simplifica la interfaz en un instante, (ojo, como todo en la programación ganamos en algo pero perdemos en otra, el cúmulo de parámetros lo seguimos teniendo, pero en un objeto de transferencia de datos es mucho más simple de mantener, ya que si las reglas cambian solamente debemos cambiar este objeto y no cambiar las interfaces de nuestra capa de aplicación)


Dado el siguiente código:

```
namespace Presentation\Http\Actions\Shipping;

class ShippingController {

    private StoreShippingAdapter $adapter;
    private ShippingService $service;

    public function __construct(
        StoreShippingAdapter $adapter,
        ShippingService $service
    )
    {
        $this->adapter = $adapter;
        $this->service = $service;
    }

    public function store(Request $request) {

        $this->shippingService->create(
            $request->input('products'),
            $request->input('address'),
            $request->input('address_number'),
            $request->input('addressee'),
            $request->input('price'),
            $request->input('taxes')
        );

        return new JsonResponse(
            [ 
                'message' => 'El envio se a guardado con éxito'
            ], 
            HttpCodes::OK
        );
    }
}
```

observe la cantidad de parámetros que recibe el método `create` de este servicio, es bastante molesto de mantener.

ahora imagine que soy el dueño del proyecto 
y necesito que se agreguen los atributos de
`address_flat`, `address_flat_number`, `time`, `date` y `company`, 
serían 5 parámetros más que necesitamos añadir a nuestra interfaz de 
`ShippingService`, ahora si recordamos lo que dice _Robert C. Martin_ en el libro de _Clean Code_ 
lo ideal sería que nuestra interfaz tenga solamente como máximo 2 parámetros.

¿Ahora, como podemos simplificar esta interfaz? si los datos los necesito delegar igualmente al servicio.

Bien, acá es donde entran las estructuras de datos, si, 
esas cosas que nos hablaban por allá en primer o segundo año de la facultad, 
esos dolores de cabeza con c++ o c# explicándonos que eran las estructuras de datos 
y como utilizarlas.

mire lo simple que esto queda con una estructura de datos simple.

```
class ShippingController {

    private StoreShippingAdapter $adapter;
    private ShippingService $service;

    public function __construct(
        StoreShippingAdapter $adapter,
        ShippingService $service
    )
    {
        $this->adapter = $adapter;
        $this->service = $service;
    }

    public function store(Request $request) {

        $command = $this->adapter->adapt($request);

        $this->service->create($command);

        return new JsonResponse(['message' => 'El envio se a guardado con éxito'], HttpCodes::OK);
    }
}
```

y nuestro adaptador queda así

```
class StoreShippingAdapter
{
    public function adapt(Request $request) {

        $this->validationService->validate(
            $request->all(),
            StoreShippingSchema::getStoreRules(),
            StoreShippingSchema::getMessages()
        );

        if ($this->validationService->isFail()) {
            throw new InvalidRequest($this->validationService->getErrors());
        }

        return new StoreShippingCommand(
            $request->input('products'),
            $request->input('address'),
            $request->input('address_number'),
            $request->input('addressee'),
        );
    }
}
```

Yo suelo agregar una validación de datos simple en donde verifico 
que el dato exista o que por ejemplo sea un número así evito problemas y 
ayudándome de un servicio de validaciones que tiene una interfaz clara y estable. 
donde no tengo que modificar estos datos jamás

##Una API simple y clara

Algo muy común es acoplarnos a una tecnología cuando trabajamos en algún sistema web
y siempre está la típica de "esto no creo que cambie jamás" pero adivinen qué? cambia 
y es un dolor de cabeza terrible cambiarlo.

me ha pasado que por ejemplo en aplicaciones de laravel en donde se hacen las vistas con blade se pasan los modelos directamente de la base de datos a la vista y terminamos teniendo esto:

```
    <td>{{ $product->getCode() }}</td>
    <td>{{ $product->getName() }}</td>
    <td>{{ $product->getPrice() }}</td>
```

y suponiendo que tenemos el requerimiento de cambiar el frontend a vue o 
tenemos que abrir la API para que nos consuman otros clientes externos, esto nos sería un problema ya que nuestros clientes no pueden consumir modelos de eloquent por ejemplo ya que estos no viajan por http

¿pero que podemos llegar a hacer para solventar esto?

algo que a mí me sirve mucho es el uso de presenters o presentadores. 

estos se encargan de iterar los datos a la forma que la API necesita exponer de la manera que necesite

esto quedaría algo así:

```
class ProductController {

    private IndexProductAdapter $adapter;
    private ProductService $service;
    private ProductPresenter $presenter;

    public function __construct(
        IndexProductAdapter $adapter,
        ProductService $service,
        ProductPresenter $presenter
    )
    {
        $this->adapter = $adapter;
        $this->service = $service;
        $this->presenter = $presenter;
    }

    public function index(Request $request) {

        $query = $this->adapter->adapt($request);

        $result = $this->service->find($query);

        return new JsonResponse([
            'data' => $this->presenter->fromResult($result)->getData()
        ], HttpCodes::OK);
    }
}
```

```
class ProductPresenter
{
    private $result;

    public function fromResult($result)
    {
        $this->result = $result;
        return $this;
    }

    public function getData()
    {
        $productList = [];

        foreach($this->result as $product)
        {
            array_push($productList, [
                'id' => $product->getId(),
                'name' => $product->getName(),
                'code' => $product->getCode(),
                'price' => $product->getPrice(),
            ]);
        }

        return $producList;
    }
}
```

haciendo este pequeño cambio en la vista podemos hacer algo tan simple como:

```
    <td>{{ $product->code }}</td>
    <td>{{ $product->name) }}</td>
    <td>{{ $product->price }}</td>
```

Así simplificamos la API y la mejoramos dejándola mucho más limpia y fácil de escalar.
