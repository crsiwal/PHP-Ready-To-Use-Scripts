The correct way to avoid SQL injection attacks, no matter which database you use, is to separate the data from SQL, so that data stays data and will never be interpreted as commands by the SQL parser. It is possible to create an SQL statement with correctly formatted data parts, but if you don't fully understand the details, you should always use prepared statements and parameterized queries. These are SQL statements that are sent to and parsed by the database server separately from any parameters. This way it is impossible for an attacker to inject malicious SQL.

You basically have two options to achieve this:

Using PDO (for any supported database driver):

$stmt = $pdo->prepare('SELECT * FROM employees WHERE name = :name');
$stmt->execute([ 'name' => $name ]);

foreach ($stmt as $row) {
    // Do something with $row
}
Using MySQLi (for MySQL):

$stmt = $dbConnection->prepare('SELECT * FROM employees WHERE name = ?');
$stmt->bind_param('s', $name); // 's' specifies the variable type => 'string'
$stmt->execute();

$result = $stmt->get_result();
while ($row = $result->fetch_assoc()) {
    // Do something with $row
}
If you're connecting to a database other than MySQL, there is a driver-specific second option that you can refer to (for example, pg_prepare() and pg_execute() for PostgreSQL). PDO is the universal option.
