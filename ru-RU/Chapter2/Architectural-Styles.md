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
![Cool](https://raw.githubusercontent.com/TalismanFR/dddinphp/master/share/cool.jpeg)

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

## Многоуровневая архитектура (Layered Architecture)
С точки зрения удобства поддержки и повторого использования кода, наилучший способ сделать код более простым в обслуживании - это разделить концепции, то есть создать слои для каждой 
отдельной задачи. В нашем предыдущем примере легко сформировать разные уровни: первй для икапсуляции доступа и манипулирования данными, второй для решения проблем 
инфраструктуры и третий для оркестрирования двух предыдущих. 
Основное правило многоуровневой архитектуры - это то, что каждый слой должен иметь связь (возможно исопльзовать) с нижестоящими слоями, 
как показано на рисунке ниже

![Layered Architecture](https://raw.githubusercontent.com/TalismanFR/dddinphp/master/share/image--004.jpg)

Многоуровневая архитектура стремится к разделению различных компонентов приложения. Например, с точки зрения 
предыдущего примера, Представление сообщения в блоге должно быть полностью незаисимым от сообщения в блоге как 
концептуальной сущности. Сообщение в блоге как концептуальная сущность может может быть связано с одним или несколькими 
представлениями,  вместо того чтобы быть тесно связанным с каким либо определенным представлением. Это принято называть 
Разделением Ответственности (Separation of Concerns).

Другой парадигмой и шаблоном архитектуры, преследующей ту же цель, является шаблон Model-View-Controller. Изначально он 
задумывался и широко использовался для создания настольных приложений с графическим интерфейсом, а теперь он в основном 
использвуется в веб-приложениях благодаря популяризации веб-фраемворков, таких как Symfony, Zend, CodeIgniter.

### Model-View-Controller

MVC - это архитектурный шаблон и парадигма, которая делит приложение на три основных уровня, описанных в следующих пунктах:

 - Модель (The Model): Содержит все поведения Доменой Модели. Этот уровень управляет всеми данными, логикой, и бизнесс-правилами 
 назависимо от уровня представления данных. Уровень Модели являетяс сердцем и душой каждого приложения MVC.
 - Контроллер (The Controller): Организует взаимодествия между другими уровнями приложения, запускает дествия в слое Модели 
 для обновления ее состояния и обновления слоя Представления связанного с этой моделью. 
 Кроме того, контроллер может отправлять сообщения на уровень Представления для изменения конкретного отображения Модели.
 - Представление (The View): Отображает различные Представления слоя Модели и предоставляет способ вызывать изменения в 
 состоянии модели.
 
![The MVC pattern](https://raw.githubusercontent.com/TalismanFR/dddinphp/master/share/image--006.jpg)
 
### Пример Многоуровневой Архитектуры
 
#### Модель

Продолжая предыдущий пример, мы упомянули, что различные задачи должны быть разделенны. 
Для этого все слои должны быть идентифицированны в нашем оригинальном запутанном коде.
На протяжении всего этого процесса мы должны уделять особое внимание коду, соотвествующему уровню Модели, который будет ядром приложения:

```php
<?php 
class Post
{
    private $title;
    private $content;

    public static function writeNewFrom($title, $content)
    {
        return new static($title, $content);
    }

    private function __construct($title, $content)
    {
        $this->setTitle($title);
        $this->setContent($content);
    }

    private function setTitle($title)
    {
        if (empty($title)) {
            throw new RuntimeException('Title cannot be empty');
        }
        $this->title = $title;
    }

    private function setContent($content)
    {
        if (empty($content)) {
            throw new RuntimeException('Content cannot be empty');
        }
        $this->content = $content;
    }
}

class PostRepository
{
    private $db;
    public function __construct()
    {
        $this->db = new PDO(
            'mysql:host=localhost;dbname=my_database',
            'a_username',
            '4_p4ssw0rd',
            [
            PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8mb4',
            ]
        );
    }

    public function add(Post $post)
    {
        $this->db->beginTransaction();
        try {
            $stm = $this->db->prepare(
                'INSERT INTO posts (title, content) VALUES (?, ?)'
            );
            $stm->execute([
                $post->title(),
                $post->content(),
            ]);
            $this->db->commit();
        } catch (Exception $e) {
            $this->db->rollback();
            throw new UnableToCreatePostException($e);
        }
    }
}
```

Слой Модели теперь определяется классом `Post` и классом `PostRepository`.
Класс `Post` представляет сообщение в блоге, а класс `PostRepository` представляет всю коллеркцию доступных
сообщений в блоге. Кроме того, еще один слой - тот, который координирует поведения Домене Модели - необходим внутри модели.
Рассмотрим Прикладной слой (Application layer):

```php
<?php
class PostService
{
    public function createPost($title, $content)
    {
        $post = Post::writeNewFrom($title, $content);
        (new PostRepository())->add($post);
        return $post;
    }
}
```

Класс `PostService` - это, так называемая Прикладная служба (Служба Приложения, Application Service), и её 
целью является организация поведения Домена. Ни один другой тип объекта не может напрямую взаимодествовать с 
внутренними слоями Модели.

#### Представление (View)
Представление - это слой, который может отправлять и получать сообщения со слоя модели и/или со слоя конетроллера. Его основная цель - 
отобразить модель пользователю на уровне пользовательского интерфейса, а также обновлять это отображение при обновлении модели.

В общем случае, слой Представления получает объект (часто это Объект Передачи Данных (DataTransferObject)), тем самым собирая 
всю необходимую информацию для успешного отображения. Для PHP есть несколько шаблонизаторов, которые могут помочь 
отделить представление Модели от самой Модели и от Контроллера. Самым популярным, на текущий момент, является Twig.
Посмотрим как будет выглядеть слой Представления с Twig.

>**DTO вместо экземпляров Модели?**
>
> Это старый холивар. Зачем создавать DTO вместо того чтобы передать экземпляр класса Модели?
> Основная причина и самый локаничный ответ, знакое нам Разделение Ответственности 
>(Separation of Concerns). Возможность Представления просматривать и использовать приводит к тесной
>связи между слоем Представления и слоем Модели. Фактически, изменение в уровне Модели может нарушить все представления,
>которые используют измененные экземпляры модели.
```twig
{% extends "base.html.twig" %}
{% block content %}
    {% if errormsg is defined %}
        <div class="alert error">{{ errormsg }}</div>
    {% else %}
    <div class="alert success">
        Bravo! Post was created successfully!
    </div>
    {% endif %}
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>TITLE</th>
                <th>ACTIONS</th>
            </tr>
        </thead>
        <tbody>
            {% for post in posts %}
                <tr>
                    <td>{{ post.id }}</td>
                    <td>{{ post.title }}</td>
                    <td><a href="{{ editPostUrl(post.id) }}">Edit Post</a></td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

В большистве случаев, когда Модель инициирует изменение состояния, она также уведомляет связанные 
Представления, чтобы обновить пользовательский интерфейс. В типичном веб-сценарии синхронизация между Моделью и 
его представлениями может быть немного сложной из-за природы клиент-серверной архитектуры.
В таких средах, обчыно требуется использовать JavaScript, чтобы поддерживать эту синхронизацию.
По этой причени, JavaScript MVC фреймворки, о которых говорится ниже, стали широко популярными в последние годы:

 - AngularJS
 - Ember.js
 - Marionette.js
 - React
 
 #### Контроллер (Controller)
 
Слой Контроллера отвечает за организацию и взаимодествие слоёв Модели и Представления.
Он получает сообщения от слоя Представления и запускает поведения модели для выполнения
желаемого действия. Кроме того, он отправляет сообщения в Представления для отрисовки отображения Модели.
Обе операции выполняются благодаря прикладному уровню, который отвечает за организацию, взаимодействие и инкапсуляцию 
поведения Домена.

В терминах веб-приложения на PHP, Контроллер, обычно, охватывает набор классов, которые для достижения
своих целей "Общаются на HTTP". Другими словами они получают HTTP запрос и HTTP запросом дают ответ.

```php
<?php
class PostsController
{
    public function updateAction(Request $request)
    {
        if (
            $request->request->has('submit') &&
            Validator::validate($request->request->post)
        ) {
            $postService = new PostService();
            try {
                $postService->createPost(
                $request->request->get('title'),
                $request->request->get('content')
            );
            $this->addFlash(
                'notice',
                'Post has been created successfully!'
            );
            } catch (Exception $e) {
                $this->addFlash(
                    'error',
                    'Unable to create the post!'
                );
            }
        }
        return $this->render('posts/update-result.html.twig');
    }
}
```

### Инверсия зависемостей. Гексогональная архитектура.
