# **Listando prestadores de serviço no GoBarber**

**Objetivo:** Realizar busca de usuários prestadores de serviço na aplicação do GoBarber.

### **1. Criação e importação de ProviderController.js**

O **ProviderController.js** será responsável por usuários prestadores de serviço na aplicação. Antes de mais nada é necessário criar o **ProviderController.js** dentro da pasta **controllers** seguindo o modelo **MVC**.

É necessário importar **ProviderController.js** em **routes.js** ```import ProviderController from './app/controllers/ProviderController';```

### **2. Configuração de ProviderController.js**
A classe contém um **método index** que tem como função filtrar todos os prestadores de serviços, conforme pode ver no código abaixo:
```
import User from '../models/User';
import File from '../models/File';

class ProviderController {
  async index (req, res) {
    const providers = await User.findAll({
      where: { provider: true },
      attributes: ['id', 'name', 'email', 'avatar_id'],
      include: [
        {
          model: File,
          as: 'avatar',
          attributes: ['name', 'path', 'url'],
        },
      ],
    });

    return res.json(providers);
  }
}

export default new ProviderController();
```
Dentro desse método **index** temos uma constante providers ```const providers = await User.findAll``` que tem como função buscar todos os usuário cadastrados no GoBarber.
```
async index (req, res) {
    const providers = await User.findAll({
      where: { provider: true },
      attributes: ['id', 'name', 'email', 'avatar_id'],
      include: [
        {
          model: File,
          as: 'avatar',
          attributes: ['name', 'path', 'url'],
        },
      ],
    });
```
E dentro dessa constante temos um filtro para buscar somente prestadores de serviços. O ```where: { provider: true }``` é responsável por buscar todos os usuários em que **provider é verdadeiro**. Logo em seguida temos um outro filtro na resposta em JSON, pois só queremos os atríbutos **id, name, email, avatar_id** ```attributes: ['id', 'name', 'email', 'avatar_id']```.
```
 where: { provider: true },
      attributes: ['id', 'name', 'email', 'avatar_id'],
      include: [
        {
          model: File,
          as: 'avatar',
          attributes: ['name', 'path', 'url'],
        },
```
Logo após temos um **include** que serve para incluir "objetos" relacionados a este usuário. Como já temos um relacionamento com o **modelo de arquivo** podemos e precisamos chama-lo para exibir a foto de perfil do usuário prestador de serviço para os demais usuários/clientes.
```
include: [
        {
          model: File,
          as: 'avatar',
          attributes: ['name', 'path', 'url'],
        },
```
Logo selecionamos o modelo ```model: File```, renomeamos como **avatar** por meio de ```as: 'avatar'``` e por fim selecionamos os atríbutos que queremos na resposta em JSON, sendo eles o **name, path, url** por meio de ```attributes: ['name', 'path', 'url']```.

### **3. Em File.js**

Não se esqueça que por meio de ```get() { return `localhost:3333/files/${this.path} }``` será o meio de manipulação de arquivos no front da aplicação.

```
  name: Sequelize.STRING,
  path: Sequelize.STRING,
  url: {
    type: Sequelize.VIRTUAL,
    get() {
      return `localhost:3333/files/${this.path}`
    }
  }
```
O método **get( )** serve para pegar o valor e formata-lo da maneira que queremos, dessa forma temos como **retorno** o **arquivo** via **url**. Em ```type: Sequelize.VIRTUAL``` estamos dizendo que não existe na tabela, somente na aplicação.

### **5. Testando no insominia**

Crie uma nova rota do tipo **Get** (é necessário colocar a **base_url** e a rota **/providers**) em seguida vá em **auth** e em **barear token**, adicione o nome do token que está setado em **Manage Environment**.