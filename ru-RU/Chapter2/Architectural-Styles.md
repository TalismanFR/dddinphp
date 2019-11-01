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
> Основная причина и самый локаничный ответ, знакомое нам Разделение Ответственности 
>(Separation of Concerns). Возможность Представления просматривать и использовать поступающие объекты приводит к тесной
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

Следую основному правилу Многоуровневой архитектуры, существует риск инкапсулирования инфраструктурных проблем в реализацию 
Доменных интерфейсов.

Например, класс PostRepository из предыдущего MVC примера, должен быть помещен  Домен предметной области.
Однако размещение инфраструктурных деталей прямо в теле нашего Домена нарушает Разделение Ответственности
(Domain Separation). Это создает проблемы; трудно избежать нарушения основных правил многоуровневой архитектуры, что приводит 
к написанию кода которых тяжело поддается тестированию, т.к. уровень Домена осведомлен о технических реализациях.

#### Принцип Инверсии Зависимостей (The Dependency Inversion Principle (DIP))

Как мы можем это исправить? Поскольку уровень Доменной Модели связан с конкретной реализаций 
инфраструктуры, можно применить принцип инверсии зависимости (DIP), переместив Инфраструктурный слой выше
всех остальных слоёв.

>**Принцип Инверсии Зависимостей**
>Модули более выского уровня не должны зависеть от нижележащий уровней. Оба должны связываться через абстракции.
>Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций. _Robert C. Martin_

При использовании принципа Инверсии Зависемостей, схема архитектуры меняется, и уровень инфраструктуры, который
можно назвать модулем низкого уровня, теперь зависит от пользовательского интерфейса, Прикладного слоя, и Доменного слоя, 
которые являются высокоуровневыми модулями. Зависимость была инвертирована.

Но что такое Гексагональная Архитектура и как она вписывается во все это? 
Гексогональная Архитектура (так же именуемая как Порты и Адаптеры) была определена Алистером 
Кокберном в его книге _"Hexagonal Architecture"_. Он изображает приложение как шестиугольник,
где каждая строна представляет Портс одним или неколькими Адаптерами. 
Порт - это коннектор с подключаемым Адаптером, который преоброзует внешний вход во что-то, что 
может понять внутренее приложение. С точки зрения DIP, Порт был бы модулем выского уровня, а Адаптер
был бы модулем низлежащего уровня. Кроме того, если приложению необходимо отправить сообщение во внешний сервис, оно 
также будет использовать порт с адаптером для преобразования сообщения в язык понятный внешнему сервису и последующей отправкой сообщения.
По этой причине Гексагональная архитектура воспитывает концепцию симметрии в приложении, а также 
является основной причиной, по которой меняется схема архитектуры.
Её часто представляют в виде шестиугольника, потому что больше не имеет смысла говорить о верхнем или нижнем слое.
Вместо этого, в Гексагональной Архитектуре говорится в внешнем слое и внутренем.

#### Применение Гексогональной Архитектуры

Продолжим рассматривать приме с блогом. Первая концепция, которая нам нужна, это Порт, через который 
внешний мир может общаться с приложением. Для этого случая мы будем использовать HTTP-порт и соответствующий ему Адаптер.
Внешним будет порт для отправки сообщений в приложение. В пример блога использовалась база данных для
хранения постов блога, поэтому нам так же необходим Порт для извлечения постов из базы данных:
```php
<?php
interface PostRepository
{
    public function byId(PostId $id);
    public function add(Post $post);
}
```
Этот интерфейс представляет собой Порт, через который приложение будет получать информацию о постах блога, и он 
будет расположен на уровне Доменного слоя. Теперь нужен Адаптер для этого Порта.
Адаптер отвечает за определение способа извлечения сообщений из блога с использованием 
определенной технологии.
```php
<?php
class PDOPostRepository implements PostRepository
{
    private $db;
    public function __construct(PDO $db)
    {
        $this->db = $db;
    }
    public function byId(PostId $id)
    {
        $stm = $this->db->prepare(
            'SELECT * FROM posts WHERE id = ?'
        );
        $stm->execute([$id->id()]);
        return recreateFrom($stm->fetch());
    }
    public function add(Post $post)
    {
        $stm = $this->db->prepare(
            'INSERT INTO posts (title, content) VALUES (?, ?)'
        );
        $stm->execute([
            $post->title(),
            $post->content(),
        ]);
    }
}
```
После того, как мы определили Порт и его Адаптер, последним шагом будет рефакторинг класса
PostService, чтобы он использовал новый механизм. Это может быть легко достигнуто с помощью
Иньекции Зависимостей (Dependency Injection):
```php
<?php
class PostService
{
    private $postRepository;
    public function __construct(PostRepositor $postRepository)
    {
        $this->postRepository = $postRepository;
    }
    public function createPost($title, $content)
    {
        $post = Post::writeNewFrom($title, $content);
        $this->postRepository->add($post);
        return $post;
    }
}
```

