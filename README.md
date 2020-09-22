## Iniciando com o laravel

Criando uma nova instalação
> composer create-project --prefer-dist laravel/laravel <nome> "5.7.*"

Alterando o namespace padrão(app) do projeto
> php artisan app:name <nome>

Alterando o timezone. Arquivo config>app.php
> 'timezone' => 'America/Recife'

Alterando a tradução
> https://github.com/UpInside/laravel-pt-BR.git 


Para criar uma View. Em _resource>views_, lembrando que é necessário usar o nome.blade.php - blade é o template padrão do laravel.

Para esse exemplo:
> form.blad.php

O laravel possui dentra da instalação padrão o bootstrap. Para inserir basta apenas colocar na view:

Para o estilo
> <link rel="stylesheet" href= {{ asset('css/app.css') }} >

Para o script
> <script src={{ asset('js/app.js') }}></script>

Vamos criar uma Rota(routes>web.php) para visualizar o formulário criado.

```
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Formulário de teste de rotas</title>

    <link rel="stylesheet" href= "{{ asset('css/app.css') }}" >
</head>
<body>

    <div class="container my-5">
        <form action="" autocomplete="off">
            <div class="form-group">
                <label for="first_name">Primeiro Nome</label>
                <input type="text" name="first_name" id="first_name" class="form-control" value="Inutil">
            </div>
            <div class="form-group">
                <label for="last_name">Segundo Nome</label>
                <input type="text" name="last_name" id="last_name" class="form-control" value="da Silva">
            </div>
            <div class="form-group">
                <label for="email">E-mail</label>
                <input type="email" name="email" id="email" class="form-control" value="inutil@email.com">
            </div>
            <button class="btn btn-primary">Enviar</button>
        </form>
    </div>


<script src="{{ asset('js/app.js') }}"></script>
</body>
</html>
```

Laravel permite por padrão o uso dos principais verbos HTTP

* Route::get($uri, $callback); 
* Route::post($uri, $callback);
* Route::put($uri, $callback);
* Route::patch($uri, $callback);
* Route::delete($uri, $callback);
* Route::options($uri, $callback);

### Criar um controlador

Forma mais simples
> php artisan make:controller UserController

# Passo a passo 

> Definir uma rota -> Criar um controlador -> Criação do método -> Camada de View

Importante

* adicionar no formulário @csrf
mecanismo de proteção do próprio laravel

Passando no formulário o verbo
> @method('PUT')

Outro mecanismo muito importante no laravel é a listagem de rotas
> php artisan route:list

Criando um controle com os métodos já definidos
> php artisan make:controller PostController --resource

Para funcionar devo colocar o resource no controlador (neste caso web.php)
> Route::resource('posts', 'PostController');

Posso também limitar os verbos no resource
> Route::resource('posts', 'PostController')->only(['index', 'show']);

Posso fazer o inverso, remover apenas o que eu quero
> Route::resource('posts', 'PostController')->except(['index', 'show']);
é possível ver a diferença usando `php artisan route:list`

Essa funcionalidade também está disponível na API.
> php artisan make:controller API\\UserController --api


Informando os verbos que iremos padronizar no projeto. Isso não mudará nossas rotas, mas a URI ficará padronizada e em português(web.php)
```
Route::resourceVerbs([
    'create' => 'cadastro',
    'edit' => 'editar'
]);
```

## Outras formas de trabalhar com rotas

Usando funções anonimas(closures)
```
Route::get('/users', function () {
    echo "Listagem dos usuários";
});

```
Closure não permite fazer cache.

Usando o método view. Usa o get
```
Route::view('/form', 'form');

```
Método para tratamento de error (Erro 404)
```
Route::fallback(function () {
    echo "<h1>Ooopss!!! Seja bem vindo a nossa página 404.<h1>";
});

```

Fazer redirect
```
Route::redirect('/users/add', url('/form'), 301);
```

### Parametrizando as rotas
Para parametrizar uma rota é necessário apenas colocar "bigodinhos"({}) nela
```
Route::get('/users/{id}/comments/{comment}', function ($id, $comment) {
    var_dump($id, $comment);
});

```
os bigodinhos tornam obrigatórios, mas colocando um sinal de "?" torna opcional
```
Route::get('/users/{id}/comments/{comment?}', function ($id, $comment = null) {
    var_dump($id, $comment);
});
```

Podemos usar validação usando regex
```
Route::get('/users/{id}/comments/{comment?}', function ($id, $comment = null) {
    var_dump($id, $comment);
})->where(['id' => '[0-9]+', 'comment' => '[a-z0-9]+']);
```
### Inspecionando Rotas

Temos três métodos que nos ajuda a saber

```
Route::get('/users/1', function () {
    $route = Route::current();
    $name = Route::currentRouteName();
    $action = Route::currentRouteAction();

    var_dump($route, $name, $action);
})->name('inspect');
```

1 - A rota que estou
> $route = Route::current();

2 - O nome da rota
> $name = Route::currentRouteName();

3 - A ação que precisa de um controlador e método responsável pela rota

web.app
> Route::get('/users/1', 'UserController@inspect')->name('inspect');

Usercontroller
```
public function inspect()
    {
        $route = Route::current();
        $name = Route::currentRouteName();
        $action = Route::currentRouteAction();

        var_dump($route, $name, $action);
    }
```
Nesse caso devemos setar a `use Illuminate\Support\Facades\Route;`

### Trabalhando com prefix

Permite agrupar as rotas

```
Route::prefix('admin')->group(function () {
    Route::view('/form', 'form');
});

```
O acesso é feito pela url/admin/form

Podemos agregar todas agrupadas com as areas e facilita a alteração rapidamente.

Agrupando pelo método name

```
Route::name('posts.')->group(function () {
    Route::get('/admin/posts/index', 'PostController@index')->name('index');
    Route::get('/admin/posts', 'PostController@show')->name('show');
});
```
Agrupamento por middleware

```
Route::middleware(['throttle:10, 1'])->group(function () {
    Route::view('/form', 'form');
});
```

Agrupando por namespace
```
Route::namespace('Admin')->group(function () {
    Route::get('/users', 'UserController@index');
});
```

Agrupando Tudo
```
Route::group(['namespace'=> 'Admin','prefix' => 'admin', 'middleware' => ['throttle:10, 1']], function () {
    Route::resource('users', 'UserController');
});
```

### Route Caching
Deixa a aplicação mais rápida. No entanto, não pode ter nenhuma Clousure

> php artisan route:cache

Se por acaso alterar alguma rota da aplicação será necessário fazer novamente o cacheamento.

Para limpar o cache
> php artisan route:clear


##################

# Trabalhando com migrations

Importante configurar o acesso ao banco de dado. Isso deve ser feito no arquivo ".env"

As migrações permite o versionamento do seu banco de dados

> php artisan make:migration create_posts


