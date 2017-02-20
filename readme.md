Angular 2: Getting Started - SnapShot 6
===================
En esta parte veremos estos contenidos del curso de Pluralshight:

 - Services & dependency injector
 - Retrieving data uning Http


----------


### 1 - Services & dependency injection


----------


Es muy habitual que en cualquier aplicación se requiera el uso de clases o servicios que encapsulen funcionalidades tales como obtención de datos, manipulación de los mismos o incluso servicios de logging.

Para ello TypeScript permite la creación de clases e interfaces para poder crear clases con funcionalidades específicas.

En la siguiente sección se definirá cómo crear un servicio y como registrarlo con el sistema de injección de dependencias de Angular para que los componentes no tengan que instanciar directamente los servicios.


#### Building a service


¿Qué se requiere para crear un servicio injectable? 

 - Importar el módulo Injectable 

 - Declarar el servicio con el decorador @Injectable()

**CURIOSIDAD**: Por qué los componentes no se decoran con **@Injectable()**?

Porque Angular ya registra de forma predeterminada los componentes en el IoC y porque el propio decorador **@Component** hereda de **@Injectable** así como **@Pipes**.


![enter image description here](https://i.imgur.com/VtULU8J.jpg)


.


#### Registring the service


Para hacer que nuestro servicio se pueda injectar en los componentes u en otros servicios es necesario registrarlo en la colección de **providers**.

Podemos hacerlo en dos ámbitos:

 - A nivel de componente: visible solo en el componente e hijos

 - A nivel de módulo: visible en todo el módulo

Bastará con añadir el servicio en el array de **providers**


![enter image description here](https://i.imgur.com/CWwMMUc.jpg)


.


#### Injecting the service


Finalmente solo se tendrá que declarar el servicio en el constructor del componente que lo vaya a utilizar:


![enter image description here](https://i.imgur.com/HZqSB76.jpg)


.


----------


### 2 - Retrieving data uning Http


----------


#### Obsevables and Reactive Extenxions


Nos ayudan a manejar la obtencion de datos asincronos 


Estan dentro de la propuesta del nuevo ES 2016


Para utilizarlos necesitamos la libreria de terceros: RxJS la qcual es usada dentro de Angular.


A pesar de esto podemos  volver al modelo clasico de promises aunque tiene unas cuantas diferencias a consierar:


![enter image description here](https://i.imgur.com/jqlhFea.png)


#### Sending and HTTP Request and exception handling


Debemos injectar el servicio Http e importar el HttpModule dentro de la definición del modulo
Utilizaremos el metodo get para obtener un **Observable < Response>**. Transformamos ese objeto en un array obsevable **Observable<IProduct[]>** medante el metodo  map de RxJS
Para manejar las excepciones utilizaremos los metodos do y catch de RxJS


#### Subscribing to an Observable


  Todo objeto observable tiene una metodo **subscribe** el cual nos notificará cuandolos datos esten disponible o se haya producido una excepción. 






![enter image description here](https://i.imgur.com/oEwLVGg.png)


----------


### 3 - Práctica


----------

El objetivo de esta practica es crear un servicio para acceder a la lista de productos.
Utilizamos el módulo el http y Obervable de RxJS para acceder a products.json.
Como complemento se requiere hacer lo mismo utilizando Promises

  
Para ello clonamos el **SnapShot 6** desde le primer commit de:


    git clone https://github.com/tc-frontend/course_angular2_day1_snapshot6
    cd course_angular2_day1_snapshot6
    git checkout tags/init
    npm install
    code .
 
----------

#### 1 - Creamos un servicio para acceder a los productos

    import {Injectable} from '@angular/core'
    import { IProduct} from './product'
    @Injectable()
    export class ProductService{

        getProducts(): IProduct[] {
            return [
            {
                "productId": 2,
                "productName": "Garden Cart",
                "productCode": "GDN-0023",
                "releaseDate": "March 18, 2016",
                "description": "15 gallon capacity rolling garden cart",
                "price": 32.99,
                "starRating": 4.2,
                "imageUrl": "http://openclipart.org/image/300px/svg_to_png/58471/garden_cart.png"
            },
            {
                "productId": 5,
                "productName": "Hammer",
                "productCode": "TBX-0048",
                "releaseDate": "May 21, 2016",
                "description": "Curved claw steel hammer",
                "price": 8.9,
                "starRating": 4.8,
                "imageUrl": "http://openclipart.org/image/300px/svg_to_png/73/rejon_Hammer.png"
            }
        ];
        }
    }

https://goo.gl/A3YaFV


----------

#### 2 - Añadimos el servicio como provider dentro del módulo principal

    providers: [ProductService]

https://goo.gl/DzlLiv

----------

#### 3 - Usamos el servicio para obtener la lista de productos

Para ello necesitamos injectar el servicio poniendolo como parametro en el constructor.
La llamada a getProducts la debemos hacer en el evento OnInit()

https://goo.gl/FhfWdX

----------

#### 4 - Utilizar el module HttpModule par acceder al JSON de datos

El JSON está en la carpeta "api". Se requiere el uso del Obsevables 

    getProducts(): Observable<IProduct[]> {
        return this._http.get(this._productUrl)
            .map((response: Response)=> <IProduct[]>response.json());
    }

https://goo.gl/tOBUKh

----------

#### 5 - Implementar manejo de excepciones 
Importamos los operadores do y catch para manejar excepciones. El operador do lo utilizamos para escribir una traza en la consola.

    import 'rxjs/add/operator/do'
    import 'rxjs/add/operator/catch'

Incluimos los operadores en get y manejamos al posible excepción:

    getProducts(): Observable<IProduct[]> {
        return this._http.get(this._productUrl)
            .map((response: Response)=> <IProduct[]>response.json())
            .do(data=> console.log('All' + JSON.stringify(data)))
            .catch(this.handleError);
    }

    private handleError(error: Response){
        return Observable.throw(error.json || 'Server error')
    }


https://goo.gl/4quYC5

----------

#### 6 - Subscribirnos al observable desde el listado de productos

Con esto recibiremos los datos de manera asincrona cuando esten disponibles y lo asignamos al modelo de datos.

    ngOnInit(): void {
            this._productService.getProducts()
            .subscribe(products =>this.products = products, 
                        error=> this.errorMessage = <any>error) ;
        }

https://goo.gl/IBYrp7

----------

#### 7 - Implementamos la lógica de obtención de productos con Promises

Importamos para transpormar Observable a Promises el operador de RxJs 'toPromie':
    
    import 'rxjs/add/operator/toPromise'

El servicio orientado a promises seria asi:

    getProductsPromise(): Promise<IProduct[]> {

        return this._http.get(this._productUrl)
                    .toPromise()
                    .then(response => <IProduct[]>response.json())
                    .catch(this.handleError);
    }

Y desde el componente lo utilizariamos así:

    //Promises
    this._productService.getProductsPromise()
        .then(products =>this.products = products)
        .catch(error=> this.errorMessage = <any>error);


https://goo.gl/2vWluc


Los pasos detallados en el historial de commits:


    https://github.com/tc-frontend/tc-frontend-angular2_day1_snapshot6/commits/master   
  

Si queremos ver la App en nuestro browser


    npm start


Si queremos ver la solucion final de este SnapShot:


    git checkout master
    npm install
    npm start
