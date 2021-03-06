# Nombre de la red inalámbrica (SSID):) {#table_engines-url}

Administra datos en un servidor HTTP/HTTPS remoto. Este motor es similar
Angeles [File](file.md) motor.

## Uso del motor en el servidor ClickHouse {#using-the-engine-in-the-clickhouse-server}

El `format` debe ser uno que ClickHouse pueda usar en
`SELECT` consultas y, si es necesario, en `INSERTs`. Para obtener la lista completa de formatos admitidos, consulte
[Formato](../../interfaces/formats.md#formats).

El `URL` debe ajustarse a la estructura de un localizador uniforme de recursos. La dirección URL especificada debe apuntar a un servidor
que utiliza HTTP o HTTPS. Esto no requiere ningún
encabezados adicionales para obtener una respuesta del servidor.

`INSERT` y `SELECT` las consultas se transforman en `POST` y `GET` Peticiones,
Respectivamente. Para el procesamiento `POST` solicitudes, el servidor remoto debe admitir
[Codificación de transferencia fragmentada](https://en.wikipedia.org/wiki/Chunked_transfer_encoding).

Puede limitar el número máximo de saltos de redirección HTTP GET utilizando el [Nombre de la red inalámbrica (SSID):](../settings/settings.md#setting-max_http_get_redirects) configuración.

**Ejemplo:**

**1.** Crear un `url_engine_table` tabla en el servidor :

``` sql
CREATE TABLE url_engine_table (word String, value UInt64)
ENGINE=URL('http://127.0.0.1:12345/', CSV)
```

**2.** Cree un servidor HTTP básico utilizando las herramientas estándar de Python 3 y
comenzarlo:

``` python3
from http.server import BaseHTTPRequestHandler, HTTPServer

class CSVHTTPServer(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/csv')
        self.end_headers()

        self.wfile.write(bytes('Hello,1\nWorld,2\n', "utf-8"))

if __name__ == "__main__":
    server_address = ('127.0.0.1', 12345)
    HTTPServer(server_address, CSVHTTPServer).serve_forever()
```

``` bash
$ python3 server.py
```

**3.** Solicitar datos:

``` sql
SELECT * FROM url_engine_table
```

``` text
┌─word──┬─value─┐
│ Hello │     1 │
│ World │     2 │
└───────┴───────┘
```

## Detalles de la implementación {#details-of-implementation}

-   Las lecturas y escrituras pueden ser paralelas
-   No soportado:
    -   `ALTER` y `SELECT...SAMPLE` operación.
    -   Índices.
    -   Replicación.

[Artículo Original](https://clickhouse.tech/docs/es/operations/table_engines/url/) <!--hide-->