Criando uma coluna para uma tabela já existente
> php artisan make:migration add_column_posts_title --table=posts

Criando um modelo com as migrations
> php artisan make:model Post -m

Para Rodar a migration criadas
> php artisan migrate

Ps: Se tivermos um erro de ... max key length is 787 bytes

Erro devido ao uso do utf8mb4. Alterar AppServiceProvider:

```
use Illuminate\Support\Facades\Schema;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Schema::defaultStringLength(191);
}
```

Comando para adicionar uma coluna numa nova migrations
>php artisan make:migration add_column_post_slug --table=posts

Código no up
> $table->string('slug')->unique()->after('title');

Criando uma chave estrangeira

Precisamos criar uma nova tabela. Para facilitar podemos rodar um comando que já criar um controlador, model e as migrations.
> php artisan make:model categories -rm 

Criando um usuário no banco de dados e pegar os dados do arquivo .env.

1 - Criamos uma migration
> php artisan make:migration insert_user_admin

2 - Vamos criar as constantes no arquivo .env
```
ADMIN_EMAIL=admin@inutil.com
ADMIN_PASSWORD=admin
```

*Lembrando de criar no arquivo .env.example também*

Migration criada:
```
public function up()
    {
        $email = env('ADMIN_EMAIL', 'admin@inutil.com');
        $password = env('ADMIN_PASSWORD', 'admin');
        DB::table('users')->insert([
            'name' => 'admin',
            'email' => $email,
            'password' => $password
        ]);
    }
```

down da migração
```
public function down()
    {
        $email = env('ADMIN_EMAIL', 'admin@inutil.com');
        DB::delete('DELETE FROM users WHERE email = ?', [$email]);
    }
```

Vamos criptografar o password
> $password = bcrypt(env('ADMIN_PASSWORD', 'admin'));


### Controle de migração

Excluindo a última migração
> php artisan migrate:rollback --step=1

Desfazer todas as migrations de uma única vez
> php artisan migrate:rollback

Fazer em ordem todos os métodos down() e depois todos os métodos up()
> php artisan migrate:refresh

Equivale a fazer a deleção das tabelas e refazer todas as migracões
> php artisan migrate:fresh


### Seeding - criando dados fake para testar a aplicação

Criar uma seed
> php artisan make:seeder UsersTableSeeder

Exemplo:
```
 DB::table('users')->insert([
        'name' => str_random(10),
        'email' => str_random(10) . '@inutil.com',
        'password' => bcrypt('demo')
    ]);
```
No entanto, é preciso fazer referência ao arquivo 'DatabaseSeeders.php', pois ele é quem controla
>$this->call(UsersTableSeeder::class);

Para rodar basta o comando no terminal
> php artisan db:seed

Caso tenha dado algum erro:
> composer dump-autoload -o

Rodando uma seeder específica, ou sem precisar sem ter que chamar pelo DatabaseSeeder
> php artisan db:seed --class=UsersTableSeeder

Para rodar as migrations com a seeders
> php artisan migrate::refresh --seed

### Usando factoreis para criar com dados mais reais as seeds

Criando uma factory
> php artisan make:factory PostFactory

----
ORM 
Como referenciar outra table no controlador

```
class Post extends Model
{
    protected $table = "posts";
    protected $primaryKey = "id";

    protected $timestamps = true;
    // public const CREATED_AT = "creation_date";
    // public const UPDATED_AT = "last_update";

}
```

Para resolver erros de arquivos externos adicionado a estrutura do Laravel, execute o comando:
> composer dump-autoload

Para executar as seedes neste caso
> php artisan db:seed --class=PostTableSeeder


Possibilidades

- take - limita o número de registro exibido
- orderBy - ordenação por campo e por ordem
- first - o resultado é um array
- firstOrFail - lança um erro e joga para página 404 se dê erro
- find - usa sempre a chave primária para localizar os registros
- findOrFail - enviar para a página 404 caso não encontre o registro
- exists - retorna true ou false caso o arquivo exista ou não
- exists só podem ser utilizadas em coleções (get por exemplo)
- exists - retorna true ou false caso o arquivo exista ou não
- count - conta a quantidade
- max - o maior
- min - o menor
- sum - a soma
- avg - a média
- all - todos os registros

Salvando dados com firstOrNew
```
/**
 * Procura o primeiro registro e se não achar junto com os dados da condição(segundo array)
*/
$post = Post::firstOrNew([
   'title' => 'teste2'
], [
    'subtitle' => 'teste2',
    'description' => 'teste2'
]);
$post->save();
```

Salvando dados com firstOrCreate
Não é necessário usar a opção save
```
/**
 * Criar por padrão um novo registro se não existir
 */
$post = Post::firstOrCreate([
    'title' => 'teste4'
], [
    'subtitle' => 'teste4',
    'description' => 'teste4'
]);
```

Atualizando um registro
```
/**
         * Opção 1 Deixando o próprio modelo sobrescrever
         */
        $post = new Post; - Omitir para não salvar como novo
        $post->title = $request->title;
        $post->subtitle = $request->subtitle;
        $post->description = $request->description;
        $post->save();
        var_dump($request, $post);

        /**
         * Opção 2 - usando o método find
         */
        $post = Post::find($post->id);
        $post->title = $request->title;
        $post->subtitle = $request->subtitle;
        $post->description = $request->description;
        $post->save();

        /**
         * Opção 2 - usando o método updateOrCreate
         */
        $post = Post::updateOrCreate([
            'title' => 'title5'
        ], [
            'subtitle' => 'teste6',
            'description' => 'teste6'
        ]);

        /**
         * Atualização em massa
         */
        Post::where('created_at', '>=', date('Y-m-d H:i:s'))->update([
            'description' => 'teste'
        ]);

```

Exclusão de registro
```
/**
         * find com delete
         */
        Post::find($post->id)->delete();

        /**
         * usando o destroy, passando um array de itens para deleção
         */
        Post::destroy([2, 3]);

        /**
         * maneira mais simples de excluir
         */
        Post::destroy($post->id);

        /**
         * Exclusão em massa
         */
        Post::where('created_at', '>=', date('Y-m-d H:i:s'))->delete();

```

Software Delete
Marcação do dado como excluido

1 - Configurar o modelo para aceitar
```
use Illuminate\Database\Eloquent\SoftDeletes;

use SoftDeletes;
```
2 - no terminal
> php artisan make:migration add_soft_delete_posts --table=posts

3 - Código da migration
```
Schema::table('posts', function (Blueprint $table) {
            $table->softDeletes();
        });
```

Rodar novamente as migrations
> php artisan migrate:fresh

Rodando as seeds
> php artisan db:seed --class=PostTableSeeder

A partir de agora o campo deleted_at fica preenchido.
Showwww


Para trabalhar com o excluidos

```

```

