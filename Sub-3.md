Ở bài viết này, mình sẽ nói về cách làm việc khi bạn cần hoàn thành 1 module, ở chuỗi bài viết training này, mình sẽ lấy ví dụ về `account`.

Như đã nói ở bài 1, mình sẽ cố gắng diễn đạt để sao cho dù người trong ngành hay ngoài ngành đều có thể hiểu được.

Chắc hẳn, ai ở đây cũng học qua môn Văn, dù giỏi hay ngu thì bạn cũng biết 1 bài văn thì có 3 phần: Mở bài, Thân bài và Kết bài.
- Mở bài: Giới thiệu bài văn đó nói về cái qq gì.
- Thân bài: Mô tả chi tiết về cái qq đó.
- Kết bài: Tóm gọn lại cái qq đó có những gì (Tinh hoa của cái qq đó).

Ở đây cũng vậy, với các file `controller`, `services`, `validate` hay `index.js`,... Thông thường đều có 3 phần.
- Mở bài: Khai báo biến được sử dụng.
- Thân bài: Mô tả cách xử lý các biến đó như thế nào.
- Kết bài: Xuất ra (Export) cái tinh hoa của file đó ra ngoài để có thể sử dụng được ở chỗ khác.

Ví dụ mình có 1 file `index.js` như sau:
```js
  /* Mở bài */
  const router = require('express').Router();
  const accountCtl = require('./account.controller');
  const {
      validateCreate,
      validateUpdate,
  } = require('./account.validate');
  const { checkPermission } = require('../auth/auth.permission');
  const auth = require('../auth/passport.strategy')();


  /* Thân bài */
  router.use(auth.authenticate);
  
  router.get('/', checkPermission(['admin']), accountCtl.all);
  router.post('/create', checkPermission(['admin']), validateCreate, accountCtl.create);

  /* Kết bài */
  module.exports = router;
```
Như bên trên, mình có 3 phần như vậy:
- Phần mở bài được sử dụng để khai báo các biến sẽ sử dụng.
- Phần thân bài sẽ mô tả các biến được khai báo sẽ sử dụng như thế nào. Ở đây, nó sẽ mô tả các api con sẽ được thực hiện thông qua những hàm nào.
- Phần kết bài, sẽ public ra cho thằng cha sử dụng.

Tuy nhiên, khi xem các file `controller` hay `services`, bạn sẽ không thấy rõ 3 phần như trên mà có thể như bên dưới:
```js
  /* Mở bài */
  const userService = require('./account.service');
  const AppError = require('../../common/error/error');
  const { httpStatus } = require('../../common/error/http-status');

  /* Thân bài */
  const create = async (req, res, next) => {
    try {
      const user = await userService.create(req.body);

      res.json(user);
    } catch (error) {
      next(error);
    }
  };

  const update = async (req, res, next) => {
    try {
      const user = await userService.updateAcc(req.params.id, req.body);

      res.json(user);
    } catch (error) {
      next(error);
    }
  };

  /* Kết bài */
  module.exports = { create, update };

```
Dễ thấy, các phần `create` hay `update` bị lặp lại. Nên để viết nhanh, thì mình sẽ viết gộp lại thành như sau:
```js
  const userService = require('./account.service');
  const AppError = require('../../common/error/error');
  const { httpStatus } = require('../../common/error/http-status');

  module.exports = {
    create: async (req, res, next) => {
      try {
        const user = await userService.create(req.body);

        res.json(user);
      } catch (error) {
        next(error);
      }
    },

    update: async (req, res, next) => {
      try {
        const user = await userService.updateAcc(req.params.id, req.body);

        res.json(user);
      } catch (error) {
        next(error);
      }
    },
  };
```
Ở cách viết thứ 2, nó chỉ còn 2 phần là Phần khai báo và Phần xuất, tuy nhiên, Phần xuất lại bao gồm cả Phần mô tả các biến được khai báo cách sử dụng như thế nào.

Khi bạn đọc code, bạn sẽ thấy thắc một số điểm, mình nói `controller` và `services` cấu trúc là như nhau, vậy tại sao ở `controller` lại sử dụng 1 cú pháp `try { ... } catch (e) { ... }` trong khi ở `services` lại không có. Tại sao, phải sử dụng `async ... await ...`. Nếu không sử dụng thì sao. Trong `controller` các hàm đều sử dụng các biến đầu vào là `req`, `res` và `next` nhưng trong `services` lại không có. Vậy thì `req`, `res` và `next` là qq gì. Đặt tên khác có được hay không?

