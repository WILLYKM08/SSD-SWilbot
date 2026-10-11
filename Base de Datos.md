# DEFINICION DE BASE DE DATOS

Construccion del Esquema de la base de datos

## Utilizar GraphQL Schema Definition Language
```[graphql]
type Usuario {
  id: ID!
  nombre: String!
  pedidos: [Pedido!]! # Relacion: Un usuario tiene muchos pedidos (1:N)
}

type Pedido {
  id: ID!
  total: Float!
  usuario: Usuario!   # Relacion: El pedido pertenece a un usuario
}
```
- **Entidades:** `Usuario` y `Pedido`.
- **Atributos:** `id`, `nombre`, `total`.
- **Relación (1:N):** Se deduce porque Usuario.pedidos devuelve una lista (`[Pedido!]!`) y `Pedido.usuario` devuelve un único objeto Usuario.

## Tipos de Datos Nativos en GraphQL

### Tipos Primitivos (Escalar)
- **`Int`**: Entero 32 bits con signo.
- **`Float`**: Valor de punto flotante de doble precisión con signo.
- **`String`**: Secuencia de caracteres UTF-8.
- **`Boolean`**: `true`or `false`.
- **`ID`**:Un identificador único (se serializa como un `String`, pero sirve para denotar llaves primarias o campos de búsqueda).

### Modificadores de Tipo
- **`No Nulo ( ! )`**: Indica que el campo es obligatorio. Ejemplo: `String!` (*no puede ser null*).
- **`Listas ( [] )`**: Define arreglos. Ejemplo: `[String]` (una lista de textos).

### Tipos Compuestos Comunes (Atributos semi estructurados)
- **`Enum`**: Tipos de enumeración para valores fijos.(Ya que luego no se podrán agregar ni quitar opciones cuando este implimentada la Base de Datos)
```[graphql]
enum Rol {
  ADMIN
  CLIENTE
  SOPORTE
}
```
- **`JSON Binario`**: Cuando el backend no necesita validar la estructura interna del objeto, solo guardarlo y entregarlo tal cual.
```[graphql]
scalar JSON

type Producto {
  id: ID!
  nombre: String!
  # Atributo semiestructurado: puede guardar cualquier objeto clave-valor
  especificaciones_tecnicas: JSON 
}
```
- **`Objeto`**:Cuando se requiere mantener el tipado fuerte y la auto-documentación de GraphQL, pero permitiendo que el objeto se llene parcialmente.

```[graphql]
type MetadatosVehiculo {
  color: String       # Opcional
  numero_puertas: Int # Opcional
  es_electrico: Boolean # Opcional
}

type Vehiculo {
  id: ID!
  placa: String!
  caracteristicas: MetadatosVehiculo # El contenedor existe, pero sus campos varían
}
```