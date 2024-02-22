# Laboratorio MongoDB

## Introducción

En este base de datos puedes encontrar un montón de alojamientos y sus reviews, esto está sacado de hacer webscrapping.

**Pregunta**. Si montaras un sitio real, ¿Qué posibles problemas pontenciales les ves a como está almacenada la información?

```md
Creo no es bueno tener toda la información en un sólo documento, habría que estudiar qué datos son más usados y menos y aplicar los patrones adecuados. 

Utilizaría el SUBSET PATTERN para extraer en un primer documento la información que iría en la página principal y workset y obtener una respuesta rápida, como ejemplo:

{
    "_id": "10006546",
    "name": "Ribeira Charming Duplex",
    "summary": "Fantastic duplex apartment with thre ...",
    "property_type": "House",
    "price": {
    "$numberDecimal": "80.00"
            },
    "resume_reviews"
}

En otro documento tendría el resto de información, a excepción de todas las reviews, que pueden ser muchas, estudiaría si es conveniente llevarlas a otro. En una primera búsqueda aparece que el *#Private Studio - Waikiki Dream* tiene *533* reviews, es el que más tiene seguido de *Near Airport private room* con *469* reviews, todo embebido en el mismo documento podría ser demasiada información y nos podríamos quedar sin espacio. Solo mostraría las primeras más importantes y el resto las separaría.

Habría que ver también qué cantidad de alojamientos podrían llegar a haber en un sólo documento, en este caso hay *5555*, pensaría en palicar también el BUCKET PATTERN, organizándolos o bien por fecha de registro o por popularidad. Aunque mejor utilizar la *address* ya podríamos ir también al TREE PATTERN y organizarlos por país por ejemplo.
```

### Consultas

- Saca en una consulta cuantos alojamientos hay en España.

```js
use('airbnb')
db.listingsAndReviews.find(
    {
        "address.country": "Spain"
    }
).count()
```

- Lista los 10 primeros:
  - Ordenados por precio de forma ascendente.
  - Sólo muestra: nombre, precio, camas y la localidad (`address.market`).

```js
use('airbnb')
db.listingsAndReviews.find(
    {
        "address.country": "Spain"
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        'address.market': 1 
    }
).sort({price: 1}).limit(10)
```
### Filtrando

- Queremos viajar cómodos, somos 4 personas y queremos:
  - 4 camas.
  - Dos cuartos de baño o más.
  - Sólo muestra: nombre, precio, camas y baños.

```js
use('airbnb')
db.listingsAndReviews.find(
    {
        beds: { $eq: 4 },
        bathrooms: { $gte: 2 }
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1
    }
)
```

- Aunque estamos de viaje no queremos estar desconectados, así que necesitamos que el alojamiento también tenga conexión wifi. A los requisitos anteriores, hay que añadir que el alojamiento tenga wifi.
- Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
use('airbnb')
db.listingsAndReviews.find(
    {
        amenities: "Wifi",
        beds: { $eq: 4 },
        bathrooms: { $gte: 2 }
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        amenities: 1
    }
)
```

- Y bueno, un amigo trae a su perro, así que tenemos que buscar alojamientos que permitan mascota (_Pets allowed_).
 - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
use('airbnb')
db.listingsAndReviews.find(
    {
        amenities: { $all: ["Wifi", "Pets allowed"] },
        beds: { $eq: 4 },
        bathrooms: { $gte: 2 }
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        amenities: 1
    }
)
```

- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen. Pero queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews (campo `review_scores.review_scores_rating` igual o superior a 88).
  - Sólo muestra: nombre, precio, camas, baños, rating y localidad.

```js
use('airbnb')
db.listingsAndReviews.find(
    {
        price: { $lte: 50 },
        "review_scores.review_scores_rating": { $gte: 88 },
        $or: [
            { "address.market": "Barcelona" },
            { "address.country": "Portugal" }
        ]
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        "review_scores.review_scores_rating": 1,
        "address.market": 1
    }
)
```

