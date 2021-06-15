# SE0-101

## 1 - Documentación SE0101 - app.py - v. 1.0

### _El desarollo está basado en programación funcional._

#### Archivo app.py

- __Entry point__ de la aplicación:

    Dentro del entry point se llama la función __get_file_xml__, la cual manda por parámetro la variable __path_file__ que conteiene la ubicación donde se almacenará el archivo XML.

```
if __name__ == "__main__":
    
    path_file = "/home/santiago/Projects/VisionAPI/xml/"
    get_file_xml(path_file)

```

- Función __get_file_xml__:

    Esta función se encarga de listar archivos del directorio. Una vez lista los archivos dentro del directorio, valida si existe un archivo con la extensión xml. En caso de encontrar, retornara la función __download_xml__, que por sí lleva un parámetro booleano que se encargará de establecer si se descarga o no el archivo xml.

    La función __download_xml__ se está importando desde el archivo __get_xml.py__:

```
def get_file_xml(path_file): # Con esta función vamos a validar que exista algún archivo dentro de la carpeta xml.
  import os
  from get_xml import download_xml

  files = os.listdir(path_file) # In the variable, an array of length is obtained in how many files it finds in the directory.   

  if len(files) == 0: # If the length is equal to zero, it must return the value of false to indicate that there is no file.
      return download_xml(False)
  elif len(files) > 0: # 
      for f in files:
          valor = f.endswith(".xml")
          if valor:
              return download_xml(True)
          else:
              return download_xml(False)
```
__________________________________________________________________________________________________________________________________________________________________

## 2 - Documentación SE0101 - get_xml.py - v. 1.0

#### Archivo get_xml.py

- Función __donwload_xml__:
  
  Esta función recibe por parametro un valor booleano(True o False) nombrado como: _validation_file_ desde la función __get_xml__ en el archivo __app.py__.

    - True: si recibe el valor true, el flujo se irá por la condición else del condicional donde solo tomará el nombre del          archivo ya que el archivo se encuentra guardado en el directorio.

    - False: si recibe el valor de false, el flujo se irá por el if del condicional, que irá a la url donde se encuentra el XML y   lo descargará. 

    Cada flujo del condicional retornará el llamado de la función __handle_xml__ que pasará como parámetro _xml_name_ que contiene el nombre del archivo xml. 

  ```
  def download_xml(validation_file):
      import wget
      import io, os

      path_file = "/home/santiago/Projects/VisionAPI/xml/tennis.xml"
      
      if validation_file is False:
          url = 'https://www.tennis.com.co/XMLData/adbid_facebook_junio.xml'
          file_xml = wget.download(url, path_file)
          f = io.open(path_file, 'rb')
          xml_name = os.path.basename(f.name)

          return handle_xml(xml_name)
      else:
          f = io.open(path_file, 'rb')
          xml_name = os.path.basename(f.name)
          
          return handle_xml(xml_name)
  ```
  
- Función __handle_xml__:

  La función recibe como parámentro la variable nombrada _xml_name_, que contiene el nombre del archivo.

  Dentro de esta función, se accederá por medio de un FOR el archivo XML, que buscará los nodos donde se encuentren las imágenes para agregarlas en lista _images_urls_.

  La función retorna el llamado la función __download_images__ que se está importando desde el archivo __get_images.py__

  ```
  def handle_xml(xml_name):
    import xml.etree.ElementTree as ET
    from get_images import download_images

    tree = ET.parse("/home/santiago/Projects/VisionAPI/xml/" + xml_name)
    root = tree.getroot()

    images_urls = []

    for child in root.findall('{http://www.w3.org/2005/Atom}entry'):
        image = child.find('{http://base.google.com/ns/1.0}image_link')
        images_urls.append(image.text)

    path_img = "/home/santiago/Projects/VisionAPI/resources/images/tennis/"

    return download_images(images_urls, path_img)
  ```   
__________________________________________________________________________________________________________________________________________________________________


## 3 - Documentación SE0101 - get_images.py - v. 1.0

#### Archivo get_images.py

- Función __download_images__:
  
  La función recibe dos parámetros; _images_urls_ y _path_. _images_url_ contiene la lista de urls de las imágenes en el feed XML y _path_ contiene la ruta donde se almacenarán las imágnes del XML.

  Retorna el llamado de la función __detect_labels__ que pasará como parámetro _path_ la ubicación donde estarán descargadas las imágenes. Se está importando desde el archivo __get_lables.py__.

  ```
  def download_images(images_urls, path):
    import urllib.request
    import io
    from get_labels import detect_labels


    i = 0
    for image in images_urls:
        filename = 'tennis_' + str(i) + '.jpg'
        urllib.request.urlretrieve(image, path+filename)
        i += 1
        if i == 10:
            break

    return detect_labels(path)
  ```
__________________________________________________________________________________________________________________________________________________________________

## 4 - Documentación SE0101 - get_labels.py - v. 1.0

#### Archivo get_labels.py

