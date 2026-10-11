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

### Escalares Nativos (Tipos Primitivos)
- **`Int`**: Entero 32 bits con signo.
- **`Float`**: Valor de punto flotante de doble precisión con signo.
- **`String`**: Secuencia de caracteres UTF-8.
- **`Boolean`**: `true`or `false`.
- **`ID`**:Un identificador único (se serializa como un `String`, pero sirve para denotar llaves primarias o campos de búsqueda).

### Modificadores de Tipo
- **`No Nulo ( ! )`**: Entero 32 bits con signo.
- **`Listas ( [] )`**: Entero 32 bits con signo.