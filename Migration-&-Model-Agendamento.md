# **Migration e model de agendamento**

**Objetivo:** Toda vez que um usuário marcar um agendamento de serviço com algum dos prestadores de serviços, será gerado um registro no banco de dados.

### **1. Criando nova tabela para o banco de dados**

Agora vamos criar uma nova tabela no banco de dados para registros de agendamentos, logo:
```javascript
yarn sequelize migration:create --name=create-appointments
```
### **2. Configurando a nova migration**

Em um agendamento temos em comum um **id** que é a chave primária e única desse agendamento. A **data** do agendamento no fomato **Sequelize.DATE**. O **id do usuário e do prestador de serviços**. E por fim as datas de **criação**, **cancelamento** e **atualização** do agendamento.
```javascript
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
      return queryInterface.createTable('appointments', {
        id: {
          type: Sequelize.INTEGER,
          allowNull: false,
          autoIncrement: true,
          primaryKey: true,
        },
        date: {
          allowNull: false,
          type: Sequelize.DATE,
        },
        user_id: {
          type: Sequelize.INTEGER,
          references: { model: 'users', key: 'id' },
          onUpdate: 'CASCADE',
          onDelete: 'SET NULL',
          allowNull: true,
        },
        provider_id: {
          type: Sequelize.INTEGER,
          references: { model: 'users', key: 'id' },
          onUpdate: 'CASCADE',
          onDelete: 'SET NULL',
          allowNull: true,
        },
        canceled_at: {
          type: Sequelize.DATE,
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
      return queryInterface.dropTable('appointments');
  }
};
```
E por meio do desses dois campos temos um relacionamento com **usuário** e **prestador de serviço**.
```javascript
  user_id: {
    type: Sequelize.INTEGER,
    references: { model: 'users', key: 'id' },
    onUpdate: 'CASCADE',
    onDelete: 'SET NULL',
    allowNull: true,
  },
  provider_id: {
    type: Sequelize.INTEGER,
    references: { model: 'users', key: 'id' },
    onUpdate: 'CASCADE',
    onDelete: 'SET NULL',
    allowNull: true,
  },
```
A relação é feita por conta de ```references: { model: 'users', key: 'id' }```.

Agora podemos migrar a tabela para o banco de dados:
```javascript
yarn sequelize db:migrate
```

### **3. Configurando o model appointments.js**

```javascript
import Sequelize, { Model } from 'sequelize';
import { isBefore, subHours } from 'date-fns';

class Appointment extends Model {
  static init(sequelize){
    super.init(
      {
        date: Sequelize.DATE,
        canceled_at: Sequelize.DATE,
        past: {
          type: Sequelize.VIRTUAL,
          get() {
            return isBefore(this.date, new Date());
          }
        },
      },
      {
        sequelize,
      }
    );

    return this;
  }

  static associate(models) {
    this.belongsTo(models.User, { foreignKey: 'user_id', as: 'user' });
    this.belongsTo(models.User, { foreignKey: 'provider_id', as: 'provider' });
  }
}

export default Appointment;
```
Por meio desse método é gerado um contrato de relacionamento entre provedor de serviços e usuário cliente.
```javascript
static init(sequelize){
    super.init(
      {
        date: Sequelize.DATE,
        canceled_at: Sequelize.DATE,
        past: {
          type: Sequelize.VIRTUAL,
          get() {
            return isBefore(this.date, new Date());
          }
        },
```
Aqui temos o relacionamento de prestador de serviços e usuário cliente:
```javascript
static associate(models) {
    this.belongsTo(models.User, { foreignKey: 'user_id', as: 'user' });
    this.belongsTo(models.User, { foreignKey: 'provider_id', as: 'provider' });
  }
```
Agora é necessário configruar **index.js** e importar o modelo **Appointments.js**:

```javascript
import Appointments from '../app/models/Appointment';
```
Dentro de **Appointment.js** adicione-o dentro da **constante models**: 
```javascript
const models = [User, File, Appointment];
```
Agora estamos com três models que podem ser mapeados por nossa aplicação.