---
Para projetos em que os arquivos externos foram adicionados, devemos primeiro fazer com que o laravel saiba da existência desses arquivos.
Primeiro rodamos o comando
> composer dump-autoload

O composer ler novamente todos os arquivos que estão no nosso projeto e coloca no autoload as novas classes.
> php artisan migrate:fresh --seed

---
Criando um controle com todos os métodos resource
> php artisan make:controller UserController -r

Criar o relacionamento 1 para 1 do usuário com endereço.

No model User
```
/**
     * Relacionamento 1 para 1 entre usuário e endereço
     */
    public function addressDelivery()
    {
        // possibilidade 1
        // return $this->hasOne('App\Address');

        // possibilidade 2 - mais fácil para ide trabalhar
        //classe, chave_estrangeira, chaveprimaria
        return $this->hasOne(Address::class, 'user', 'id');
    }
```

Caminho inverso para Address to User
```
/**
     * fazendo o caminho inverso do usuário para endereço
     * belongsTo(Classe, ChaveEstrangeira, chavedoRelacinamento)
     */
    public function user()
    {
        return $this->belongsTo(User::class, 'user', 'id');
    }
```

É possível usar o método para salvar um novo registro.
Salvando um novo endereço no usuário
```
 /**
     * Vinculação automática do endereço ao usuário 
     */
    $address = new Address([
        'address' => 'Rua dos bobos',
        'number' => '0',
        'complement' => 'Apto 123',
        'zipcode' => '12345-123',
        'city' => 'João Pessoa',
        'state' => 'PB'
    ]);

    $user->addressDelivery()->save($address);

```
Usando método dd para exibir os relacionamento
```
/**
     * Usando o método dd para exibir os relacionamentos
     */
    $users = User::with('addressDelivery')->get();
    dd($users);
```
Maneira muito simples de manipular o modelo

```
/**
     * Inserindo uma categoria
     * Muito show
     */
    $post->categories()->attach([3]);

    /**
     * Remover a categoria
     */
    $post->categories()->detach([3]);

    /**
     * Remover e inserir de uma vez
     * O que não for passado será retirado
     */
    $post->categories()->sync([5,10]);

/**
     * Sincronização sem remoção
     */
    $post->categories()->syncWithoutDetaching([5, 6, 7]);
```
Relacionamento polimórfico

Precisamos colocar na migration da tabela
> $table->morphs('item');

Precisamos configurar no modelo 

```
public function comments()
    {
        return $this->morphMany(Comment::class, 'item');
    }
```
Trabalhando com Scopo

Útil para fazer verificação automática do nível do usuário

No Model, é obrigatório o uso da palavra reservada "scope" no nome do método:
```
public function scopeStudents($query)
    {
        return $query->where('level', '<=', 5);
    }

    public function scopeAdmins($query)
    {
        return $query->where('level', '>', 5);
    }
```

No controller:
```
$students = User::students()->get();
    if ($students) {
        echo "<h1>Alunos</h1>";

        foreach ($students as $student) {
            echo "Nome do Usuário: {$student->name}<br>";
            echo "Email: {$student->email}<br><hr>";
        }
    }

    $admins = User::admins()->get();
    if ($admins) {
        echo "<h1>Administradores</h1>";

        foreach ($admins as $admin) {
            echo "Nome do Usuário: {$admin->name}<br>";
            echo "Email: {$admin->email}<br><hr>";
        }
    }
```

Mutator e Accessor
Utilizado para modificar a exibição dos dados

No controller é obrigatório a chamada constando *getNomeReferenciadoAttribute* :
```
public function getCreatedFmtAttribute()
    {
        return date('d/m/Y H:i', strtotime($this->created_at));
    }
```
Podemos formatar nomes e outros atributos no model sem precisar tá formatando no view

setAttribute
Model:
```
public function setTitleAttribute($value)
    {
        $this->attributes['title'] = $value;
        $this->attributes['slug'] = str_slug($value);
    }
```

Controller
```
/**
 * criando um título pelo setAttribute
 */
    post->title = 'Título de Teste do meu artigo!';
    $post->save();
```

Serializando os dados para trabalhar com API

- ToArray
- ToJson

```
/**
     * Serializando os dados
     */
    $users = User::all();
    var_dump($users->toArray());
    var_dump($users->toJson(JSON_PRETTY_PRINT));
 ```
 
 Para ocultar
 ```/**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
```

Para permitir um método

Criamos um método no controler
```
public function getAdminAttribute()
    {
        return ($this->attributes['level'] > 5 ? true : false);
    }
```
Adiciono no appends
> protected $appends = ['admin'];

Adiciono no $visible
> protected $visible = ['name', 'email', 'admin'];

Permitindo que campos que preciso para situações específicas
```
$users = User::all();
    var_dump($users->makeVisible('created_at')->toArray());
    var_dump($users->makeHidden('created_at')->toJson(JSON_PRETTY_PRINT));

```

# Trabalhando com Query Builder

Lembrando que precisamos fazer com que o laravel reconheça os arquivos que foram adicionados externamentes(factores, migrations e seeds)

> composer dump-autoload

Rodando as migrates

> php artisan migrate --seed

Para corrigir o erro vamos criar o model user

> php artisan make:model Address

Rodando novamente as migrations. Dropando todas as tabelas e criando novmaente

> php artisan migrate:fresh --seed

Precisamos criar um controler para trabalhar com os dados. Não precisamos criar as rotas... nos criaremos manualmente.

> php artisan make:controller UserController

```
$users = DB::table('users')
        ->select('users.*')
        ->select('users.id', 'users.name', 'users.status')
        ->where('users.status', '=', '1')
        ->orderBy('users.name', 'DESC')
        // se eu não definir ele pega a coluna de criação
        ->oldest('users.name') //ASC
        ->latest('name') //DESC
        ->inRandomOrder() //sempre que atualiza ele altera a ordem
        ->limit(10)
        ->offset(10)
        ->get();

        foreach ($users as $user) {
            echo "#{$user->id} Nome: {$user->name} {$user->status}<br>";
        }

```
Consulta RAW

```
/**
     * Tratando os dados que vem do banco de dados, via método selectRaw.
     * Lembrando que esse método não é tratado pelo construtor
     */
    // $users = DB::table('users')
    //             ->selectRaw('
    //                 users.id,
    //                 users.name,
    //                 CASE WHEN users.status = 1 THEN "ATIVO" ELSE "INATIVO" END status_description')
    //             ->whereRaw('(SELECT count(1) FROM addresses a WHERE a.user = users.id) > 2')
    //             ->whereRaw('users.status = 1')
    //             ->orderByRaw('updated_at - created_at', 'ASC')
    //             ->get();

        /**
         * Forma Raw de fazer a consulta anterior
         * Com passagem de parâmetros
         */
        $users = DB::select(DB::raw('SELECT users.id,
                            users.name,
                            CASE
                                WHEN users.status = 1 THEN "ATIVO"
                                ELSE "INATIVO"
                            END status_description
                        FROM users
                        where (SELECT count(1) FROM addresses a WHERE a.user = users.id) > 2
                            AND users.status = :userStatus
                        ORDER BY updated_at - created_at ASC'), ['userStatus' => '1']);


        foreach ($users as $user) {
            echo "#{$user->id} Nome: {$user->name} {$user->status_description}<br>";
        }
```

