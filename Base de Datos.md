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
## Entidades Transaccionales
Son aquellas que representan eventos, acciones o interacciones dinámicas que ocurren en un momento específico dentro de un sistema. A diferencia de las entidades maestras (que guardan información estática como un cliente, un producto o un empleado), las entidades transaccionales registran el intercambio de valor o la operación que ocurre entre ellas.

### Caracteristicas principales
- Son dinámicas: Crecen constantemente con cada nueva operación del negocio.
- Tienen una marca temporal: Cuentan con una fecha y hora exacta (timestamp) de cuándo ocurrió el evento.
- Relacionan datos maestros: Suelen contener claves foráneas (IDs) que apuntan a tablas de entidades maestras (por ejemplo, el `ID_Cliente` y el `ID_Producto)`.
- Garantizan integridad (ACID): Las operaciones asociadas a estas entidades se rigen por principios como atomicidad y consistencia (o se ejecutan por completo, o se revierten con un ROLLBACK).

### Diseño de Entidades Transaccionales
```[graphql]
# --- ENTIDADES MAESTRAS (Información de catálogo / estática) ---

"""
Representa a los clientes del sistema.
"""
type Cliente {
  id: ID!
  nombre: String!
  email: String!
  # Relación inversa: Un cliente puede tener muchas transacciones (compras)
  historialCompras: [Transaccion!]!
}

"""
Representa los productos disponibles.
"""
type Producto {
  id: ID!
  nombre: String!
  precio: Float!
  stock: Int!
}


# --- ENTIDAD TRANSACCIONAL (Información dinámica con marca temporal) ---

"""
Representa el evento de una venta u operación.
"""
type Transaccion {
  id: ID!
  fecha: String!       # Marca temporal del evento
  montoTotal: Float!
  
  # Relaciones con entidades maestras (los actores del evento)
  cliente: Cliente!    
  detalles: [DetalleTransaccion!]!
}

"""
Entidad de soporte transaccional para manejar el "muchos a muchos" entre venta y producto.
"""
type DetalleTransaccion {
  id: ID!
  producto: Producto!
  cantidad: Int!
  precioUnitario: Float!
}
```
### Operaciones con Entidades relacionales
```[graphql]
# Operaciones del Esquema (Queries y Mutations)

type Query {
  # Obtener una transacción específica y sus relaciones
  obtenerTransaccion(id: ID!): Transaccion
  
  # Obtener el catálogo de productos
  listarProductos: [Producto!]!
}

type Mutation {
  # Registra la acción/transacción en el sistema
  crearTransaccion(
    clienteId: ID!
    productos: [InputProductoDetalle!]!
  ): Transaccion!
}

# Input para enviar los datos de la compra
input InputProductoDetalle {
  productoId: ID!
  cantidad: Int!
}
```
## Buenas Prácticas para implementación Web
### El Escalar DateTime en la Web (Buenas Prácticas)
JavaScript en el navegador entiende este formato de forma nativa. Al recibir un string ISO 8601 desde tu API de GraphQL, el frontend puede transformarlo instantáneamente a la hora local del usuario con una sola línea de código: `new Date(fechaString)`.

### Diseñar pensando en la Paginación (Crucial para Web)
En desarrollo web, mostrar listas infinitas de datos rompe el rendimiento del navegador. Tu esquema debe estar preparado desde el diseño para paginar los datos. El estándar de la industria web para GraphQL es el patrón **Connections (Relay-style)**.

Desarrollo de un Ejemplo con los registros de unas ventas hechas por un vendedor
#### Diseño de esquema
Primero definimos los tipos siguiendo la especificación de Relay. Cada vendedor (Seller) tendrá un campo sales que devolverá una conexión de ventas (SaleConnection).
```[graphql]
type Sale {
  id: ID!
  amount: Float!
  product: String!
  createdAt: String!
}

# El Nodo envuelto en una Arista (Edge) con su cursor
type SaleEdge {
  cursor: String!
  node: Sale!
}

# Metadatos de la página actual
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# El contenedor principal de la conexión
type SaleConnection {
  edges: [SaleEdge!]!
  pageInfo: PageInfo!
  totalCount: Int! # Campo opcional muy útil
}

type Seller {
  id: ID!
  name: String!
  # El campo de ventas recibe los argumentos de paginación de Relay
  sales(first: Int, after: String): SaleConnection!
}

type Query {
  seller(id: ID!): Seller
}
```
#### Implementación del Resolver (Lógica en Node.js)
Para que el cursor sea estable y opaco, lo ideal es codificar en Base64 el ID del registro o su fecha de creación. En este ejemplo, convertiremos el ID de la venta.
```[javascript]
// Funciones utilitarias para codificar/decodificar cursores opacos
const toCursor = (id) => Buffer.from(`sale:${id}`).toString('base64');
const fromCursor = (cursor) => {
  if (!cursor) return null;
  const decoded = Buffer.from(cursor, 'base64').toString('utf-8');
  return decoded.split(':')[1]; // Retorna el ID numérico o string original
};

const resolvers = {
  Seller: {
    sales: async (seller, { first = 10, after }, context) => {
      // 1. Decodificar el cursor si el cliente lo envió
      const afterId = fromCursor(after);

      // 2. Consultar la base de datos (Ejemplo conceptual con ORM/SQL)
      // Pedimos 'first + 1' elementos para saber si hay una página siguiente sin hacer otro conteo
      const limit = first + 1; 
      
      const salesFromDb = await context.db.sale.findMany({
        where: {
          sellerId: seller.id,
          // Si hay cursor, buscamos solo los registros posteriores a ese ID
          ...(afterId && { id: { gt: afterId } }) 
        },
        orderBy: { id: 'asc' },
        take: limit
      });

      // 3. Determinar si hay una página siguiente
      const hasNextPage = salesFromDb.length > first;
      
      // Si había un elemento extra, lo removemos para devolver solo los solicitados
      const slicedSales = hasNextPage ? salesFromDb.slice(0, first) : salesFromDb;

      // 4. Transformar los registros en Edges (Aristas) con sus cursores
      const edges = slicedSales.map(sale => ({
        cursor: toCursor(sale.id),
        node: sale
      }));

      // 5. Construir el objeto PageInfo
      const startCursor = edges.length > 0 ? edges[0].cursor : null;
      const endCursor = edges.length > 0 ? edges[edges.length - 1].cursor : null;

      return {
        edges,
        pageInfo: {
          hasNextPage,
          hasPreviousPage: !!after, // Si usó "after", asumimos que hay páginas hacia atrás
          startCursor,
          endCursor
        }
      };
    }
  }
};
```
#### Consulta desde el Front-End
Si quieres obtener las primeras 2 ventas de un vendedor, la consulta se vería así
```[graphql]
query {
  seller(id: "seller_99") {
    name
    sales(first: 2) {
      edges {
        cursor
        node {
          product
          amount
        }
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
}
```
#### Respuesta del servidor
```[json]
{
  "data": {
    "seller": {
      "name": "Carlos Mendoza",
      "sales": {
        "edges": [
          { "cursor": "c2FsZToxMDE=", "node": { "product": "Laptop Pro", "amount": 1200.00 } },
          { "cursor": "c2FsZToxMDI=", "node": { "product": "Teclado Mecánico", "amount": 95.00 } }
        ],
        "pageInfo": {
          "hasNextPage": true,
          "endCursor": "c2FsZToxMDI="
        }
      }
    }
  }
}
```
Para pedir la siguiente página, el frontend solo tendría que repetir la query enviando el endCursor obtenido (c2FsZToxMDI=) en el argumento after