- Función __detect_labels_:

  Recibe como parámetro el path donde están almancenadas las imáganes, que luego se listarán para leer cada imagen con VISION API.

  Por medio del paquete _google.cloud import vision_ se están usando los métodos para leer las imágenes y agregar los labels en la lista _label_list_. 

  Retorna el llamado de la función _translate_label_ que pasa como parámetros a _labels_list_, _file_name_ y _lang_.

  ```
  def detect_labels(path):
    import io
    import os  
    from google.cloud import vision
    from google.oauth2 import service_account

    credentials = service_account.Credentials.from_service_account_file("/home/folder/folder/folder/key.json")

    client = vision.ImageAnnotatorClient(credentials=credentials)
    files = os.listdir(path)
    counter = 0
    for file_ in files:
        image_path = path + str(files[counter])
        f = io.open(image_path, 'rb')
        file_name = os.path.basename(f.name)

        with f as image_file:
            content = image_file.read()

        image = vision.Image(content=content)

        response = client.label_detection(image=image, max_results=8)
        labels = response.label_annotations

        labels_list = []
        for label in labels:
            labels_list.append(label.description)
        lang = "es"
        translate_label(labels_list, file_name, lang)
        counter += 1
  ```

- Función __translate_label__:

  Recibe los parámetros __list_label_, __file_name_ y _lang_. _list_label_ contiene los atributos leídos de cada imagen por medio de _VISION API_. _file_name contiene el nombre de la imagen, y _lang_ especifica el estandar de la sigla asociada para el idioma español "es".

  Por medio del paquete _google.cloud import translate_v2_ se están usando los métodos para traducir cada uno de los atributos obtenidos por _VISION_API_.

  Invoca función _insert_db_ que pasa como parámetros a _kw_translate_ y _file_name_.

  ```
  def translate_label(list_label, file_name, lang):
    import six
    from google.cloud import translate_v2 as translate
    from google.oauth2 import service_account

    credentials = service_account.Credentials.from_service_account_file("/home/santiago/Projects/VisionAPI/vison--API-1cbf8599fbce.json")
    translate_client = translate.Client(credentials=credentials)
    
    counter = 0
    kw_translate = []
    for label in list_label:
        result = translate_client.translate(label, target_language=lang)
        kw_translate.append(result['translatedText'])
        counter += 1
        
    insert_db(kw_translate, file_name)
    
  ```

- Función __insert_db__:

  Recibe los parámetros _list_label_ y _file_name_. _list_label_ almacena los atributos obtenidos por _VISION API_ y _file_name_ almacena el nombre del archivo.

  Por medio del paquete _google.cloud import bigquery_ se están usando los métodos para ingresar los datos en _BigQuery_.

  ```
  def insert_db(list_label, file_name):
    from google.cloud import bigquery
    from google.oauth2 import service_account

    credentials = service_account.Credentials.from_service_account_file("/home/santiago/Projects/VisionAPI/vison--API-1cbf8599fbce.json")

    client = bigquery.Client(credentials=credentials)

    table_id = "vison-api-304615.seo_101.images"

    length_labels = len(list_label)
    quantity_labels = []
    for i in range(length_labels):
        quantity_labels.append("_"+str(i))

    attr_value = dict(zip(quantity_labels, list_label))
    rows_to_insert = [
        {
        u"image": file_name,
        u"image_attribute":
            [
                attr_value
            ], 
        }  
    ]
   
    errors = client.insert_rows_json(table_id, rows_to_insert)  # Make an API request.
    if errors == []:
        print("New rows have been added.")
    else:
        print("Encountered errors while inserting rows: {}".format(errors))

    return ""
  ```