### Trabalhando com o Chunk
Quando temos muito dados para processar ou quando ultrapassa os 30 segundos. Para que a aplicação não parar com esse erro.

```
/**
     * Trabalhando com chunk
     */

    DB::table('users')->where('id', '<', '500')->chunkById('50', function ($users) {
        foreach ($users as $user) {
            echo "#{$user->id} Nome: {$user->name} {$user->status}<br>";
        }

        echo "<p>ENCERROU UM CICLO</p>";
        sleep(1);
    });
```
outra forma de usar o chunk. Que dará o mesmo resultado

```
DB::table('users')->where('id', '<', '500')->orderBy('id')->chunk('50', function ($users) {
        foreach ($users as $user) {
            echo "#{$user->id} Nome: {$user->name} {$user->status}<br>";
        }

        echo "<p>ENCERROU UM CICLO</p>";
        sleep(1);
    });
       
```

Trabalhando com os parâmetros parse strings

Filtrar os registros com a cláusula where
```
/**
         * parse string para filtar os registros
         */
        $users = DB::table('users')
                // ->whereIn('users.status', [0,1])
                // ->whereNotIn('users.status', [0,1])
                // ->whereNull()
                // ->whereNotNull('users.name')
                // ->whereColumn('created_at', '=', 'updated_at') //compara duas colunas
                // ->whereDate('created_at', '>', '2020-09-08 22:00:01')
                // ->whereDay('created_at', '=', '09')
                // ->whereMonth('created_at', '=', '01')
                ->whereYear('created_at', '=', '2021')
                ->whereTime('created_at', '>', '19:11:00')
                ->get();

        foreach ($users as $user) {
            echo "#{$user->id} Nome: {$user->name} {$user->status}<br>";
        }
```

Trabalhando com join


```
/**
         * Trabalhando com Join
         * join, leftJoin, rigthJoin
         */

        $users = DB::table('users')
                ->select('users.id', 'users.name', 'users.status', 'addresses.address')
                // ->leftJoin('addresses', 'users.id', '=', 'addresses.user')
                ->join('addresses', function ($join) {
                    $join->on('users.id', '=', 'addresses.user')
                         ->where('addresses.status', '=', '1');
                })
                ->orderBy('users.id', 'ASC')
                ->get();


        echo "Total de Registros: {$users->count()}<br><br>";

        foreach ($users as $user) {
            echo "#{$user->id} Nome: {$user->name} {$user->status} - {$user->address}<br>";
        }
```

Cadastro, atualização e deleção de registros

Criando

```
//criando
        DB::table('users')->insert([
            'name' => 'Jaqueline Fernandes',
            'email' => 'inutil@email.com',
            'password' => bcrypt('123456'),
            'status' => '1'
        ]);
```

Atualizando

```
//atualizando
        DB::table('users')->where('id', '=', '1001')->update([
            'email' => 'inutilupdate@email.com'
        ]);
```

Excluindo

```
//exclusão
        DB::table('users')->where('id', '=', '1001')->delete();
```

Paginação dos resultados

```
/**
     * Paginação
     * método simplePaginate exibe de maneira mais compactar e o método paginate com a paginação
     * e para termos os links usamos links()
     */
    // $users = DB::table('users')->simplePaginate(50);
    $users = DB::table('users')->paginate(50);

    foreach ($users as $user) {
        echo "#{$user->id} Nome: {$user->name} {$user->status}<br>";
    }

    echo $users->links();
```