- También queremos que el huésped sea un superhost (`host.host_is_superhost`) y que no tengamos que pagar depósito de seguridad (`security_deposit`).
  - Sólo muestra: nombre, precio, camas, baños, rating, si el huésped es superhost, depósito de seguridad y localidad.

```js
use('airbnb')
db.listingsAndReviews.find(
    {
        price: { $lte: 50 },
        "review_scores.review_scores_rating": { $gte: 88 },
        "host.host_is_superhost": true,
        security_deposit: { $eq: 0 },
        $or: [
            { "address.market": "Barcelona" },
            { "address.country": "Portugal" }
        ]
    },
    {
        _id: 0,
        name: 1,
        price: 1,
        beds: 1,
        bathrooms: 1,
        "review_scores.review_scores_rating": 1,
        "address.market": 1,
        security_deposit: 1,
        "host.host_is_superhost": 1
    }
)
```

### Agregaciones

- Queremos mostrar los alojamientos que hay en España, con los siguientes campos:
  - Nombre.
  - Localidad (no queremos mostrar un objeto, sólo el string con la localidad).
  - Precio

```js
use('airbnb')
db.listingsAndReviews.aggregate([
    {
        $match: { "address.country": "Spain" }
    },
    {
        $project: {
            _id: 0,
            name: 1,
            market: {
                $concat: [
                    "$address.market"
                ]
            },
            price: 1
        }
    }
])
```

- Queremos saber cuantos alojamientos hay disponibles por pais.

```js
use('airbnb')
db.listingsAndReviews.aggregate([
    {
        $group: {
            _id: "$address.country",
            count: { $sum: 1 }
        }
    }
])
```

## Opcional

- Queremos saber el precio medio de alquiler de airbnb en España.

```js
use('airbnb')
db.listingsAndReviews.aggregate([
    {
        $match: { "address.country": "Spain" }
    },
    {
        $group: {
            _id: "$address.country",
            precio_medio: { $avg: "$price" }
        }
    }
])
```
- ¿Y si quisieramos hacer como el anterior, pero sacarlo por paises?

```js
use('airbnb')
db.listingsAndReviews.aggregate([
    {
        $group: {
            _id: "$address.country",
            count: { $sum: 1 },
            precio_medio: { $avg: "$price" }
        }
    }
])
```
- Repite los mismos pasos pero agrupando también por numero de habitaciones.

```js
use('airbnb')
db.listingsAndReviews.aggregate([
    {
        $group: {
            _id: {
                country: "$address.country",
                num_rooms: "$bedrooms"
            },
            count: { $sum: 1 },
            precio_medio: { $avg: "$price" }
        }
    },{
        $sort: {
            "_id.country": 1, "_id.num_rooms": 1
        }
    }
])
```

## Desafio

Queremos mostrar el top 5 de alojamientos más caros en España, con los siguentes campos:

- Nombre.
- Precio.
- Número de habitaciones
- Número de camas
- Número de baños
- Ciudad.
- Servicios, pero en vez de un array, un string con todos los servicios incluidos.

```js
use('airbnb')
db.listingsAndReviews.aggregate([
    {
        $match: {
            "address.country": "Spain"
        }
    },
    {
        $sort: {
            price: -1,
        }
    },
    {
        $limit: 5
    },
    {
        $project: {
            _id: 0,
            name: 1,
            price: 1,
            bedrooms: 1,
            beds: 1,
            bathrooms: 1,
            market: {
                $concat: [
                    "$address.market"
                ]
            },
            amenities: {
                $substr: [
                    {
                        $reduce: {
                            input: "$amenities",
                            initialValue: "",
                            in: { $concat: ["$$value", ", ", "$$this"] }
                        }
                    },
                    2,
                    { $strLenCP: { $reduce: { input: "$amenities", initialValue: "", in: { $concat: ["$$value", ", ", "$$this"] } } } }
                ]
            }
        }
    }
])
```