Это простой пример Гегсогональной архитектуры. Это гибкая архитектура, которая способствует Разделению
Ответственности, как в Многоуровневой Архитектуре. Это также способствует симметрии, благодаря
наличию внутренней части приложения, которая связывается с внешним слоем через Порты.
Отныне, это будет основополгающая архитектура, используемая для построения и объяснения CQRS и
Event Sourcing.

Более детальный разбор этой архитектуры вы можете найти в главе _"Приложение. Гексагональная архитектура в PHP"_.
Для более подробного примера вам следует перейти к _Главе 11 "Приложение"_, в которой объясняются такие сложные темы, 
как транзакционность и другие сквозные вопросы.

### Command Query Responsibility Segregation (CQRS)

Гексагональная архитектура - это хорошая основопологающая архитектура, но она имеет некоторые ограничения.
Например, сложные пользовательские интерфейсы могут требовать Агрегированной информации, отображаемой
в различных формах (Глава 8, Агрегаты), или они могут требовать данных, полученных из нескольких агрегатов.
И в этом сценарии мы могли бы реализовать множество методов поиска в Репозиториях (возможно, столько же,
сколько представлений пользовательского интерфейса есть в приложении). Или, может быть, мы решим
перенести эту сложность в Службу Приложений (Application Service), используя сложные структуры для 
сбора и объедининя данных из различных Агрегатов. Пример:

```php
<?php
interface PostRepository
{
    public function save(Post $post);
    public function byId(PostId $id);
    public function all();
    public function byCategory(CategoryId $categoryId); 
    public function byTag(TagId $tagId);
    public function withComments(PostId $id);
    public function groupedByMonth();
    // ...
}
```

