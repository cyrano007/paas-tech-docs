## Verbinden Sie eine PHP-App mit PostgreSQL oder MySQL

Wenn Ihre PHP-App eine PostgreSQL- oder MySQL-Datenbank verwendet, muss sie über SSL eine sichere Verbindung zu dieser Datenbank herstellen.

> Bei diesen Anweisungen wird davon ausgegangen, dass Ihre App die PHP Data Objects (PDO)-Bibliothek verwendet, um eine Verbindung zu einer MySQL- oder PostgreSQL-Backing-Service-Datenbank herzustellen.

Sie müssen Ihre App für die Verwendung einer SSL-Verbindung konfigurieren, indem Sie den folgenden Code in die Datei `config.ini` , die sich in `.bp-config/php/php.ini.d/` :

```
extension=pdo.so
extension=pdo_mysql.so OR pdo_pgsql.so
extension=openssl.so
```

Sie sollten diese Methode anstelle der inzwischen veralteten Methode zum Definieren von PHP-Erweiterungen in der `.bp-config/options.json` verwenden.

Weitere Informationen zum Konfigurieren des PHP-Buildpacks finden Sie in der [PHP-Buildpack-Konfigurationsdokumentation](https://docs.cloudfoundry.org/buildpacks/php/gsg-php-config.html) .

Im folgenden Code finden Sie Beispiele dafür, wie Sie Ihre App mit MySQL oder PostgreSQL verbinden.

### Beispielcode – MySQL

```
$vcapServices = json_decode(getenv('VCAP_SERVICES'), true);
$creds = $vcapServices['mysql'][0]['credentials'];

try {
  $pdo = new PDO(
    sprintf('mysql:host=%s;port=%d;dbname=%s', $creds['host'], $creds['port'], $creds['name']),
    $creds['username'],
    $creds['password'],
    array(PDO::MYSQL_ATTR_SSL_CAPATH => '/etc/ssl/certs')
  );
  printf("Result was: %s\n", $pdo->query('SELECT 1')->fetchColumn());
} catch(Expection $e) {
  printf("Error: %s\n", $e->getMessage());
}
```

### Beispielcode – PostgreSQL

```
$vcapServices = json_decode(getenv('VCAP_SERVICES'), true);
$creds = $vcapServices['postgres'][0]['credentials'];

try {
  $pdo = new PDO(
    sprintf('pgsql:host=%s;port=%d;dbname=%s', $creds['host'], $creds['port'], $creds['name']),
    $creds['username'],
    $creds['password']
  );
  printf("Result was: %s\n", $pdo->query('SELECT 1')->fetchColumn());
} catch(Expection $e) {
  printf("Error: %s\n", $e->getMessage());
}
```

## Verbinden Sie Drupal mit MySQL

Wenn Ihre Drupal-App MySQL verwendet, muss sie sich sicher mit SSL mit der Datenbank verbinden. Sie müssen Drupal für die Verwendung einer SSL-Verbindung konfigurieren, indem Sie:

- Erforderliche PHP-Erweiterungen aktivieren
- Einrichten der Datenbankverbindung

### Aktivieren Sie die erforderlichen PHP-Erweiterungen

1. Erstellen Sie eine `mysql.ini` -Datei innerhalb `.bp-config/php/php.ini.d/` .

2. Fügen Sie dieser INI-Datei den folgenden Code hinzu:

    ```
    extension=pdo.so
    extension=pdo_mysql.so
    extension=openssl.so
    ```

Sie sollten diese Methode anstelle der inzwischen veralteten Methode zum Definieren von PHP-Erweiterungen in der `.bp-config/options.json` verwenden.

Weitere Informationen zum Konfigurieren des PHP-Buildpacks finden Sie in der [PHP-Buildpack-Konfigurationsdokumentation](https://docs.cloudfoundry.org/buildpacks/php/gsg-php-config.html) .

### Richten Sie die Datenbankverbindung ein

Fügen Sie den folgenden Code in Ihre Drupal-Konfigurationsdatei ein, die sich standardmäßig unter `sites/default/settings.php` befindet:

```
$vcapServices = json_decode(getenv('VCAP_SERVICES'), true);
$mysqlCreds = $vcapServices['mysql'][0]['credentials'];

$databases['default']['default'] = array(
  'driver' => 'mysql',
  'database' => $mysqlCreds['name'],
  'username' => $mysqlCreds['username'],
  'password' => $mysqlCreds['password'],
  'host' => $mysqlCreds['host'],
  'port' => $mysqlCreds['port'],
  'prefix' => 'drupal_',
  'collation' => 'utf8mb4_general_ci', // For Drupal 8
  // 'collation' => 'utf8_general_ci', // For Drupal 7 or earlier
  'pdo' => array(PDO::MYSQL_ATTR_SSL_CAPATH => '/etc/ssl/certs')
);
```

## Verbinden Sie Wordpress mit MySQL

Ihre Wordpress-App muss eine sichere SSL-Verbindung zu MySQL herstellen. Sie müssen WordPress so konfigurieren, dass es eine SSL-Verbindung verwendet, indem Sie:

- Erforderliche PHP-Erweiterungen aktivieren
- Einrichten der Datenbankverbindung
- Wordpress patchen, um SSL-Verbindungen zu aktivieren

### Aktivieren Sie die erforderlichen PHP-Erweiterungen

1. Erstellen Sie eine `mysql.ini` -Datei innerhalb `.bp-config/php/php.ini.d/` .
2. Fügen Sie dieser INI-Datei den folgenden Code hinzu:

```
extension=mysqli.so
extension=openssl.so
```

Sie sollten diese Methode anstelle der veralteten Methode zum Definieren von PHP-Erweiterungen in der `.bp-config/options.json` verwenden.

Weitere Informationen zum Konfigurieren des PHP-Buildpacks finden Sie in der [PHP-Buildpack-Konfigurationsdokumentation](https://docs.cloudfoundry.org/buildpacks/php/gsg-php-config.html) .

### Richten Sie die Datenbankverbindung ein

Ersetzen Sie den Datenbankkonfigurationscode in Ihrer `wp-config.php` Datei durch den folgenden Code:

```
$vcapServices = json_decode(getenv('VCAP_SERVICES'), true);
$mysqlCreds = $vcapServices['mysql'][0]['credentials'];

define('DB_NAME', $mysqlCreds["name"]);
define('DB_USER', $mysqlCreds["username"]);
define('DB_PASSWORD', $mysqlCreds["password"]);
define('DB_HOST', $mysqlCreds["host"]);
define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);
define('MYSQL_SSL_CAPATH', "/etc/ssl/certs");
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
```

### Patchen Sie Wordpress, um SSL-Verbindungen zu aktivieren

Fügen Sie den folgenden Code vor dem Funktionsaufruf `mysqli_real_connect` in die Datei `wp-includes/wp-db.php` :

```
[...]

// Included block start
mysqli_ssl_set($this->dbh, null, null, null, MYSQL_SSL_CAPATH, null);
// Included block end

if ( WP_DEBUG ) {
    mysqli_real_connect( $this->dbh, $host, $this->dbuser, $this->dbpassword, null, $port, $socket, $client_flags );
} else {
    @mysqli_real_connect( $this->dbh, $host, $this->dbuser, $this->dbpassword, null, $port, $socket, $client_flags );
}
[...]
```
