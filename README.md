# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) SOFTWARE ENGINEERING IMMERSIVE

## Getting started

1. Fork
1. Clone

# Sequelize Migrations

> Take five minutes and read the Sequelize docs on migrations:
>
> - https://sequelize.org/master/manual/migrations.html

Migrations are an important feature to have while managing a database. They allow us to add, remove or change columns or tables without destroying our stored information. Database integrity is key when you're building applications ready for public use. Every time you make an update to your database you should not lose any stored information.

##

Let's go into the repo:

```sh
cd sequelize-migrations
npm install
```

Create your database:

```sh
sequelize db:create
```

Run the existing migrations:

```sh
sequelize db:migrate
```

Take a moment to look at the User model that already exists in your codebase:

models/user.js

```js
'use strict'
const { Model } = require('sequelize')
module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
    }
  }
  User.init(
    {
      firstName: DataTypes.STRING,
      lastName: DataTypes.STRING,
      email: DataTypes.STRING,
      password: DataTypes.STRING
    },
    {
      sequelize,
      modelName: 'User',
      tableName:'users
    }
  )
  return User
}
```

Notice how there is no userName attribute, what if we wanted to add `username` to the User model? How do we use [Sequelize Migrations](https://sequelize.org/master/manual/migrations.html) to do this?

```sh
sequelize migration:generate --name add-username-to-users
```

> Want to know more about generating migrations using the Sequelize CLI? Run `sequelize migration:generate --help`

Use the `addColumn` method in the migration:

```sh
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.addColumn('users','userName',{
      type:Sequelize.STRING
      }
    );
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.removeColumn('users', 'userName');
  }
};
```

Make sure your model matches your migration, if we add a column in our migration our model needs to reflect that in order to use that field:

```js
'use strict'
const { Model } = require('sequelize')
module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
    }
  }
  User.init(
    {
      firstName: DataTypes.STRING,
      lastName: DataTypes.STRING,
      email: DataTypes.STRING,
      password: DataTypes.STRING,
      userName: Datatypes.STRING
    },
    {
      sequelize,
      modelName: 'User',
      tableName:'users
    }
  )
  return User
}
```

Apply the migration:

```sh
sequelize db:migrate
```

> If you made a mistake, you can always rollback: sequelize db:migrate:undo

Let's make sure the column was added:

```sh
psql sequelize_migrations_development
SELECT * FROM users;
```

Oh no! I made a mistake! I wanted `username` not `userName` as the column name. How do I rename an already existing column using migrations?

```sh
sequelize migration:generate --name rename-userName-to-username
```

And write the following code in your migration:

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.renameColumn('users', 'userName', 'username')
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.renameColumn('users', 'username', 'userName')
  }
}
```

Run it!

```sh
sequelize db:migrate
```

Make the adjustment in your user model:

```js
'use strict'
const { Model } = require('sequelize')
module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
    }
  }
  User.init(
    {
      firstName: DataTypes.STRING,
      lastName: DataTypes.STRING,
      email: DataTypes.STRING,
      password: DataTypes.STRING,
      username:Datatypes.STRING
    },
    {
      sequelize,
      modelName: 'User',
      tableName:'users
    }
  )
  return User
}
```

Make sure the change was applied:

```sh
psql sequelize_migrations_development
SELECT * FROM users;
```

So now you have one last change you'd like to make to your database. You want to have `email` be required (no nulls). That means we need to create a migration to change our already existing `email` column to not allow nulls.

```sh
sequelize migration:generate --name change-email-to-not-allow-nulls
```

Here is the code using the sequelize `changeColumn` method:

```sh
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.changeColumn(
      'users',
      'email',
      {
        type: Sequelize.STRING,
        allowNull: false
      }
    );
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.changeColumn(
      'users',
      'email',
      {
        type: Sequelize.STRING,
        allowNull: true
      }
    )
  }
};
```

Apply the changes:

```sh
sequelize db:migrate
```

Reflect these changs in your `User` model:

```js
'use strict'
const { Model } = require('sequelize')
module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
    }
  }
  User.init(
    {
      firstName: DataTypes.STRING,
      lastName: DataTypes.STRING,
      email: {
        type:DataTypes.STRING,
        allowNull:false
      },
      password: DataTypes.STRING,
      username:Datatypes.STRING
    },
    {
      sequelize,
      modelName: 'User',
      tableName:'users
    }
  )
  return User
}
```

Test it out:

```sql
psql sequelize_migrations_development
INSERT INTO users VALUES (2, 'Jane', 'Dee');
```

The result should be `null value in column "email" violates not-null constraint`. Awesome.

> Want to successfully `INSERT`? Try `INSERT INTO users VALUES (Default, 'Jane', 'Dee', 'jane@dee.com', 'STRONG', now(), now(), 'janeD');`

## Conclusion

There are [several other methods available](https://sequelize.readthedocs.io/en/latest/docs/migrations/#changecolumntablename-attributename-datatypeoroptions-options) to you e.g. `dropAllTables()`, `renameTable()`, and `removeColumn()` to name a few.

## Resources

- https://sequelize.readthedocs.io/en/latest/docs/migrations/#changecolumntablename-attributename-datatypeoroptions-options
- https://sequelize.org/master/manual/migrations.html
