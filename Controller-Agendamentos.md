# **Agendamento de serviços**

**Objetivo:** Explicar a lógica de agendamentos de serviços do GoBarber.

### **1. Criação do controlador de agendamentos**

Crie um arquivo com o nome **AppointmentController.js** dentro da pasta **controllers**.

### **2. Configuração do controlador de agendamentos**

```javascript
import * as Yup from 'yup';
import User from '../models/User';
import Appointment from '../models/Appointment';

class AppointmentController {
  async store(req, res) {
    const schema = Yup.object().shape({
      provider_id: Yup.number().required(),
      date: Yup.date().required(),
    });

    if(!(await schema.isValid(req.body))){
      return res.status(401).json({ error: 'validation fails' })
    }

    const { provider_id, date } = req.body;

    if(provider_id === req.userId){
      return res.status(401).json({ error: "Isn't not possible to appointment" });
    }

    //Verifica se o alvo é um prestador de serviços
    const CheckIsProvider = await User.findOne({
      where: { id: provider_id, provider: true },
    });

    if(!CheckIsProvider){
      return res
        .status(401)
        .json({ error: 'You can only create appointments with providers'});
    }

    const appointment = await Appointment.create({
      user_id: req.userId,
      provider_id,
      date,
    });

    return res.json(appointment);
  }
}

export default new AppointmentController();

```
O [Yup](https://github.com/jquense/yup) ```import * as Yup from 'yup';``` serve para validações na aplicação. 
```javascript
import * as Yup from 'yup';
```
Uma observação, como o Yup não exporta seu conteúdo como padrão, precisamos importar tudo com ```*``` e dar um nome ```as Yup```. Diferentemente do [express](https://devdocs.io/express/), por exemplo, quando importamos o express ```import express from 'express';```, essa lib **exporta o express como default**, e por conta disso podemos importar dessa maneira ```import express from 'express';```.
```javascript
class AppointmentController {
  async store(req, res) {
    const schema = Yup.object().shape({
      provider_id: Yup.number().required(),
      date: Yup.date().required(),
    });

    if(!(await schema.isValid(req.body))){
      return res.status(401).json({ error: 'validation fails' })
    }

    const { provider_id, date } = req.body;

    if(provider_id === req.userId){
      return res.status(401).json({ error: "Isn't not possible to appointment" });
    }

    //Verifica se o alvo é um prestador de serviços
    const CheckIsProvider = await User.findOne({
      where: { id: provider_id, provider: true },
    });

    if(!CheckIsProvider){
      return res
        .status(401)
        .json({ error: 'You can only create appointments with providers'});
    }

    const appointment = await Appointment.create({
      user_id: req.userId,
      provider_id,
      date,
    });

    return res.json(appointment);
  }
}
```
Por meio do método ```async store(req, res) { ... }``` temos uma constante **schema** que utiliza o Yup para validação e manipulação. Contém um atributo **provider_id** que é a chave primária do usuário prestador de serviço, sendo ela do tipo **numérico** e **obrigatório**. Também possui um atributo **date** que **indica a data do agendamento**, sendo ela do tipo **date** e **obrigatória**.

```javascript
async store(req, res) {
    const schema = Yup.object().shape({
      provider_id: Yup.number().required(),
      date: Yup.date().required(),
    });
``` 
Se o corpo da requisição não for válido, ou melhor, não existir **provider_id** e **date**, entrará dentro desta estrutura de decisão que será responsável por tratar este erro, retornando código 401 (**unauthorized**) e emitindo o erro **_validation fails_**.
```javascript
if(!(await schema.isValid(req.body))){
      return res.status(401).json({ error: 'validation fails' })
    }
```
Em seguida é selecionado do corpo da requisição por meio de desestrturação, os atríbutos **provider_id** e **date**.
```javascript
const { provider_id, date } = req.body;
```
Em seguida temos a seguinte estrutura de decisão que tem como função validar se o usuário prestador de serviço está tentando marcar serviço com ele mesmo.
```javascript
if(provider_id === req.userId){
  return res.status(401).json({ error: "Isn't not possible to appointment" });
}
```
Como não é possível marcar o usuário prestador de serviço marcar um agendamento com ele próprio, é retornado o seguinte erro 401 (**unauthorized**) e emitindo o erro **_Isn't not possible to appointment_**.
```javascript
return res.status(401).json({ error: "Isn't not possible to appointment" });
```
A próxima validação verifica se o alvo é um prestador de serviços:
```javascript
const CheckIsProvider = await User.findOne({
  where: { id: provider_id, provider: true },
});
```
Se não for um prestador de serviços, tratamos o erro dentro desta estrutura de decisão, onde retorna um erro 401 (**unauthorized**) e emitindo o erro **_You can only create appointments with providers_**.
```javascript
if(!CheckIsProvider){
      return res
        .status(401)
        .json({ error: 'You can only create appointments with providers'});
    }
```
Se tudo ocorrer perfeitamente é criado um novo agendamento, utilizando **user_id**, **provider_id** e **date** o agendamento será criado com sucesso.
```javascript
 const appointment = await Appointment.create({
      user_id: req.userId,
      provider_id,
      date,
    });
```
O ```user_id``` é o usuário que está marcando o agendamento do serviço, onde é atribuido por ```req.userId``` que é o **middleware** de autenticação que é setado automaticamente quando o usuário efetua uma nova sessão (ou login).
##
Por fim retornamos como resposta a constante appointment em formato JSON.
```javascript
return res.json(appointment);
```

### **3. Testando com o insominia**

Para efetuar o teste com o insominia é necessário que já tenha criado dois usuários no banco de dados, um como prestador de serviços e outro como usuário cliente. 

É necessário criar a rota e fazer toda a configuração que já estamos careca de saber.

Agora no corpo da requisição do tipo POST em JSON:
```javascript
{
  "provider_id": 3,
  "date": "2019-09-06T18:00:00",
}
```
**Obs**: Foi utilizado ```"provider_id": 3``` como um exemplo de usuário prestador de serviços.

Pronto, agendamento criado com sucesso!