Để giải đáp thắc mắc đó, đọc tiếp nào.

### Try {...} catch (e) {...} là gì? Và tại sao chỉ thấy ở `controller` mà không thấy ở `services`.
Giải thích một cách dễ hiểu, khi chạy, nó chỉ chạy những gì xảy ra trong `try {...}`. Nếu mọi thứ suôn sẻ và không bị lỗi gì thì nó sẽ bỏ qua `catch`. Còn nếu trong khi thực hiện các thao tác trong `catch` mà phát sinh lỗi thì nó sẽ bỏ qua những bước phía sau mà xuất ra lỗi luôn.

Ví dụ, bạn hẹn crush đi chơi.
```js
  try {
    Chờ crush rep tin nhắn.
    Sửa soạn áo quần.
    Vuốt tóc các thứ.
    Lên đường
    .
    .
    .
    Chở crush về.
  } catch ( Crush từ chối ) {
    Ngủ
  }
```
Trong trường hợp, crush đồng ý và mọi chuyên suôn sẻ, thì bạn sẽ bỏ qua cái `Ngủ`, còn trong trường Crush từ chối thì bạn sẽ `Ngủ`.

Vậy khi nào thì sử dụng, khi nào bạn thực hiện thao tác và thao tác đó có thể gây ra lỗi thì bạn phải sử dụng `try...catch...` để handle cái lỗi đó ra, trong trường hợp bạn không sử dụng thì lỗi đó có thể làm đứng chương trình. Và nó có thể gây ra lỗi toàn hệ thống.

Các thao tác trong `services` là thao tác với cơ sở dữ liệu, nên có thể sẽ gây ra lỗi. Vì vậy, ở trong `controller` cần phải handle lỗi trong trường hợp các `services` bị lỗi. Trong một số trường hợp, ở `services` vẫn có thể sử dụng `try...catch` nếu bạn thấy thao tác cần thực hiện có thể gây ra lỗi.
### Async...Await là gì? Tại sao phải sử dụng và nếu không sử dụng thì sao.
Trong Javascript có một khái niệm mang tên _Bất đồng bộ_, hiểu theo cách đơn giản, là nó sẽ chạy không đồng bộ với nhau. Hơi khó hiểu.

Ví dụ, bạn nấu cơm trưa cần có canh, cá kho, thịt luộc, dọn chén bát và ăn. Bạn chỉ có thể **Ăn** sau khi đã nấu các món ăn và dọn chén bát. Bạn chỉ có thể nấu canh khi bạn đã luộc thịt, vì bạn cần nước luộc thịt để nấu canh. Vậy ở các ngôn ngữ đồng bộ, có nghĩa là lệnh sẽ chạy từ trên xuống dưới theo tuần tự, xong lệnh này thì mới thực hiện được lệnh tiếp theo.
```js
  const prepareLunch = () => {
    const boiledMeat = Luộc_thịt() // 10p
    const soup = Nấu_canh(boiledMeat.water) // 10p
    const fish = Kho_cá() // 10p
    const cup = Dọn_chén() // 3p
    const statusEat = Eat(boiledMeat.meat, soup, cup);
    return statusEat;
  }
```
Nếu thực hiện tuần tự thì bạn mất đâu đó khoảng 33p cuộc đời chỉ cho 1 bữa ăn 10p. Thay vào đó, trong khi bạn Luộc thịt, bạn có thể kho cá và dọn chén, sau đó nấu canh. Như vậy bạn chỉ mất đâu đó lâu nhất là 20p cho 1 bữa ăn. Lúc đó bạn cần sử dụng `async...await`.
```js
  const prepareLunch = async () => {
    const boiledMeat = await Luộc_thịt();
    const soup = await Nấu_canh(boiledMeat.water);
    const fish = await Kho_cá();
    const cup = await Dọn_chén();
    const statusEat = await Eat(boiledMeat.meat, soup, cup);
    return statusEat;
  }
```
Giải thích một chút ở bên trên, `Nấu canh` chỉ được thực hiện khi đã `Luộc thịt`, bởi vì hàm `Nấu canh` cần một biến là kết quả trả về của hàm `Luộc_thịt()`. Trong khi đó hàm `Kho_cá` và hàm `Dọn_chén()` sẽ được chạy vì nó không phải đợi thằng nào cả. Và hàm `Eat` chỉ được chạy khi tất cả các hàm trên đã hoàn tất vì nó yêu cầu 3 biến là kết quả trả về của 3 hàm trước nó.