Когда этими конструкциями злоупотребляют, построение пользовательских представлений может стать
действительно болезненым. Мы должны принять компромисс между тем, чтобы заставлять Службу Приложений 
возвращать Доменную Модель или возвращать некие DTO`s. В последнем вариант (DTO), мы избегаем тесной связи между
Доменной модели и кодом Инфраструктуры (веб-контроллеры, cli контроллеры и т.д.).

К счастью, есть другой подход. Если проблема заключается в множественных и разрозненных представлениях, то мы можем исключить 
их из Доменным Моделей и начать рассматривать их как чисто инфраструктурную проблему. Эта опция базируется
на принципе разработки, Разделение Командных Запросов (CQS, Command Query Separation). Этот принцпи был определен
Бертраном Мейером (Bertrand Meyer), и, в свою очередь, он породил новый архитектурный паттерн под названием
"Разделение Ответственности на Запросы и Команды" (CQRS, Command Query Responsibility Segregation), как это определенно
Грегом Янгом (Greg Young).

>**Command Query Separation (CQS)**
>Задание вопроса не должно менять ответ - Бертран Мейер.
>Этот принцип разработки гласит, что каждый метод должен быть либо командой, выполняющей действие, либо 
>запросом, возвращающим данные вызывающей стороне, но не обоими сразу. Wikipedia

CQRS стремится к еще более агрессивному Разделению Проблем, деля модель на две части:
 - Модель Записи (The Write Model): также известная как Командная модель (Command Model), она выполняет запись
и несет ответственность за истинное поведение домена.
 - Модель Чтения (The Read Model): она берет на себя ответственность за чтение в приложении и расматривается как 
 нечто что должно выходить за пределы предметной области.
 
Каждый раз, когда кто-то запускает команду для модели записи, выполняется запись в нужное хранилище данных.
Кроме того, она запускает обновление Модели Чтения, чтобы в ней отобразились последние изменения.

Это строгое разделение вызывает еще одно проблему: Согласованность в Конечном Итоге (Eventual Consistency).
Согласованность модели чтения теперь зависит от команд, выполняемых Моделью Записи. Другими словами, модель чтения
в конечном итоге непротиворечива. То есть каждый раз, когда Модель Записи выполняет команду, 
она запускает процесс, который будет отвечать за обновление модели чтения в соответствии с последними 
обновлениями модели записи. Есть некоторый лаг во времени, когда пользовательский интерфейс может представить
устаревшую информацию пользователю. В веб-сценарии это происходит часто, поскольку мы несколько ограничены текущими
технологиями.

Подумайте о системе кэширования стоящим перед веб-приложением. Каждый раз, когда база данных обновляется новой информацией,
данные на уровен кэша потенциально могут быть устаревшими, поэтому каждый раз, когда она обновляется,
должен быть процесс, который обновляет систему кэша. Системы кэширования в конечном итоге становятся согласованными.

Такие процессы, в терминалогии CQRS, называются Проекции Модели Записи (Write Model Projections)
или просто Проекции. Мы проецируем Модели Записи на Модель Чтения. Этот процесс может быть синхронным или
асинхронным, в зависимости от ваших потребностей, и этом может быть сделанно благодаря полезному тактическому
шаблону проектирования - Глава "События Домена" - который будет подробно объяснен позже в книге.
Основной проекций Модели Записи является сбор всех опубликованных событий домена и обновление модели чтения всей информацией, 
поступившей из событий.

#### Модель записи
Модель записи является истиным владельцем поведения Домена. Продолжая
наш пример, интерфейс репозитория будет урощен до следующего:
```php
<?php
interface PostRepository
{
    public function save(Post $post);
    public function byId(PostId $id);
}
```

Теперь `PostRepository` освобожден от всех задач чтения, кроме одной: функция byId, которая
отвечает за загрузку Агрегата по его ID, для дальнейшей работы с ним.
Так же будут удалены все методы запросов (query) из модели `Post`, оставив только
методы команд. Это приводит к тому, что мы избавляемся от всех методов
получения данных и любых других методов, предоставляющих
информацию о Агрегате Post. Вместо этого будут опубликоны Доменные События, чтобы запустить
проекции Модели Записи использую подписку на них (события):

```php
<?php
class AggregateRoot
{
    private $recordedEvents = [];
    protected function recordApplyAndPublishThat(
        DomainEvent $domainEvent
    ) {
        $this->recordThat($domainEvent);
        $this->applyThat($domainEvent);
        $this->publishThat($domainEvent);
    }

    protected function recordThat(DomainEvent $domainEvent)
    {
        $this->recordedEvents[] = $domainEvent;
    }

    protected function applyThat(DomainEvent $domainEvent)
    {
        $modifier = 'apply' . get_class($domainEvent);
        $this->$modifier($domainEvent);
    }

    protected function publishThat(DomainEvent $domainEvent)
    {
        DomainEventPublisher::getInstance()->publish($domainEvent);
    }

    public function recordedEvents()
    {
        return $this->recordedEvents;
    }

    public function clearEvents()
    {
        $this->recordedEvents = [];
    }
}

class Post extends AggregateRoot
{
    private $id;
    private $title;
    private $content;
    private $published = false;
    private $categories;
    private function __construct(PostId $id)
    {
        $this->id = $id;
        $this->categories = new Collection();
    }

    public static function writeNewFrom($title, $content)
    {
        $postId = PostId::create();
        $post = new static($postId);
        $post->recordApplyAndPublishThat(
            new PostWasCreated($postId, $title, $content)
        );
    }

    public function publish()
    {
        $this->recordApplyAndPublishThat(
            new PostWasPublished($this->id)
        );
    }

    public function categorizeIn(CategoryId $categoryId)
    {
        $this->recordApplyAndPublishThat(
            new PostWasCategorized($this->id, $categoryId)
        );
    }

