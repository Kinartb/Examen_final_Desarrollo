

## Parte 1: Estas preguntas corresponden a las actividades desarrolladas en clase.

### 1
Produce un conflicto de fusión (merge) en algún repositorio de tus actividades realizadas. 
Establece los pasos y comandos que usas para resolver un conflicto de fusión en Git. Si intentas git push y falla con un mensaje como :
Non-fast-forward (error): failed to push some refs esto significa que algún archivo contiene un conflicto de fusión entre la versión de tu repositorio y la versión del repositorio origen. 
Para este ejercicio debes presentar el conflicto dado, los pasos y comandos para resolver el problema y las solución.

se tienen estos el repo

https://github.com/Kinartb/TDD

y se creara un archivo example.rb

Se inicia el git con `git init`

Se conect el repositorio con:

```
git remote add origin https://github.com/Kinartb/TDD.git
```

Si nos aparece un error de que ya se esta usando origin en un remoto

```
git remote set-url origin https://github.com/Kinartb/TDD.git
```

lo que se hara a cotinuacion es tratar de subir los archivos:
```
git push -u origin main
```
Nos aparece un error y que para solucionarlo debemos hacer `git pull`

Configuramos el tipo de solucion que queremos q tenga por defecto ` git config pull.rebase false` (Esto descarga el repositorio al local) y luego hacemos `git pull` nuevamente

Hacemos `git push` y vemos que los archivos se suben con normalidad

Este conflicto se debia a que en el repositorio local no se encontraban los archivos `Readme.md` ni `copia_de_readme.md`, ademas de que de manera local se tenia `example.rb` creado por lo que para poder comparar
los archivos (para este caso unirlos) se debe realizar el pull respectivo y bajar los archivos de manera local para despues subirlo

2.
```
class User < ActiveRecord::Base
validates :username, :presence => true
validate :username_format
end
```
Si tienes un @user sin nombre de usuario y llamas a @user.valid?, la validación validates :username, :presence => true fallará, ya que impone que el nombre de usuario no puede estar ausente. Por lo tanto, @user.valid? devolverá false. Si intentas guardar el usuario mediante @user.save, el guardado no se llevará a cabo debido a la falla en la validación y @user.save devolverá false. Además, puedes verificar los errores de validación utilizando @user.errors.full_messages para obtener mensajes detallados sobre por qué la validación ha fallado.
```
class User < ActiveRecord::Base
  validates :username, presence: true
  validate :username_format

  private

  def username_format
    return unless username.present?

    unless username.match(/\A[A-Za-z][A-Za-z0-9]{0,9}\z/)
      errors.add(:username, 'debe comenzar con una letra y tener como máximo 10 caracteres')
    end
  end
end

```
3.

Recuerda, los filtros nos ayudan a verificar si ciertas condiciones se cumplen antes de permitir que se ejecute una acción del controlador. Para el modelo de User, digamos que queremos verificar si @user era administrador de todos los métodos en AdminController. Completa el método before_filter:check_admin a continuación que verifica si el campo de administrador en @user es verdadero. De lo contrario, redirija a la página admin_login con un mensaje que indica acceso restringido.
```
class AdminController < ApplicationController
  	        before_filter :check_admin
      # Completa el codigo
```

solucion:

```
class AdminController < ApplicationController
  before_filter :check_admin

  def check_admin
    unless @user&.admin?
      flash[:alert] = 'Acceso restringido. Debe ser administrador.'
      redirect_to admin_login_path
    end
  end
end

```
Este método verifica si el campo admin en @user es verdadero. Si no lo es, redirige a la página admin_login con un mensaje de error.

4. no
### Parte 2. Pruebas

![](fina5)

a partir de ahora se utilizara 
```
bundle _2.0.0.pre.3_ install
```
Ejecutaremos 
```
rails server
```
Y verificamos la url asignada en el output de pantall

Nos aparece un error donde no se ha creado la tabla ejecutaremos

```
rake db:migrate
```
al ejecutar `rails server` nos aparece la pagina cargada satisfactoriamente.

Para comenzar con el trabajao agregamos dichas gemas al archivo gemfile
```
gem 'faraday'  
group :test do
  gem 'rails-controller-testing'
  gem 'guard-rspec'                 
end
```
ejecutamos `bundle install`

