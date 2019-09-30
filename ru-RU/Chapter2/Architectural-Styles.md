Глава 2. Архитектурные стили
==

Чтобы иметь возможность создавать сложные приложения, одним из ключевых требований является наличие архитектурного дизайна, 
который соответствует потребностям приложения. Одно из преимущества DDD заключается в том, что он не привязан к какому-либо 
конкретному стилю архитектуры. Вместо этого мы можем свободно выбирать архитектуру, которая наилучшим образом соответствует 
потребностям каждого Ограниченного Контекста внутри основного Домена, который предлагает разнообразный набор архитектурных 
решений для каждого конкретной проблемы Домена.

Например, Система Обработки Заказов может использовать подход Event Sourcing для отслеживания всех различных операций с заказом; 
Каталог продуктов может использовать CQRS для предоставления информации о продуктах различным клиентам; Система Управления  Контентом 
может использовать Гексогональную Архитектуру (Hexagonal Architecture) для представления требований, таких как блоги, статичные страницы и т.д.

В этой главе представленно введение во все соответствующие архитектурные стили в контексте PHP, следуя эволюции от традиционного 
(старого) PHP-кода к более сложной архитектуре. Обратите внимание, что, хотя существует много дургих существующих архитектурных стилей, 
таких как Data Fabric или SOA, некоторые из них оказались слишком сложными для представления средствами PHP.

## Старые, добрые времена

До релиза PHP версии 4, язык не охватывал Объектно-Ориентированную Парадигму. В то время обычным способом написания приложения 
было использвоание процедур и глобального состояния. Такие понятия, как Разделение Ответственности (Separation of Concerns (SoC)) и 
Модель-Представление-Контролер (Model-View-Controller (MVC)) были широко распространены среди сообщества PHP.

Пример ниже представляет собой приложение, написанное традиционным способом, где приложение состоят из множества фронтальных контроллеров, смешанных 
с кодом HTML. В те времена слой Инфраструктуры, Представления, UI, и Доменные слои были перемешаны.

```php
<?php
include __DIR__ . '/bootstrap.php';
$link = mysql_connect('localhost', 'a_username', '4_p4ssw0rd');
if (!$link) {    
    die('Could not connect: ' . mysql_error());
}
mysql_set_charset('utf8', $link);
mysql_select_db('my_database', $link);
$errormsg = null;
if (isset($_POST['submit']) && isValid($_POST['post'])) {
    $post = getFrom($_POST['post']);
    mysql_query('START TRANSACTION', $link);
    $sql = sprintf(
        "INSERT INTO posts (title, content) VALUES ('%s','%s')",
        mysql_real_escape_string($post['title']),
        mysql_real_escape_string($post['content']
    ));
    $result = mysql_query($sql, $link);
    if ($result) {
        mysql_query('COMMIT', $link);
    } else {
        mysql_query('ROLLBACK', $link);
        $errormsg = 'Post could not be created! :(';
    }}
$result = mysql_query('SELECT id, title, content FROM posts', $link);?>
<html>
    <head></head>
    <body>
        <?php if (null !== $errormsg) : ?>
            <div class="alert error"><?php echo $errormsg; ?></div>
        <?php else: ?>
            <div class="alert success">
                Bravo! Post was created successfully!
            </div>
        <?php endif; ?>
        <table>
            <thead><tr><th>ID</th><th>TITLE</th>
            <th>ACTIONS</th></tr></thead>
            <tbody>
            <?php while($post = mysql_fetch_assoc($result)) : ?>
                <tr>
                    <td><?php echo $post['id']; ?></td>
                    <td><?php echo $post['title']; ?></td>
                    <td><?php editPostUrl($post['id']); ?></td>
                </tr>
            <?php endwhile; ?>
            </tbody>
        </table>
   </body>
 </html>
 <?php mysql_close($link); ?>
```
![Cool](link)

Этот стиль кода часто называют Большой Ком Грязи (Big Ball of Mud).
Однако в этом стиле улучшение стало то, что заголовок и нижний колонтитул веб-страницы были заключены в отдельные файлы.
Это позвонило избежать дублирования и способствовало повторному использованию:

```php
<?php
include __DIR__ . '/bootstrap.php';
$link = mysql_connect('localhost', 'a_username', '4_p4ssw0rd');
if (!$link) {
    die('Could not connect: ' . mysql_error());
}
mysql_set_charset('utf8', $link);
mysql_select_db('my_database', $link);
$errormsg = null;
if (isset($_POST['submit']) && isValid($_POST['post'])) {
    $post = getFrom($_POST['post']);
    mysql_query('START TRANSACTION', $link);
    $sql = sprintf(
        "INSERT INTO posts(title, content) VALUES('%s','%s')",
        mysql_real_escape_string($post['title']),
        mysql_real_escape_string($post['content'])
    );
    $result = mysql_query($sql, $link);
    if ($result) {
        mysql_query('COMMIT', $link);
    } else {
        mysql_query('ROLLBACK', $link);
        $errormsg = 'Post could not be created! :(';
    }
}
$result = mysql_query('SELECT id, title, content FROM posts', $link);
?>
<?php 
include __DIR__ . '/header.php';
 ?>
<?php 
if (null !== $errormsg) : ?>
    <div class="alert error">
<?php echo $errormsg; ?>
</div>
<?php else: ?>
    <div class="alert success">
        Bravo! Post was created successfully!
    </div>
<?php endif;
 ?>
<table>
    <thead>
        <tr>
            <th>ID</th>
            <th>TITLE</th>
            <th>ACTIONS</th>
        </tr>
    </thead>
    <tbody>
    <?php while($post = mysql_fetch_assoc($result)): ?>
        <tr>
            <td><?php echo $post['id']; ?></td>
            <td><?php echo $post['title']; ?></td>
            <td><?php editPostUrl($post['id']); ?></td>
        </tr>
    <?php endwhile; ?>
    </tbody>
</table>
<?php include __DIR__ . '/footer.php'; ?>
```

В настоящее время, хотя это крайн нежелательно, все еще встречаются приложение которые используют подобный процедурный подход. (скорре всего legacy код)
Основным недостатком этого стиля архитектуры является то, что нет реального Разделения Ответственности (Separation of Concerns). Обслуживание и 
стоимость разработки приложения, разработанного таким образом, резко возрастает по сравнению с другими известными и проверенными архитектурами.

## Многоуровневая архитектура
С точки зрения удобства поддержки и повторого использования кода, наилучший способ сделать код более простым в обслуживании - это разделить концепции, то есть создать слои для каждой 
отдельной задачи. В нашем предыдущем примере легко сформировать разные уровни: первй для икапсуляции доступа и манипулирования данными, второй для решения проблем 
инфраструктуры и третий для оркестрирования двух предыдущих. 
Основное правило многоуровневой архитектуры - это то, что каждый слой должен иметь связь (возможно исопльзовать) с нижестоящими слоями, 
как показано на рисунке ниже

![Layered Architecture]()