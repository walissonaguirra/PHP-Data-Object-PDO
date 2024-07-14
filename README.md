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


### Executando instrução SQL
As etapas para executar uma instrução SQL são:

 1. **Connect** conexão com o banco de dados (ver ponto anterior)
 2. **Prepare** prepara uma instrução
 3. **Execute** executar a instrução (associando parâmetros se necessário)
 4. Opcional: processe o resultado da instrução executada.

### Preparando uma instrução
Podemos preparar a instrução para ser executada de forma simples 
```php
$stmt = $pdo->prepare("SELECT name, name_social FROM students");
```

Se precisarmos incluir parâmetros na instrução, podemos usar a sintaxe `:name`
```php
$stmt = $pdo->prepare("SELECT name, name_social FROM students WHERE age > :age");
```

Da mesma forma que preparamos uma consulta do tipo `SELECT`, também podemos preparar uma consulta `INSERT`, `UPDATE`, etc.
```php
$stmt = $pdo->prepare("INSERT INTO students(name, name_social) values (:name, :name_social)");
```

### Executar á instrução
Para executar uma instrução simples que não requer parâmetros, podemos invocar diretamente o método `execute()`.
```php
$stmt = $pdo->prepare("SELECT name, name_social FROM students");
$stmt->execute();
```
Se precisarmos incluir parâmetros na instrução, passaremos as informações em um array associativo para o método `execute()`:
```php
$data = [ ':name' => 'Walisson Aguirra', ':age' => 24 ];
$stmt = $pdo->prepare("SELECT name, name_social FROM students WHERE name = :name AND age = :age");
$stmt->execute($data);
```
Da mesma forma que preparamos uma consulta do tipo `SELECT`, também podemos preparar uma consulta `INSERT`, `UPDATE`, etc.
```php
$data = [ ':name' => 'Walisson Aguirra', ':age' => 24 ];
$stmt = $pdo->prepare("INSERT INTO students(name, age) values (:name, :age)");
$stmt->execute($data);
```

### Processar os resultados de uma consulta SELECT

Uma vez executado o metodo `execute()`, podemos acessar os resultados obtidos do banco de dados. O PDO nos oferece a possibilidade de receber os resultados em diversos formatos: objetos de uma classe, arrays associativos, etc. Para dizer como queremos coletar os resultados, usaremos o método `setFetchMode(String mode)`.

Estes são os 3 valores mais utilizados que nos ajudarão a cobrir praticamente todas as nossas necessidades:

 - **PDO::FETCH_ASSOC:** retorna um array onde os índices serão os nomes das colunas.
 - **PDO::FETCH_CLASS:** Atribui os valores das colunas às propriedades da classe. Criar novas propriedades caso não existam para as colunas.
 - **PDO::FETCH_OBJ:** Retorna objetos anônimos com propriedades que correspondem aos nomes das colunas.

Depois de definirmos como queremos os dados, usaremos o método `fetch()` para acessar as informações. O método `fetch()` busca a próxima linha de um conjunto de resultados, para que possamos iterar pelos resultados conforme mostrado nos exemplos a seguir:

**FETCH ASSOC:**
```php
// Este tipo de busca cria um array associativo, indexado pelo nome da coluna.
$data = array( 'name' => 'Walisson Aguirra', 'age' => 24 );
$stmt = $pdo->prepare("SELECT name, name_social, age FROM students WHERE name = :name AND age = :age");

// Definimos a forma como queremos receber os dados
$stmt->setFetchMode(PDO::FETCH_ASSOC);

// Executamos a instrução sql
$stmt->execute($data);

// Mostramos os resultados obtidos
while($row = $stmt->fetch()) {
    echo $row['name'] . PHP_EOL;
    echo $row['name_social'] . PHP_EOL;
    echo $row['age'] . PHP_EOL;
}
```