luego ` bundle exec guard init rspec` y posteriormente para utilizar el entorno guard
```
bundle exec guard

```
![](9)

Creamos la tabla pero se me olvido crear las peliculas, para esto usamos
```
rake db:seed
```
Ejecutamos y luego haremos `rails server`

![](10)

Despues de implementar lo pedido vimos que las 3 pruebas realizadas fallan

![](11)


Vamos a implementar los errores

![](12)

Vemos que las 3 pruebas fallan pero ahora el tipo derror es diferentes nos aparece un Thread asi que lo agregamos. Al agregar `(usando guard activamente)` el siguiente codigo

```
if RUBY_VERSION>='2.6.0'
  if Rails.version < '5'
    class ActionController::TestResponse < ActionDispatch::TestResponse
      def recycle!
        # hack to avoid MonitorMixin double-initialize error:
        @mon_mutex_owner_object_id = nil
        @mon_mutex = nil
        initialize
      end
    end
  else
    puts "Monkeypatch for ActionController::TestResponse no longer needed"
  end
end
```

Si bien se implemento en el paso 1, recien aparece este problema en el paso 2.

Al momento de ejecutarlo vemos ahora que solamente esta fallando 2 de las 3 pruebas como vemos en el `guard`

![](13)


¿Puedes ir a /search_tmdb ahora? 
En este punto, RSpec debería informa Green para el primer ejemplo, pero eso no es realmente exacto porque el ejemplo en sí está incompleto: en realidad no se ha verificado si search_tmdb en el controlador llama a un método modelo para buscar TMDb, como lo requiere la especificación. 

![](14)

modificamos `movies_controller_spec`
```
it 'calls the model method that performs TMDb search' do
  fake_results = [double('movie1'), double('movie2')]
  expect(Movie).to receive(:find_in_tmdb).with('hardware').
    and_return(fake_results)
  get :search_tmdb, {:search_terms => 'hardware'}
end
```

la prueba sigue fallando pero por la razon correcta

![](15)

¿Por qué la expectativa receive debe preceder  a la acción get en la prueba?.

Cuando  se utiliza expect con receive se configura una expectativa en un objeto para que reciba un mensaje particular. 
En este caso, estás configurando la expectativa de que el método find_in_tmdb de la clase Movie se llame con el argumento 'hardware'
cuando se ejecute la acción get :search_tmdb.

La razón por la que la expectativa debe preceder a la acción get en la prueba es porque estás definiendo las condiciones que esperas que ocurran durante la ejecución de esa acción. 

```rb
# Configurando la expectativa antes de la acción
expect(Movie).to receive(:find_in_tmdb).with('hardware').and_return(fake_results)
get :search_tmdb, { search_terms: 'hardware' }

# Ejecutando la acción sin la configuración de expectativa
get :search_tmdb, { search_terms: 'hardware' }
expect(Movie).to receive(:find_in_tmdb).with('hardware').and_return(fake_results)
```
!()[15]

### 3

```
it 'selects the Search Results template for rendering' do
  fake_results = [double('movie1'), double('movie2')]
  allow(Movie).to receive(:find_in_tmdb).and_return(fake_results)
  get :search_tmdb, {:search_terms => 'hardware'}
  expect(response).to render_template('search_tmdb')
end

```
Explicación del código:

`fake_results = [double('movie1'), double('movie2')]:` Se crean resultados falsos que simulan lo que se esperaría de una llamada exitosa al método find_in_tmdb.

`allow(Movie).to receive(:find_in_tmdb).and_return(fake_results):` Se configura un doble para el método find_in_tmdb en la clase Movie, indicando que cuando este método se llame, debe devolver los resultados falsos configurados

`get :search_tmdb, { search_terms: 'hardware' }:` Se realiza la acción del controlador, simulando una búsqueda en TMDb con el término 'hardware'.

`expect(response).to render_template('search_tmdb')` Se verifica que la respuesta de la acción del controlador indica que se seleccionó el template 'search_tmdb' para renderizar.

Modificamos y ahora la prueba aparece en verde

![](16)

¿De qué tipo de objeto crees que @fake_results es una variable de instancia? (Dicho de otra manera, ¿cuál crees que es el valor de self dentro de un bloque de código de prueba?)