Vậy nếu trong trường hợp không sử dụng thì chuyện gì xảy ra, đó là bạn sẽ không có gì để ăn, vì lúc đó đồ ăn và chén bát vẫn chưa sẵn sàng để bạn ăn.

Khi nào thì sử dụng `async...await...`, `async...await` chỉ là 1 trong 3 cách được sử dụng để xử lý bất đồng bộ trong js, bạn có thể tham khảo một số cách khác. Các cách xử lý bất đồng bộ này được sử dụng trong trường hợp bạn cần xử lý 1 thao tác và thao tác đó cần thời gian để chạy nhằm lấy dữ liệu hoặc cập nhật thông tin thì bạn cần sử dụng các hàm xử lý bất đồng bộ.

### Các biến `req`, `res` và `next` là gì? Có thể đặt tên khác hay không?
Thực ra, các biến `req`, `res` và `next` mà bạn thấy ở controller là các tham số đầu vào để thực hiện các thao tác. Cụ thể:
- `req`: Là viết của `request` - Là request của người dùng, ở đây, tùy từng phương thức, bạn sẽ lấy dữ liệu từ request của người dùng.
- `res`: Là viết tắt của `response` - Cái gì sẽ được trả về cho người dùng, thông thường, được sử dụng cho hàm `res.send()` hoặc `res.json()`.
- `next`: Đây là một hàm được sử dụng để xử lý promise. Hàm này thường được gọi khi xử lý các thao tác handle Error trong `try...catch`.

Vì đây chỉ là tên biến thôi, nên bạn có thể thay đổi nó thanh gì bạn muốn, có thể, bạn không thích `req, res, next` mà thích `gui_di, phan_hoi, tiep` thì vẫn không sao, như mình nói nó chỉ là tên biến mà thôi, nội dung không hề thay đổi.

## Cách làm việc như nào, khi cần thực hiện 1 module.
Khi bạn cần hoàn tất 1 module, bạn cần vào xem module đó sẽ có những models gì, những models đó sẽ lưu những thông tin gì, loại gì. Module này cần thực hiện những thao tác gì.

Mình đã giới thiệu chi tiết về các request method và mô tả khi client call apis thì chuyện gì xảy ra ở bài 2. Vậy thì, các phương thức và tên các api đó có thống nhất không, hay làm đến đâu thì suy nghĩ đến đấy? Để làm việc này, bạn leader như mình sẽ có 2 file chính mà bạn cần quan tâm:
- File README: Mình sẽ mô tả module nào sẽ có thực hiện thao tác gì?
- File APIs docs: Mình sẽ mô tả các APIs sẽ được gọi từ client, API đó sẽ gửi kèm theo thông tin gì, nó sẽ lấy thông tin gì từ máy chủ.

Để xem APIs docs, sau khi chạy ứng dụng, xem ứng dụng đó chạy trên port bao nhiêu (thông thường là 3000), sau đó bạn truy cập vào `http://localhost:3000/docs`. Tìm đến module mà bạn cần làm, thì ở đó mình đã mô tả rõ ràng cái gì sẽ gửi đi, ai có quyền thực hiện thao tác đó, và sẽ cần gửi lại thông tin gì cho client.

Từ các mô tả đó, bạn sẽ code được file `controllers.js` của module đó. Từ đó, sẽ code file `index.js` và file `services.js`. Ở đây, bạn cần lưu ý, là các hàm ở services không chỉ gọi từ module mà nó sẽ được gọi đến từ các module khác nếu cần thực hiện thao tác nào đó liên quan đến module này. Nên không chỉ đọc phần mô tả module của mình mà bạn cần phải đọc phần mô tả trong các module khác (Thông thường mình sẽ note bên dưới module hiện tại). OK!

Một lỗi mà dù bạn trong ngành hay ngoài ngành đều dễ mắc phải là code một lèo xong mới chạy thử, nhưng mình thật lòng khuyên, bạn làm xong APIs nào, chạy APIs đó trong mọi trường hợp, xem có được hay không? Ngoài ra, ở trong phần APIs Docs của mình đã mô tả các mã lỗi và tên lỗi, bạn cần phải handle các lỗi đó.