_____________________________________________________________________________________________________________________________________________________

  ### Diagrama de procesos SEO101
  
  1. Proceso abstracto del proyecto SEO101

  ![alt](https://drive.google.com/uc?export=view&id=16scS1BpsOj5zytZfFh5gEqrq5hjkQxU8)
 
 ### Diagrama de procesos para lectura de imágenes
  
  1. Proceso para obtener atributos por medio de Vision API, traducirlos con Translate API y cargarlos a BigQuery. Todo en entorno GCP.
  
 ![alt](https://lucid.app/publicSegments/view/82593e67-3d12-4f6e-bf2d-f9e8b44699cc/image.jpeg)
 
 _____________________________________________________________________________________________________________________________________________________
 
 # Bases de datos
 
 ## Modelo entidad relación

 ### ¿Qué es el modelo entindad relación?

Es un diagrama de flujo que describe como diseñar una base de datos que contendrá la lógica de como se relacionan las entidades en la base de datos.

### Está compuesto por:

- Entidades
- Atributos 
- Relacioes
- Tipos de relaciones

### ¿Qué es una entidad?

- Una entidad es un objeto que representa algo de la vida real. Una entidad puede representar dos tipos de objetos: objetos   con existencia física y conceptual.
  - Fisica: Es un objeto que existe fisicamente en el mundo real, ejm: carro, casa. Un objeto con existencia fisica se le     denomina entidad concreta.
  - Conceptual: Es un objeto que no existe fisicamente pero existe, ejm: una asignatura, un cargo. Un objeto con entidad      conceptual se le denomina entidad abstracta.

- ¿Como se representa una entidad en un diagrama de bases de datos entidad relación?

  En el diagrama, se identifica una entidad con la figura rectángulo.

![Image](https://lh3.googleusercontent.com/proxy/-LHyB3uAVpP5NkYN64PqPwiWttRdZfe0U_-KGM-wHPcWnPGXPN_M43HKK73bkMY5jXn7si4uRyMwlV1hyRCHlVAZkuhouH3Em_yT3pXNKU-KuZEHQEbeQxAmXTretANXyo0Ix7uCu4MW_yVOR-yX2Qe2F9sk0rH2)


### ¿Qué es un atributo?

- Un atributo es una propiedad o una característica que describre a una entidad.
  Ejem: El automóvil tiene un color, ese color es un atributo o propiedad de la entidadd automóvil.
  Una entidad puede tener varios atributos y un atributo puede tener varios registros.

  En el diagrama, se identifica un atributo con un círculo u óvalo.

  ![Image](https://www.ecured.cu/images/thumb/6/6a/%C3%93valo.jpeg/260px-%C3%93valo.jpeg)

- Si se debe identificar cada registro por un tipo de atributo especifico que identifica cada registro como único y           especial, se denominarácomo atributo clave.


### ¿Qué es una relación?

  - Las entidades se relacionan y las relaciones nos dicen como se relacionan las entidades. 

    En el diagrama, se identifica un atributo con un círculo u óvalo.

    ![Image](https://www.caracteristicass.de/wp-content/uploads/2018/10/caracteristicas-del-rombo.jpg)


    __Tipos de relaciones__:
    
    Existen varios tipos de relaciones:

      - 1:1 -> La relación uno a uno es cuando una entidad se relaciona exclusivamente con otra entidad.
      - 1:N -> La relación uno a muchos es cuando una entidad se relaciona con varias entidades. 
      - N:N -> La relación muchos a muchos es cuando varias entidades se relacionan con varias entidades.

      N = Muchos

      Ejemplos:

      ![Image](https://lucid.app/publicSegments/view/9677614a-a322-4625-83ee-1b6d46d5614b/image.png)
      
 ### Diseño de base de datos
 
   1. El diseño de la base está basada en entidad-relacion.

   En ella encontraremos 7 entidades que se relacionan entre sí.
   
   ![Image](https://drive.google.com/uc?export=view&id=1MjQ8xDH0NV-OU52HRq6e7hTjLVBfArJw)
  
#### En la entidad _PRODUCTS_ encontraremos:
   
- _PRODUCT_ID_: campo como primary key que almacena el id único del producto.
- _PRODUCT_NAME_: campo como foreign key que almacena el nombre del producto asociado al id del producto.
- _PRODUCT_PRICE_: campo que almacena el valor del producto asociado al id del producto.
- _PRODUCT_IMAGE_: campo que almacena la imagen del producto asociado al id del producto. (Las imágenes como tal se almacenarán en un bucket)
- _PRODUCT_DESCRIPTION_: campo que almacena la descripción del producto asociado al id del producto.
   
#### En la entidad _CATEGORIES_ encontraremos:
   
- _CATEGORY_ID_: campo como primary key que almacena el id único de la categoría.
- _CATEGORY_NAME_: campo como foreign key que almacena el nombre asociado al id de la categoría.

#### En la entidad _CATEGORIES_AND_PRODUCTS_ encontraremos:
  
- _PRODUCT_ID_: campo como foreign key de la entidad __PRODUCTS__ que almacena el id único asociado al producto.
- _CATEGORY_ID_: campo como foreign key de la entidad __CATEGORIES__ que almacena el id único asociado a la categoría.
- _STATUS_: campo que almacena el estado del producto. El estado varía si el producto tiene stock. Si tiene stock el valor booleano será uno. Si no hay stock, el   valor booleano será de cero.

#### En la entidad _PRODUCT_DESCRIPTIONS_ encontraremos:

- _PRODUCT_ID_: campo compo foreign key de la entidad __PRODUCTS__ que almacena el id único asociado al producto.
- _PRODUCT_DESCRIPTION_: campo que almcena la descripción del producto del feed XML.

#### En la entidad _PRODUCT_LABELS_ encontraremos:

- _PRODUCT_ID_: campo compo foreign key de la entidad __PRODUCTS__ que almacena el id único asociado al producto.
- _PRODUCT_LABEL_: campo que almacenará cada etiqueta leída por _VISION_API_.

#### En la entidad _PRODUCT_OBJECTS_ encontraremos:

- _PRODUCT_ID_: campo compo foreign key de la entidad __PRODUCTS__ que almacena el id único asociado al producto.
- _PRODUCT_LABEL_: campo que almacenará cada objeto leído por _VISION_API_.

#### En la entidad _PRODUCT_PROPERTIES_ encontraremos:

- _PRODUCT_ID_: campo compo foreign key de la entidad __PRODUCTS__ que almacena el id único asociado al producto.
- _PRODUCT_PROPERTY_RGB_: campo que almacenará el color dominante de la imagen en formato RGB.
- _PRODUCT_PROPERTY_HEX_: campo que almacenará el color dominante de la imagnen en formato HEX.

