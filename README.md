## PHP Data Object (PDO) <img src="https://github.com/user-attachments/assets/6754d142-a1b1-4313-bc94-6e7ea990c0e0" width="14%" height="14%" align="right" valign="center" alt="PHP Data Object"/> 

![PHP](https://img.shields.io/badge/PHP-%5E8.2-blue)
![License](https://img.shields.io/badge/Code%20GNU-License-blue.svg)
![learning](https://img.shields.io/badge/PDO-learning-blue.svg)

Guia básico de utilização da class PDO do PHP.

> _[PHP Data Objects](https://www.php.net/manual/pt_BR/intro.pdo.php). Trata-se de uma extensão do PHP para prover acesso a diferentes modelos de bancos de dados através de uma interface única de classes e métodos._

### Conexão com o banco de dados


#### Método 1: Embutido
```php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=database;charset=utf8mb4', 'username', 'password');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

#### Método 2: Função
```php
function connect(): PDO
{
  try {

    # MariaDB
    $host    = '127.0.0.1';
    $dbname  = 'database';
    $user    = 'username';
    $pass    = 'password';
    $options = [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
    ];

    return new PDO("mysql:host=$host;dbname=$dbname;charset=utf8mb4", $user, $pass);
   
    # SQLite Database
    # return new PDO("sqlite:database.db");

  } catch(PDOException $e) {
      echo "Erro na conexão com banco de dados: " . $e->getMessage();
  }
}
```
[_PDO Options_](https://www.php.net/manual/en/pdo.setattribute.php)
`ATTR_ERRMODE` Lança exceções em erros.


### Executando estruções SQL
Los pasos para ejecutar una sentencia SQL son:

 1. **Establecer** conexión con la base de datos (ver punto anterior)
 2. **Preparar** la sentencia
 3. **Ejecutar** sentencia (asociando parámetros si fuese necesario)
 4. Opcional: tratar el resultado de la sentencia ejecutada.

### Preparar la sentencia
Podemos preparar la sentencia a ejecutar de forma sencilla 
```php
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos");
```
En caso de necesitar incluir parámetros en la sentencia, podemos realizarlo utilizando la sintaxis `:nombre`

```php
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE edad > :edad");
```

De la misma forma que hemos preparado una consulta de tipo SELECT, también podemos preparar un INSERT, UPDATE, etc.

```php
$stmt= $dbh->prepare("INSERT INTO alumnos(nombre, apellidos) values (:nombre, :apellidos)");
```

### Ejecutar la sentencia
Para ejecutar una sentencia simple que no requiera de parámetros, podemos invocar directamente el método `execute()`. 
```php
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos");
$stmt->execute();
```
En caso de necesitar incluir parámetros en la sentencia, pasaremos al método `execute()` la información en un array asociativo:

```php
$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE nombre = :nombre AND edad = :edad");
$stmt->execute($data);
```
De la misma forma que hemos preparado una consulta de tipo SELECT, también podemos preparar un INSERT, UPDATE, etc.

```php
$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
$stmt = $dbh->prepare("INSERT INTO alumnos(nombre, edad) values (:nombre, :edad)");
$stmt->execute($data);
```

## Tratar los resultados de una consulta SELECT

Una vez ejecutada la sentencia `execute()` podremos acceder a los resultados obtenidos de la base de datos. PDO nos ofrece la posiblidad de recibir los resultados en distintos formatos: objetos de una clase, arrays asociativos, etc. Para indicarle cómo queremos recoger los resultados utilizaremos el método `setFetchMode(String mode)`. 

Estos son los 3 valores más utilizados y que nos servirán para cubrir prácticamente todas nuestras necesidades:

 - PDO::FETCH_ASSOC: devuelve un array cuyos índices serán los nombres de las columnas.
 - PDO::FETCH_CLASS: Asigna los valores de las columnas a las propiedades de la clase. Creará nuevas propiedades en caso de que no existan para las columnas.
 - PDO::FETCH_OBJ: devuelve objetos anónimos con propiedades que corresponden a los nombres de columna.

Una vez indicado el cómo queremos los datos, utilizaremos el método `fetch()` para acceder a la información. El método `fetch()` obtiene la siguiente fila de un conjunto de resultados, por lo que podremos iterar por los resultados tal y como se muestra en los ejemplos siguientes:

```php
function fetchAssoc(){
	// Este tipo de fetch crea un array asociativo, indexado por el nombre de la columna.
	
	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos, edad FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_ASSOC);
	// Ejecutamos la sentencia
	$stmt->execute($data);
	// Mostramos los resultados obtenidos
	while($row = $stmt->fetch()) {
		echo $row['nombre'] . "\n";
		echo $row['apellidos'] . "\n";
		echo $row['edad '] . "\n";
	}
}

function fetch_obj(){
	// Este método crea un objeto por cada fila obtenida de la base de datos.

	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos, edad FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_OBJ);
	// Ejecutamos la sentencia
	$stmt->execute($data);
	
	// Mostramos los resultados obtenidos
	while($row = $stmt->fetch()) {
		echo $row->nombre . "\n";
		echo $row->apellidos . "\n";
		echo $row->edad . "\n";
	}

}

function fetch_class(){
	// Este método devuelve los datos como objetos de la clase que nosotros le hayamos indicado.
	// Las propiedades del objeto se inicializarán con los datos de la base de datos antes de llamar al constructor.

	// Si hubiese nombres de columnas que no tienen una propiedad en la clase, se crearán como propiedades de tipo public

	// Se pueden realizar transformaciones sobre esos datos en el constructor de la clase.

	class Alumno {

		public $nombre;
		public $apellidos;
		public $edad;
		public $otraInformacion;

		function __construct($otraInformacion= '') {
			// El constructor se ejecutará después de asociar los valores obtenidos de la base de datos al objeto. Por lo tanto, podemos tratar esos valores dentro del constructor.
			$this->nombre = strtoupper($this->nombre);
			$this->otraInformacion = $otraInformacion;
		}
	}

	$data = array( 'nombre' => 'Mikel', 'edad' => 15 );
	$stmt = $dbh->prepare("SELECT nombre, apellidos FROM alumnos WHERE nombre = :nombre AND edad = :edad");
	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_CLASS, 'Alumno');
	// Ejecutamos la sentencia
	$stmt->execute($data);

	// Mostramos los resultados
	while($obj = $stmt->fetch()) {
		echo $obj->nombre;
	}
}

```
## Método abreviado query()
En consultas que no reciban parámetros, podemos utilizar el método abreviado `query()` el cual ejecutará la sentencia y nos devolverá el conjunto de resultados directamente. En otras palabras, no es necesario hacer la operación en 2 pasos (`prepare()` y `execute()`) como hacíamos hasta ahora.

```php
	$stmt = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	// Establecemos el modo en el que queremos recibir los datos
	$stmt->setFetchMode(PDO::FETCH_ASSOC);

	while($row = $stmt->fetch()) {
		echo $row['nombre'] . "\n";
		echo $row['apellidos'] . "\n";
		echo $row['edad '] . "\n";
	}
```
Por razones de seguridad (evitar [SQL Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL)) es recomendable evitar el método `query()` cuando la sentencia incluya valores variables. Por razones de rendimiento también se recomienda utilizar `prepare()` y `execute()` en sentencias que vayan a ejecutarse varias veces.

## Método fetchObject()
Existe una alternativa al método `fetch()` la cual devolverá los resultados cómo objetos anónimos (**`PDO::FETCH_OBJ`** ) u objetos de la clase indicada ( **`PDO::FETCH_CLASS`**). Este método se llama `fetchObject()`.

```php
	$stmt = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	while($persona = $stmt ->fetchObject()) {
		echo $persona->nombre;
		echo $persona->apellidos;
	}
```
En caso de que queremos que los objetos pertenezcan a una clase en concreto, es suficiente con indicárselo en la llamada:

```php
	$stmt = $dbh->query('SELECT nombre, apellidos, edad from empleado');

	while($persona = $stmt ->fetchObject('Alumno')) {
		echo $persona->nombre;
		echo $persona->apellidos;
	}
```

## Obtener todos los resultados con fetchAll()
A diferencia del método `fetch()`, `fetchAll()` te trae todos los datos de golpe, sin abrir ningún puntero, almacenándolos en un array. Se recomienda cuando no se esperan demasiados resultados que podrían provocar problemas de memoria al querer guardar de golpe en un array miles de filas provenientes de un SELECT.

```php
	// En este caso $resultado será un array asociativo con todos los datos de la base de datos
	$resultado = $stmt->fetchAll(PDO::FETCH_ASSOC);
	
	// Para leer las filas podemos recorrer el array y acceder a la información.
	foreach ($resultado as $row){
	    echo $row["nombre"]." ".$row["apellido"].PHP_EOL;
	}
```
Es casi idéntico que en `fetch()`, sólo que aquí no estamos actuando sobre un recorrido de los datos a través de un puntero, sino sobre los datos ya almacenados en una variable. Si por ejemplo antes de esto nosotros cerramos el statement, ya tenemos los datos en $resultado y podremos leerlos. En `fetch` si cerramos el statement no podremos leer los datos.

## Licencia

Puedes utilizar esta guía para lo que quieras. El objetivo de esta guía es ayudarte a tí y a todas las personas que lo necesiten a utilizar PDO de forma correcta. Puedes utilizar este contenido de la forma que lo creas conveniente, sin ser necesario citar al autor o el origen de la fuente (aunque se agradece ;-)

Happy coding!