self es la instancia de la clase que está siendo probada. Por lo tanto, @fake_results es una variable de instancia de la clase MoviesController.
@fake_results se utiliza para almacenar resultados falsos que se esperan al llamar al método find_in_tmdb en la clase Movie. 
```
  1 require 'rails_helper'
  2 
  3 describe MoviesController do
  4   describe 'searching TMDb' do
  5     before :each do
  6       @fake_results = [double('movie1'), double('movie2')]
  7     end
  8     it 'calls the model method that performs TMDb search' do
  9       expect(Movie).to receive(:find_in_tmdb).with('hardware').
 10         and_return(@fake_results)
 11       get :search_tmdb, {:search_terms => 'hardware'}
 12     end
 13     describe 'after valid search' do
 14       before :each do
 15         allow(Movie).to receive(:find_in_tmdb).and_return(@fake_results)
 16         get :search_tmdb, {:search_terms => 'hardware'}
 17       end
 18       it 'selects the Search Results template for rendering' do
 19         expect(response).to render_template('search_tmdb')
 20       end
 21       it 'makes the TMDb search results available to that template' do
 22         expect(assigns(:movies)).to eq(@fake_results)
 23       end
 24     end
 25   end
 26 end
```
Ejecuta este código y explica las línes 5-7, 14-17, 18-20. ¿Hacemos stubbing?.

Especifica cuales son las  construccionesde  RSpec se utiliza para (a) crear un seam, (b) determinar el comportamiento de un seam.

¿Por qué suele ser preferible utilizar before(:each) en lugar de before(:all)?. Explica un caso de ejemplo.


`Líneas 5-7:`

En estas líneas, se define un bloque before :each. Este bloque se ejecutará antes de cada ejemplo.
En el bloque before, se crea un conjunto de resultados falsos @fake_results utilizando la función double de RSpec. @fake_results es un array que contiene dos objetos dobles ('movie1' y 'movie2') que simulan resultados de búsqueda de TMDb.

Líneas 14-17:`

Aquí hay otor bloque before :each dentro de un describe secundario llamado 'after valid search'. Este bloque se ejecutará antes de cada ejemplo.
Dentro de este bloque, se utiliza allow para hacer un stubbing del método find_in_tmdb de la clase Movie. 
Se configura para devolver @fake_results cuando se llama con cualquier argumento.
Luego, se realiza una llamada a get :search_tmdb con un conjunto de términos.

`Líneas 18-20:`

Estas líneas correspodnen a un ejemplo de prueba que verifica si la respuesta es el template 'search_tmdb'.
Se utiliza la expectativa expect(response).to render_template('search_tmbd') para asegurarse de que la acción search_tmdb enderice el template esperado.

`Líneas 21-23:`

Estas líneas corresponden a otro ejemplo de prueba que verifica si los resultados de búsqueda de TMDb están disponibles para el template.
Se utiliza la expectativa expect(assigns(:movies)).to eq(@fake_results) para asegurarse de que los resultados de la búsqueda (@fake_results) se asignen correctamente a la variable de instancia @movies en el controlador.

`¿Hacemos stubbing?`

Sí, se está utilizando stubbing en este código. En las líneas 15 y 16, se utiliza allow para hacer un stubbing del método find_in_tmdb de la clase Movie. Esto se hace para controlar el comportamiento de find_in_tmdb durante las pruebas y asegurarse de que devuelva los resultados falsos (@fake_results).

`Construcciones de RSpec:`

Para crear un seam (un punto de entrada que permite cambiar el comportamiento del sistema), se utiliza el método allow para hacer stubbing en el método find_in_tmdb de la clase Movie.
Para determinar el comportamiento de un seam, se utiliza el método expect para establecer expectativas sobre cómo se llamará y comportará el método find_in_tmdb durante la ejecución de la acción search_tmdb.

`¿Por qué suele ser preferible utilizar before(:each) en lugar de before(:all)?`

Usar before(:each) es preferible a before(:all). En la mayoría de los casos porque before(:each) garantiza que  la prueba se restablezca antes de cada ejemplo individual de prueba. Si se utilizara before(:all), el estado se compartiría entre todos los ejemplos de prueba en ese grupo, lo que podría llevar a dependencias no deseadas entre pruebas.


Ahora todas las pruebas fallan esto e debe a que it aun no se ha implementado

![](17)

### 4



!
