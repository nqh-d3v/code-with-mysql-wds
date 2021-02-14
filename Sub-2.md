Ở bài này, mình sẽ nói về 2 phần chính:
- [1. Các thao tác CRUD với cơ sở dữ liệu](#các-thao-tác-crud-với-cơ-sở-dữ-liệu).
- [2. Các phương thức request từ client](#các-phương-thức-request-từ-client).

Như đã nói ở bài trước, các dự án clb nhận đa phần đều sử dụng MySQL nên các ví dụ trong chuỗi training này mình sẽ sử dụng MySQL.

Để sử dụng MySQL trong NodeJS, CLB sử dụng Sequelize (một ORM NodeJS dựa trên promise), chi tiết bạn có thể tham khảo tại [đây](https://sequelize.org/master/)

### Các thao tác CRUD với cơ sở dữ liệu:
#### Create
```sql
  INSERT INTO `account` ('username', 'password', 'name') VALUES ('admin', 'admin', 'admin');
```
Khi sử dụng kèm Sequelize, thao tác như sau:
```js
  const account = require('../../models');

  module.exports = {
    /** accountDTO
      {
        username: string(255),
        password: text,
        name: string(255)
      }
      Phía bên dưới, mình sử dụng một thao tác của ES6
      Bạn có thể viết theo một cách dài dòng hơn như sau:
        const newAcc = new account();
        newAcc.username = accountDTO.username;
        newAcc.password = accountDTO.password;
        newAcc.name = accountDTO.name;
        await newAcc.save();
      hoặc:
        const newAcc = await account.create({
          username: accountDTO.username,
          password: accountDTO.password,
          name: accountDTO.name,
        });

      Tuy nhiên, dễ nhận ra, có các trường lặp lại là username, password và name, vì vậy ở ES6, bạn có thể có tương đương như sau:
        const a = function({'username': username}) <=> const a = function({ username });
    */
    create: async function (accountDTO) {
      const newAcc = await account.create(accountDTO);
      return newAcc;
    },
  }
```
#### READ
```sql
  -- Lấy thông tin tài khoản có id = 1
  SELECT * FROM `account` WHERE id = 1;
```
Sử dụng cùng Sequelize có dạng như sau:
```js
  const account = require('../../models');
  module.exports = {
    readV1: async function (accountId) {
      const acc = await account.findAll({
        where: {
          id: 1,
        },
      });
      return acc;
    },
    readV2: async function (accountId) {
      const acc = await account.findByPk(accountId);
      return acc;
    },
  };
```

#### UPDATE
```sql
  UPDATE `account` SET 'name' = 'admin update' WHERE id = 1;
```
Sử dụng cùng Sequelize có dạng như sau:
```js
  // ~services

  const account = require('../../models');
  module.exports = {
    updateV1: async function (accountId, updateDTO) {
      const acc = await account.update(
        updateDTO,
        {
          where: {
            id: 1,
          },
        }
      );
      return acc;
    },
    updateV2: async function (accountId, updateDTO) {
      const acc = await account.findByPk(accountId);
      acc.name = updateDTO.name;
      await acc.save();
      return acc;
    },
  };
```
#### DELETE
```sql
  DELETE * FROM account WHERE id = 1;
```
Khi sử dụng cùng Sequelize có dạng như sau:
```js
  // ~services

  const account = require('../../models');
  module.exports = {
    delete: async function (accountId) {
      const acc = await account.destroy({
        where: {
          id: 1,
        },
      });
      return acc;
    },
  };
```
### Các phương thức request từ client:
Các phương thức này được sử dụng trong file `index.js` của các module.
#### GET
```js
  router.get('/', accountCtl.all);
```
**Chỉ nên sử dụng để lấy dữ liệu.**

Trong ví dụ trên: Phương thức GET đến api `'/'` yêu cầu hàm controller all() trong controller để thực hiện yêu cầu của nó.
#### POST
```js
  router.post('/', validateCreate, accountCtl.create);
```
**Thường được sử dụng để gửi data lên máy chủ để thực hiện một thay đổi thông tin trên máy chủ.**

Trong ví dụ trên: Phương thức POST cho api `'/'` dùng để tạo mới một tài khoản
#### PUT
```js
  router.put('/:id', validateUpdate, accountCtl.update);
```
**Được sử dụng để gửi data lên máy chủ và thực hiện thay đổi thông tin trên máy chủ bằng chính dữ liệu được gửi lên.**

Trong ví dụ trên: Phương thức PUT cho api `'/:id'` dùng để cập nhật thông tin cho tài khoản.
### PATCH
```js
  router.patch('/:id', validateUpdatePwd, accountCtl.updatePass);
```
**Được sử dụng để gửi data lên máy chủ và thực hiện thay đổi một phần thông tin trên máy chủ mà không tác động đến các thông tin khác.**

Điểm giống nhau giữa PUT và POST là đều có thể được sử dụng để tạo mới một bản ghi, điểm khác nhau là PUT thì thực hiện ghi đè trong trường hợp bản ghi đó đã tồn tại trong cơ sở dữ liệu.

Điểm giống nhau giữa PUT và PATCH là đều được thực hiện để cập nhật thông tin cho một bản ghi trong trường hợp nó đã tồn tại, tuy nhiên, PATCH chỉ thực hiện thay đổi một phần mà không ảnh hưởng các thông tin khác trong khi PUT lại ghi đè toàn bộ.

-----

Bài này mình đã nói về 2 vấn đề chính như bên trên, ở bài tiếp theo, mình sẽ nói về cách code một module như thế nào, cụ thể là cách bạn làm việc với controller, services và validate như thế nào.