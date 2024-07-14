## PHP Data Object (PDO) <img src="https://github.com/user-attachments/assets/6754d142-a1b1-4313-bc94-6e7ea990c0e0" width="14%" height="14%" align="right" valign="center" alt="PHP Data Object"/> 

![PHP](https://img.shields.io/badge/PHP-%5E8.2-blue)
![License](https://img.shields.io/badge/Code%20GNU-License-blue.svg)
![learning](https://img.shields.io/badge/PDO-learning-blue.svg)

Guia básico de utilização da class PDO do PHP. Os exemplos que você encontrará neste guia são projetados para MariaDB e SQLite3, mas você é livre para adaptá-los a qualquer outro gerenciador de banco de dados suportado.

> **O que é PDO?** <br/>
> É a sigla para [PHP Data Objects](https://www.php.net/manual/pt_BR/intro.pdo.php). Trata-se de uma extensão do PHP para prover acesso a diferentes modelos de bancos de dados através de uma interface única de classes e métodos.

### Conexão com o banco de dados


**Método 1: embutido**
```php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=database;charset=utf8mb4', 'username', 'password');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

**Método 2: Função**
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