É possível resgatar os parâmetros. Mais informações na documentação oficial
[Laravel Documentation Pagination](https://laravel.com/docs/5.7/pagination) 


### Serviços essenciais do dia a dia


Gerando log. O arquivo

As opção do log está na config > logging.php.
No arquivo .env temos a definição
```
/**
 * Lançando uma mensagem de log
 */
Route::get('/log', function () {
    //envio para uma pilha
    Log::stack(['telegram', 'stack'])->info('log');

    //envio individual
    // Log::channel('telegram')->info('Test caption 🏠');
});
```
Trabalhanco com Sessão

- Facede (Illuminate\Support\Facades\Session)
- helper session

Salvando os dados
```
session([
        'course' => 'Laradev'
    ]);

session()->put('name', 'jaqueline');

```

Resgatando dados da sessão

```
 // resgatando
echo session('name');

// valor padrão
echo session('studante', 'Valor padrão se não encontrar');

echo session()->get('name');
```


Salvando se não existir
```
echo session('student', function () {
	session()->put('student', 'jaqueline');
	return session('student');
});
```

Exclusão

```
$student = session()->pull('student');
    echo $student;

    //sem retorno
    session()->forget('name');

    //deleta tudo da sessão
    session()->flush();
```

Verificando a existencia de uma key na sessao

```
    //se não for null
    // if (session()->has('course')) {
    //     echo "O curso selecionado foi ". session()->get('course');
    // } else {
    //     echo "Esse curso não existe";
    // }

    // //se apenas existir
    // if (session()->exists('student')) {
    //     echo "Esse Indice existe";
    // } else {
    //     echo "Esse Indice não existe";
    // }
```


Trabalhar com Redis

Necessário instalar o library externa
> composer require predis/predis

Configurar a sessão para salvar no redis e pronto!


Trabalhar com a sessao no banco de dados
Configurar a session_driver para database

Criando as migrations
> php artisan session:table

Para criar
> php artisan migration


Envio de email
criando um novo objeto
> php artisan make:mail welcomeLaraDev

Devemos configurar o arquivo *.env* e adicionar os que faltam e constam no config>mail.php
```
MAIL_DRIVER=smtp
MAIL_HOST=smtp
MAIL_PORT=2121
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=
MAIL_FROM_NAME=

```

Fazendo um teste inicial
```
    public function build()
    {
        $this->subject("Seja muito bem vindo ao Laravel Developer");
        return $this->view('mail.welcomeLaraDev');
    }
```

Usando o markdown para formatar o email

Na classe do email
```
$this->subject("Seja muito bem vindo ao Laravel Developer");
        $this->to($this->user->email, $this->user->name);

        return $this->markdown('mail.welcomeLaraDev')->with([
            'user' => $this->user,
            'order' => $this->order
        ]);
    }
```

No web.php

```
$user = new stdClass();
    $user->name = "Jaqueline Developer";
    $user->email = "dhelly@gmail.com";

    $order = new stdClass();
    $order->type = "billet";
    $order->due_at = "2019-01-10";
    $order->value = 678;

    return new welcomeLaraDev($user, $order);
```
Na view welcomeLaraDev.blade.php
```
@component('mail::message')
<h1>Parabéns por garantir sua vaga na laradev</h1>
<p>
    Para fazer login na plataforma utilize seu email ({{ $user->email }})
    juntamente com sua senha de cadastro.
</p>
<p>
    Para garantir a sua vaga, vote tem até o dia {{ date('d/m/y', strtotime($order->due_at)) }} para
    conseguir o seu desconto e pagar somente R${{ number_format($order->value, 2, ',', '.') }} e ter acesso completo ao
    ao nosso conteúdo.
</p>
@endcomponent
```


Podemos adicionar ao projeto todos os assets para facilitar diretamente no laravel
> php artisan vendor:publish --tag=laravel-mail


Rodando via lista de jobs

> php artisan queue:table

Para rodar a migrate
> php artisan migrate

vamos tirar a carga do cliente e jogar para a tabela auxiliar

1 - duplicamos a rota de envio
2 - Removemos o método enviar(send) por queue
3 - Alterar a propiedade QUEUE_CONNECTION=database
4 - Ativar o serviço de database queue do laravel
> php artisan queue:work 

Há a possiblidade de enviar via *send* para a fila
Para isso precisamos alterar a assinatura da classe *welcomeLaraDev.php* e inserirmos:
> ShouldQueue
```
class welcomeLaraDev extends Mailable implements ShouldQueue
```
Se acontecer erro no envio da fila ela será executada novamente e precisamos determinar a quantidade de vezes. Para isso Criamos um Job

> php artisan make:job welcomeLaraDev

Um novo diretório foi criado. Vamos passar a responsabilidade para o processamento  para ese job.

*welcomeLaraDev.php - Job*
```
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Support\Facades\Mail as SendMail;

class welcomeLaraDev implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $tries = 1;

    public $timeout = 20;

    private $user;
    private $order;

    /**
     * Create a new job instance.
     *
     * @return void
     */
    public function __construct(\stdClass $user, \stdClass $order)
    {

        $this->user = $user;
        $this->order = $order;
    }

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        SendMail::send(new \App\Mail\welcomeLaraDev($this->user, $this->order));
    }
```
*web.php - Route*

```
Route::get('/mail-queue-job', function () {

    /**
     * Teste de envio de email para fila via job
     */

    $user = new stdClass();
    $user->name = "Jaqueline Developer";
    $user->email = "dhelly@gmail.com";

    $order = new stdClass();
    $order->type = "billet";
    $order->due_at = "2019-01-10";
    $order->value = 678;

    // Mail::queue(new welcomeLaraDev($user, $order));
    //removemos a chamada do método e passamos para o job
    \App\Jobs\welcomeLaraDev::dispatch($user, $order)->delay(now()->addSecond(15));
});
```
*welcomeLaraDev.php - Mail*
```
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;

class welcomeLaraDev extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    private $user;
    private $order;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct(\stdClass $user, \stdClass $order)
    {
        $this->user = $user;
        $this->order = $order;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        /**
         * envio de email simples
         */
        // $this->subject("Seja muito bem vindo ao Laravel Developer");
        // $this->to($this->user->email, $this->user->name);
        // return $this->view('mail.welcomeLaraDev')->with([
        //     'user' => $this->user,
        //     'order' => $this->order
        // ]);

        /**
         * Envio de email com markdown
         */
        $this->subject("Seja muito bem vindo ao Laravel Developer");
        $this->to($this->user->email, $this->user->name);

        return $this->markdown('mail.welcomeLaraDev')->with([
            'user' => $this->user,
            'order' => $this->order
        ]);
    }
}
```


Criando a table responsável por salvar os jobs errados
> php artisan queue:failed-table

> php artisan migrate

---
Gerenciando os arquivos no Laravel

Arquivo de configuração respansável(config)
> filesystems.php

Importante precisamos criar um atalho para a pasta  *storage*

```
Route::get('/files', function () {
    //apenas os arquivos do diretório atual
    $files = Storage::files('');

    //busca recursivamente
    $allFiles = Storage::allFiles('');

    //busca no diretório sem especificar pega o root
    $directories = Storage::directories('');

    //todos os diretório recursivamente
    $allDirectories = Storage::allDirectories('');

    //criando um diretório
    // Storage::makeDirectory('public/course');

    //excluir diretório
    // Storage::deleteDirectory('public/course');

    //criando um arquivo especificando o disco 'public'
    // Storage::disk('public')->put('teste.txt', 'O melhor curso de laravel do mercado');

    //criando um arquivo especificando o disco 'app' que é o padrão
    // Storage::put('upinside.txt', 'O melhor curso de laravel do mercado');

    //resgatando um arquivo
    // echo Storage::get('upsinde.txt');

    //baixar um arquivo do servidor
    //necessário usar o return
    // return Storage::download('upsinde.txt');

    //usando o método exists
    // if (Storage::exists('upsinde1.txt')) {
    //     echo "Arquivo existe!";
    // } else {
    //     echo "Arquivo não existe";
    // }

    //validando arquivos pelo tamanho
    // echo Storage::size('upsinde.txt');

    //validando arquivos pelo tamanho
    // echo date('d/m/Y H:i:s', Storage::lastModified('upsinde.txt')) ;

    //manipulando o arquivo txt - adicionando no início
    // Storage::prepend('upsinde.txt', 'Inutil Treinamentos');

    //manipulando o arquivo txt - adicionando no final
    // Storage::append('upsinde.txt', 'Vem estudar com a gente');

    //fazendo uma copia do arquivo
    // Storage::copy('upsinde.txt', 'upinside.txt');

    //movendo o arquivo
    // Storage::move('upsinde.txt', 'public/upinside.txt');

    //deletando um arquivo
    Storage::delete('public/teste.txt');

    // var_dump($files, $allFiles, $directories, $allDirectories);


});
```

Trabalhando com formulário

Criar o resource
> php artisan make:model Property -rm 

Estamos criando o model, o controlador com o resourse e a migration

Migration
```
Schema::create('properties', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title');
            $table->double('rental_price');
            $table->string('cover');
            $table->timestamps();
        });
```
Rodar a migration
> php artisan migrate

Criamos a rota
```
Route::resource('/imoveis', 'PropertyController');
```
Para verificar se as rotas estão funcionando
> php artisan route:list

Formulário de teste *create.blade.php* que ficará na pasta *properties*
```
<form action="{{ route('imoveis.store') }}" method="POST" enctype="multipart/form-data">
@csrf

<label for="title">Título do imóvel</label>
<input type="text" name="title">

<br>

<label for="rental_price">Preço de Locação</label>
<input type="text" name="rental_price">

<br>

<label for="cover">Imagem de Destaque</label>
<input type="file" name="cover">

<br>

<button type="submit">Gravar Imóvel</button>

</form>
```

Método store do PropertyController

```
        $property = new Property();
        $property->title = $request->title;
        $property->rental_price = $request->rental_price;
        $property->cover = $request->file('cover')->store('public/properties');
        $property->save();

```

No entanto, temos um problema: O arquivo fica salvo na pasta storage, que nossa aplicação não tem acesso. Para resolver executamos o seguinte comando:
> php artisan storage:link 

Criamos um link simbólico
Precisamos adiconar no arquivo *.env* 
>FILESYSTEM_DRIVER=public

Importante: se não estiver usando o servidor do laravel e usando o apache, precisamos mudar no arquivo *.env*
>APP_URL=http://localhost/curso_laradev/servico_essencial/public/

Atualizando o store para visualizar a imagem
```
$property = new Property();
        $property->title = $request->title;
        $property->rental_price = $request->rental_price;
        $property->cover = $request->file('cover')->store('properties');
        $property->save();
        
        //métodos que posso usar para validar
        var_dump(
            $request->file('cover')->getMimeType(),
            $request->file('cover')->getClientOriginalName(),
            $request->file('cover')->getClientOriginalExtension(),
            $request->file('cover')->getExtension(),
            $request->file('cover')->getSize(),
            $request->file('cover')->isValid()
        );
        
        echo "<img src='". Storage::url($property->cover) . "'>";
        
        //varias formas de acessar o arquivo
        // var_dump($request->cover, $request->file('cover'), $request->allFiles());
```
---
### Trabalhando com Middleware

Intercepta as requisitos ou trata as respostas dessas requisições.
Devemos ter cuidado pois pode gerar muito overhead na aplicação

Criando um middleware
> php artisan make:middleware checkParam

Criando um método simples no *checkParam.php*
```
public function handle($request, Closure $next)
    {
        Log::info('Foi invocado meu primeiro middleware!');
        return $next($request);
    }
```
Parametriznado o *kernel.php* 

- para que TODA requisição passe pelo nosso checkParam. Adicionar a classe no **protected $middleware**.
- para group, que atribui para grupos
- por apelido para o middleware
- podemos determinar a prioridade

Vamos testar pelo apelido
```
'testeMiddleware' => \App\Http\Middleware\checkParam::class
```
Vamos aplicar diretamente na nossa rota

```
// Route::get('/teste-middleware', 'PropertyController@middle')
        // ->middleware(\App\Http\Middleware\checkParam::class);
Route::get('/teste-middleware', 'PropertyController@middle')->middleware('testemiddleware');
```
Nossa função
```
public function middle()
    {
        echo "Seja bem vindo ao nosso teste de middleware!";
    }
```
Para passar mais informações pela rota
> Route::get('/teste-middleware', 'PropertyController@middle')->middleware('testemiddleware:admin,paid');

*checkParam.php*
```
public function handle($request, Closure $next, $param, $param_2)
    {
        //user logado
        //atributo que armazenda
        //if (att == param) ? next($request) : redirect(login)
        Log::info('Foi invocado meu primeiro middleware! ['. $param .' - '. $param_2.']');
        Log::info('TESTE 1');

        $process = $next($request);

        Log::info('TESTE 3');
        return $process;
    }
```
## Compreendendo o blade
- Os arquivos devem possuir a extensão **.blade.php**
- As pastas devem ser referenciada como '.' na chamada

Passagem de parâmetros
```
    $user = new stdClass();
    $user->name = 'Aluno Inutil';
    $user->birth = '1980-12-31';

    /**
     * Passagem de parâmetro
     */
    // return view('front.home', [
    //     'user' => $user
    //     ]);

    /**
     * Outra forma de passar parâmetro
     */
    // return view('front.home')->with([
    //     'user' => $user
    // ]);

    /**
     * Mais uma outra forma de passar parâmetro
     */
    // return view('front.home')->with('user', $user)->with('tutor', 'Prof Inutil');

    /**
     * Forma compacta que usar o próprio nome da variável como indice
     */
    return view('front.home', compact('user'));
```
O blade possui helps que ajudam
```
{{ $user->name }}
```
Basicamente o blade por ser uma template engine converte nosso código para uma estrutura php antes de enviar para o cliente. Isso é fácil de ser verificado indo na pasta *storage>framework>views*. Lá fica salvo as versões convertidas de nossas views

- O blade no uso d o `{{ conteudo }}` faz o escape dos caracteres

Para permitir que seja exibido usamos `{!! conteudo !!}`

Exemplo 
```
 $alert = "<div style='background-color:red;'>Teste</div>";

    /**
     * Passagem de parâmetro
     */
    return view('front.home', [
        'user' => $user,
        'alert' => $alert
        ]);
```
Exemplo do blade
```
Meu nome é: {{ $user->name }}
<br>
{{ $alert }}
<br>
{!! $alert !!}

```

Comenátrio em php no blade. Não é exibido no browser
```
{{-- Comentário --}}
```
Exemplos:
```
{{-- Por padrão temos o escape --}}
Meu nome é: {{ $user->name }}
<br>
{{ $alert }}
<br>

{{-- Exibindo o código html --}}
{!! $alert !!}

{{-- Entendendo os help --}}

{{-- Como não fazer --}}
<?php
echo "<h1>Sintaxe PHP</h1>";

if (!empty($user->email)){
    echo "<p>[IF] O usuário não informou o email</p>";
} elseif ($user) {
    echo "<p>[ELSEIF] Existe o objeto usuário</p>";
} else {
    echo "<p>[ELSEIF] NÃO existe objeto usuário</p>";
}

echo "<p>[TERNÁRIA]". (!empty($user->email) ? 'O usuário não informou o email' : ' Existe o objeto usuário') ."</p>";
echo "<p>[TERNÁRIA DUPLA]". (!empty($user->email) ? 'O usuário não informou o email' : ($user ? ' Existe o objeto usuário' : ' NÃO existe objeto usuário')) ."</p>";

if(isset($user)){
    echo "<p> [ISSET] Existe um usuário</p>";
}

if(empty(!$user)){
    echo "<p> [empty] Usuário vazio</p>";
}

$var = '1';
switch ($var) {
    case '1':
        echo "<p>[CASE] 1</p>";
    break;

    case '2':
    echo "<p>[CASE] 2</p>";
    break;

    default:
        echo "<p>[CASE] default</p>";
        break;
}
?>

{{-- Como fazer --}}
<h1>Sintaxe do Blade</h1>
@if (!empty($user->email))
    <p>[IF] O usuário não informou o email</p>
@elseif ($user)
    <p>[ELSEIF] Existe o objeto usuário</p>
@else
    <p>[ELSEIF] NÃO existe objeto usuário</p>
@endif

<p>[TERNÁRIA] {{ (!empty($user->email) ? 'O usuário não informou o email' : ' Existe o objeto usuário') }}</p>
<p>[TERNÁRIA DUPLA] {{ (!empty($user->email) ? 'O usuário não informou o email' :
($user ? ' Existe o objeto usuário' : ' NÃO existe objeto usuário')) }}</p>

@isset($user)
    <p>[ISSET] Existe um usuário</p>
@endisset

@empty(!$user)
    <p> [empty] Usuário vazio</p>
@endempty

@switch($var)
    @case(1)
        <p>[CASE] 1</p>
        @break
    @case(2)
        <p>[CASE] 2</p>
        @break
    @default
        <p>[CASE] Default</p>
@endswitch

```
Laços de repetiãço

controller

```
    $user = new stdClass();
    $user->name = 'Aluno Inutil';
    $user->birth = '1980-12-31';

    $courses = [
        [
            "id" => 1,
            "name" => "Laravel Developer",
            "tutor" => "Gustavo Web"
        ],
        [
            "id" => 2,
            "name" => "Bootstrap Builder",
            "tutor" => "Gustavo Web"
        ],
        [
            "id" => 3,
            "name" => "FullStack PHP Developer",
            "tutor" => "Robson V. Leite"
        ]
    ];
    
    return view('front.home', [
        'user' => $user,
        'alert' => $alert,
        'courses' => $courses
        ]);
```


view
```
{{-- Por padrão temos o escape --}}
Meu nome é: {{ $user->name }}
<br>
{{ $alert }}
<br>

{{-- Exibindo o código html --}}
{!! $alert !!}

{{-- Entendendo os help --}}

{{-- Como não fazer --}}
<?php
echo "<h1>Sintaxe PHP</h1>";

if (!empty($user->email)){
    echo "<p>[IF] O usuário não informou o email</p>";
} elseif ($user) {
    echo "<p>[ELSEIF] Existe o objeto usuário</p>";
} else {
    echo "<p>[ELSEIF] NÃO existe objeto usuário</p>";
}

echo "<p>[TERNÁRIA]". (!empty($user->email) ? 'O usuário não informou o email' : ' Existe o objeto usuário') ."</p>";
echo "<p>[TERNÁRIA DUPLA]". (!empty($user->email) ? 'O usuário não informou o email' : ($user ? ' Existe o objeto usuário' : ' NÃO existe objeto usuário')) ."</p>";

if(isset($user)){
    echo "<p> [ISSET] Existe um usuário</p>";
}

if(empty(!$user)){
    echo "<p> [empty] Usuário vazio</p>";
}

$var = '1';
switch ($var) {
    case '1':
        echo "<p>[CASE] 1</p>";
    break;

    case '2':
    echo "<p>[CASE] 2</p>";
    break;

    default:
        echo "<p>[CASE] default</p>";
        break;
}

echo "<h2>Listagem de Cursos</h2>";

for ($i=0; $i < count($courses); $i++) {
    echo "<p> [FOR]". $courses[$i]['name']." - ". $courses[$i]['tutor'] ."</p>";
}
echo "<br>";
foreach ($courses as $course) {
    echo "<p> [FOREACH]". $course['name']." - ". $course['tutor'] ."</p>";
}

?>

{{-- Como fazer --}}
<h1>Sintaxe do Blade</h1>
@if (!empty($user->email))
    <p>[IF] O usuário não informou o email</p>
@elseif ($user)
    <p>[ELSEIF] Existe o objeto usuário</p>
@else
    <p>[ELSEIF] NÃO existe objeto usuário</p>
@endif

<p>[TERNÁRIA] {{ (!empty($user->email) ? 'O usuário não informou o email' : ' Existe o objeto usuário') }}</p>
<p>[TERNÁRIA DUPLA] {{ (!empty($user->email) ? 'O usuário não informou o email' :
($user ? ' Existe o objeto usuário' : ' NÃO existe objeto usuário')) }}</p>

@isset($user)
    <p>[ISSET] Existe um usuário</p>
@endisset

@empty(!$user)
    <p> [empty] Usuário vazio</p>
@endempty

@switch($var)
    @case(1)
        <p>[CASE] 1</p>
        @break
    @case(2)
        <p>[CASE] 2</p>
        @break
    @default
        <p>[CASE] Default</p>
@endswitch

<h2>Listagem de Cursos</h2>

@for ($i=0; $i < count($courses); $i++)
    <p> [FOR] {{ $courses[$i]['name'] }} - {{$courses[$i]['tutor']}}</p>
@endfor
<br>
@foreach ($courses as $course)
    <p style="background-color: {{ $loop->index % 2 === 0 ? 'grey' : 'yellow' }}"> [FOREACH] {{ $course['name'] }} - {{$course['tutor']}}</p>
    @php
        // variável criada pelo foreach
        // var_dump($loop)
    @endphp
@endforeach
```

Trabalhando com a master

1 - Criamos a a página master e inserirmos 

> @yield('content')

```
<!DOCTYPE html>
<html lang="pt_br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>LaraDev - </title>
</head>
<body>

@yield('content')

</body>
</html>
```
2 - Na página que faremos a injeção do conteúdo

No topo para referenciar a master
> @extends('front.master.master')

Para envolver o conteúdo que será inserido nessa dobra
> @session('content')
> @endsession

Podemos criar includes para segmentar melhor e reutilizar nosso código em diferentes páginas

Inserimos o conteúdo dos módulos que se repetem nos includes
> @include('front.includes.header')

Criação de componentes

Componentes são uma forma de modularizar pequenos pedaços de código que serão usados em diferentes lugares de nossa aplicação

1 - Criando uma pasta para os componentes do front

Arquivo *view>front>compoents>alert.blade.php*
```
<div class="alert alert-{{ $type }}" role="alert">
    {{ $slot }} <br>
    <small>Mensagem Gerada em {{ $datetime }}</small>
</div>
```
A variável `$slot` é fornecida pelo blade(confirmar) para injeção dos dados; podemos criar outras variáveis como `$type`;

Chamando o componente
```
<div class="row">
    <div class="col-12">
        @component('front.components.alert', ['type' => 'danger', 'datetime' => date('d/m/Y H:i:s')])
            Mensagem de Teste
        @endcomponent
    </div>
</div>
```
Uma outra solução é criar um alias para nosso componente. Para isso, precisamos criar um no método register do *AppServiceProvider.php*

> Blade::component('front.components.alert', 'alert');

Nós temos um componente `@alert`
```
<div class="row">
    <div class="col-12">
        @alert(['type' => 'info', 'datetime' => date('d/m/Y H:i:s')])
        Essa Mensagem é do alias do componente criando no AppServiceProvide
        @endalert
    </div>
</div>
```
Outra opção interessante é permitir o carregamento de arquivos de *CSS* e *JS* para uma página específica, como por exemplo: carregamento de um gráfico. Podemos inserir um na Master

```
@hasSection ('css')
    @yield('css')
@endif
```
E na página responsável pelo conteúdo

```
@section('css')
    <style>
        .teste {
            color: purple;
        }
    </style>
@endsection
```
### Herdando configurações de blocos com mesmo nome

Vamos dicionar um sider bar na Master
```
<div class="col-4 py-3">
    {{-- @yield('siderbar') --}}
    @section('siderbar')
        <h1>[Master] Últimos postados</h1>
        <ul>
            <li>Lorem ipsum dolor sit amet.</li>
            <li>Porro veritatis totam tempora officiis!</li>
            <li>Ut ducimus iusto accusantium qui?</li>
            <li>Accusantium quam ullam veritatis earum.</li>
            <li>Deserunt, eligendi! Doloremque, quisquam optio?</li>
        </ul>
    @show
</div>
```
A opção *@show* faz com que o atual seja exibido

Na página de conteúdo fazemos referência *@parent*

```
@section('siderbar')
    @parent
    <h1 class="teste">[Page] Listagem de Arquivos</h1>
    <ul>
        <li>Lorem, ipsum dolor.</li>
        <li>Blanditiis, cupiditate ratione.</li>
        <li>Aperiam, quisquam quos.</li>
    </ul>
@endsection
```
---
## NPM e LaraveLMix

NPM responsável pelo gerenciamento das bibliotecas JS. LaravelMix é uma abstração do WebPack.

Rodando as instalações do NPM
> npm install

Para atualizar e recompilar as biblitecas
> npm run dev

Devemos trabalhar nos arquivos que estão na pasta **resource**, pois eles irão alimentar nossa pasta *public*

Exemplo do arquivo *app.css*
```

// Fonts
@import url('https://fonts.googleapis.com/css?family=Nunito');

// Variables
@import 'variables';

// Bootstrap
@import '~bootstrap/scss/bootstrap';

.navbar-laravel {
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.04);
}
```
Vamos adicionar uma classe

```
.jaqueline {
    color: darkblue;
}
```
Para que a aplicação veja a modificação devemos rodar novamente
> npm run dev

Para evitar digitar o tempo todo deixaremos o npm observando nossas modificações
> npm run watch-poll


Mimificação e versionamento

Só geramos o arquivo mimificado quando enviarmos para produção.
> npm run production

Para versionar é necessário ir apenas no arquivo *webpack.mix.js* e inserimos o método version. Fica assim:
```
mix.js('resources/js/app.js', 'public/js')
   .sass('resources/sass/app.scss', 'public/css')
   .version();
```
Rodamos novamente o `npm run production`

Na pasta **public** temos um arquivo *mix-manifest.json*. Nele temos uma hash.
Precisamos agora configurar nossa master para fazer referência a hash. Para isso, alterar o help *assets* para *mix*.
Antes:
> <link rel="stylesheet" href="{{ asset('css/app.css') }}">

Depois:
> <link rel="stylesheet" href="{{ mix('css/app.css') }}">

Obs: Se usarmos o laravel diretamente no apache. Não irá funcionar e será preciso fazer algumas alterações. Para ver funcionando.
> php artisanserve

---
## Validação e Recursos

Criando um controlador
> php artisan make:controller CourseController --resource

Criamos um formulário
```
@extends('template.master.master')

@section('content')

    <div class="container">
        <div class="row">
            <div class="col-12 mt-4">

                @foreach ($errors->all() as $error)
                <div class="alert alert-danger" role="alert">
                    {{ $error }}
                </div>
                @endforeach

                <form action="{{ route('couser.store') }}" method="post" autocomplete="off">
                    @csrf
                    <div class="form-group">
                        <label for="name">Curso:</label>
                        <input type="text" name="name" id="name" class="form-control {{ ($errors->has('name') ? 'is-invalid' : 'is-valid') }}"  placeholder="Insira o nome do curso" value="{{ old('name')}}">
                        @if ($errors->has('name'))
                        <div class="invalid-feedback">
                            {{ $errors->first('name') }}
                        </div>
                        @endif
                    </div>

                    <div class="form-group">
                        <label for="name">Tutor:</label>
                        <input type="text" name="tutor" id="tutor" class="form-control {{ ($errors->has('tutor') ? 'is-invalid' : 'is-valid') }}" placeholder="Insira o tutor do curso" value="{{ old('tutor')}}">
                        @if ($errors->has('tutor'))
                        <div class="invalid-feedback">
                            {{ $errors->first('tutor') }}
                        </div>
                        @endif
                    </div>

                    <div class="form-group">
                        <label for="name">Email:</label>
                        <input type="text" name="email" id="email" class="form-control {{ ($errors->has('email') ? 'is-invalid' : 'is-valid') }}" placeholder="Insira o email do tutor" value="{{ old('email')}}">
                        @if ($errors->has('email'))
                        <div class="invalid-feedback">
                            {{ $errors->first('email') }}
                        </div>
                        @endif
                    </div>

                    <div class="form-group">
                        <button class="btn btn-primary" type="submit">Cadastrar</button>
                    </div>

                </form>

            </div>
        </div>
    </div>

@endsection

```

Controlador

```
public function store(Request $request)
    {
        $rules = [
            'name' => 'required',
            'tutor' => 'required | min:3 | max:8',
            'email' => 'required | email'
        ];

        $messages = [
            'name.required' => 'Por favor, insira o nome do curso',
            'email.email' => 'Por favor, insira um endereço de email válido',
            'email.required' => 'Email é requerido'
        ];

        $request->validate($rules, $messages);
        // var_dump($request->all());
    }
```
### Validando com injeção de dependência

Vamos criar uma classe que fará a injeção, ou seja, que seja do tipo request

> php artisan make:request Course

Temos dois métodos criados: authorize e o rules. Para fazer funcionar, vamos passar todas as regras que haviamos criado no controlador. E colocamos **true** no `authorize` para que usuários que não estejam autenticados consigam acessar o método.

Para que seja permitido alteramos o método **Store** por **Course**

```
public function rules()
    {
        return [
            'name' => 'required',
            'tutor' => 'required | min:3 | max:8',
            'email' => 'required | email'
        ];
    }
```
> public function store(Course $request)

Para as mensagens:

```
public function messages() {
        return [
            'name.required' => 'Por favor, insira o nome do curso',
            'email.email' => 'Por favor, insira um endereço de email válido',
            'email.required' => 'Email é requerido'
        ];
    }
```

### Validando com Facede

Precisamos voltar o método store
> public function store(Request $request)

```
//validando com a facede
        $validate = Validator::make($request->all(), $rules, $messages);
        var_dump($validate->fails());

        if ($validate->fails()) {
            return redirect()->route('course.create')
                             ->withInput()
                             ->withErrors($validate);
        }
```
