# **Upload de arquivos no GoBarber**

**Objetivo:** Realizar uploads de arquivos na aplicação do GoBarber.

**Obs:** Quando precisamos lidar com arquivos, é necessário utilizar o formato **multpart form data** para testes com o **Insominia**.

## **Instalando o Multer via package manager:**

Intalação via **npm**:
> [npm](https://npmjs.com) install [multer](https://www.npmjs.com/package/multer)

Intalação via **yarn**:
> [yarn](https://yarnpkg.com) add [multer](https://yarnpkg.com/en/package/multer)

### **1. Criação de pastas**

Como boas práticas, fora da pasta ```src``` foi criado a pasta ```tmp``` que conterá os arquivos temporários, e dentro desta, contém outra pasta criada ```uploads```, que por sinal conterá todos os arquivos de uploads do usuário diante da aplicação **GoBarber**. Ficando da seguinte maneira: 

  ![caminho-upload](https://user-images.githubusercontent.com/49616761/64479208-3a944b80-d18a-11e9-84f0-7722765d2618.png)


  ### **2. Configuração de uploads**

  Dentro da pasta **config** crie um arquivo chamado **multer.js**. Esse arquivo será responsável por toda a configuração de uplaods na aplicação. Conteúdo do arquivo **multer.js**:

```
import multer from 'multer';
import crypto from 'crypto';
import { extname, resolve } from 'path';

export default {
  storage: multer.diskStorage({
    destination: resolve(__dirname, '..', '..', 'tmp', 'uploads'),
    filename: (req, file, cb) => {
      crypto.randomBytes(16, (err, res) => {
        if(err) return cb(err);

        return cb(null, res.toString('hex') + extname(file.originalname));
      });
    },
  }),
};

```

Para a utilização da lib **multer**, é necessário importar o mesmo no arquivo da seguinte forma ```import multer from 'multer';```. Logo em seguida é importado da lib **path** ```import {extname, resolve} from 'path';```. 
```
import multer from 'multer';
import crypto from 'crypto';
import { extname, resolve } from 'path';
```
O [**path**](https://nodejs.org/api/path.html) é uma lib padrão do node, assim como o **Crypto**. O **extname** retorna a extensão do arquivo, já o **resolve** serve para percorrer caminhos dentro da aplicação.


```
storage: multer.diskStorage({
// A linha acima serve para salvar informações utilizando o multer

    destination: resolve(__dirname, '..', '..', 'tmp', 'uploads'),
// A destination serve para passar onde queremos salvar

    filename: (req, file, cb) => {
// A filename serve para passar uma formatação para o nome do arquivo

      crypto.randomBytes(16, (err, res) => {
// Por meio de crypto.randomBytes é gerado um nome unico de 16 bytes para cada upload

        if(err) return cb(err);
        // Se der erro, retorna um callback com o erro

        return cb(null, res.toString('hex') + extname(file.originalname));
      });
```
Por meio do próximo bloco, passamos **null** como **primeiro parâmetro**, porque não queremos que dê erro. No **segundo parâmetro** estamos convertendo 16 bytes de conteúdo aleatório do tipo string em hexadecimal e em seguida estamos concatenando-o com a extensão do arquivo original.
```
return cb(null, res.toString('hex') + extname(file.originalname));
```

### **3. Criando o controle de arquivos**

É necessário a criação de um controller para os arquivos de upload, por conta disso foi criado o FileController.js, apresentando a seguinte configuração:
```
import File from '../models/File';

class FileController {
  async store(req, res) {

    const { originalname: name, filename: path } = req.file;

    const file = await File.create({
      name,
      path,
    });

    return res.json(File);
  }
}

export default new FileController();
```
No código abaixo, por meio de desestruturação, estamos obtendo os seguintes dados de acordo com **originalname** e **filename**.
```
const { originalname: name, filename: path } = req.file;
```
Em seguida selecionamos os atríbutos por short syntax e criamos um arquivo. Por fim enviamos a constante **File**.
```
 const file = await File.create({
      name,
      path,
    });

    return res.json(File);
```

  ### **4. Importando o multer em routes.js**
```
import multer from 'multer';
import multerConfig from './config/multer';
```
Em seguida craimos uma **constante upload** para ser atribuida pelo método da lib multer, onde receberá a configuração de **multer.js**. Ficando da seguinte maneira:
```
const upload = multer(multerConfig);
```
Em seguida a rota para uploads de arquivos foi criada da seguinte maneira:
```
rouste.post('/files', upload.single('file'), FileController.store);
```
O ```upload.single('file')``` é responsável para fazer upload de somente um único arquivo por vez. Uma **observação**, se precisar fazer uploads de vários arquivos, um exemplo seria ```upload.multiple('files')```.

  ### **5. Configurando o Insominia**

  O insominia é uma ferramenta que nos permite testar a aplicação. Primeiramente é necessário configurar-lo, então vá em **No Environment** (no canto superior esquedo) e em seguida clique em **Manage Environment**. Agora configure da seguinte forma:
```
{
  "base_url": "localhost:3333",
  "token": "token-gerado-na-sessao-do-usuario"
}
```
E a rota ficará da seguinte maneira:

![rota-insominia-files-post](https://user-images.githubusercontent.com/49616761/64479256-bf7f6500-d18a-11e9-96af-7e2d40304e78.png)

A rota deve ser em **Multpart Form Data**, selecione a opção file, e escolha o arquivo para testar a lógica de uplaods. 

![multparformdata](https://user-images.githubusercontent.com/49616761/64479264-cf974480-d18a-11e9-8525-c5e69c4e0765.png)

O **token** gerado na sessão do usuário é necessário ir na aba em **auth** e em seguida em **barear token**, lá coloque o nome para o token que está setado em **Manage Environment**.

![toekn](https://user-images.githubusercontent.com/49616761/64479286-1dac4800-d18b-11e9-97ff-a1f2c6ec5344.png)

 ### **6. Criando uma nova migration com o sequelize**

 **Lembrete**: Para voltar uma migration, faça da seguinte forma:
 > ```yarn sequelize db:migrate:undo```

 Se precisar voltar tudo ao início, digite:
> ```yarn sequelize db:migrate:undo:all```
#
 Primeiramente precisamos criar uma nova tabela no banco de dados para salvamento desses uploads, então antes de tudo é necessario criar uma migration: 
 ```
 yarn sequelize migration:create --name=create-files
 ```
 Na nova migration foi necessario criar uma nova coluna para armazenar os arquivos, logo:
```
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
      return queryInterface.createTable('files', {
        id: {
          type: Sequelize.INTEGER,
          allowNull: false,
          autoIncrement: true,
          primaryKey: true,
        },
        name: {
          type: Sequelize.STRING,
          allowNull: false,
        },
        path: {
          type: Sequelize.STRING,
          allowNull: false,
          unique: true,
        },
        created_at: {
          type: Sequelize.DATE,
          allowNull: false,
        },
        updated_at: {
          type: Sequelize.DATE,
          allowNull: false,
        }

      });
  },

  down: queryInterface => {
      return queryInterface.dropTable('files');
  }
};
```
Por meio dessa migration temos um **id** como primary key, um **nome**, que por sinal será o **nome original** do arquivo, em seguida temos o **path** que será o nome gerado pelo **crypto.randomBytes** que por sinal será um **nome único**, a **data de criação** e de **updates**.

Agora com a migration finalizada, podemos criar nossa tabela no banco de dados com o seguinte comando:
 ```
 yarn sequelize db:migrate
 ```

  ### **7. Criando o model de arquivos**

  O **model de arquivos** tem a função de manipulação dos arquivos. Podemos ver que por meio desse **Model de arquivos** temos a importação do **Sequelize** e de **Model** ```import Sequelize, { Model } from 'sequelize';```.
```
import Sequelize, { Model } from 'sequelize';

class File extends Model {
  static init(sequelize){
    super.init(
      {
        name: Sequelize.STRING,
        path: Sequelize.STRING,
        url: {
          type: Sequelize.VIRTUAL,
          get() {
            return `${process.env.APP_URL}/files/${this.path}`
          }
        }
      },
      {
        sequelize,
      }
    );

    return this;
  }
}

export default File;
```
A classe **File** herda atríbutos da classe **Model** de **sequelize**, onde possuí um método estático ```static init(sequelize) {...}``` que faz uma chamada a classe mãe **Model** com ```super.init({ ... })```.
```
.
.
.

class File extends Model {
  static init(sequelize){
    super.init(
      {
        name: Sequelize.STRING,
        path: Sequelize.STRING,
        url: {
          type: Sequelize.VIRTUAL,
          get() {
            return `localhost:3333/files/${this.path}`
          }
        }
      },

      .
      .
      .
```
E em seguida é passado o que é para o modelo de arquivos o que deve ser retornado como resposta. 
```
{
        name: Sequelize.STRING,
        path: Sequelize.STRING,
        url: {
          type: Sequelize.VIRTUAL,
          get() {
            return `localhost:3333/files/${this.path}`
          }
        }
}
```
Em  ```name: Sequelize.STRING``` é retornado o nome original do arquivo. No ```path: Sequelize.STRING``` é passado o nome aleatório de 16 bytes gerado por **crypto.randomBytes**, que por sinal esse **path** é adicionado em:
```
url: {
          type: Sequelize.VIRTUAL,
          get() {
            return `localhost:3333/files/${this.path}`
          }
      }
```
Que tem como função retornar uma **url** do arquivo, onde facilitará a manipulação de arquivos no frontend da aplicação.
###
Agora em **index.js** é necessário importar o **modelo de arquivos**, ficando da seguinte maneira:
```
import File from '../app/models/file';
```
E consequentemente alterar a constante models, adicionando no vetor, o modelo de arquivos:
```
const models = [User, File];
```

**Observação:** Toda vez que precisar alterar sua tabela, é recomendavel que crie sempre uma nova migration. **Migrations** funcionam como **linhas do tempo**, mesmo se a **alteração for mínima** é necessário criar uma nova migration.

  ### **8. Migration para relacionamento de usuários e upload de arquivos**

  Primeiramente é necessário criar uma nova migration, por tanto:
  ```
  yarn sequelize migration:create --name=add-avatar-field-to-users
  ```
Por meio de  ```return queryInterface.addColumn('users', 'avatar_id', { ... }``` é adicionado uma nova coluna na tabela **users** no banco de dados com o nome **avatar_id**.
A configuração desta migration ficará da seguinte maneira:
```
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.addColumn('users', 'avatar_id', {
      type: Sequelize.INTEGER,
      references: { model: 'files', key: 'id' },
      onUpdate: 'CASCADE',
      onDelete: 'SET NULL',
      allowNull: true,
     });

  },

  down: queryInterface => {
    return queryInterface.removeColumn('users', 'avatar_id');
  }
};

```
Logo é passado alguns atributos para essa coluna:
```
      type: Sequelize.INTEGER,
      references: { model: 'files', key: 'id' },
      onUpdate: 'CASCADE',
      onDelete: 'SET NULL',
      allowNull: true,
```
Como podemos ver ```type: Sequelize.INTEGER``` é do tipo inteiro, onde tem referência com o **modelo de arquivos** onde será relacionado por meio da chave **id** ```references: { model: 'files', key: 'id' }```. Em seguida temos ```onUpdate: 'CASCADE'```, caso o usuário atualize sua foto de perfil, atualizara a tabela inteira do banco de dados. Em ```onDelete: 'SET NULL'```, caso o usuário delete sua foto de perfil, no banco de dados esse campo ficará como **null**, entretanto, por **default** quando a conta é criada esse campo já vem com o valor **null**.
###
Agora é só fazer a migração para o banco de dados:
```
yarn sequelize db:migrate
```
  ### **9. Relacionamento do modelos ```User``` e ```File```**

No model **User** é necessário adicionar:
```
 static associate(models) {
    this.belongsTo(models.File, { foreignKey: 'avatar_id', as: 'avatar' });
  }
  ```
O método estático ```static associate(models) { ... }``` será responsável por receber todos os models da aplicação. Em ```this.belongsTo(models.File, { foreignKey: 'avatar_id', as: 'avatar' });``` estamos dizendo que pertence a um **model File** que vai armazenar a referência do arquivo, e logo em seguida setamos um nome para manipulação ```as: 'avatar'```.

### **10. Alterando index.js**

O **index.js** é a "ponte" entre a aplicação e o banco de dados, sua configuração atual:

```
import Sequelize from 'sequelize';

import User from '../app/models/User';
import File from '../app/models/File';

import databaseConfig from '../config/database';

const models = [User, File];

class Database {
  constructor() {
    this.init();
  }

  init() {
    this.connection = new Sequelize(databaseConfig);

    models
      .map(model => model.init(this.connection))
      .map(model => model.associate && model.associate(this.connection.models));
  }
}

export default new Database();
```
O método **init( )** é responsável por executar a configuração de **databaseConfig** e de percorrer todos os models da aplicação.
```
init() {
    this.connection = new Sequelize(databaseConfig);

    models
      .map(model => model.init(this.connection))
      .map(model => model.associate && model.associate(this.connection.models));
}
```
O código abaixo vai mapear todos os models, entrentanto ele também é responsável por chamar o **model.associate** somente se ele existir na conexão dos models. Portanto só será chamado por meio do model de **User** (ou **user.js**).
```
.map(model => model.associate && model.associate(this.connection.models));
```

### **11. Alterando app.js**

E agora para **finalizar de vez** a **relação de usuário e upload de arquivos**, no arquivo **app.js** precisamos adicionar ```this.server.use('/files', express.static(path.resolve(__dirname, '..', 'tmp', 'uploads')));``` no método **middleware**. Ficará da seguinte maneira:
```
// Não se esqueça de importar o path

 middlewares() {
    {
      // código
    }

    this.server.use('/files', express.static(path.resolve(__dirname, '..', 'tmp', 'uploads')));
  }
  ```
O ```this.server.use('/files', express.static(path.resolve(__dirname, '..', 'tmp', 'uploads')));``` será responsável pela navegação de arquivos dentro da aplicação.

### **12. Um pouco mais sobre o app.js**

Em **app.js** deve-se adicionar no **middlewares( )** a seguinte configuração:
```
middlewares( ) {
  this.server.use(express.json());
  this.server.use('/files', express.static(
    path.resolve(__dirname, '..', 'tmp', 'uploads'))
  );
}
```
Por meio da rota **/files** e ```express.static``` é passado o nome do diretório que contém os ativos estáticos para a função de middleware, para iniciar entrega direta dos arquivos.
```
this.server.use('/files', express.static(
    path.resolve(__dirname, '..', 'tmp', 'uploads'))
  );
```  
E por meio de ```path.resolve(__dirname, '..', 'tmp', 'uploads'))``` fazemos a navegação no domínio de arquivos da API.

### **13. Testando com insominia**

Com base na requisição de uploads de arquivos pelo insominia que por sinal já está criada e upload de arquivo já feito, precisamos passar pelo corpo da requisição (em JSON) da seguinte forma:

```
{
  "name": "fulano",
  "email": "fulanoDaSilva@email.com",
  "oldpassword": "12345",
  "password": "docker",
  "confirmPassword": "docker",
  "avatar_id": 1,
}
```
Agora temos o relacionamento do arquivo com o usuário.