**FETCH OBJ:**
```php
// Este método cria um objeto para cada linha obtida do banco de dados.
$data = array( 'name' => 'Walisson Aguirra', 'age' => 24 );
$stmt = $pdo->prepare("SELECT name, name_social, age FROM students WHERE name = :name AND age = :age");

// Definimos a forma como queremos receber os dados
$stmt->setFetchMode(PDO::FETCH_OBJ);

// Executamos a instrução sql
$stmt->execute($data);

// Mostramos os resultados obtidos
while($row = $stmt->fetch()) {
    echo $row->name . PHP_EOL;
    echo $row->name_social . PHP_EOL;
    echo $row->age . PHP_EOL;
}

```

**FETCH CLASS:**
```php
// Este método retorna os dados como objeto da class que indicamos.
// As propriedades do objeto serão inicializadas com dados do banco de dados antes de chamar o construtor.

// Se houver nomes de colunas que não possuem uma propriedade na class, elas serão criadas como propriedades de tipo público

// As modificações desses dados podem ser feito no construtor da class.

class Student {
    public $name;
    public $name_social;
    public $age;
    public $otherInformation;

    function __construct($otherInformation = '') {
        // O construtor será executado após associar os valores obtidos do banco de dados ao objeto. Portanto, podemos tratar esses valores dentro do construtor.
	$this->name = strtoupper($this->name);
        $this->otherInformation = $otherInformation;
    }
}

$data = [ 'name' => 'Walisson Aguirra', 'age' => 24 ];
$stmt = $pdo->prepare("SELECT name, name_social FROM students WHERE name = :name AND age = :age");

// Definimos a forma como queremos receber os dados
$stmt->setFetchMode(PDO::FETCH_CLASS, 'Student');

// Executamos a instrução sql
$stmt->execute($data);

// Mostramos os resultados obtidos
while($obj = $stmt->fetch()) {
    echo $obj->name;
}
```

### Método abreviado query()
Em consultas que não recebem parâmetros, podemos utilizar o método de atalho `query()` que executará a instrução e retornará o conjunto de resultados diretamente. Ou seja, não é necessário fazer a operação em 2 passo (`prepare()` e `execute()`) como fizemos até agora.

```php
$stmt = $pdo->query('SELECT name, name_social, age from students');

// Definimos a forma como queremos receber os dados
$stmt->setFetchMode(PDO::FETCH_ASSOC);

while($row = $stmt->fetch()) {
    echo $row['name'] . PHP_EOL;
    echo $row['name_social'] . PHP_EOL;
    echo $row['age'] . PHP_EOL;
}
```
Por razões de segurança (evite [injeção de SQL](https://pt.wikipedia.org/wiki/Inje%C3%A7%C3%A3o_de_SQL)) é aconselhável evitar o método `query()` quando a instrução inclui valores de variáveis. Por razões de desempenho também é recomendado usar `prepare()` e `execute()` em instruções que serão executadas múltiplas vezes.

### Método fetchObject()
Existe uma alternativa ao método `fetch()` que retornará os resultados como objetos anônimos (**`PDO::FETCH_OBJ`**) ou objetos da classe indicada (**`PDO::FETCH_CLASS`**) . Este método é chamado `fetchObject()`.

```php
$stmt = $pdo->query('SELECT name, name_social, age from students');

while($student = $stmt->fetchObject()) {
    echo $student->name;
    echo $student->name_social;
}
```

Se quisermos que os objetos pertençam a uma classe específica, podemos passa como paramento na chamada do método:
```php
$stmt = $pdo->query('SELECT name, name_social, age from students');

while($student = $stmt ->fetchObject('Studant')) {
    echo $student->name;
    echo $student->name_social;
}
```

### Obter todos os resultados com o método fetchAll()
Ao contrário do método `fetch()`, `fetchAll()` traz todos os dados de uma vez, sem abrir nenhum ponteiro, armazenando-os em um array. É recomendado quando você não espera muitos resultados que possam causar problemas de memória, quando você deseja salvar milhares de linhas de um SELECT em um array de uma só vez.
```php
// Neste caso $result será um array associativo com todos os dados do banco de dados
$result = $stmt->fetchAll(PDO::FETCH_ASSOC);

// Para ler as linhas podemos percorrer o array e acessar as informações.
foreach ($result as $row){
    echo $row["name"]." ".$row["name_social"] . PHP_EOL;
}
```