    public function changeContentFor($newContent)
    {
        $this->recordApplyAndPublishThat(
            new PostContentWasChanged($this->id, $newContent)
        );
    }

    public function changeTitleFor($newTitle)
    {
        $this->recordApplyAndPublishThat(
            new PostTitleWasChanged($this->id, $newTitle)
        );
    }
}
```

Все действия, которые инициируют изменение состояния, реализуются
через события Домена. Для каждого опубликованного Доменого События существует
соотвествующий метод apply, отвечающий за отражение изменения состояния:

```php
<?php
class Post extends AggregateRoot
{
    // ...
    protected function applyPostWasCreated(
        PostWasCreated $event
    ) {
        $this->id = $event->id();
        $this->title = $event->title();
        $this->content = $event->content();
    }

    protected function applyPostWasPublished(
        PostWasPublished $event
    ) {
        $this->published = true;
    }

    protected function applyPostWasCategorized(
        PostWasCategorized $event
    ) {
        $this->categories->add($event->categoryId());
    }

    protected function applyPostContentWasChanged(
        PostContentWasChanged $event
    ) {
        $this->content = $event->content();
    }

    protected function applyPostTitleWasChanged(
        PostTitleWasChanged $event
    ) {
        $this->title = $event->title();
    }
}
```

#### Модель чтения

Модель Чтения, так же известная как модель запросов (Query Model), является 
денормализованной, в интересах Домена, моделью данных.
Фактически, в CQRS все задачи чтения считаются процессами отчетности в 
инфраструктурной задаче. Как правило, при использовании CQRS Модель Чтения зависит от 
потребностей пользовательского интерфейса и сложности представлений, состовляющих
пользовательский интерфейс. В ситуации, когда модель чтения определяется в терминах реляционных баз данных, 
простейшим подходом было бы установить взаимно-однозначные отношения между
таблицами базы данных и представлениями пользовательского интерфейса. Эти
таблицы базы данных и представления пользовательского интерфейса будут
обновлены с использованием проекций Модели Записи, инициированных событиями домена,
 опубликованными стороной записи: 
 
```sql
-- Определение представляния UI для поста с его комментариями
CREATE TABLE single_post_with_comments (
    id INTEGER NOT NULL,
    post_id INTEGER NOT NULL,
    post_title VARCHAR(100) NOT NULL,
    post_content TEXT NOT NULL,
    post_created_at DATETIME NOT NULL,
    comment_content TEXT NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Вставка данных
INSERT INTO single_post_with_comments VALUES
    (1, 1, "Layered" , "Some content", NOW(), "A comment"),
    (2, 1, "Layered" , "Some content", NOW(), "The comment"),
    (3, 2, "Hexagonal" , "Some content", NOW(), "No comment"),
    (4, 2, "Hexagonal", "Some content", NOW(), "All comments"),
    (5, 3, "CQRS", "Some content", NOW(), "This comment"),
    (6, 3, "CQRS", "Some content", NOW(), "That comment");

-- Запрос на получения данных статьи
SELECT * FROM single_post_with_comments WHERE post_id = 1;
```

Важной особенностью этого архитектурного стиля является то, что
 Модель Чтения должна быть полностью одноразовой, поскольку истинное состояние приложения 
определяется Моделью Записи. Это означает что Модель Чтения может быть удалено
и пересоздано при необходимости использую проекции Модели Записи.

Ниже мы можеи увидеть некоторые примеры возможных представлений в преложении блога:
```sql
SELECT * FROM
posts_grouped_by_month_and_year
ORDER BY month DESC,year ASC;

SELECT * FROM
posts_by_tags
WHERE tag = "ddd";

SELECT * FROM
posts_by_author
WHERE author_id = 1;
```

Важно отметить, что CQRS не ограничевается реализацией модели для реляционной
базы данных. Это зависит исключительно от потребностей создаваемого приложения.
Это может быть реляционная база данных, документно-ориентированная база данных, хранилище типа ключ-значение,
или что-либо, что лучше всего соответствует потребностям вашего приложения. После приложения для публикации постов в блоге мы
будет использовать `Elasticsearch` (базу данных, ориентированную на документы)
 - для реализации модели чтения.
 
```php
<?php
class PostsController
{
    public function listAction()
    {
        $client = new ElasticsearchClientBuilder::create()->build();
        $response = $client->search([
            'index' => 'blog-engine',
            'type' => 'posts',
            'body' => [
                'sort' => [
                    'created_at' => ['order' => 'desc']
                ]
            ]
        ]);
        return [
            'posts' => $response
        ];
    }
}
```

Код Модели Чтения был существенно упрощен до одного запроса к индексу
Elasticsearch.

Этот код показывает, что Модель Чтения на самом деле не нуждается в
объектно-реляционном преобразователе, посколько это может быть излишним.
Однако Модель Записи может выиграть от использования объектно-реляционого
преобразования, поскольку это позволит вам организовать и структурировать Модель Чтения
в соответствии с потребностями приложения.

#### Синхронизация Модели Записи с Моделью Чтения

Здесь начинается сложная часть. Как мы синхронизируем Модель Чтения с 
Моделью Записи? Мы уже говорили, что сделаем это с помощью Событий Домена,
захваченных в тразакции Модели Записи. Для каждого типа захваченного События Домена
будет выполнена соответствующая проекция. Таким образом, будет установлено 
взаимно-однозначное отношение между Событиями Домена и проекциями.

Давайте посмотрим на пример настройки проекций, для лучшего понимания идеи.
Прежде всего, нам нужно определить каркас для проекций:

```php
<?php
interface Projection
{
    public function listensTo();
    public function project($event);
}
```

Определение проекции `Elasticsearch` для события `PostWasCreated` достаточно просто:

```php
<?php
namespace Infrastructure\Projection\Elasticsearch;
use Elasticsearch\Client;
use PostWasCreated;
class PostWasCreatedProjection implements Projection
{
    private $client;
    public function __construct(Client $client)
    {
        $this->client = $client;
    }

    public function listensTo()
    {
        return PostWasCreated::class;
    }

    public function project($event)
    {
        $this->client->index([
        'index' => 'posts',
        'type' => 'post',
        'id' => $event->getPostId(),
        'body' => [
            'content' => $event->getPostContent(),
                // ...
            ]
        ]);
    }
}
```

Реализация Проекции является своего рода специализированным слушателем
Событий Домена. Основное различие между этой Проекций и слушателем Доменых Событий по умолчанию в том,
что Проекция реагирует на группу Доменных Событий, а не только на одно.

```php
<?php
namespace Infrastructure\Projection;
class Projector
{
    private $projections = [];
    public function register(array $projections)
    {
        foreach ($projections as $projection) {
            $this->projections[$projection->eventType()] = $projection;
        }
    }

    public function project( array $events)
    {
        foreach ($events as $event) {
            if (isset($this->projections[get_class($event)])) {
                $this->projections[get_class($event)]
                    ->project($event);
            }
        }
    }
}
```

Следующий код показывает, как будет выглядеть поток между проекцией и событиями:

```php
<?php
$client = new ElasticsearchClientBuilder::create()->build();

$projector = new Projector();
$projector->register([
    new Infrastructure\Projection\Elasticsearch\
    PostWasCreatedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasPublishedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasCategorizedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostContentWasChangedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostTitleWasChangedProjection($client),
]);

$events = [
    new PostWasCreated(/* ... */),
    new PostWasPublished(/* ... */),
    new PostWasCategorized(/* ... */),
    new PostContentWasChanged(/* ... */),
    new PostTitleWasChanged(/* ... */),
];

$projector->project($event);
```

Этот код является своего рода синхронным, но процесс может быть и 
асинхронным, если это необходимо. И вы могли бы информировать своих клиентов об этих
не синхроннизированных данных, разместив несколько предупреждений в слое представления.

В следующем примере мы будем использовать PHP-расширение amqplib в сочетании с 
ReactPHP:

```php
<?php
// Connect to an AMQP broker
$cnn = new AMQPConnection();
$cnn->connect();

// Создание канала
$ch = new AMQPChannel($cnn);

// Declare a new exchange
$ex = new AMQPExchange($ch);
$ex->setName('events');
$ex->declare();

// Create an event loop
$loop = ReactEventLoopFactory::create();

// Создание поставщика, который будет отправлять
// любые ожидающие сообщения каждые полсекунды
$producer = new Gos\Component\React\AMQPProducer($ex, $loop, 0.5);
$serializer = JMS\Serializer\SerializerBuilder::create()->build();
$projector = new AsyncProjector($producer, $serializer);
$events = [
    new PostWasCreated(/* ... */),
    new PostWasPublished(/* ... */),
    new PostWasCategorized(/* ... */),
    new PostContentWasChanged(/* ... */),
    new PostTitleWasChanged(/* ... */),
];
$projector->project($event);
```

Чтобы это работало, нам нужен асинхронная проекция. Вот наивная реализация этого:

```php
<?php
namespace Infrastructure\Projection;
use Gos\Component\React\AMQPProducer;
use JMS\Serializer\Serializer;
class AsyncProjector
{
    private $producer;
    private $serializer;
    public function __construct(
        Producer $producer,
        Serializer $serializer
    ) {
        $this->producer = $producer;
        $this->serializer = $serializer;
    }
    public function project(array $events)
    {
        foreach ($events as $event) {
            $this->producer->publish(
                $this->serializer->serialize(
                    $event, 'json'
                )
            );
        }
    }
}
```

И потребитель событий с использованием брокера RabbitMQ будет 
выглядеть примерно так:

```php
<?php
// Connect to an AMQP broker
$cnn = new AMQPConnection();
$cnn-> connect();

// Create a channel
$ch = new AMQPChannel($cnn);

// Create a new queue
$queue = new AMQPQueue($ch);
$queue->setName('events');
$queue->declare();

// Create an event loop
$loop = React\EventLoop\Factory::create();
$serializer = JMS\Serializer\SerializerBuilder::create()->build();
$client = new Elasticsearch\ClientBuilder::create()->build();

$projector = new Projector();
$projector->register([
    new Infrastructure\Projection\Elasticsearch\
    PostWasCreatedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasPublishedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasCategorizedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostContentWasChangedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostTitleWasChangedProjection($client),
]);

// Create a consumer
$consumer = new Gos\Component\ReactAMQP\Consumer($queue, $loop, 0.5, 10);

// Check for messages every half a second and consume up to 10 at a time.
$consumer->on(
    'consume',
    function ($envelope, $queue) use ($projector, $serializer) {
        $event = $serializer->unserialize($envelope->getBody(), 'json');
        $projector->project($event);
    }
);
$loop->run();
```

Отсюда все становится проще. Мы заставляем все репозитории использовать
экземпляр проекции, а затем так же запускаем процесс проецирования:

```php
<?php
class DoctrinePostRepository implements PostRepository
{
    private $em;
    private $projector;
    public function __construct(EntityManager $em, Projector $projector)
    {
        $this->em = $em;
        $this->projector = $projector;
    }

    public function save(Post $post)
    {
        $this->em->transactional(
            function (EntityManager $em) use ($post)
            {
                $em->persist($post);
                foreach ($post->recordedEvents() as $event) {
                    $em->persist($event);
                }
            }
        );
        $this->projector->project($post->recordedEvents());
    }

    public function byId(PostId $id)
    {
        return $this->em->find($id);
    }
}
```

Экземпляр `Post` и записанные события запускаются и сохраняются в одной транзакции. Это гарантирует, что никакие
события не будут потеряны, так как мы спроецируем их на модель чтения, если транзакция прошла успешно. 
В результате между Моделью Записи и Моделью чтения не будет никаких несоответствий.

>**ORM или без ORM**
>
>Один из наиболее распространных вопросов при реализации CQRS - действительно ли нужен объектно-реляционный маппер
> (ORM)? Мы твердо верим, что использование ORM для модели записи прекрасно и дает все преимущества использования
>инструмента, который поможет нам сэкономить много работы в случае использования реляционной базы данных.
>Но мы не должны забывать, что нам все еще нужно сохранять и извлекать состояние Модели Записи используя 
>реляционную базу данных

### Event Sourcing
